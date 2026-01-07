---
title: "Polymarket API Guide: Building Your First Trading Bot"
date: 2026-01-05
description: "A technical guide to the Polymarket API. Learn how to fetch market data, place orders, and build your first automated trading bot."
keywords: "polymarket api, polymarket bot, polymarket trading bot, prediction market api, automated trading polymarket"
tags: ["bots"]
image: "/images/uploads/pm-08-api-guide.jpg"
---

Manual trading is fine for casual participation. But if you want to capture edges consistently, you need automation.

Bots can watch markets 24/7. They react in milliseconds. They don't get emotional. They execute your strategy with perfect discipline.

This guide walks you through the Polymarket API from scratch. By the end, you'll have a working foundation to build your own trading bot.

## Prerequisites

Before you start, you'll need:

- Python 3.8+ installed
- Basic programming knowledge
- A Polymarket account with some USDC deposited
- A crypto wallet (MetaMask or similar)
- Comfort with command line operations

If you're missing any of these, get them sorted first. This guide assumes technical competence.

## Understanding Polymarket's Architecture

Polymarket runs on the Polygon blockchain. But you don't need to interact with the blockchain directly for most operations.

The platform provides:

**REST API** - For fetching market data, order book info, and historical data. Read-only. No authentication required.

**CLOB API** - The Central Limit Order Book API. For placing, canceling, and managing orders. Requires authentication.

**WebSocket feeds** - Real-time updates on prices and order book changes.

We'll start with the REST API (reading data), then move to the CLOB API (placing orders).

## Setting Up Your Environment

Create a new project directory and virtual environment:

```bash
mkdir polymarket-bot
cd polymarket-bot
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

Install required packages:

```bash
pip install requests websocket-client python-dotenv eth-account
```

Create a `.env` file for your credentials (never commit this to git):

```
PRIVATE_KEY=your_wallet_private_key
API_KEY=your_polymarket_api_key
API_SECRET=your_polymarket_api_secret
API_PASSPHRASE=your_polymarket_api_passphrase
```

## Fetching Market Data

Let's start by pulling market data. No authentication needed.

```python
import requests

BASE_URL = "https://clob.polymarket.com"

def get_markets():
    """Fetch all active markets"""
    response = requests.get(f"{BASE_URL}/markets")
    return response.json()

def get_market(condition_id):
    """Fetch a specific market by condition ID"""
    response = requests.get(f"{BASE_URL}/markets/{condition_id}")
    return response.json()

def get_orderbook(token_id):
    """Fetch order book for a specific token"""
    response = requests.get(f"{BASE_URL}/book", params={"token_id": token_id})
    return response.json()

# Example usage
markets = get_markets()
print(f"Found {len(markets)} markets")

# Print first market
if markets:
    market = markets[0]
    print(f"Market: {market['question']}")
    print(f"Condition ID: {market['condition_id']}")
```

Each market has two tokens: one for Yes, one for No. The `token_id` is what you use to place orders.

## Understanding Market Structure

A Polymarket market response looks something like this:

```python
{
    "condition_id": "0x...",
    "question": "Will X happen by Y date?",
    "tokens": [
        {
            "token_id": "123...",
            "outcome": "Yes",
            "price": "0.65"
        },
        {
            "token_id": "456...",
            "outcome": "No",
            "price": "0.35"
        }
    ],
    "volume": "1500000",
    "liquidity": "250000"
}
```

The `tokens` array contains the Yes and No outcomes. Prices should roughly sum to 1 (minus spread).

## Reading the Order Book

The order book shows all outstanding buy and sell orders:

```python
def analyze_orderbook(token_id):
    """Analyze order book depth and spread"""
    book = get_orderbook(token_id)

    bids = book.get('bids', [])
    asks = book.get('asks', [])

    if bids and asks:
        best_bid = float(bids[0]['price'])
        best_ask = float(asks[0]['price'])
        spread = best_ask - best_bid

        print(f"Best Bid: {best_bid}")
        print(f"Best Ask: {best_ask}")
        print(f"Spread: {spread}")

        # Calculate depth at each level
        bid_depth = sum(float(b['size']) for b in bids[:5])
        ask_depth = sum(float(a['size']) for a in asks[:5])

        print(f"Bid depth (top 5 levels): {bid_depth}")
        print(f"Ask depth (top 5 levels): {ask_depth}")

    return book
```

Understanding order book depth is crucial. It tells you how much you can trade without moving the price.

## Setting Up Authentication

To place orders, you need to authenticate. Polymarket uses API keys tied to your wallet.

First, generate API credentials through the Polymarket interface or their API. Then set up authentication headers:

```python
import hmac
import hashlib
import time
import base64
from dotenv import load_dotenv
import os

load_dotenv()

API_KEY = os.getenv('API_KEY')
API_SECRET = os.getenv('API_SECRET')
API_PASSPHRASE = os.getenv('API_PASSPHRASE')

def get_auth_headers(method, path, body=""):
    """Generate authentication headers for CLOB API"""
    timestamp = str(int(time.time() * 1000))

    message = timestamp + method + path + body
    signature = hmac.new(
        base64.b64decode(API_SECRET),
        message.encode('utf-8'),
        hashlib.sha256
    ).digest()
    signature_b64 = base64.b64encode(signature).decode('utf-8')

    return {
        "POLY_API_KEY": API_KEY,
        "POLY_SIGNATURE": signature_b64,
        "POLY_TIMESTAMP": timestamp,
        "POLY_PASSPHRASE": API_PASSPHRASE
    }
```

## Placing Orders

Now the interesting part. Let's place a limit order:

```python
import json

def place_limit_order(token_id, side, price, size):
    """
    Place a limit order

    Args:
        token_id: The token to trade
        side: "BUY" or "SELL"
        price: Price in USDC (0-1 for prediction markets)
        size: Number of shares
    """
    path = "/order"

    order_data = {
        "tokenID": token_id,
        "side": side,
        "price": str(price),
        "size": str(size),
        "type": "GTC"  # Good Till Canceled
    }

    body = json.dumps(order_data)
    headers = get_auth_headers("POST", path, body)
    headers["Content-Type"] = "application/json"

    response = requests.post(
        f"{BASE_URL}{path}",
        headers=headers,
        data=body
    )

    return response.json()

# Example: Buy 100 Yes shares at 0.55
result = place_limit_order(
    token_id="your_token_id_here",
    side="BUY",
    price=0.55,
    size=100
)
print(result)
```

## Canceling Orders

You'll need to cancel orders when your strategy changes:

```python
def cancel_order(order_id):
    """Cancel an existing order"""
    path = f"/order/{order_id}"
    headers = get_auth_headers("DELETE", path)

    response = requests.delete(
        f"{BASE_URL}{path}",
        headers=headers
    )

    return response.json()

def cancel_all_orders():
    """Cancel all open orders"""
    path = "/orders"
    headers = get_auth_headers("DELETE", path)

    response = requests.delete(
        f"{BASE_URL}{path}",
        headers=headers
    )

    return response.json()
```

## Getting Your Positions

Track what you own:

```python
def get_positions():
    """Get all current positions"""
    path = "/positions"
    headers = get_auth_headers("GET", path)

    response = requests.get(
        f"{BASE_URL}{path}",
        headers=headers
    )

    return response.json()

def get_open_orders():
    """Get all open orders"""
    path = "/orders"
    headers = get_auth_headers("GET", path)

    response = requests.get(
        f"{BASE_URL}{path}",
        headers=headers
    )

    return response.json()
```

## Real-Time Data with WebSockets

For trading bots, you want real-time updates, not polling:

```python
import websocket
import json

def on_message(ws, message):
    data = json.loads(message)
    print(f"Received: {data}")
    # Handle price updates, trade notifications, etc.

def on_error(ws, error):
    print(f"Error: {error}")

def on_close(ws, close_status_code, close_msg):
    print("Connection closed")

def on_open(ws):
    # Subscribe to market updates
    subscribe_msg = {
        "type": "subscribe",
        "channel": "market",
        "assets": ["your_token_id_here"]
    }
    ws.send(json.dumps(subscribe_msg))

def start_websocket():
    ws = websocket.WebSocketApp(
        "wss://ws-subscriptions-clob.polymarket.com/ws",
        on_open=on_open,
        on_message=on_message,
        on_error=on_error,
        on_close=on_close
    )
    ws.run_forever()
```

## A Simple Market Making Bot

Let's put it together with a basic market making strategy:

```python
import time

class SimpleMarketMaker:
    def __init__(self, token_id, spread=0.02, size=100):
        self.token_id = token_id
        self.spread = spread  # 2 cents on each side
        self.size = size
        self.buy_order_id = None
        self.sell_order_id = None

    def get_mid_price(self):
        """Calculate mid price from order book"""
        book = get_orderbook(self.token_id)

        if book['bids'] and book['asks']:
            best_bid = float(book['bids'][0]['price'])
            best_ask = float(book['asks'][0]['price'])
            return (best_bid + best_ask) / 2
        return None

    def update_quotes(self):
        """Update buy and sell orders around mid price"""
        mid = self.get_mid_price()

        if mid is None:
            print("No market data available")
            return

        buy_price = round(mid - self.spread/2, 2)
        sell_price = round(mid + self.spread/2, 2)

        # Cancel existing orders
        if self.buy_order_id:
            cancel_order(self.buy_order_id)
        if self.sell_order_id:
            cancel_order(self.sell_order_id)

        # Place new orders
        buy_result = place_limit_order(
            self.token_id, "BUY", buy_price, self.size
        )
        sell_result = place_limit_order(
            self.token_id, "SELL", sell_price, self.size
        )

        self.buy_order_id = buy_result.get('orderID')
        self.sell_order_id = sell_result.get('orderID')

        print(f"Quotes updated: Buy {buy_price} / Sell {sell_price}")

    def run(self, update_interval=30):
        """Run the market maker"""
        print("Starting market maker...")

        while True:
            try:
                self.update_quotes()
                time.sleep(update_interval)
            except KeyboardInterrupt:
                print("Shutting down...")
                cancel_all_orders()
                break
            except Exception as e:
                print(f"Error: {e}")
                time.sleep(5)

# Usage
# maker = SimpleMarketMaker("your_token_id", spread=0.02, size=50)
# maker.run()
```

This is a toy example. A real market making bot needs:

- Better inventory management
- Risk limits
- Multiple market support
- Proper logging
- Error recovery
- Position monitoring

But this gives you the foundation.

## Important Considerations

**Rate limits**

Polymarket has rate limits. Don't spam the API. Use WebSockets for real-time data instead of polling.

**Gas fees**

Trading on Polygon has small gas fees. Factor these into your calculations.

**Testnet first**

If available, test on testnet before using real money. Bugs in trading bots can be expensive.

**Error handling**

APIs fail. Networks hiccup. Your bot needs to handle errors gracefully without losing money.

**Monitoring**

Run your bot with proper monitoring. Know when it's failing. Have kill switches.

## Security Best Practices

**Never expose private keys**

Use environment variables. Never commit credentials to git. Use .gitignore religiously.

**Limit API permissions**

Only give your API keys the permissions they need. Read-only keys for monitoring bots.

**Position limits**

Code hard limits on position sizes. A bug shouldn't be able to bet your entire bankroll.

**Regular audits**

Review your bot's behavior regularly. Check logs. Verify it's doing what you expect.

<div class="cta-box">
  <h3>Master Prediction Market Bots</h3>
  <p>Get the complete playbook for building and deploying trading bots on Polymarket.</p>
  <a href="[YOUR-AFFILIATE-LINK]" class="cta-button">Get the Course</a>
</div>

## Next Steps

This guide gets you started. Where you go next depends on your goals:

**For market making:** Learn about inventory risk, adverse selection, and spread optimization.

**For arbitrage:** Build systems to monitor multiple platforms simultaneously.

**For signals-based trading:** Integrate news feeds, sentiment analysis, or other data sources.

**For latency optimization:** Move to compiled languages, colocate servers, optimize network paths.

The Polymarket API is your gateway to systematic prediction market trading. The basics are here. The advanced stuff is where the real money is.

Start simple. Test thoroughly. Scale gradually.

The bot traders making real money didn't build their systems overnight. They iterated, failed, learned, and improved.

Your turn.
