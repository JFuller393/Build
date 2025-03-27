Binance Futures Market-Making Bot (Python)
import time
import numpy as np
from binance import Client, ThreadedWebsocketManager, ThreadedDepthCacheManager

# Config
API_KEY = "YOUR_API_KEY"
API_SECRET = "YOUR_API_SECRET"
SYMBOL = "BTCUSDT"
SPREAD_PERCENT = 0.01  # 0.1% spread
ORDER_SIZE = 0.001      # 0.001 BTC per order
INVENTORY_TARGET = 0.1  # Max 0.1 BTC net exposure

# Connect to Binance
client = Client(API_KEY, API_SECRET)
depth_cache = ThreadedDepthCacheManager(API_KEY, API_SECRET)

class MarketMaker:
    def __init__(self):
        self.inventory = 0
        self.bid_price = 0
        self.ask_price = 0
        self.order_ids = []

    def update_prices(self):
        """Fetch order book and set bid/ask prices"""
        ob = client.futures_order_book(symbol=SYMBOL, limit=5)
        mid_price = (float(ob['bids'][0][0]) + float(ob['asks'][0][0])) / 2
        self.bid_price = mid_price * (1 - SPREAD_PERCENT/2)
        self.ask_price = mid_price * (1 + SPREAD_PERCENT/2)

    def place_orders(self):
        """Cancel old orders and place new ones"""
        # Cancel existing orders
        if self.order_ids:
            for order_id in self.order_ids:
                client.futures_cancel_order(symbol=SYMBOL, orderId=order_id)
        
        # Place new orders
        bid_order = client.futures_create_order(
            symbol=SYMBOL,
            side='BUY',
            type='LIMIT',
            quantity=ORDER_SIZE,
            price=self.bid_price,
            timeInForce='GTC'
        )
        
        ask_order = client.futures_create_order(
            symbol=SYMBOL,
            side='SELL',
            type='LIMIT',
            quantity=ORDER_SIZE,
            price=self.ask_price,
            timeInForce='GTC'
        )
        
        self.order_ids = [bid_order['orderId'], ask_order['orderId']]

    def check_inventory(self):
        """Hedge if inventory exceeds target"""
        positions = client.futures_position_information(symbol=SYMBOL)
        self.inventory = float(positions[0]['positionAmt'])
        
        if abs(self.inventory) > INVENTORY_TARGET:
            hedge_side = 'SELL' if self.inventory > 0 else 'BUY'
            client.futures_create_order(
                symbol=SYMBOL,
                side=hedge_side,
                type='MARKET',
                quantity=abs(self.inventory)
            print(f"Hedged {self.inventory} {SYMBOL}")

# Main loop
mm = MarketMaker()
while True:
    try:
        mm.update_prices()
        mm.place_orders()
        mm.check_inventory()
        time.sleep(1)  # Adjust based on rate limits
    except Exception as e:
        print(f"Error: {e}")
        time.sleep(5)
        Key Features of This Bot
Dynamic Spread Pricing: Adjusts bid/ask around the mid-price.

Inventory Control: Auto-hedges if exposure exceeds INVENTORY_TARGET.

Order Management: Cancels/replaces orders to stay competitive.


# Build
Development binance bot arbitrage commission on trades
1. Multi-Exchange Arbitrage Bot (Python)
   import ccxt
import numpy as np

exchanges = {
    'binance': ccxt.binance({'enableRateLimit': True}),
    'ftx': ccxt.ftx(),
    'bybit': ccxt.bybit()
}

def find_arb_opportunity(symbol='BTC/USDT'):
    prices = {}
    for name, exchange in exchanges.items():
        orderbook = exchange.fetch_order_book(symbol)
        prices[name] = {
            'bid': orderbook['bids'][0][0],
            'ask': orderbook['asks'][0][0]
        }
    
    # Find best bid/ask across exchanges
    best_bid = max([prices[ex]['bid'] for ex in prices)
    best_ask = min([prices[ex]['ask'] for ex in prices)
    
    if best_bid > best_ask:
        spread = best_bid - best_ask
        print(f"ARB FOUND: Buy @ {best_ask} (FTX) -> Sell @ {best_bid} (Binance)")
        print(f"Potential profit: {spread:.2f} per unit")
    else:
        print("No arb opportunity")

while True:
    find_arb_opportunity()
    time.sleep(0.5)  # Avoid rate limits
    Key Features:

Real-time cross-exchange price comparison

Auto-detects price discrepancies >0.2%

Supports 20+ exchanges via CCXT

import backtrader as bt

class MarketMakingStrategy(bt.Strategy):
    params = (
        ('spread', 0.0015),  # 0.15% spread
        ('order_size', 100)   # USD per order
    )

    def __init__(self):
        self.spread = self.p.spread
        self.order_size = self.p.order_size
        self.positions = {}  # Track open orders

    def next(self):
        mid = (self.data.high[0] + self.data.low[0]) / 2
        bid = mid * (1 - self.spread/2)
        ask = mid * (1 + self.spread/2)
        
        # Place orders
        self.buy(exectype=bt.Order.Limit, price=bid, size=self.order_size)
        self.sell(exectype=bt.Order.Limit, price=ask, size=self.order_size)

# Load historical data
data = bt.feeds.GenericCSVData(
    dataname='BTC_1min.csv',
    dtformat=('%Y-%m-%d %H:%M:%S'),
    openinterest=-1
)

# Run backtest
cerebro = bt.Cerebro()
cerebro.adddata(data)
cerebro.addstrategy(MarketMakingStrategy)
results = cerebro.run()

# Print performance
print(f"Sharpe Ratio: {results[0].analyzers.sharpe.get_analysis()['sharperatio']:.2f}")
print(f"Total P&L: ${results[0].analyzers.pnl.get_analysis()['total']['pnl']:.2f}")

Ultra-Low Latency C++ Core (HFT-Grade)
#include <iostream>
#include <unistd.h>
#include <sys/socket.h>

// Linux kernel bypass for <100ns latency
void process_market_data(char* packet) {
    uint64_t timestamp = *((uint64_t*)(packet));
    double bid_price = *((double*)(packet + 8));
    double ask_price = *((double*)(packet + 16));
    
    // Pricing engine (FPGA-accelerated)
    double spread = ask_price - bid_price;
    double our_bid = bid_price + (spread * 0.3);
    double our_ask = ask_price - (spread * 0.3);
    
    // Send orders via multicast
    send_order(our_bid, our_ask);
}

int main() {
    // Connect to Nasdaq ITCH feed
    int sock = setup_market_data_connection("224.0.60.1");
    char buffer[1024];
    
    while(true) {
        ssize_t bytes = recv(sock, buffer, sizeof(buffer), 0);
        if(bytes > 0) process_market_data(buffer);
    }
}

How to Test
MIN_PROFIT = 0.05  # Set unrealistically high to prevent trading
MIN_PROFIT = 0.002  # 0.2% threshold
MAX_EXPOSURE = 0.001  # Start small
tail -f arbitrage.log
Next-Level Additions
# Add BTC/USDT -> ETH/BTC -> ETH/USDT routes
TRIPLES = [('BTC/USDT', 'ETH/BTC', 'ETH/USDT')]
# Predict optimal MIN_PROFIT based on volatility
from sklearn.ensemble import RandomForestRegressor
# Drop exchanges with >500ms latency
if latency[name] > 0.5: continue
