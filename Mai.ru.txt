Мобільний API-сервер для додатку криптоаналітики ETH/USDT

Працює локально і може бути підключений до Android через HTTP

from fastapi import FastAPI from fastapi.middleware.cors import CORSMiddleware from fastapi.responses import JSONResponse from binance.client import Client from prophet import Prophet import pandas as pd import numpy as np import uvicorn

API_KEY = "mg3Pg6Vm8EOF7kIdZqmI4wyI044Zy0aWPs82LFqkC8xrlUbv86qXlnw1yCp2bL7B" API_SECRET = ""

app = FastAPI(title="Крипто Аналітика Mobile", description="Аналіз ETH/USDT для мобільного додатку")

app.add_middleware( CORSMiddleware, allow_origins=[""], allow_credentials=True, allow_methods=[""], allow_headers=["*"], )

client = Client(API_KEY, API_SECRET)

def get_klines(symbol="ETHUSDT", interval="1h", limit=1000): klines = client.get_klines(symbol=symbol, interval=interval, limit=limit) df = pd.DataFrame(klines, columns=[ 'timestamp', 'open', 'high', 'low', 'close', 'volume', 'close_time', 'quote_asset_volume', 'number_of_trades', 'taker_buy_base_volume', 'taker_buy_quote_volume', 'ignore' ]) df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms') df['close'] = df['close'].astype(float) return df[['timestamp', 'close']].rename(columns={"timestamp": "ds", "close": "y"})

def forecast_price(df): model = Prophet() model.fit(df) future = model.make_future_dataframe(periods=24, freq='H') forecast = model.predict(future) result = forecast[['ds', 'yhat']].tail(24) return result

@app.get("/api/forecast") def api_forecast(): try: df = get_klines() прогноз = forecast_price(df) data = прогноз.to_dict(orient='records') return JSONResponse(content={ "валюта": "ETH/USDT", "інтервал": "1h", "прогноз_на_24_години": data }) except Exception as e: return JSONResponse(status_code=500, content={"error": str(e)})

if name == "main": uvicorn.run(app, host="0.0.0.0", port=8000)

