# AURIUM_GOLD_bot
main.py
import requests
import json
from datetime import datetime

TOKEN = "PUT_TOKEN"
CHANNEL = "@AURIUMGOLD"

STATE_FILE = "state.json"

def load_last():
    try:
        with open(STATE_FILE, "r") as f:
            return json.load(f)
    except:
        return {}

def save_last(data):
    with open(STATE_FILE, "w") as f:
        json.dump(data, f)

def get_xau():
    return float(requests.get("https://api.gold-api.com/price/XAU").json()["price"])

def get_usd():
    return float(requests.get("https://api.exchangerate.host/latest?base=USD&symbols=IRR").json()["rates"]["IRR"])

def percent_change(new, old):
    if not old:
        return 0
    return ((new - old) / old) * 100

def send(msg):
    url = f"https://api.telegram.org/bot{TOKEN}/sendMessage"
    requests.post(url, data={"chat_id": CHANNEL, "text": msg})

def run():
    last = load_last()

    xau = get_xau()
    usd = get_usd()

    xau_change = percent_change(xau, last.get("xau"))
    usd_change = percent_change(usd, last.get("usd"))

    trend = "📈 صعودی" if xau_change > 0 else "📉 نزولی"

    gold_18 = (xau * usd) / 31.1 / 4.25

    time = datetime.now().strftime("%Y-%m-%d %H:%M")

    msg = f"""
💎 AURIUM GOLD | PREMIUM SYSTEM

🌍 اونس جهانی: {xau:.2f} $
📊 تغییر: {xau_change:+.2f}%

🇮🇷 دلار: {usd:,.0f} تومان
📊 تغییر: {usd_change:+.2f}%

🪙 طلای ۱۸ عیار: {gold_18:,.0f} تومان

⚡ وضعیت بازار: {trend}

🕒 {time}
"""

    send(msg)
    save_last({"xau": xau, "usd": usd})

run()
requirements.txt
requests
