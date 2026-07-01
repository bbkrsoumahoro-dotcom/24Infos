# 24Infos
#!/usr/bin/env python3
"""NEXUS DIAMOND v5 — Fonctionnel immédiat. Tout modifiable depuis l'interface."""

import os, sys, json, time, hashlib, re, html, threading, queue, sqlite3
from datetime import datetime, timezone, timedelta
from collections import Counter
from urllib.parse import urlparse

# Auto-install
for pkg, imp in [("flask","flask"),("feedparser","feedparser"),("bs4","bs4"),("requests","requests")]:
    try:
        __import__(imp)
    except:
        os.system(f"{sys.executable} -m pip install {pkg} --quiet")
        __import__(imp)

from flask import Flask, Response, request, jsonify, render_template_string
import feedparser
from bs4 import BeautifulSoup
import requests

# DB
conn = sqlite3.connect("nexus.db", check_same_thread=False)
conn.row_factory = sqlite3.Row
conn.execute("PRAGMA journal_mode=WAL")
conn.executescript("""
CREATE TABLE IF NOT EXISTS articles(id TEXT PRIMARY KEY, title TEXT, url TEXT, descr TEXT, source TEXT, slevel INT DEFAULT 70, region TEXT DEFAULT 'inter', level TEXT DEFAULT 'info', score INT DEFAULT 0, cat TEXT DEFAULT 'General', pub TEXT, col TEXT DEFAULT (datetime('now')), img TEXT, lu INT DEFAULT 0);
CREATE TABLE IF NOT EXISTS alertes(id TEXT PRIMARY KEY, title TEXT, descr TEXT, niveau TEXT DEFAULT 'haute', score INT DEFAULT 0, cat TEXT DEFAULT 'General', sources TEXT, scount INT DEFAULT 1, url TEXT, created TEXT DEFAULT (datetime('now')), lu INT DEFAULT 0);
CREATE TABLE IF NOT EXISTS settings(key TEXT PRIMARY KEY, val TEXT);
CREATE TABLE IF NOT EXISTS custom_sources(name TEXT, url TEXT, region TEXT, level INT);
CREATE TABLE IF NOT EXISTS stats(id INTEGER PRIMARY KEY AUTOINCREMENT, total INT, crit INT, haut INT, moy INT, src INT, alrt INT, ts TEXT DEFAULT (datetime('now')));
INSERT OR IGNORE INTO settings VALUES('theme','dark'),('interval','180'),('hours','24'),('max_articles','50000'),('extract_images','1');
""")
conn.commit()

KW_CRIT = ["meurtre","assassinat","massacre","attentat","explosion","attaque","fusillade","coup d'etat","putsch","insurrection","guerre","conflit arme","terrorisme","epidemie","pandemie","ebola","inondation","seisme","accident mortel","naufrage","crash","incendie","drogue","cocaine","arme de guerre","enlevement","viol","execution","etat d'urgence","couvre-feu","cyberattaque","brèche","anonymous","hacktivisme"]
KW_HAUT = ["securite","armee","police","gendarmerie","presidence","president","ministre","gouvernement","corruption","detournement","scandale","cybercriminalite","piratage","election","vote","crise politique","diplomatie","defense","renseignement","manifestation","greve","sanction","embargo","droits humains","anonymous","data leak","fuite","ddos","ransomware","hack"]
KW_MOY = ["sante","hopital","education","ecole","universite","economie","investissement","agriculture","culture","sport","technologie","environnement","nomination","reforme","loi","decret","accident","religion"]
ALL_KW = list(set(KW_CRIT + KW_HAUT + KW_MOY))

RSS = [
    ("Fraternité Matin","https://www.fratmat.info/rss","nat",100),("RTI","https://www.rti.ci/rss","nat",100),("Abidjan.net","https://news.abidjan.net/rss","nat",100),("AIP","https://www.aip.ci/feed/","nat",100),("7Info","https://7info.ci/rss","nat",95),("Soir Info","https://www.soirinfo.com/rss","nat",95),("L'Inter","https://www.linterci.com/rss","nat",95),("Koaci","https://www.koaci.com/rss","nat",95),("Linfodrome","https://www.linfodrome.com/24h?format=feed","nat",95),("Connection Ivoirienne","https://www.connectionivoirienne.net/feed/","nat",90),("Jeune Afrique","https://www.jeuneafrique.com/feed/","cont",100),("RFI Afrique","https://www.rfi.fr/fr/afrique/rss","cont",100),("BBC Afrique","https://www.bbc.com/afrique/rss.xml","cont",100),("Le Monde Afrique","https://www.lemonde.fr/afrique/rss_full.xml","cont",100),("AfricaNews","https://www.africanews.com/feed/","cont",95),("France24 Afrique","https://www.france24.com/fr/afrique/rss","cont",95),("Al Jazeera","https://www.aljazeera.com/xml/rss/all.xml","cont",95),("Crisis Group","https://www.crisisgroup.org/rss.xml","inter",95),("HRW","https://www.hrw.org/rss/africa","cont",95),("ISS Africa","https://issafrica.org/rss.xml","cont",95),("ONU Info","https://news.un.org/feed/subscribe/en/news/region/africa/feed/rss.xml","cont",100),("Interpol","https://www.interpol.int/rss","inter",100),("Krebs","https://krebsonsecurity.com/feed/","inter",90),("The Hacker News","https://feeds.feedburner.com/TheHackersNews","inter",85),("Bleeping Computer","https://www.bleepingcomputer.com/feed/","inter",85),("Cisco Talos","https://blog.talosintelligence.com/feed/","inter",90),("CrowdStrike","https://www.crowdstrike.com/blog/feed/","inter",85),("Anonymous News","https://www.anonymous.news/feed/","inter",80),("HackRead","https://www.hackread.com/feed/","inter",80),("Security Affairs","https://securityaffairs.com/feed","inter",80),("Schneier","https://www.schneier.com/feed/","inter",85),("Troy Hunt","https://www.troyhunt.com/rss/","inter",85),("AnonHQ","https://anonhq.com/feed/","inter",75),("GNews CI","https://news.google.com/rss/search?q=C%C3%B4te+d%27Ivoire+apr%C3%A8s+2025&hl=fr&gl=CI&ceid=CI:fr","nat",95),("GNews Securite","https://news.google.com/rss/search?q=s%C3%A9curit%C3%A9+attaque+C%C3%B4te+d%27Ivoire&hl=fr&gl=CI&ceid=CI:fr","nat",95),("GNews Cyber","https://news.google.com/rss/search?q=cybercriminalit%C3%A9+Afrique+hacking&hl=fr&gl=CI&ceid=CI:fr","cont",90),("GNews Terrorisme","https://news.google.com/rss/search?q=terrorisme+Sahel+Afrique+attentat&hl=fr&gl=CI&ceid=CI:fr","cont",95),("GNews Conflits","https://news.google.com/rss/search?q=conflit+guerre+Afrique+monde&hl=fr&gl=CI&ceid=CI:fr","inter",90),
]

session = requests.Session()
session.headers.update({"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"})
sse_queue = queue.Queue()

def nml(t):
    if not t: return ""
    t = t.lower()
    for a,b in [('é','e'),('è','e'),('ê','e'),('ë','e'),('à','a'),('â','a'),('ä','a'),('ù','u'),('û','u'),('ü','u'),('ô','o'),('ö','o'),('î','i'),('ï','i'),('ç','c')]:
        t = t.replace(a,b)
    return t

def detect_lv(t,d=""):
    txt = nml(f"{t} {d[:200]}")
    for k in KW_CRIT:
        if k in txt: return "critique", [k]
    for k in KW_HAUT:
        if k in txt: return "haute", [k]
    for k in KW_MOY:
        if k in txt: return "moyenne", [k]
    return "info", []

def calc_score(t,d,sl,lv):
    s = sl
    txt = nml(f"{t} {d[:200]}")
    if lv=="critique": s+=50
    elif lv=="haute": s+=30
    elif lv=="moyenne": s+=15
    for k in ALL_KW:
        if k in txt: s+=2
    return min(s,200)

def detect_cat(t,d):
    txt = nml(f"{t} {d[:200]}")
    cats = {"Politique":["gouvernement","president","ministre","election","vote","parti","loi","reforme"],"Securite":["armee","police","gendarmerie","securite","defense","terrorisme","attaque","criminalite"],"Cyber":["cyberattaque","piratage","hack","ransomware","virus","malware","breche","faille","anonymous","ddos"],"Economie":["economie","finance","banque","marche","investissement","pib","inflation","bceao","brvm"],"Diplomatie":["diplomatie","ambassade","onu","ua","cedeao","cooperation","sommet"],"Societe":["societe","education","sante","culture","religion","jeunesse"],"Justice":["justice","tribunal","proces","mandat","enquete","juge","prison"]}
    for c,m in cats.items():
        if any(k in txt for k in m): return c
    return "General"

def detect_reg(t,d,sr,sn=""):
    txt = nml(f"{t} {d[:100]}")
    ci = ["cote d'ivoire","abidjan","ivoirien","yopougon","cocody","bouake","gbagbo","ouattara","bedie","soro"]
    af = ["afrique","mali","burkina","niger","guinee","senegal","ghana","nigeria","cameroun","rdc","sahel"]
    for k in ci:
        if k in txt: return "nat"
    for k in af:
        if k in txt: return "cont"
    return sr

def get_set(k, defv=""):
    r = conn.execute("SELECT val FROM settings WHERE key=?",(k,)).fetchone()
    return r["val"] if r else defv

def set_set(k,v):
    conn.execute("INSERT OR REPLACE INTO settings VALUES(?,?)",(k,v))
    conn.commit()

def scrape():
    arts = []
    sources = RSS
    cs = conn.execute("SELECT * FROM custom_sources").fetchall()
    for s in cs:
        sources.append((s["name"],s["url"],s["region"] or "nat",s["level"] or 70))
    hours = int(get_set("hours","24"))
    cutoff = datetime.now(timezone.utc) - timedelta(hours=hours)
    for name,url,region,level in sources:
        try:
            time.sleep(0.03)
            r = session.get(url, timeout=10)
            if r.status_code!=200: continue
            feed = feedparser.parse(r.content)
            for e in feed.entries[:20]:
                t = (e.get("title","") or "").strip()
                l = (e.get("link","") or "").strip()
                if not t or not l: continue
                d = BeautifulSoup(e.get("summary","") or e.get("description","") or "","html.parser").get_text()[:400]
                pub = datetime.now(timezone.utc)
                if e.get("published_parsed"):
                    try: pub = datetime(*e.published_parsed[:6], tzinfo=timezone.utc)
                    except: pass
                if pub < cutoff: continue
                txt = nml(f"{t} {d}")
                matched = [k for k in ALL_KW if k in txt]
                if not matched: continue
                uid = hashlib.sha256(f"{name}:{l}".encode()).hexdigest()[:16]
                lv,kw = detect_lv(t,d)
                reg = detect_reg(t,d,region,name)
                cat = detect_cat(t,d)
                sc = calc_score(t,d,level,lv)
                arts.append({"id":uid,"title":t[:300],"url":l,"descr":d[:400],"source":name,"slevel":level,"region":reg,"level":lv,"score":sc,"cat":cat,"pub":pub.isoformat(),"img":""})
        except: pass
    return arts

def save(arts):
    saved=0
    for a in arts:
        try:
            conn.execute("INSERT OR IGNORE INTO articles VALUES(?,?,?,?,?,?,?,?,?,?,?,datetime('now'),?,0)",(a["id"],a["title"],a["url"],a["descr"],a["source"],a["slevel"],a["region"],a["level"],a["score"],a["cat"],a["pub"],a["img"]))
            if conn.total_changes: saved+=1
        except: pass
    conn.commit()
    return saved

def detect_alert():
    hours = int(get_set("hours","24"))
    cutoff = (datetime.now(timezone.utc)-timedelta(hours=hours)).isoformat()
    rows = conn.execute("SELECT * FROM articles WHERE level IN ('critique','haute') AND col > ? ORDER BY score DESC",(cutoff,)).fetchall()
    groups = {}
    for r in rows:
        words = set(nml(r["title"]).split())
        sw = sorted([w for w in words if len(w)>=4])[:5]
        if not sw: continue
        key = "_".join(sw)
        if key not in groups: groups[key]=[]
        groups[key].append(r)
    nv=0
    for k,items in groups.items():
        if len(items)<2: continue
        sources = list(set(i["source"] for i in items))
        if len(sources)<2: continue
        best = max(items, key=lambda x:x["score"])
        aid = hashlib.sha256(f"al:{k}".encode()).hexdigest()[:16]
        if conn.execute("SELECT id FROM alertes WHERE id=?",(aid,)).fetchone(): continue
        nv_ = "critique" if any(i["level"]=="critique" for i in items) else "haute"
        conn.execute("INSERT OR IGNORE INTO alertes VALUES(?,?,?,?,?,?,?,?,?,datetime('now'),0)",(aid,best["title"][:200],best["descr"][:300],nv_,best["score"]+len(sources)*10,best["cat"],json.dumps(sources),len(sources),best["url"]))
        if conn.total_changes: nv+=1
    conn.commit()
    return nv

def update_stats():
    hours = int(get_set("hours","24"))
    cutoff = (datetime.now(timezone.utc)-timedelta(hours=hours)).isoformat()
    t = conn.execute("SELECT COUNT(*) FROM articles WHERE col>?",(cutoff,)).fetchone()[0]
    c = conn.execute("SELECT COUNT(*) FROM articles WHERE level='critique' AND col>?",(cutoff,)).fetchone()[0]
    h = conn.execute("SELECT COUNT(*) FROM articles WHERE level='haute' AND col>?",(cutoff,)).fetchone()[0]
    m = conn.execute("SELECT COUNT(*) FROM articles WHERE level='moyenne' AND col>?",(cutoff,)).fetchone()[0]
    s = conn.execute("SELECT COUNT(DISTINCT source) FROM articles WHERE col>?",(cutoff,)).fetchone()[0]
    a = conn.execute("SELECT COUNT(*) FROM alertes WHERE created>?",(cutoff,)).fetchone()[0]
    conn.execute("INSERT INTO stats(total,crit,haut,moy,src,alrt) VALUES(?,?,?,?,?,?)",(t,c,h,m,s,a))
    conn.commit()
    return {"total":t,"critiques":c,"hautes":h,"moyennes":m,"sources":s,"alertes":a}

class Scraper(threading.Thread):
    def run(self):
        while True:
            start=time.time()
            print(f"[{datetime.now().strftime('%H:%M:%S')}] Scraping {len(RSS)} sources...")
            arts=scrape()
            s=save(arts)
            a=detect_alert()
            st=update_stats()
            print(f"  +{s} articles | {a} alertes | {st['total']} total")
            if s>0: sse_queue.put({"type":"new_articles","data":{"count":s,"stats":st}})
            if a>0: sse_queue.put({"type":"new_alertes","data":{"count":a}})
            sse_queue.put({"type":"stats","data":st})
            slp=max(1,int(get_set("interval","180"))-(time.time()-start))
            time.sleep(slp)

app = Flask(__name__)

@app.route("/api/stats")
def api_stats():
    st=update_stats()
    anl=conn.execute("SELECT COUNT(*) FROM alertes WHERE lu=0").fetchone()[0]
    st["alertes_non_lues"]=anl
    st["themes"]=["dark","clair","noir","bleu","vert","rouge","violet","or"]
    st["current_theme"]=get_set("theme","dark")
    st["interval"]=get_set("interval","180")
    st["hours"]=get_set("hours","24")
    return jsonify(st)

@app.route("/api/articles")
def api_articles():
    lim=min(int(request.args.get("limit",50)),500)
    off=int(request.args.get("offset",0))
    lv=request.args.get("level","all")
    reg=request.args.get("region","all")
    cat=request.args.get("category","all")
    src=request.args.get("source","all")
    q=request.args.get("search","").strip()
    sort=request.args.get("sort","score")
    hours=int(get_set("hours","24"))
    cutoff=(datetime.now(timezone.utc)-timedelta(hours=hours)).isoformat()
    query="SELECT * FROM articles WHERE col>?"
    params=[cutoff]
    if lv!="all": query+=" AND level=?"; params.append(lv)
    if reg!="all": query+=" AND region=?"; params.append(reg)
    if cat!="all": query+=" AND cat=?"; params.append(cat)
    if src!="all": query+=" AND source=?"; params.append(src)
    if q: query+=" AND (title LIKE ? OR descr LIKE ?)"; params.extend([f"%{q}%",f"%{q}%"])
    total=conn.execute(query.replace("SELECT *","SELECT COUNT(*)"),params).fetchone()[0]
    query+=f" ORDER BY {'pub' if sort=='date' else 'score'} DESC LIMIT ? OFFSET ?"
    params.extend([lim,off])
    rows=conn.execute(query,params).fetchall()
    return jsonify({"articles":[dict(r) for r in rows],"total":total})

@app.route("/api/alertes")
def api_alertes():
    hours=int(get_set("hours","24"))
    cutoff=(datetime.now(timezone.utc)-timedelta(hours=hours)).isoformat()
    rows=conn.execute("SELECT * FROM alertes WHERE created>? ORDER BY score DESC LIMIT 100",(cutoff,)).fetchall()
    al=[]
    for r in rows:
        a=dict(r)
        a["sources"]=json.loads(a["sources"]) if a["sources"] else []
        al.append(a)
    return jsonify({"alertes":al})

@app.route("/api/sources")
def api_sources():
    hours=int(get_set("hours","24"))
    cutoff=(datetime.now(timezone.utc)-timedelta(hours=hours)).isoformat()
    rows=conn.execute("""SELECT source,slevel,region,COUNT(*) as ct,SUM(CASE WHEN level='critique' THEN 1 ELSE 0 END) as crit,SUM(CASE WHEN level='haute' THEN 1 ELSE 0 END) as haut FROM articles WHERE col>? GROUP BY source ORDER BY ct DESC""",(cutoff,)).fetchall()
    cs=conn.execute("SELECT * FROM custom_sources").fetchall()
    return jsonify({"sources":[dict(r) for r in rows],"custom":[dict(c) for c in cs]})

@app.route("/api/categories")
def api_categories():
    hours=int(get_set("hours","24"))
    cutoff=(datetime.now(timezone.utc)-timedelta(hours=hours)).isoformat()
    rows=conn.execute("SELECT cat,COUNT(*) as ct,SUM(CASE WHEN level='critique' THEN 1 ELSE 0 END) as crit FROM articles WHERE col>? GROUP BY cat ORDER BY ct DESC",(cutoff,)).fetchall()
    return jsonify({"categories":[dict(r) for r in rows]})

@app.route("/api/historique")
def api_historique():
    rows=conn.execute("SELECT * FROM stats ORDER BY ts DESC LIMIT 50").fetchall()
    return jsonify({"historique":[dict(r) for r in rows]})

@app.route("/api/settings",methods=["GET","POST"])
def api_settings():
    if request.method=="POST":
        d=request.json
        for k,v in d.items():
            set_set(k,str(v))
        return jsonify({"success":True})
    return jsonify({k:get_set(k) for k in ["theme","interval","hours","max_articles","extract_images"]})

@app.route("/api/sources/add",methods=["POST"])
def api_add_source():
    d=request.json
    conn.execute("INSERT INTO custom_sources VALUES(?,?,?,?)",(d["name"],d["url"],d.get("region","nat"),int(d.get("level",70))))
    conn.commit()
    return jsonify({"success":True})

@app.route("/api/sources/delete/<name>",methods=["DELETE"])
def api_del_source(name):
    conn.execute("DELETE FROM custom_sources WHERE name=?",(name,))
    conn.commit()
    return jsonify({"success":True})

@app.route("/api/alertes/read-all",methods=["POST"])
def api_alertes_read():
    conn.execute("UPDATE alertes SET lu=1")
    conn.commit()
    return jsonify({"success":True})

@app.route("/api/alerte/<id>/read",methods=["POST"])
def api_alerte_read(id):
    conn.execute("UPDATE alertes SET lu=1 WHERE id=?",(id,))
    conn.commit()
    return jsonify({"success":True})

@app.route("/api/scan",methods=["POST"])
def api_scan():
    threading.Thread(target=lambda:(save(scrape()),detect_alert(),update_stats(),sse_queue.put({"type":"scan_done"})),daemon=True).start()
    return jsonify({"success":True})

@app.route("/api/logs")
def api_logs():
    hours=int(get_set("hours","24"))
    cutoff=(datetime.now(timezone.utc)-timedelta(hours=hours)).isoformat()
    rows=conn.execute("SELECT strftime('%H:%M',ts) as time,total,crit,haut FROM stats ORDER BY ts DESC LIMIT 24").fetchall()
    return jsonify({"logs":[dict(r) for r in reversed(rows)]})

@app.route("/stream")
def stream():
    def gen():
        while True:
            try:
                e=sse_queue.get(timeout=30)
                yield f"data: {json.dumps(e,ensure_ascii=False)}\n\n"
            except queue.Empty:
                yield ": keepalive\n\n"
    return Response(gen(),mimetype="text/event-stream",headers={"Cache-Control":"no-cache","X-Accel-Buffering":"no"})

@app.route("/")
def index():
    return render_template_string(HTML)

# ═══════════════════════════════════════════
# FRONTEND HTML COMPLET
# ═══════════════════════════════════════════

HTML = r"""<!DOCTYPE html><html lang="fr"><head>
<meta charset="UTF-8"><meta name="viewport" content="width=device-width,initial-scale=1">
<title>⟡ NEXUS DIAMOND v5</title>
<style>
*{margin:0;padding:0;box-sizing:border-box;font-family:system-ui,-apple-system,sans-serif;transition:background .2s,color .2s}
:root{--bg:#060810;--sf:#0c0f1c;--cd:#121828;--cd2:#181f38;--tx:#eef2fa;--tx2:#8898c8;--ac:#00e5ff;--ac2:#ff6b35;--gr:#00ff88;--rd:#ff0030;--or:#ff6b00;--ye:#ffcc00;--pu:#aa66ff;--gd:#ffd700;--br:#1e2a4a;--r:12px}
.theme-clair{--bg:#f0f2f8;--sf:#e4e7f0;--cd:#fff;--cd2:#f4f6fc;--tx:#1a1d2e;--tx2:#5a6a8a;--br:#d0d6e4}
.theme-noir{--bg:#000;--sf:#0a0a0a;--cd:#111;--cd2:#1a1a1a;--tx:#fff;--tx2:#888;--br:#222}
.theme-bleu{--bg:#0a1628;--sf:#0f1f3a;--cd:#142850;--cd2:#1a3366;--tx:#e0eeff;--tx2:#7a9ec8;--br:#1e4a7a}
.theme-vert{--bg:#0a1a0a;--sf:#0f2a0f;--cd:#143a14;--cd2:#1a4a1a;--tx:#e0ffe0;--tx2:#7ac87a;--br:#1e4a1e}
.theme-rouge{--bg:#1a0a0a;--sf:#2a0f0f;--cd:#3a1414;--cd2:#4a1a1a;--tx:#ffe0e0;--tx2:#c87a7a;--br:#4a1e1e}
.theme-violet{--bg:#100a1a;--sf:#1a0f2a;--cd:#24143a;--cd2:#2e1a4a;--tx:#eee0ff;--tx2:#9a7ac8;--br:#3a1e5a}
.theme-or{--bg:#1a180a;--sf:#2a2610;--cd:#3a3416;--cd2:#4a421c;--tx:#fffae0;--tx2:#c8b87a;--br:#5a4e1e}
body{background:var(--bg);color:var(--tx);min-height:100vh}
.nav{position:fixed;top:0;left:0;right:0;height:48px;background:rgba(8,11,22,.97);border-bottom:1px solid var(--br);display:flex;align-items:center;padding:0 12px;z-index:1000;gap:8px}
.nav-logo{font-size:15px;font-weight:800;background:linear-gradient(135deg,#00e5ff,#aa66ff);-webkit-background-clip:text;-webkit-text-fill-color:transparent;white-space:nowrap}
.nav-logo span{display:inline-block;width:28px;height:28px;line-height:28px;text-align:center;background:linear-gradient(135deg,#00e5ff,#aa66ff);border-radius:6px;color:#fff;font-size:12px;-webkit-text-fill-color:#fff;margin-right:4px}
.nav-items{display:flex;gap:2px;overflow-x:auto;flex:1;scrollbar-width:none}
.nav-items::-webkit-scrollbar{display:none}
.nav-items button{padding:5px 10px;border-radius:6px;border:none;background:transparent;color:var(--tx2);font-size:10px;cursor:pointer;white-space:nowrap;font-weight:500}
.nav-items button:hover{color:var(--tx);background:var(--cd)}
.nav-items button.active{color:var(--ac);background:var(--cd)}
.nav-info{display:flex;align-items:center;gap:6px;font-size:9px;color:var(--tx2);flex-shrink:0}
.nav-info .dot{width:6px;height:6px;border-radius:50%;background:var(--gr);animation:pulse 1.5s infinite}
@keyframes pulse{0%,100%{opacity:1}50%{opacity:.3}}
.content{padding:60px 12px 12px;max-width:1400px;margin:0 auto}
.page{display:none}.page.active{display:block;animation:fade .15s}
@keyframes fade{from{opacity:0}to{opacity:1}}
.grid{display:grid;gap:6px}.g2{grid-template-columns:repeat(auto-fit,minmax(320px,1fr))}.g3{grid-template-columns:repeat(auto-fit,minmax(250px,1fr))}.g4{grid-template-columns:repeat(auto-fit,minmax(200px,1fr))}
.card{background:var(--cd);border:1px solid var(--br);border-radius:var(--r);padding:12px}
.card:hover{border-color:var(--ac);transform:translateY(-1px);box-shadow:0 4px 20px rgba(0,0,0,.4)}
.stats{display:grid;grid-template-columns:repeat(auto-fit,minmax(90px,1fr));gap:6px;margin-bottom:8px}
.stat-card{background:var(--cd);border:1px solid var(--br);border-radius:10px;padding:10px 6px;text-align:center}
.stat-card .num{font-size:20px;font-weight:800;color:var(--ac)}
.stat-card .label{font-size:8px;color:var(--tx2);text-transform:uppercase;margin-top:1px}
.badge{display:inline-flex;padding:1px 8px;border-radius:12px;font-size:8px;font-weight:700}
.badge.critique{background:var(--rd);color:#fff}.badge.haute{background:var(--or);color:#fff}.badge.moyenne{background:var(--ye);color:#000}.badge.info{background:var(--ac);color:#000}
.item{padding:6px 8px;background:var(--cd2);border-radius:6px;margin-bottom:3px;border-left:3px solid var(--ac);cursor:pointer}
.item:hover{background:var(--cd)}
.item.critique{border-left-color:var(--rd)}.item.haute{border-left-color:var(--or)}.item.moyenne{border-left-color:var(--ye)}
.item-title{font-size:10px;font-weight:600;overflow:hidden;display:-webkit-box;-webkit-line-clamp:2;-webkit-box-orient:vertical}
.item-meta{font-size:8px;color:var(--tx2);margin-top:1px;display:flex;gap:4px;flex-wrap:wrap}
.alert-box{padding:8px 12px;background:rgba(255,0,48,.08);border:1px solid var(--rd);border-radius:8px;margin-bottom:4px;cursor:pointer}
.alert-box:hover{background:rgba(255,0,48,.12)}
.alert-box.haute{background:rgba(255,107,0,.08);border-color:var(--or)}
.alert-box.haute:hover{background:rgba(255,107,0,.12)}
.tabs{display:flex;gap:2px;margin-bottom:8px;overflow-x:auto;scrollbar-width:none;border-bottom:1px solid var(--br)}
.tabs button{padding:5px 12px;border:none;background:transparent;color:var(--tx2);font-size:9px;cursor:pointer;white-space:nowrap}
.tabs button:hover{color:var(--tx)}
.tabs button.active{background:var(--cd);color:var(--ac);border-radius:6px 6px 0 0}
.search-bar{display:flex;gap:6px;flex-wrap:wrap;align-items:center;margin-bottom:8px;padding:8px 12px;background:var(--cd);border-radius:var(--r);border:1px solid var(--br)}
.search-bar input,.search-bar select{flex:1;min-width:80px;background:var(--sf);border:1px solid var(--br);padding:5px 8px;border-radius:6px;color:var(--tx);font-size:9px}
.search-bar input:focus,.search-bar select:focus{border-color:var(--ac);outline:none}
.btn{padding:5px 12px;border-radius:6px;border:none;font-size:9px;font-weight:600;cursor:pointer}
.btn-primary{background:linear-gradient(135deg,#00e5ff,#aa66ff);color:#fff}
.btn-primary:hover{box-shadow:0 2px 12px rgba(0,229,255,.3)}
.btn-outline{background:transparent;border:1px solid var(--br);color:var(--tx2)}
.btn-outline:hover{border-color:var(--ac);color:var(--ac)}
.btn-danger{background:var(--rd);color:#fff}
.btn-sm{padding:3px 8px;font-size:8px}
.btn-lg{padding:8px 16px;font-size:11px}
.floating-label{position:relative;margin-bottom:6px}
.floating-label input,.floating-label textarea,.floating-label select{width:100%;background:var(--sf);border:1px solid var(--br);padding:10px 10px 4px;border-radius:6px;color:var(--tx);font-size:9px;outline:none}
.floating-label input:focus,.floating-label textarea:focus,.floating-label select:focus{border-color:var(--ac);box-shadow:0 0 0 2px rgba(0,229,255,.1)}
.floating-label label{position:absolute;top:8px;left:10px;font-size:9px;color:var(--tx2);pointer-events:none;transition:all .15s}
.floating-label input:focus~label,.floating-label input:not(:placeholder-shown)~label,.floating-label textarea:focus~label,.floating-label textarea:not(:placeholder-shown)~label{top:2px;font-size:7px;color:var(--ac)}
.modal{display:none;position:fixed;top:0;left:0;right:0;bottom:0;background:rgba(0,0,0,.8);z-index:2000;align-items:center;justify-content:center;backdrop-filter:blur(8px)}
.modal.open{display:flex}
.modal-content{background:var(--cd);border:1px solid var(--br);border-radius:14px;padding:16px;max-width:500px;width:92%;max-height:80vh;overflow-y:auto;animation:slide .2s}
@keyframes slide{from{transform:translateY(10px);opacity:0}to{transform:translateY(0);opacity:1}}
.modal-close{position:absolute;top:8px;right:8px;background:var(--cd2);border:1px solid var(--br);color:var(--tx2);width:26px;height:26px;border-radius:50%;cursor:pointer;display:flex;align-items:center;justify-content:center;font-size:11px}
.modal-close:hover{background:var(--rd);color:#fff;border-color:var(--rd)}
.toast{position:fixed;bottom:60px;left:50%;transform:translateX(-50%);background:var(--cd);border:1px solid var(--ac);padding:8px 16px;border-radius:8px;font-size:9px;z-index:9999;box-shadow:0 4px 20px rgba(0,0,0,.5);animation:tin .2s}
@keyframes tin{from{opacity:0;transform:translateX(-50%) translateY(10px)}to{opacity:1;transform:translateX(-50%) translateY(0)}}
::-webkit-scrollbar{width:3px;height:3px}
::-webkit-scrollbar-track{background:transparent}
::-webkit-scrollbar-thumb{background:var(--br);border-radius:3px}
@media(max-width:768px){.nav-items{display:none}.stats{grid-template-columns:repeat(2,1fr)}.g2,.g3,.g4{grid-template-columns:1fr}}
</style></head><body>
<div class="nav">
  <div class="nav-logo"><span>⟡</span> NEXUS DIAMOND v5</div>
  <div class="nav-items" id="navItems"></div>
  <div class="nav-info"><span class="dot"></span><span id="liveInfo">0 arts</span></div>
</div>
<div class="content" id="content"></div>

<script>
// APP
let state={page:'dashboard',articles:[],alertes:[],stats:{}};
const pages=['dashboard','flux','alertes','sources','categories','settings','historique'];
const icons={'dashboard':'📊','flux':'📡','alertes':'🚨','sources':'📰','categories':'📁','settings':'⚙️','historique':'📋'};

function init(){
  const nav=document.getElementById('navItems');
  nav.innerHTML=pages.map(p=>`<button class="${p==='dashboard'?'active':''}" onclick="showPage('${p}')">${icons[p]} ${p.charAt(0).toUpperCase()+p.slice(1)}</button>`).join('');
  showPage('dashboard');
  connectSSE();
  setInterval(loadStats,10000);
}

function showPage(p){
  state.page=p;
  document.querySelectorAll('.nav-items button').forEach(b=>b.classList.remove('active'));
  document.querySelector(`.nav-items button[onclick*="'${p}'"]`)?.classList.add('active');
  const fns={dashboard:loadDashboard,flux:loadFlux,alertes:loadAlertes,sources:loadSources,categories:loadCategories,settings:loadSettings,historique:loadHistorique};
  if(fns[p]) fns[p]();
}

function toast(m){const t=document.createElement('div');t.className='toast';t.textContent=m;document.body.appendChild(t);setTimeout(()=>t.remove(),2000)}
function api(p){return fetch(p).then(r=>r.json())}
function apiPost(p,d){return fetch(p,{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(d||{})}).then(r=>r.json())}

async function loadStats(){
  const s=await api('/api/stats');
  state.stats=s;
  document.getElementById('liveInfo').textContent=`${s.total||0} arts · ${s.alertes_non_lues||0} alertes`;
}

async function loadDashboard(){
  const s=state.stats.total?state.stats:await api('/api/stats');
  Object.assign(state.stats,s);
  const al=await api('/api/alertes');
  const art=await api('/api/articles?limit=8&sort=score');
  document.getElementById('content').innerHTML=`
    <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:6px;margin-bottom:8px">
      <div><h2 style="font-size:16px">📊 Dashboard</h2><p style="font-size:9px;color:var(--tx2)">Tout est modifiable depuis l'interface</p></div>
      <button class="btn btn-primary btn-sm" onclick="apiPost('/api/scan');toast('📡 Scan lancé')">📡 Scanner</button>
    </div>
    <div class="stats">${['total','critiques','hautes','moyennes','sources','alertes_non_lues'].map(k=>`<div class="stat-card"><div class="num" style="color:${k==='critiques'?'var(--rd)':k==='alertes_non_lues'?'var(--gd)':k==='hautes'?'var(--or)':'var(--ac)'}">${s[k]||0}</div><div class="label">${k.replace('_',' ')}</div></div>`).join('')}</div>
    <div class="grid g2">
      <div class="card"><div class="flex" style="display:flex;justify-content:space-between;margin-bottom:6px"><b style="font-size:11px">🚨 Alertes</b><span class="badge critique">${s.alertes_non_lues||0}</span></div>
        <div id="dashAlertes">${(al.alertes||[]).slice(0,6).map(a=>`<div class="alert-box ${a.niveau}" onclick="apiPost('/api/alerte/${a.id}/read');loadDashboard()"><b style="font-size:9px">${a.niveau==='critique'?'🔴':'🟠'} ${(a.title||'').substring(0,70)}</b><div style="font-size:7px;color:var(--tx2)">${a.scount} sources · Score: ${a.score}</div></div>`).join('')||'<div style="font-size:9px;color:var(--tx2);text-align:center;padding:10px">✅ Aucune alerte</div>'}</div></div>
      <div class="card"><div style="display:flex;justify-content:space-between;margin-bottom:6px"><b style="font-size:11px">🔥 Top articles</b></div>
        <div id="dashArticles">${(art.articles||[]).slice(0,8).map(a=>`<div class="item ${a.level}" onclick="window.open('${a.url||''}','_blank')"><div class="item-title">${a.level==='critique'?'🔴':a.level==='haute'?'🟠':'🟡'} ${(a.title||'').substring(0,70)}</div><div class="item-meta"><span>${a.source}</span><span>·</span><span>${a.score}pts</span><span>·</span><span>${a.region}</span></div></div>`).join('')||'<div style="font-size:9px;color:var(--tx2);text-align:center;padding:10px">Lancez un scan</div>'}</div></div>
    </div>`;
}

async function loadFlux(){
  const q=document.getElementById('fSearch')?.value||'';
  const lv=document.getElementById('fLevel')?.value||'all';
  const reg=document.getElementById('fRegion')?.value||'all';
  const cat=document.getElementById('fCat')?.value||'all';
  let url=`/api/articles?limit=60&sort=score&level=${lv}&region=${reg}&category=${cat}`;
  if(q) url+=`&search=${encodeURIComponent(q)}`;
  const data=await api(url);
  document.getElementById('content').innerHTML=`
    <h2 style="font-size:15px;margin-bottom:4px">📡 Flux temps réel</h2>
    <div class="search-bar">
      <input id="fSearch" placeholder="🔍 Mots-clés..." oninput="debounce(loadFlux,300)" value="${q}">
      <select id="fLevel" onchange="loadFlux()"><option value="all">Tous</option><option value="critique" ${lv==='critique'?'selected':''}>🔴 Critique</option><option value="haute" ${lv==='haute'?'selected':''}>🟠 Haute</option><option value="moyenne" ${lv==='moyenne'?'selected':''}>🟡 Moyenne</option><option value="info" ${lv==='info'?'selected':''}>🗞️ Info</option></select>
      <select id="fRegion" onchange="loadFlux()"><option value="all">🌍 Toutes</option><option value="nat" ${reg==='nat'?'selected':''}>🔵 National</option><option value="cont" ${reg==='cont'?'selected':''}>🟢 Continental</option><option value="inter" ${reg==='inter'?'selected':''}>🔴 International</option></select>
      <select id="fCat" onchange="loadFlux()"><option value="all">📁 Toutes</option>${['Politique','Securite','Cyber','Economie','Diplomatie','Societe','Justice'].map(c=>`<option value="${c}" ${cat===c?'selected':''}>${c}</option>`).join('')}</select>
    </div>
    <div style="font-size:8px;color:var(--tx2);margin-bottom:4px">${data.total} résultats</div>
    <div id="fluxList">${(data.articles||[]).map(a=>`<div class="item ${a.level}" onclick="window.open('${a.url||''}','_blank')"><div class="item-title">${a.level==='critique'?'🔴':a.level==='haute'?'🟠':a.level==='moyenne'?'🟡':'🔵'} ${(a.title||'').substring(0,100)}</div><div class="item-meta"><span>${a.source}</span><span class="badge ${a.level}" style="font-size:7px">${a.level.toUpperCase()}</span><span>${a.cat}</span><span>${a.region}</span><span>${a.score}pts</span>${a.url?`<a href="${a.url}" target="_blank" style="color:var(--ac)">🔗</a>`:''}</div></div>`).join('')||'<div style="text-align:center;padding:20px;color:var(--tx2);font-size:10px">Aucun article</div>'}</div>`;
}

async function loadAlertes(){
  const data=await api('/api/alertes');
  document.getElementById('content').innerHTML=`
    <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:6px;margin-bottom:8px">
      <div><h2 style="font-size:15px">🚨 Alertes multi-sources</h2><p style="font-size:9px;color:var(--tx2)">Confirmées par ≥2 sources</p></div>
      <button class="btn btn-outline btn-sm" onclick="apiPost('/api/alertes/read-all');loadAlertes()">✅ Tout marquer lu</button>
    </div>
    <div id="alertList">${(data.alertes||[]).map(a=>`<div class="alert-box ${a.niveau}" onclick="apiPost('/api/alerte/${a.id}/read');loadAlertes()"><div style="display:flex;justify-content:space-between"><div><b style="font-size:10px">${a.niveau==='critique'?'🔴':'🟠'} ${(a.title||'').substring(0,100)}</b><div style="font-size:8px;color:var(--tx2);margin-top:2px">📡 ${(a.sources||[]).join(', ')} · ${a.scount} sources</div></div><span style="font-size:10px;font-weight:700;color:var(--gd)">${a.score}pts</span></div><div style="font-size:8px;color:var(--tx2);margin-top:4px">${(a.descr||'').substring(0,120)}</div><div class="item-meta" style="margin-top:2px"><span>${a.cat}</span>${a.url?`<a href="${a.url}" target="_blank" style="color:var(--ac)">🔗 Source</a>`:''}</div></div>`).join('')||'<div style="text-align:center;padding:30px;color:var(--tx2);font-size:11px">✅ Aucune alerte</div>'}</div>`;
}

async function loadSources(){
  const data=await api('/api/sources');
  document.getElementById('content').innerHTML=`
    <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:6px;margin-bottom:8px">
      <div><h2 style="font-size:15px">📰 Sources</h2><p style="font-size:9px;color:var(--tx2)">${data.sources.length} actives + ${data.custom.length} personnalisées</p></div>
      <button class="btn btn-primary btn-sm" onclick="showAddSource()">➕ Ajouter</button>
    </div>
    <div style="margin-bottom:6px;font-size:10px;font-weight:600">📡 Personnalisées :</div>
    ${data.custom.length?data.custom.map(s=>`<div class="card" style="padding:8px;margin-bottom:3px;display:flex;justify-content:space-between;align-items:center"><span><b style="font-size:10px">${s.name}</b> <span style="font-size:8px;color:var(--tx2)">${s.url.substring(0,40)}...</span></span><button class="btn btn-sm btn-danger" onclick="apiPost('/api/sources/delete/${s.name}',{}).then(loadSources)">✕</button></div>`).join(''):'<div style="font-size:9px;color:var(--tx2);padding:6px">Aucune source personnalisée</div>'}
    <div style="margin:8px 0 6px;font-size:10px;font-weight:600">📡 Actives :</div>
    ${data.sources.slice(0,80).map(s=>`<div class="card" style="padding:6px 8px;margin-bottom:2px"><div style="display:flex;justify-content:space-between;font-size:9px"><span><b>${s.source}</b> <span style="color:var(--tx2)">${s.region} · Niv.${s.slevel}</span></span><span>📄 ${s.ct} ${s.crit>0?`<span style="color:var(--rd)">🔴${s.crit}</span>`:''} ${s.haut>0?`<span style="color:var(--or)">🟠${s.haut}</span>`:''}</span></div></div>`).join('')}
    <div style="text-align:center;padding:10px;font-size:9px;color:var(--tx2)">${data.sources.length>80?'... et '+(data.sources.length-80)+' autres':''}</div>`;
}

async function loadCategories(){
  const data=await api('/api/categories');
  document.getElementById('content').innerHTML=`
    <h2 style="font-size:15px;margin-bottom:4px">📁 Catégories</h2>
    <p style="font-size:9px;color:var(--tx2);margin-bottom:8px">Répartition thématique</p>
    <div class="grid g3">${(data.categories||[]).map(c=>`<div class="card" style="text-align:center;padding:14px;cursor:pointer" onclick="document.getElementById('fCat')?document.getElementById('fCat').value='${c.cat}':null;showPage('flux')"><div style="font-size:22px;margin-bottom:4px">${c.cat==='Cyber'?'💻':c.cat==='Securite'?'🛡️':c.cat==='Politique'?'🏛️':c.cat==='Economie'?'📈':c.cat==='Diplomatie'?'🌍':c.cat==='Justice'?'⚖️':c.cat==='Societe'?'👥':'📋'}</div><div style="font-weight:600;font-size:11px">${c.cat}</div><div style="font-size:9px;color:var(--tx2)">${c.ct} articles</div>${c.crit>0?`<div style="font-size:8px;color:var(--rd)">🔴 ${c.crit} critiques</div>`:''}</div>`).join('')||'<div style="grid-column:1/-1;text-align:center;padding:20px;color:var(--tx2)">Aucune donnée</div>'}</div>`;
}

async function loadSettings(){
  const s=state.stats.total?state.stats:await api('/api/stats');
  document.getElementById('content').innerHTML=`
    <h2 style="font-size:15px;margin-bottom:4px">⚙️ Paramètres</h2>
    <p style="font-size:9px;color:var(--tx2);margin-bottom:8px">Tout est modifiable en temps réel</p>
    <div class="grid g2">
      <div class="card"><h3 style="font-size:11px;margin-bottom:8px">🎨 Thème</h3>
        <div style="display:flex;gap:4px;flex-wrap:wrap">
          ${['dark','clair','noir','bleu','vert','rouge','violet','or'].map(t=>`<button class="btn ${s.current_theme===t?'btn-primary':'btn-outline'} btn-sm" onclick="applyTheme('${t}')">${t}</button>`).join('')}
        </div>
      </div>
      <div class="card"><h3 style="font-size:11px;margin-bottom:8px">⏱️ Scraping</h3>
        <div class="floating-label"><input id="setInterval" placeholder=" " value="${s.interval||'180'}"><label>Intervalle (secondes)</label></div>
        <div class="floating-label"><input id="setHours" placeholder=" " value="${s.hours||'24'}"><label>Profondeur (heures)</label></div>
        <button class="btn btn-primary btn-sm mt-8" onclick="saveSettings()" style="margin-top:6px">💾 Sauvegarder</button>
      </div>
      <div class="card"><h3 style="font-size:11px;margin-bottom:8px">➕ Ajouter une source RSS</h3>
        <div class="floating-label"><input id="srcName" placeholder=" "><label>Nom *</label></div>
        <div class="floating-label"><input id="srcUrl" placeholder=" "><label>URL RSS *</label></div>
        <div class="floating-label"><select id="srcReg" style="width:100%;background:var(--sf);border:1px solid var(--br);padding:10px 10px 4px;border-radius:6px;color:var(--tx);font-size:9px"><option value="nat">🔵 National</option><option value="cont">🟢 Continental</option><option value="inter">🔴 International</option></select></div>
        <button class="btn btn-primary btn-sm mt-8" onclick="addSource()" style="margin-top:6px">✅ Ajouter</button>
      </div>
      <div class="card"><h3 style="font-size:11px;margin-bottom:8px">📊 Actions</h3>
        <button class="btn btn-primary w-full" onclick="apiPost('/api/scan');toast('📡 Scan lancé')" style="width:100%;margin-bottom:4px">📡 Lancer un scan</button>
        <button class="btn btn-outline w-full" onclick="apiPost('/api/alertes/read-all');toast('✅ Alertes marquées lues')" style="width:100%">✅ Tout marquer lu</button>
      </div>
    </div>`;
}

async function loadHistorique(){
  const data=await api('/api/historique');
  document.getElementById('content').innerHTML=`
    <h2 style="font-size:15px;margin-bottom:4px">📋 Historique</h2>
    <p style="font-size:9px;color:var(--tx2);margin-bottom:8px">Évolution des statistiques</p>
    ${(data.historique||[]).length?data.historique.map(h=>`<div class="card" style="padding:8px;margin-bottom:3px"><div style="display:flex;justify-content:space-between;flex-wrap:wrap;gap:4px;font-size:9px"><span>🕐 ${h.ts}</span><span>📄 ${h.total}</span><span style="color:var(--rd)">🔴 ${h.crit}</span><span style="color:var(--or)">🟠 ${h.haut}</span><span>📡 ${h.src}</span><span style="color:var(--gd)">🚨 ${h.alrt}</span></div></div>`).join(''):'<div style="text-align:center;padding:20px;color:var(--tx2)">Aucun historique</div>'}`;
}

function debounce(fn,ms){let t;return(...args)=>{clearTimeout(t);t=setTimeout(()=>fn(...args),ms)}}

async function saveSettings(){
  const iv=document.getElementById('setInterval')?.value;
  const hr=document.getElementById('setHours')?.value;
  await apiPost('/api/settings',{interval:iv,hours:hr});
  toast('✅ Paramètres sauvegardés');
}

async function addSource(){
  const n=document.getElementById('srcName')?.value.trim();
  const u=document.getElementById('srcUrl')?.value.trim();
  const r=document.getElementById('srcReg')?.value||'nat';
  if(!n||!u) return toast('❌ Nom et URL requis');
  await apiPost('/api/sources/add',{name:n,url:u,region:r,level:70});
  toast('✅ Source ajoutée: '+n);
  loadSettings();
}

async function applyTheme(t){
  await apiPost('/api/settings',{theme:t});
  document.body.className='theme-'+t;
  toast('🎨 Thème: '+t);
  loadSettings();
}

function connectSSE(){
  const src=new EventSource('/stream');
  src.onmessage=function(e){
    try{const ev=JSON.parse(e.data);if(ev.type==='stats'||ev.type==='scan_done'){loadDashboard();}}catch(err){}
  };
  src.onerror=function(){setTimeout(connectSSE,5000)};
}

// Init theme
api('/api/stats').then(s=>{
  if(s.current_theme){document.body.className='theme-'+s.current_theme}
  state.stats=s;
});

document.addEventListener('DOMContentLoaded',init);
</script></body></html>
"""

# ═══════════════════════════════════════════
# MAIN
# ═══════════════════════════════════════════

if __name__=="__main__":
    print("\n"+"="*50)
    print("  ⟡ NEXUS DIAMOND v5 — FONCTIONNEL IMMÉDIAT")
    print("="*50)
    print(f"  📡 {len(RSS)} sources RSS + personnalisables")
    print(f"  🔑 {len(ALL_KW)} mots-clés")
    print(f"  🌐 http://0.0.0.0:5000")
    print("  ⏎ Ctrl+C pour arrêter\n")
    
    Scraper().start()
    app.run(host="0.0.0.0", port=5000, debug=False, use_reloader=False)
NEXUSFINAL

# COLLE CE SCRIPT DANS UN FICHIER ET LANCE
cat > nexus.py << 'NEXUSFINAL'
#!/usr/bin/env python3
"""NEXUS DIAMOND v5 — Fonctionnel immédiat. Tout modifiable depuis l'interface."""

import os, sys, json, time, hashlib, re, html, threading, queue, sqlite3
from datetime import datetime, timezone, timedelta
from collections import Counter
from urllib.parse import urlparse

# Auto-install
for pkg, imp in [("flask","flask"),("feedparser","feedparser"),("bs4","bs4"),("requests","requests")]:
    try:
        __import__(imp)
    except:
        os.system(f"{sys.executable} -m pip install {pkg} --quiet")
        __import__(imp)

from flask import Flask, Response, request, jsonify, render_template_string
import feedparser
from bs4 import BeautifulSoup
import requests

# DB
conn = sqlite3.connect("nexus.db", check_same_thread=False)
conn.row_factory = sqlite3.Row
conn.execute("PRAGMA journal_mode=WAL")
conn.executescript("""
CREATE TABLE IF NOT EXISTS articles(id TEXT PRIMARY KEY, title TEXT, url TEXT, descr TEXT, source TEXT, slevel INT DEFAULT 70, region TEXT DEFAULT 'inter', level TEXT DEFAULT 'info', score INT DEFAULT 0, cat TEXT DEFAULT 'General', pub TEXT, col TEXT DEFAULT (datetime('now')), img TEXT, lu INT DEFAULT 0);
CREATE TABLE IF NOT EXISTS alertes(id TEXT PRIMARY KEY, title TEXT, descr TEXT, niveau TEXT DEFAULT 'haute', score INT DEFAULT 0, cat TEXT DEFAULT 'General', sources TEXT, scount INT DEFAULT 1, url TEXT, created TEXT DEFAULT (datetime('now')), lu INT DEFAULT 0);
CREATE TABLE IF NOT EXISTS settings(key TEXT PRIMARY KEY, val TEXT);
CREATE TABLE IF NOT EXISTS custom_sources(name TEXT, url TEXT, region TEXT, level INT);
CREATE TABLE IF NOT EXISTS stats(id INTEGER PRIMARY KEY AUTOINCREMENT, total INT, crit INT, haut INT, moy INT, src INT, alrt INT, ts TEXT DEFAULT (datetime('now')));
INSERT OR IGNORE INTO settings VALUES('theme','dark'),('interval','180'),('hours','24'),('max_articles','50000'),('extract_images','1');
""")
conn.commit()

KW_CRIT = ["meurtre","assassinat","massacre","attentat","explosion","attaque","fusillade","coup d'etat","putsch","insurrection","guerre","conflit arme","terrorisme","epidemie","pandemie","ebola","inondation","seisme","accident mortel","naufrage","crash","incendie","drogue","cocaine","arme de guerre","enlevement","viol","execution","etat d'urgence","couvre-feu","cyberattaque","brèche","anonymous","hacktivisme"]
KW_HAUT = ["securite","armee","police","gendarmerie","presidence","president","ministre","gouvernement","corruption","detournement","scandale","cybercriminalite","piratage","election","vote","crise politique","diplomatie","defense","renseignement","manifestation","greve","sanction","embargo","droits humains","anonymous","data leak","fuite","ddos","ransomware","hack"]
KW_MOY = ["sante","hopital","education","ecole","universite","economie","investissement","agriculture","culture","sport","technologie","environnement","nomination","reforme","loi","decret","accident","religion"]
ALL_KW = list(set(KW_CRIT + KW_HAUT + KW_MOY))

RSS = [
    ("Fraternité Matin","https://www.fratmat.info/rss","nat",100),("RTI","https://www.rti.ci/rss","nat",100),("Abidjan.net","https://news.abidjan.net/rss","nat",100),("AIP","https://www.aip.ci/feed/","nat",100),("7Info","https://7info.ci/rss","nat",95),("Soir Info","https://www.soirinfo.com/rss","nat",95),("L'Inter","https://www.linterci.com/rss","nat",95),("Koaci","https://www.koaci.com/rss","nat",95),("Linfodrome","https://www.linfodrome.com/24h?format=feed","nat",95),("Connection Ivoirienne","https://www.connectionivoirienne.net/feed/","nat",90),("Jeune Afrique","https://www.jeuneafrique.com/feed/","cont",100),("RFI Afrique","https://www.rfi.fr/fr/afrique/rss","cont",100),("BBC Afrique","https://www.bbc.com/afrique/rss.xml","cont",100),("Le Monde Afrique","https://www.lemonde.fr/afrique/rss_full.xml","cont",100),("AfricaNews","https://www.africanews.com/feed/","cont",95),("France24 Afrique","https://www.france24.com/fr/afrique/rss","cont",95),("Al Jazeera","https://www.aljazeera.com/xml/rss/all.xml","cont",95),("Crisis Group","https://www.crisisgroup.org/rss.xml","inter",95),("HRW","https://www.hrw.org/rss/africa","cont",95),("ISS Africa","https://issafrica.org/rss.xml","cont",95),("ONU Info","https://news.un.org/feed/subscribe/en/news/region/africa/feed/rss.xml","cont",100),("Interpol","https://www.interpol.int/rss","inter",100),("Krebs","https://krebsonsecurity.com/feed/","inter",90),("The Hacker News","https://feeds.feedburner.com/TheHackersNews","inter",85),("Bleeping Computer","https://www.bleepingcomputer.com/feed/","inter",85),("Cisco Talos","https://blog.talosintelligence.com/feed/","inter",90),("CrowdStrike","https://www.crowdstrike.com/blog/feed/","inter",85),("Anonymous News","https://www.anonymous.news/feed/","inter",80),("HackRead","https://www.hackread.com/feed/","inter",80),("Security Affairs","https://securityaffairs.com/feed","inter",80),("Schneier","https://www.schneier.com/feed/","inter",85),("Troy Hunt","https://www.troyhunt.com/rss/","inter",85),("AnonHQ","https://anonhq.com/feed/","inter",75),("GNews CI","https://news.google.com/rss/search?q=C%C3%B4te+d%27Ivoire+apr%C3%A8s+2025&hl=fr&gl=CI&ceid=CI:fr","nat",95),("GNews Securite","https://news.google.com/rss/search?q=s%C3%A9curit%C3%A9+attaque+C%C3%B4te+d%27Ivoire&hl=fr&gl=CI&ceid=CI:fr","nat",95),("GNews Cyber","https://news.google.com/rss/search?q=cybercriminalit%C3%A9+Afrique+hacking&hl=fr&gl=CI&ceid=CI:fr","cont",90),("GNews Terrorisme","https://news.google.com/rss/search?q=terrorisme+Sahel+Afrique+attentat&hl=fr&gl=CI&ceid=CI:fr","cont",95),("GNews Conflits","https://news.google.com/rss/search?q=conflit+guerre+Afrique+monde&hl=fr&gl=CI&ceid=CI:fr","inter",90),
]

session = requests.Session()
session.headers.update({"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"})
sse_queue = queue.Queue()

def nml(t):
    if not t: return ""
    t = t.lower()
    for a,b in [('é','e'),('è','e'),('ê','e'),('ë','e'),('à','a'),('â','a'),('ä','a'),('ù','u'),('û','u'),('ü','u'),('ô','o'),('ö','o'),('î','i'),('ï','i'),('ç','c')]:
        t = t.replace(a,b)
    return t

def detect_lv(t,d=""):
    txt = nml(f"{t} {d[:200]}")
    for k in KW_CRIT:
        if k in txt: return "critique", [k]
    for k in KW_HAUT:
        if k in txt: return "haute", [k]
    for k in KW_MOY:
        if k in txt: return "moyenne", [k]
    return "info", []

def calc_score(t,d,sl,lv):
    s = sl
    txt = nml(f"{t} {d[:200]}")
    if lv=="critique": s+=50
    elif lv=="haute": s+=30
    elif lv=="moyenne": s+=15
    for k in ALL_KW:
        if k in txt: s+=2
    return min(s,200)

def detect_cat(t,d):
    txt = nml(f"{t} {d[:200]}")
    cats = {"Politique":["gouvernement","president","ministre","election","vote","parti","loi","reforme"],"Securite":["armee","police","gendarmerie","securite","defense","terrorisme","attaque","criminalite"],"Cyber":["cyberattaque","piratage","hack","ransomware","virus","malware","breche","faille","anonymous","ddos"],"Economie":["economie","finance","banque","marche","investissement","pib","inflation","bceao","brvm"],"Diplomatie":["diplomatie","ambassade","onu","ua","cedeao","cooperation","sommet"],"Societe":["societe","education","sante","culture","religion","jeunesse"],"Justice":["justice","tribunal","proces","mandat","enquete","juge","prison"]}
    for c,m in cats.items():
        if any(k in txt for k in m): return c
    return "General"

def detect_reg(t,d,sr,sn=""):
    txt = nml(f"{t} {d[:100]}")
    ci = ["cote d'ivoire","abidjan","ivoirien","yopougon","cocody","bouake","gbagbo","ouattara","bedie","soro"]
    af = ["afrique","mali","burkina","niger","guinee","senegal","ghana","nigeria","cameroun","rdc","sahel"]
    for k in ci:
        if k in txt: return "nat"
    for k in af:
        if k in txt: return "cont"
    return sr

def get_set(k, defv=""):
    r = conn.execute("SELECT val FROM settings WHERE key=?",(k,)).fetchone()
    return r["val"] if r else defv

def set_set(k,v):
    conn.execute("INSERT OR REPLACE INTO settings VALUES(?,?)",(k,v))
    conn.commit()

def scrape():
    arts = []
    sources = RSS
    cs = conn.execute("SELECT * FROM custom_sources").fetchall()
    for s in cs:
        sources.append((s["name"],s["url"],s["region"] or "nat",s["level"] or 70))
    hours = int(get_set("hours","24"))
    cutoff = datetime.now(timezone.utc) - timedelta(hours=hours)
    for name,url,region,level in sources:
        try:
            time.sleep(0.03)
            r = session.get(url, timeout=10)
            if r.status_code!=200: continue
            feed = feedparser.parse(r.content)
            for e in feed.entries[:20]:
                t = (e.get("title","") or "").strip()
                l = (e.get("link","") or "").strip()
                if not t or not l: continue
                d = BeautifulSoup(e.get("summary","") or e.get("description","") or "","html.parser").get_text()[:400]
                pub = datetime.now(timezone.utc)
                if e.get("published_parsed"):
                    try: pub = datetime(*e.published_parsed[:6], tzinfo=timezone.utc)
                    except: pass
                if pub < cutoff: continue
                txt = nml(f"{t} {d}")
                matched = [k for k in ALL_KW if k in txt]
                if not matched: continue
                uid = hashlib.sha256(f"{name}:{l}".encode()).hexdigest()[:16]
                lv,kw = detect_lv(t,d)
                reg = detect_reg(t,d,region,name)
                cat = detect_cat(t,d)
                sc = calc_score(t,d,level,lv)
                arts.append({"id":uid,"title":t[:300],"url":l,"descr":d[:400],"source":name,"slevel":level,"region":reg,"level":lv,"score":sc,"cat":cat,"pub":pub.isoformat(),"img":""})
        except: pass
    return arts

def save(arts):
    saved=0
    for a in arts:
        try:
            conn.execute("INSERT OR IGNORE INTO articles VALUES(?,?,?,?,?,?,?,?,?,?,?,datetime('now'),?,0)",(a["id"],a["title"],a["url"],a["descr"],a["source"],a["slevel"],a["region"],a["level"],a["score"],a["cat"],a["pub"],a["img"]))
            if conn.total_changes: saved+=1
        except: pass
    conn.commit()
    return saved

def detect_alert():
    hours = int(get_set("hours","24"))
    cutoff = (datetime.now(timezone.utc)-timedelta(hours=hours)).isoformat()
    rows = conn.execute("SELECT * FROM articles WHERE level IN ('critique','haute') AND col > ? ORDER BY score DESC",(cutoff,)).fetchall()
    groups = {}
    for r in rows:
        words = set(nml(r["title"]).split())
        sw = sorted([w for w in words if len(w)>=4])[:5]
        if not sw: continue
        key = "_".join(sw)
        if key not in groups: groups[key]=[]
        groups[key].append(r)
    nv=0
    for k,items in groups.items():
        if len(items)<2: continue
        sources = list(set(i["source"] for i in items))
        if len(sources)<2: continue
        best = max(items, key=lambda x:x["score"])
        aid = hashlib.sha256(f"al:{k}".encode()).hexdigest()[:16]
        if conn.execute("SELECT id FROM alertes WHERE id=?",(aid,)).fetchone(): continue
        nv_ = "critique" if any(i["level"]=="critique" for i in items) else "haute"
        conn.execute("INSERT OR IGNORE INTO alertes VALUES(?,?,?,?,?,?,?,?,?,datetime('now'),0)",(aid,best["title"][:200],best["descr"][:300],nv_,best["score"]+len(sources)*10,best["cat"],json.dumps(sources),len(sources),best["url"]))
        if conn.total_changes: nv+=1
    conn.commit()
    return nv

def update_stats():
    hours = int(get_set("hours","24"))
    cutoff = (datetime.now(timezone.utc)-timedelta(hours=hours)).isoformat()
    t = conn.execute("SELECT COUNT(*) FROM articles WHERE col>?",(cutoff,)).fetchone()[0]
    c = conn.execute("SELECT COUNT(*) FROM articles WHERE level='critique' AND col>?",(cutoff,)).fetchone()[0]
    h = conn.execute("SELECT COUNT(*) FROM articles WHERE level='haute' AND col>?",(cutoff,)).fetchone()[0]
    m = conn.execute("SELECT COUNT(*) FROM articles WHERE level='moyenne' AND col>?",(cutoff,)).fetchone()[0]
    s = conn.execute("SELECT COUNT(DISTINCT source) FROM articles WHERE col>?",(cutoff,)).fetchone()[0]
    a = conn.execute("SELECT COUNT(*) FROM alertes WHERE created>?",(cutoff,)).fetchone()[0]
    conn.execute("INSERT INTO stats(total,crit,haut,moy,src,alrt) VALUES(?,?,?,?,?,?)",(t,c,h,m,s,a))
    conn.commit()
    return {"total":t,"critiques":c,"hautes":h,"moyennes":m,"sources":s,"alertes":a}

class Scraper(threading.Thread):
    def run(self):
        while True:
            start=time.time()
            print(f"[{datetime.now().strftime('%H:%M:%S')}] Scraping {len(RSS)} sources...")
            arts=scrape()
            s=save(arts)
            a=detect_alert()
            st=update_stats()
            print(f"  +{s} articles | {a} alertes | {st['total']} total")
            if s>0: sse_queue.put({"type":"new_articles","data":{"count":s,"stats":st}})
            if a>0: sse_queue.put({"type":"new_alertes","data":{"count":a}})
            sse_queue.put({"type":"stats","data":st})
            slp=max(1,int(get_set("interval","180"))-(time.time()-start))
            time.sleep(slp)

app = Flask(__name__)

@app.route("/api/stats")
def api_stats():
    st=update_stats()
    anl=conn.execute("SELECT COUNT(*) FROM alertes WHERE lu=0").fetchone()[0]
    st["alertes_non_lues"]=anl
    st["themes"]=["dark","clair","noir","bleu","vert","rouge","violet","or"]
    st["current_theme"]=get_set("theme","dark")
    st["interval"]=get_set("interval","180")
    st["hours"]=get_set("hours","24")
    return jsonify(st)

@app.route("/api/articles")
def api_articles():
    lim=min(int(request.args.get("limit",50)),500)
    off=int(request.args.get("offset",0))
    lv=request.args.get("level","all")
    reg=request.args.get("region","all")
    cat=request.args.get("category","all")
    src=request.args.get("source","all")
    q=request.args.get("search","").strip()
    sort=request.args.get("sort","score")
    hours=int(get_set("hours","24"))
    cutoff=(datetime.now(timezone.utc)-timedelta(hours=hours)).isoformat()
    query="SELECT * FROM articles WHERE col>?"
    params=[cutoff]
    if lv!="all": query+=" AND level=?"; params.append(lv)
    if reg!="all": query+=" AND region=?"; params.append(reg)
    if cat!="all": query+=" AND cat=?"; params.append(cat)
    if src!="all": query+=" AND source=?"; params.append(src)
    if q: query+=" AND (title LIKE ? OR descr LIKE ?)"; params.extend([f"%{q}%",f"%{q}%"])
    total=conn.execute(query.replace("SELECT *","SELECT COUNT(*)"),params).fetchone()[0]
    query+=f" ORDER BY {'pub' if sort=='date' else 'score'} DESC LIMIT ? OFFSET ?"
    params.extend([lim,off])
    rows=conn.execute(query,params).fetchall()
    return jsonify({"articles":[dict(r) for r in rows],"total":total})

@app.route("/api/alertes")
def api_alertes():
    hours=int(get_set("hours","24"))
    cutoff=(datetime.now(timezone.utc)-timedelta(hours=hours)).isoformat()
    rows=conn.execute("SELECT * FROM alertes WHERE created>? ORDER BY score DESC LIMIT 100",(cutoff,)).fetchall()
    al=[]
    for r in rows:
        a=dict(r)
        a["sources"]=json.loads(a["sources"]) if a["sources"] else []
        al.append(a)
    return jsonify({"alertes":al})

@app.route("/api/sources")
def api_sources():
    hours=int(get_set("hours","24"))
    cutoff=(datetime.now(timezone.utc)-timedelta(hours=hours)).isoformat()
    rows=conn.execute("""SELECT source,slevel,region,COUNT(*) as ct,SUM(CASE WHEN level='critique' THEN 1 ELSE 0 END) as crit,SUM(CASE WHEN level='haute' THEN 1 ELSE 0 END) as haut FROM articles WHERE col>? GROUP BY source ORDER BY ct DESC""",(cutoff,)).fetchall()
    cs=conn.execute("SELECT * FROM custom_sources").fetchall()
    return jsonify({"sources":[dict(r) for r in rows],"custom":[dict(c) for c in cs]})

@app.route("/api/categories")
def api_categories():
    hours=int(get_set("hours","24"))
    cutoff=(datetime.now(timezone.utc)-timedelta(hours=hours)).isoformat()
    rows=conn.execute("SELECT cat,COUNT(*) as ct,SUM(CASE WHEN level='critique' THEN 1 ELSE 0 END) as crit FROM articles WHERE col>? GROUP BY cat ORDER BY ct DESC",(cutoff,)).fetchall()
    return jsonify({"categories":[dict(r) for r in rows]})

@app.route("/api/historique")
def api_historique():
    rows=conn.execute("SELECT * FROM stats ORDER BY ts DESC LIMIT 50").fetchall()
    return jsonify({"historique":[dict(r) for r in rows]})

@app.route("/api/settings",methods=["GET","POST"])
def api_settings():
    if request.method=="POST":
        d=request.json
        for k,v in d.items():
            set_set(k,str(v))
        return jsonify({"success":True})
    return jsonify({k:get_set(k) for k in ["theme","interval","hours","max_articles","extract_images"]})

@app.route("/api/sources/add",methods=["POST"])
def api_add_source():
    d=request.json
    conn.execute("INSERT INTO custom_sources VALUES(?,?,?,?)",(d["name"],d["url"],d.get("region","nat"),int(d.get("level",70))))
    conn.commit()
    return jsonify({"success":True})

@app.route("/api/sources/delete/<name>",methods=["DELETE"])
def api_del_source(name):
    conn.execute("DELETE FROM custom_sources WHERE name=?",(name,))
    conn.commit()
    return jsonify({"success":True})

@app.route("/api/alertes/read-all",methods=["POST"])
def api_alertes_read():
    conn.execute("UPDATE alertes SET lu=1")
    conn.commit()
    return jsonify({"success":True})

@app.route("/api/alerte/<id>/read",methods=["POST"])
def api_alerte_read(id):
    conn.execute("UPDATE alertes SET lu=1 WHERE id=?",(id,))
    conn.commit()
    return jsonify({"success":True})

@app.route("/api/scan",methods=["POST"])
def api_scan():
    threading.Thread(target=lambda:(save(scrape()),detect_alert(),update_stats(),sse_queue.put({"type":"scan_done"})),daemon=True).start()
    return jsonify({"success":True})

@app.route("/api/logs")
def api_logs():
    hours=int(get_set("hours","24"))
    cutoff=(datetime.now(timezone.utc)-timedelta(hours=hours)).isoformat()
    rows=conn.execute("SELECT strftime('%H:%M',ts) as time,total,crit,haut FROM stats ORDER BY ts DESC LIMIT 24").fetchall()
    return jsonify({"logs":[dict(r) for r in reversed(rows)]})

@app.route("/stream")
def stream():
    def gen():
        while True:
            try:
                e=sse_queue.get(timeout=30)
                yield f"data: {json.dumps(e,ensure_ascii=False)}\n\n"
            except queue.Empty:
                yield ": keepalive\n\n"
    return Response(gen(),mimetype="text/event-stream",headers={"Cache-Control":"no-cache","X-Accel-Buffering":"no"})

@app.route("/")
def index():
    return render_template_string(HTML)

# ═══════════════════════════════════════════
# FRONTEND HTML COMPLET
# ═══════════════════════════════════════════

HTML = r"""<!DOCTYPE html><html lang="fr"><head>
<meta charset="UTF-8"><meta name="viewport" content="width=device-width,initial-scale=1">
<title>⟡ NEXUS DIAMOND v5</title>
<style>
*{margin:0;padding:0;box-sizing:border-box;font-family:system-ui,-apple-system,sans-serif;transition:background .2s,color .2s}
:root{--bg:#060810;--sf:#0c0f1c;--cd:#121828;--cd2:#181f38;--tx:#eef2fa;--tx2:#8898c8;--ac:#00e5ff;--ac2:#ff6b35;--gr:#00ff88;--rd:#ff0030;--or:#ff6b00;--ye:#ffcc00;--pu:#aa66ff;--gd:#ffd700;--br:#1e2a4a;--r:12px}
.theme-clair{--bg:#f0f2f8;--sf:#e4e7f0;--cd:#fff;--cd2:#f4f6fc;--tx:#1a1d2e;--tx2:#5a6a8a;--br:#d0d6e4}
.theme-noir{--bg:#000;--sf:#0a0a0a;--cd:#111;--cd2:#1a1a1a;--tx:#fff;--tx2:#888;--br:#222}
.theme-bleu{--bg:#0a1628;--sf:#0f1f3a;--cd:#142850;--cd2:#1a3366;--tx:#e0eeff;--tx2:#7a9ec8;--br:#1e4a7a}
.theme-vert{--bg:#0a1a0a;--sf:#0f2a0f;--cd:#143a14;--cd2:#1a4a1a;--tx:#e0ffe0;--tx2:#7ac87a;--br:#1e4a1e}
.theme-rouge{--bg:#1a0a0a;--sf:#2a0f0f;--cd:#3a1414;--cd2:#4a1a1a;--tx:#ffe0e0;--tx2:#c87a7a;--br:#4a1e1e}
.theme-violet{--bg:#100a1a;--sf:#1a0f2a;--cd:#24143a;--cd2:#2e1a4a;--tx:#eee0ff;--tx2:#9a7ac8;--br:#3a1e5a}
.theme-or{--bg:#1a180a;--sf:#2a2610;--cd:#3a3416;--cd2:#4a421c;--tx:#fffae0;--tx2:#c8b87a;--br:#5a4e1e}
body{background:var(--bg);color:var(--tx);min-height:100vh}
.nav{position:fixed;top:0;left:0;right:0;height:48px;background:rgba(8,11,22,.97);border-bottom:1px solid var(--br);display:flex;align-items:center;padding:0 12px;z-index:1000;gap:8px}
.nav-logo{font-size:15px;font-weight:800;background:linear-gradient(135deg,#00e5ff,#aa66ff);-webkit-background-clip:text;-webkit-text-fill-color:transparent;white-space:nowrap}
.nav-logo span{display:inline-block;width:28px;height:28px;line-height:28px;text-align:center;background:linear-gradient(135deg,#00e5ff,#aa66ff);border-radius:6px;color:#fff;font-size:12px;-webkit-text-fill-color:#fff;margin-right:4px}
.nav-items{display:flex;gap:2px;overflow-x:auto;flex:1;scrollbar-width:none}
.nav-items::-webkit-scrollbar{display:none}
.nav-items button{padding:5px 10px;border-radius:6px;border:none;background:transparent;color:var(--tx2);font-size:10px;cursor:pointer;white-space:nowrap;font-weight:500}
.nav-items button:hover{color:var(--tx);background:var(--cd)}
.nav-items button.active{color:var(--ac);background:var(--cd)}
.nav-info{display:flex;align-items:center;gap:6px;font-size:9px;color:var(--tx2);flex-shrink:0}
.nav-info .dot{width:6px;height:6px;border-radius:50%;background:var(--gr);animation:pulse 1.5s infinite}
@keyframes pulse{0%,100%{opacity:1}50%{opacity:.3}}
.content{padding:60px 12px 12px;max-width:1400px;margin:0 auto}
.page{display:none}.page.active{display:block;animation:fade .15s}
@keyframes fade{from{opacity:0}to{opacity:1}}
.grid{display:grid;gap:6px}.g2{grid-template-columns:repeat(auto-fit,minmax(320px,1fr))}.g3{grid-template-columns:repeat(auto-fit,minmax(250px,1fr))}.g4{grid-template-columns:repeat(auto-fit,minmax(200px,1fr))}
.card{background:var(--cd);border:1px solid var(--br);border-radius:var(--r);padding:12px}
.card:hover{border-color:var(--ac);transform:translateY(-1px);box-shadow:0 4px 20px rgba(0,0,0,.4)}
.stats{display:grid;grid-template-columns:repeat(auto-fit,minmax(90px,1fr));gap:6px;margin-bottom:8px}
.stat-card{background:var(--cd);border:1px solid var(--br);border-radius:10px;padding:10px 6px;text-align:center}
.stat-card .num{font-size:20px;font-weight:800;color:var(--ac)}
.stat-card .label{font-size:8px;color:var(--tx2);text-transform:uppercase;margin-top:1px}
.badge{display:inline-flex;padding:1px 8px;border-radius:12px;font-size:8px;font-weight:700}
.badge.critique{background:var(--rd);color:#fff}.badge.haute{background:var(--or);color:#fff}.badge.moyenne{background:var(--ye);color:#000}.badge.info{background:var(--ac);color:#000}
.item{padding:6px 8px;background:var(--cd2);border-radius:6px;margin-bottom:3px;border-left:3px solid var(--ac);cursor:pointer}
.item:hover{background:var(--cd)}
.item.critique{border-left-color:var(--rd)}.item.haute{border-left-color:var(--or)}.item.moyenne{border-left-color:var(--ye)}
.item-title{font-size:10px;font-weight:600;overflow:hidden;display:-webkit-box;-webkit-line-clamp:2;-webkit-box-orient:vertical}
.item-meta{font-size:8px;color:var(--tx2);margin-top:1px;display:flex;gap:4px;flex-wrap:wrap}
.alert-box{padding:8px 12px;background:rgba(255,0,48,.08);border:1px solid var(--rd);border-radius:8px;margin-bottom:4px;cursor:pointer}
.alert-box:hover{background:rgba(255,0,48,.12)}
.alert-box.haute{background:rgba(255,107,0,.08);border-color:var(--or)}
.alert-box.haute:hover{background:rgba(255,107,0,.12)}
.tabs{display:flex;gap:2px;margin-bottom:8px;overflow-x:auto;scrollbar-width:none;border-bottom:1px solid var(--br)}
.tabs button{padding:5px 12px;border:none;background:transparent;color:var(--tx2);font-size:9px;cursor:pointer;white-space:nowrap}
.tabs button:hover{color:var(--tx)}
.tabs button.active{background:var(--cd);color:var(--ac);border-radius:6px 6px 0 0}
.search-bar{display:flex;gap:6px;flex-wrap:wrap;align-items:center;margin-bottom:8px;padding:8px 12px;background:var(--cd);border-radius:var(--r);border:1px solid var(--br)}
.search-bar input,.search-bar select{flex:1;min-width:80px;background:var(--sf);border:1px solid var(--br);padding:5px 8px;border-radius:6px;color:var(--tx);font-size:9px}
.search-bar input:focus,.search-bar select:focus{border-color:var(--ac);outline:none}
.btn{padding:5px 12px;border-radius:6px;border:none;font-size:9px;font-weight:600;cursor:pointer}
.btn-primary{background:linear-gradient(135deg,#00e5ff,#aa66ff);color:#fff}
.btn-primary:hover{box-shadow:0 2px 12px rgba(0,229,255,.3)}
.btn-outline{background:transparent;border:1px solid var(--br);color:var(--tx2)}
.btn-outline:hover{border-color:var(--ac);color:var(--ac)}
.btn-danger{background:var(--rd);color:#fff}
.btn-sm{padding:3px 8px;font-size:8px}
.btn-lg{padding:8px 16px;font-size:11px}
.floating-label{position:relative;margin-bottom:6px}
.floating-label input,.floating-label textarea,.floating-label select{width:100%;background:var(--sf);border:1px solid var(--br);padding:10px 10px 4px;border-radius:6px;color:var(--tx);font-size:9px;outline:none}
.floating-label input:focus,.floating-label textarea:focus,.floating-label select:focus{border-color:var(--ac);box-shadow:0 0 0 2px rgba(0,229,255,.1)}
.floating-label label{position:absolute;top:8px;left:10px;font-size:9px;color:var(--tx2);pointer-events:none;transition:all .15s}
.floating-label input:focus~label,.floating-label input:not(:placeholder-shown)~label,.floating-label textarea:focus~label,.floating-label textarea:not(:placeholder-shown)~label{top:2px;font-size:7px;color:var(--ac)}
.modal{display:none;position:fixed;top:0;left:0;right:0;bottom:0;background:rgba(0,0,0,.8);z-index:2000;align-items:center;justify-content:center;backdrop-filter:blur(8px)}
.modal.open{display:flex}
.modal-content{background:var(--cd);border:1px solid var(--br);border-radius:14px;padding:16px;max-width:500px;width:92%;max-height:80vh;overflow-y:auto;animation:slide .2s}
@keyframes slide{from{transform:translateY(10px);opacity:0}to{transform:translateY(0);opacity:1}}
.modal-close{position:absolute;top:8px;right:8px;background:var(--cd2);border:1px solid var(--br);color:var(--tx2);width:26px;height:26px;border-radius:50%;cursor:pointer;display:flex;align-items:center;justify-content:center;font-size:11px}
.modal-close:hover{background:var(--rd);color:#fff;border-color:var(--rd)}
.toast{position:fixed;bottom:60px;left:50%;transform:translateX(-50%);background:var(--cd);border:1px solid var(--ac);padding:8px 16px;border-radius:8px;font-size:9px;z-index:9999;box-shadow:0 4px 20px rgba(0,0,0,.5);animation:tin .2s}
@keyframes tin{from{opacity:0;transform:translateX(-50%) translateY(10px)}to{opacity:1;transform:translateX(-50%) translateY(0)}}
::-webkit-scrollbar{width:3px;height:3px}
::-webkit-scrollbar-track{background:transparent}
::-webkit-scrollbar-thumb{background:var(--br);border-radius:3px}
@media(max-width:768px){.nav-items{display:none}.stats{grid-template-columns:repeat(2,1fr)}.g2,.g3,.g4{grid-template-columns:1fr}}
</style></head><body>
<div class="nav">
  <div class="nav-logo"><span>⟡</span> NEXUS DIAMOND v5</div>
  <div class="nav-items" id="navItems"></div>
  <div class="nav-info"><span class="dot"></span><span id="liveInfo">0 arts</span></div>
</div>
<div class="content" id="content"></div>

<script>
// APP
let state={page:'dashboard',articles:[],alertes:[],stats:{}};
const pages=['dashboard','flux','alertes','sources','categories','settings','historique'];
const icons={'dashboard':'📊','flux':'📡','alertes':'🚨','sources':'📰','categories':'📁','settings':'⚙️','historique':'📋'};

function init(){
  const nav=document.getElementById('navItems');
  nav.innerHTML=pages.map(p=>`<button class="${p==='dashboard'?'active':''}" onclick="showPage('${p}')">${icons[p]} ${p.charAt(0).toUpperCase()+p.slice(1)}</button>`).join('');
  showPage('dashboard');
  connectSSE();
  setInterval(loadStats,10000);
}

function showPage(p){
  state.page=p;
  document.querySelectorAll('.nav-items button').forEach(b=>b.classList.remove('active'));
  document.querySelector(`.nav-items button[onclick*="'${p}'"]`)?.classList.add('active');
  const fns={dashboard:loadDashboard,flux:loadFlux,alertes:loadAlertes,sources:loadSources,categories:loadCategories,settings:loadSettings,historique:loadHistorique};
  if(fns[p]) fns[p]();
}

function toast(m){const t=document.createElement('div');t.className='toast';t.textContent=m;document.body.appendChild(t);setTimeout(()=>t.remove(),2000)}
function api(p){return fetch(p).then(r=>r.json())}
function apiPost(p,d){return fetch(p,{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(d||{})}).then(r=>r.json())}

async function loadStats(){
  const s=await api('/api/stats');
  state.stats=s;
  document.getElementById('liveInfo').textContent=`${s.total||0} arts · ${s.alertes_non_lues||0} alertes`;
}

async function loadDashboard(){
  const s=state.stats.total?state.stats:await api('/api/stats');
  Object.assign(state.stats,s);
  const al=await api('/api/alertes');
  const art=await api('/api/articles?limit=8&sort=score');
  document.getElementById('content').innerHTML=`
    <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:6px;margin-bottom:8px">
      <div><h2 style="font-size:16px">📊 Dashboard</h2><p style="font-size:9px;color:var(--tx2)">Tout est modifiable depuis l'interface</p></div>
      <button class="btn btn-primary btn-sm" onclick="apiPost('/api/scan');toast('📡 Scan lancé')">📡 Scanner</button>
    </div>
    <div class="stats">${['total','critiques','hautes','moyennes','sources','alertes_non_lues'].map(k=>`<div class="stat-card"><div class="num" style="color:${k==='critiques'?'var(--rd)':k==='alertes_non_lues'?'var(--gd)':k==='hautes'?'var(--or)':'var(--ac)'}">${s[k]||0}</div><div class="label">${k.replace('_',' ')}</div></div>`).join('')}</div>
    <div class="grid g2">
      <div class="card"><div class="flex" style="display:flex;justify-content:space-between;margin-bottom:6px"><b style="font-size:11px">🚨 Alertes</b><span class="badge critique">${s.alertes_non_lues||0}</span></div>
        <div id="dashAlertes">${(al.alertes||[]).slice(0,6).map(a=>`<div class="alert-box ${a.niveau}" onclick="apiPost('/api/alerte/${a.id}/read');loadDashboard()"><b style="font-size:9px">${a.niveau==='critique'?'🔴':'🟠'} ${(a.title||'').substring(0,70)}</b><div style="font-size:7px;color:var(--tx2)">${a.scount} sources · Score: ${a.score}</div></div>`).join('')||'<div style="font-size:9px;color:var(--tx2);text-align:center;padding:10px">✅ Aucune alerte</div>'}</div></div>
      <div class="card"><div style="display:flex;justify-content:space-between;margin-bottom:6px"><b style="font-size:11px">🔥 Top articles</b></div>
        <div id="dashArticles">${(art.articles||[]).slice(0,8).map(a=>`<div class="item ${a.level}" onclick="window.open('${a.url||''}','_blank')"><div class="item-title">${a.level==='critique'?'🔴':a.level==='haute'?'🟠':'🟡'} ${(a.title||'').substring(0,70)}</div><div class="item-meta"><span>${a.source}</span><span>·</span><span>${a.score}pts</span><span>·</span><span>${a.region}</span></div></div>`).join('')||'<div style="font-size:9px;color:var(--tx2);text-align:center;padding:10px">Lancez un scan</div>'}</div></div>
    </div>`;
}

async function loadFlux(){
  const q=document.getElementById('fSearch')?.value||'';
  const lv=document.getElementById('fLevel')?.value||'all';
  const reg=document.getElementById('fRegion')?.value||'all';
  const cat=document.getElementById('fCat')?.value||'all';
  let url=`/api/articles?limit=60&sort=score&level=${lv}&region=${reg}&category=${cat}`;
  if(q) url+=`&search=${encodeURIComponent(q)}`;
  const data=await api(url);
  document.getElementById('content').innerHTML=`
    <h2 style="font-size:15px;margin-bottom:4px">📡 Flux temps réel</h2>
    <div class="search-bar">
      <input id="fSearch" placeholder="🔍 Mots-clés..." oninput="debounce(loadFlux,300)" value="${q}">
      <select id="fLevel" onchange="loadFlux()"><option value="all">Tous</option><option value="critique" ${lv==='critique'?'selected':''}>🔴 Critique</option><option value="haute" ${lv==='haute'?'selected':''}>🟠 Haute</option><option value="moyenne" ${lv==='moyenne'?'selected':''}>🟡 Moyenne</option><option value="info" ${lv==='info'?'selected':''}>🗞️ Info</option></select>
      <select id="fRegion" onchange="loadFlux()"><option value="all">🌍 Toutes</option><option value="nat" ${reg==='nat'?'selected':''}>🔵 National</option><option value="cont" ${reg==='cont'?'selected':''}>🟢 Continental</option><option value="inter" ${reg==='inter'?'selected':''}>🔴 International</option></select>
      <select id="fCat" onchange="loadFlux()"><option value="all">📁 Toutes</option>${['Politique','Securite','Cyber','Economie','Diplomatie','Societe','Justice'].map(c=>`<option value="${c}" ${cat===c?'selected':''}>${c}</option>`).join('')}</select>
    </div>
    <div style="font-size:8px;color:var(--tx2);margin-bottom:4px">${data.total} résultats</div>
    <div id="fluxList">${(data.articles||[]).map(a=>`<div class="item ${a.level}" onclick="window.open('${a.url||''}','_blank')"><div class="item-title">${a.level==='critique'?'🔴':a.level==='haute'?'🟠':a.level==='moyenne'?'🟡':'🔵'} ${(a.title||'').substring(0,100)}</div><div class="item-meta"><span>${a.source}</span><span class="badge ${a.level}" style="font-size:7px">${a.level.toUpperCase()}</span><span>${a.cat}</span><span>${a.region}</span><span>${a.score}pts</span>${a.url?`<a href="${a.url}" target="_blank" style="color:var(--ac)">🔗</a>`:''}</div></div>`).join('')||'<div style="text-align:center;padding:20px;color:var(--tx2);font-size:10px">Aucun article</div>'}</div>`;
}

async function loadAlertes(){
  const data=await api('/api/alertes');
  document.getElementById('content').innerHTML=`
    <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:6px;margin-bottom:8px">
      <div><h2 style="font-size:15px">🚨 Alertes multi-sources</h2><p style="font-size:9px;color:var(--tx2)">Confirmées par ≥2 sources</p></div>
      <button class="btn btn-outline btn-sm" onclick="apiPost('/api/alertes/read-all');loadAlertes()">✅ Tout marquer lu</button>
    </div>
    <div id="alertList">${(data.alertes||[]).map(a=>`<div class="alert-box ${a.niveau}" onclick="apiPost('/api/alerte/${a.id}/read');loadAlertes()"><div style="display:flex;justify-content:space-between"><div><b style="font-size:10px">${a.niveau==='critique'?'🔴':'🟠'} ${(a.title||'').substring(0,100)}</b><div style="font-size:8px;color:var(--tx2);margin-top:2px">📡 ${(a.sources||[]).join(', ')} · ${a.scount} sources</div></div><span style="font-size:10px;font-weight:700;color:var(--gd)">${a.score}pts</span></div><div style="font-size:8px;color:var(--tx2);margin-top:4px">${(a.descr||'').substring(0,120)}</div><div class="item-meta" style="margin-top:2px"><span>${a.cat}</span>${a.url?`<a href="${a.url}" target="_blank" style="color:var(--ac)">🔗 Source</a>`:''}</div></div>`).join('')||'<div style="text-align:center;padding:30px;color:var(--tx2);font-size:11px">✅ Aucune alerte</div>'}</div>`;
}

async function loadSources(){
  const data=await api('/api/sources');
  document.getElementById('content').innerHTML=`
    <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:6px;margin-bottom:8px">
      <div><h2 style="font-size:15px">📰 Sources</h2><p style="font-size:9px;color:var(--tx2)">${data.sources.length} actives + ${data.custom.length} personnalisées</p></div>
      <button class="btn btn-primary btn-sm" onclick="showAddSource()">➕ Ajouter</button>
    </div>
    <div style="margin-bottom:6px;font-size:10px;font-weight:600">📡 Personnalisées :</div>
    ${data.custom.length?data.custom.map(s=>`<div class="card" style="padding:8px;margin-bottom:3px;display:flex;justify-content:space-between;align-items:center"><span><b style="font-size:10px">${s.name}</b> <span style="font-size:8px;color:var(--tx2)">${s.url.substring(0,40)}...</span></span><button class="btn btn-sm btn-danger" onclick="apiPost('/api/sources/delete/${s.name}',{}).then(loadSources)">✕</button></div>`).join(''):'<div style="font-size:9px;color:var(--tx2);padding:6px">Aucune source personnalisée</div>'}
    <div style="margin:8px 0 6px;font-size:10px;font-weight:600">📡 Actives :</div>
    ${data.sources.slice(0,80).map(s=>`<div class="card" style="padding:6px 8px;margin-bottom:2px"><div style="display:flex;justify-content:space-between;font-size:9px"><span><b>${s.source}</b> <span style="color:var(--tx2)">${s.region} · Niv.${s.slevel}</span></span><span>📄 ${s.ct} ${s.crit>0?`<span style="color:var(--rd)">🔴${s.crit}</span>`:''} ${s.haut>0?`<span style="color:var(--or)">🟠${s.haut}</span>`:''}</span></div></div>`).join('')}
    <div style="text-align:center;padding:10px;font-size:9px;color:var(--tx2)">${data.sources.length>80?'... et '+(data.sources.length-80)+' autres':''}</div>`;
}

async function loadCategories(){
  const data=await api('/api/categories');
  document.getElementById('content').innerHTML=`
    <h2 style="font-size:15px;margin-bottom:4px">📁 Catégories</h2>
    <p style="font-size:9px;color:var(--tx2);margin-bottom:8px">Répartition thématique</p>
    <div class="grid g3">${(data.categories||[]).map(c=>`<div class="card" style="text-align:center;padding:14px;cursor:pointer" onclick="document.getElementById('fCat')?document.getElementById('fCat').value='${c.cat}':null;showPage('flux')"><div style="font-size:22px;margin-bottom:4px">${c.cat==='Cyber'?'💻':c.cat==='Securite'?'🛡️':c.cat==='Politique'?'🏛️':c.cat==='Economie'?'📈':c.cat==='Diplomatie'?'🌍':c.cat==='Justice'?'⚖️':c.cat==='Societe'?'👥':'📋'}</div><div style="font-weight:600;font-size:11px">${c.cat}</div><div style="font-size:9px;color:var(--tx2)">${c.ct} articles</div>${c.crit>0?`<div style="font-size:8px;color:var(--rd)">🔴 ${c.crit} critiques</div>`:''}</div>`).join('')||'<div style="grid-column:1/-1;text-align:center;padding:20px;color:var(--tx2)">Aucune donnée</div>'}</div>`;
}

async function loadSettings(){
  const s=state.stats.total?state.stats:await api('/api/stats');
  document.getElementById('content').innerHTML=`
    <h2 style="font-size:15px;margin-bottom:4px">⚙️ Paramètres</h2>
    <p style="font-size:9px;color:var(--tx2);margin-bottom:8px">Tout est modifiable en temps réel</p>
    <div class="grid g2">
      <div class="card"><h3 style="font-size:11px;margin-bottom:8px">🎨 Thème</h3>
        <div style="display:flex;gap:4px;flex-wrap:wrap">
          ${['dark','clair','noir','bleu','vert','rouge','violet','or'].map(t=>`<button class="btn ${s.current_theme===t?'btn-primary':'btn-outline'} btn-sm" onclick="applyTheme('${t}')">${t}</button>`).join('')}
        </div>
      </div>
      <div class="card"><h3 style="font-size:11px;margin-bottom:8px">⏱️ Scraping</h3>
        <div class="floating-label"><input id="setInterval" placeholder=" " value="${s.interval||'180'}"><label>Intervalle (secondes)</label></div>
        <div class="floating-label"><input id="setHours" placeholder=" " value="${s.hours||'24'}"><label>Profondeur (heures)</label></div>
        <button class="btn btn-primary btn-sm mt-8" onclick="saveSettings()" style="margin-top:6px">💾 Sauvegarder</button>
      </div>
      <div class="card"><h3 style="font-size:11px;margin-bottom:8px">➕ Ajouter une source RSS</h3>
        <div class="floating-label"><input id="srcName" placeholder=" "><label>Nom *</label></div>
        <div class="floating-label"><input id="srcUrl" placeholder=" "><label>URL RSS *</label></div>
        <div class="floating-label"><select id="srcReg" style="width:100%;background:var(--sf);border:1px solid var(--br);padding:10px 10px 4px;border-radius:6px;color:var(--tx);font-size:9px"><option value="nat">🔵 National</option><option value="cont">🟢 Continental</option><option value="inter">🔴 International</option></select></div>
        <button class="btn btn-primary btn-sm mt-8" onclick="addSource()" style="margin-top:6px">✅ Ajouter</button>
      </div>
      <div class="card"><h3 style="font-size:11px;margin-bottom:8px">📊 Actions</h3>
        <button class="btn btn-primary w-full" onclick="apiPost('/api/scan');toast('📡 Scan lancé')" style="width:100%;margin-bottom:4px">📡 Lancer un scan</button>
        <button class="btn btn-outline w-full" onclick="apiPost('/api/alertes/read-all');toast('✅ Alertes marquées lues')" style="width:100%">✅ Tout marquer lu</button>
      </div>
    </div>`;
}

async function loadHistorique(){
  const data=await api('/api/historique');
  document.getElementById('content').innerHTML=`
    <h2 style="font-size:15px;margin-bottom:4px">📋 Historique</h2>
    <p style="font-size:9px;color:var(--tx2);margin-bottom:8px">Évolution des statistiques</p>
    ${(data.historique||[]).length?data.historique.map(h=>`<div class="card" style="padding:8px;margin-bottom:3px"><div style="display:flex;justify-content:space-between;flex-wrap:wrap;gap:4px;font-size:9px"><span>🕐 ${h.ts}</span><span>📄 ${h.total}</span><span style="color:var(--rd)">🔴 ${h.crit}</span><span style="color:var(--or)">🟠 ${h.haut}</span><span>📡 ${h.src}</span><span style="color:var(--gd)">🚨 ${h.alrt}</span></div></div>`).join(''):'<div style="text-align:center;padding:20px;color:var(--tx2)">Aucun historique</div>'}`;
}

function debounce(fn,ms){let t;return(...args)=>{clearTimeout(t);t=setTimeout(()=>fn(...args),ms)}}

async function saveSettings(){
  const iv=document.getElementById('setInterval')?.value;
  const hr=document.getElementById('setHours')?.value;
  await apiPost('/api/settings',{interval:iv,hours:hr});
  toast('✅ Paramètres sauvegardés');
}

async function addSource(){
  const n=document.getElementById('srcName')?.value.trim();
  const u=document.getElementById('srcUrl')?.value.trim();
  const r=document.getElementById('srcReg')?.value||'nat';
  if(!n||!u) return toast('❌ Nom et URL requis');
  await apiPost('/api/sources/add',{name:n,url:u,region:r,level:70});
  toast('✅ Source ajoutée: '+n);
  loadSettings();
}

async function applyTheme(t){
  await apiPost('/api/settings',{theme:t});
  document.body.className='theme-'+t;
  toast('🎨 Thème: '+t);
  loadSettings();
}

function connectSSE(){
  const src=new EventSource('/stream');
  src.onmessage=function(e){
    try{const ev=JSON.parse(e.data);if(ev.type==='stats'||ev.type==='scan_done'){loadDashboard();}}catch(err){}
  };
  src.onerror=function(){setTimeout(connectSSE,5000)};
}

// Init theme
api('/api/stats').then(s=>{
  if(s.current_theme){document.body.className='theme-'+s.current_theme}
  state.stats=s;
});

document.addEventListener('DOMContentLoaded',init);
</script></body></html>
"""

# ═══════════════════════════════════════════
# MAIN
# ═══════════════════════════════════════════

if __name__=="__main__":
    print("\n"+"="*50)
    print("  ⟡ NEXUS DIAMOND v5 — FONCTIONNEL IMMÉDIAT")
    print("="*50)
    print(f"  📡 {len(RSS)} sources RSS + personnalisables")
    print(f"  🔑 {len(ALL_KW)} mots-clés")
    print(f"  🌐 http://0.0.0.0:5000")
    print("  ⏎ Ctrl+C pour arrêter\n")
    
    Scraper().start()
    app.run(host="0.0.0.0", port=5000, debug=False, use_reloader=False)
NEXUSFINAL
