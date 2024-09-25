import alpaca_trade_api as tradeapi
import pandas as pd
from ta.trend import MACD
import time

# Alpaca API credentials
API_KEY = 'your_api_key'
API_SECRET = 'your_api_secret'
BASE_URL = 'https://paper-api.alpaca.markets'

# Initialize Alpaca API connection
api = tradeapi.REST(API_KEY, API_SECRET, BASE_URL, api_version='v2')

# Get historical data for 2-minute timeframe
def get_stock_data(symbol):
    barset = api.get_barset(symbol, '2Min', limit=100)
    bars = barset[symbol]
    
    data = {
        'Time': [bar.t for bar in bars],
        'Open': [bar.o for bar in bars],
        'High': [bar.h for bar in bars],
        'Low': [bar.l for bar in bars],
        'Close': [bar.c for bar in bars],
        'Volume': [bar.v for bar in bars]
    }
    df = pd.DataFrame(data)
    return df

# Heikin-Ashi Candle Calculation
def calculate_heikin_ashi(df):
    df['HA_Close'] = (df['Open'] + df['High'] + df['Low'] + df['Close']) / 4
    df['HA_Open'] = (df['Open'].shift(1) + df['Close'].shift(1)) / 2
    df['HA_High'] = df[['High', 'HA_Open', 'HA_Close']].max(axis=1)
    df['HA_Low'] = df[['Low', 'HA_Open', 'HA_Close']].min(axis=1)
    return df

# MACD Calculation
def calculate_macd(df):
    macd = MACD(df['Close'], window_slow=26, window_fast=12, window_sign=9)
    df['MACD'] = macd.macd()
    df['MACD_Signal'] = macd.macd_signal()
    df['MACD_Hist'] = macd.macd_diff()
    return df

# Buy Signal: 2nd Heikin-Ashi Candle Green + MACD Bullish Crossover
def check_buy_signal(df):
    second_last_candle = df.iloc[-2]
    if second_last_candle['HA_Close'] <= second_last_candle['HA_Open']:
        return False
    
    last_macd = df.iloc[-1]
    prev_macd = df.iloc[-2]
    
    if prev_macd['MACD'] <= prev_macd['MACD_Signal'] and last_macd['MACD'] > last_macd['MACD_Signal']:
        return True
    return False

# Sell Signal: 1st Heikin-Ashi Candle Red (Bearish)
def check_sell_signal(df):
    last_candle = df.iloc[-1]
    if last_candle['HA_Close'] < last_candle['HA_Open']:
        return True
    return False

# Place Buy Order
def place_buy_order(symbol, qty):
    try:
        api.submit_order(
            symbol=symbol,
            qty=qty,
            side='buy',
            type='market',
            time_in_force='gtc'
        )
        print(f"Buy order placed for {qty} shares of {symbol}")
    except Exception as e:
        print(f"Error placing buy order: {e}")

# Place Sell Order
def place_sell_order(symbol, qty):
    try:
        api.submit_order(
            symbol=symbol,
            qty=qty,
            side='sell',
            type='market',
            time_in_force='gtc'
        )
        print(f"Sell order placed for {qty} shares of {symbol}")
    except Exception as e:
        print(f"Error placing sell order: {e}")

# Real-time Trading Loop
symbol = 'AAPL'
quantity = 1
position = None

while True:
    # Fetch stock data
    df = get_stock_data(symbol)
    
    # Calculate Heikin-Ashi and MACD
    df = calculate_heikin_ashi(df)
    df = calculate_macd(df)
    
    # Check for buy signal
    if position is None and check_buy_signal(df):
        place_buy_order(symbol, quantity)
        position = 'long'
    
    # Check for sell signal
    elif position == 'long' and check_sell_signal(df):
        place_sell_order(symbol, quantity)
        position = None
    
    # Wait 2 minutes before checking again
    time.sleep(120)
- 👋 Hi, I’m @kgalal75
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
kgalal75/kgalal75 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
