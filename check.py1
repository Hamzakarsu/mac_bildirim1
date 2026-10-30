import os, json, requests
from bs4 import BeautifulSoup

URL = "https://www.ofv.swiss/ostschweizer-fussballverband/schiedsrichterin-ofv/spiele-ohne-sr-sra-ofv.aspx"
TOKEN = os.environ["TELEGRAM_TOKEN"]
CHAT = os.environ["TELEGRAM_CHAT_ID"]
START = "spiele ohne schiedsrichter"
END = "regelung spiele ohne"
EMPTY = "momentan sind keine spiele"

def tg(msg):
    requests.post(f"https://api.telegram.org/bot{TOKEN}/sendMessage",
                  data={"chat_id": CHAT, "text": msg}, timeout=30)

def load():
    try:
        return json.load(open("state.json"))
    except Exception:
        return {"lines": [], "error": False}

def save(s):
    json.dump(s, open("state.json", "w"), ensure_ascii=False)

def fetch_lines():
    r = requests.get(URL, timeout=30, headers={
        "User-Agent": "Mozilla/5.0",
        "Accept-Language": "de-CH,de;q=0.9"})
    r.raise_for_status()
    text = BeautifulSoup(r.text, "html.parser").get_text("\n", strip=True)
    low = text.lower()
    e = low.find(END)
    if e == -1:
        raise RuntimeError("sayfa yapisi anlasilamadi (bitis)")
    s = low.rfind(START, 0, e)
    if s == -1:
        raise RuntimeError("sayfa yapisi anlasilamadi (baslangic)")
    section = text[s:e]
    if EMPTY in section.lower():
        return []
    return [l for l in section.split("\n")[1:]
            if l.strip() and l.strip() != "(SR/SRA)"]

prev = load()
try:
    lines = fetch_lines()
except Exception as ex:
    if not prev.get("error"):
        tg(f"⚠️ Site okunamadi: {ex}")
    prev["error"] = True
    save(prev)
    raise SystemExit(0)

new = [l for l in lines if l not in prev["lines"]]
if new:
    tg("⚽ Yeni maç var!\n" + "\n".join(new[:15]) + "\n" + URL)
if os.environ.get("EVENT") == "workflow_dispatch":
    tg(f"✅ Sistem çalışıyor. Sayfada şu an {len(lines)} satır maç bilgisi var.")
save({"lines": lines, "error": False})
