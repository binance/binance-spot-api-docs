import ccxt
import pandas as pd
import numpy as np
from sklearn.model_selection import ParameterGrid
import time

# =========================
# تنظیمات اولیه
# =========================
symbol = 'BTC/USDT'
timeframe = '1h'
initial_balance = 1000
risk_per_trade = 0.01
update_interval = 3600  # هر یک ساعت بررسی و به‌روزرسانی

# =========================
# اتصال به Binance
# =========================
exchange = ccxt.binance({
    'rateLimit': 1200,
    'enableRateLimit': True,
    # 'apiKey': 'YOUR_API_KEY',
    # 'secret': 'YOUR_SECRET',
})

# =========================
# دریافت داده‌های تاریخی و لحظه‌ای
# =========================
def fetch_ohlcv(symbol, timeframe, limit=500):
    ohlcv = exchange.fetch_ohlcv(symbol, timeframe=timeframe, limit=limit)
    df = pd.DataFrame(ohlcv, columns=['timestamp','open','high','low','close','volume'])
    df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms')
    return df

# =========================
# افزودن اندیکاتورها
# =========================
def add_indicators(df, ema_short=20, ema_long=50, rsi_period=14):
    df['EMA_short'] = df['close'].ewm(span=ema_short, adjust=False).mean()
    df['EMA_long'] = df['close'].ewm(span=ema_long, adjust=False).mean()
    delta = df['close'].diff()
    gain = delta.clip(lower=0)
    loss = -delta.clip(upper=0)
    avg_gain = gain.rolling(window=rsi_period).mean()
    avg_loss = loss.rolling(window=rsi_period).mean()
    rs = avg_gain / avg_loss
    df['RSI'] = 100 - (100 / (1 + rs))
    return df

# =========================
# تولید سیگنال
# =========================
def generate_signal(df):
    signals = []
    for i in range(len(df)):
        if df['EMA_short'].iloc[i] > df['EMA_long'].iloc[i] and df['RSI'].iloc[i] < 70:
            signals.append('BUY')
        elif df['EMA_short'].iloc[i] < df['EMA_long'].iloc[i] and df['RSI'].iloc[i] > 30:
            signals.append('SELL')
        else:
            signals.append('HOLD')
    df['Signal'] = signals
    return df

# =========================
# Backtesting ساده
# =========================
def backtest(df, balance):
    position = 0
    equity = balance
    for i in range(1, len(df)):
        if df['Signal'].iloc[i] == 'BUY' and position == 0:
            position = equity * risk_per_trade / df['close'].iloc[i]
            equity -= position * df['close'].iloc[i]
        elif df['Signal'].iloc[i] == 'SELL' and position > 0:
            equity += position * df['close'].iloc[i]
            position = 0
    if position > 0:
        equity += position * df['close'].iloc[-1]
    return equity

# =========================
# بهینه‌سازی پارامترها با داده‌های گذشته
# =========================
def optimize_strategy(df):
    param_grid = {
        'ema_short': [10, 20, 25],
        'ema_long': [40, 50, 60],
        'rsi_period': [10, 14, 20]
    }
    best_params = None
    best_balance = -np.inf
    
    for params in ParameterGrid(param_grid):
        temp_df = add_indicators(df.copy(), **params)
        temp_df = generate_signal(temp_df)
        final_balance = backtest(temp_df, initial_balance)
        if final_balance > best_balance:
            best_balance = final_balance
            best_params = params
            
    return best_params, best_balance

# =========================
# اجرای ربات خودآموز
# =========================
def run_bot():
    balance = initial_balance
    while True:
        try:
            print("\n=== دریافت داده‌ها و به‌روزرسانی ربات ===")
            df = fetch_ohlcv(symbol, timeframe)
            
            # بهینه‌سازی استراتژی
            best_params, best_balance = optimize_strategy(df)
            print(f"بهترین پارامترها: {best_params}")
            print(f"سرمایه پیش‌بینی شده با Backtesting: {best_balance:.2f} USDT")
            
            # تولید سیگنال با بهترین پارامتر
            df = add_indicators(df, **best_params)
            df = generate_signal(df)
            last_signal = df['Signal'].iloc[-1]
            print(f"سیگنال آخرین کندل: {last_signal}")
            
            # این قسمت برای ارسال سفارش واقعی است (در صورت نیاز فعال شود)
            # if last_signal == 'BUY':
            #     # ارسال سفارش خرید
            # elif last_signal == 'SELL':
            #     # ارسال سفارش فروش
            
            # ذخیره گزارش
            df.to_csv("trade_report.csv", index=False)
            print("گزارش ذخیره شد.")

        except Exception as e:
            print("خطا در اجرا:", e)
        
        print(f"انتظار {update_interval} ثانیه تا اجرای بعدی...")
        time.sleep(update_interval)

# =========================
# شروع ربات
# =========================
if __name__ == "__main__":
    run_bot()
