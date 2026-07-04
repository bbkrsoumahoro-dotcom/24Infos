<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>CROGEND OSINT v10 — Cellule Renseignement</title>
<style>
*{margin:0;padding:0;box-sizing:border-box;font-family:system-ui,'Segoe UI',sans-serif}
:root{--bg:#060a14;--bg2:#0c1322;--bg3:#111b2e;--card:#141f35;--card2:#1a2842;--border:#1e3352;--text:#e8edf5;--text2:#b0c4de;--text3:#6b8aaf;--orange:#f97316;--blue:#3b82f6;--cyan:#06b6d4;--emerald:#10b981;--red:#ef4444;--grad:linear-gradient(135deg,#f97316,#3b82f6);--radius:14px}
body{background:var(--bg);color:var(--text);min-height:100vh}
::-webkit-scrollbar{width:4px}::-webkit-scrollbar-track{background:var(--bg2)}::-webkit-scrollbar-thumb{background:var(--border)}
.btn{display:inline-flex;align-items:center;gap:6px;padding:8px 18px;border:none;border-radius:8px;font-size:12px;font-weight:600;cursor:pointer;transition:.25s;font-family:inherit}
.btn-primary{background:var(--grad);color:#fff}.btn-primary:hover{transform:translateY(-2px);box-shadow:0 6px 24px rgba(249,115,22,.3)}
.btn-success{background:linear-gradient(135deg,#10b981,#059669);color:#fff}
.btn-danger{background:linear-gradient(135deg,#ef4444,#dc2626);color:#fff}
.btn-sm{padding:6px 14px;font-size:11px}.btn-xs{padding:3px 10px;font-size:9px}
.btn-ghost{background:transparent;border:1px solid var(--border);color:var(--text2)}.btn-ghost:hover{border-color:var(--orange);color:var(--orange)}
.btn-orange{background:linear-gradient(135deg,var(--orange),#ea580c);color:#fff}
#sidebar{position:fixed;top:0;left:0;width:250px;height:100vh;background:var(--bg2);border-right:1px solid var(--border);z-index:100;overflow-y:auto;transition:.3s}
#sidebar .brand{padding:18px 16px;font-size:20px;font-weight:900;background:var(--grad);-webkit-background-clip:text;-webkit-text-fill-color:transparent;border-bottom:1px solid var(--border);display:flex;align-items:center;gap:8px}
#sidebar .brand .live{display:inline-block;width:8px;height:8px;border-radius:50%;background:#10b981;-webkit-text-fill-color:initial;animation:pulse 2s infinite}
@keyframes pulse{0%,100%{opacity:1;box-shadow:0 0 6px #10b981}50%{opacity:.4;box-shadow:0 0 16px #10b981}}
.nav-item{display:flex;align-items:center;gap:10px;padding:10px 16px;color:var(--text2);font-size:13px;cursor:pointer;transition:.2s;border-left:3px solid transparent}
.nav-item:hover{background:var(--bg3);color:var(--text)}
.nav-item.active{background:var(--bg3);color:var(--orange);border-left-color:var(--orange)}
.nav-item .badge{margin-left:auto;background:var(--red);color:#fff;font-size:8px;padding:2px 7px;border-radius:10px}
.nav-item .badge.success{background:#10b981}.nav-item .badge.danger{background:#ef4444}
#topbar{position:fixed;top:0;left:250px;right:0;height:56px;background:var(--bg2);border-bottom:1px solid var(--border);z-index:99;display:flex;align-items:center;justify-content:space-between;padding:0 18px}
.menu-btn{display:none;background:none;border:none;color:var(--text);font-size:24px;cursor:pointer}
@media(max-width:768px){#sidebar{transform:translateX(-100%)}#sidebar.open{transform:translateX(0)}#topbar{left:0}#main{margin-left:0}.menu-btn{display:block}}
#main{margin-left:250px;margin-top:56px;padding:18px 20px;min-height:calc(100vh-56px)}
.page{display:none}.page.active{display:block}
@keyframes fade{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:translateY(0)}}
.page-title{font-size:24px;font-weight:800;margin-bottom:18px;background:var(--grad);-webkit-background-clip:text;-webkit-text-fill-color:transparent;display:flex;align-items:center;gap:10px}
.card{background:var(--card);border:1px solid var(--border);border-radius:var(--radius);padding:16px;margin-bottom:14px;transition:.2s}
.card:hover{border-color:rgba(249,115,22,.2)}
.metrics{display:grid;grid-template-columns:repeat(auto-fit,minmax(120px,1fr));gap:10px;margin-bottom:16px}
.metric{background:var(--card2);border:1px solid var(--border);border-radius:10px;padding:12px;text-align:center;cursor:pointer;transition:.2s}
.metric:hover{transform:translateY(-2px);border-color:var(--orange)}
.metric .val{font-size:24px;font-weight:800}.metric .lab{font-size:10px;color:var(--text3);margin-top:2px}
.grid-2{display:grid;grid-template-columns:1fr 1fr;gap:14px;margin-bottom:14px}
@media(max-width:768px){.grid-2{grid-template-columns:1fr}}
.result-card{background:var(--card2);border:1px solid var(--border);border-radius:10px;padding:12px 14px;margin-bottom:8px;cursor:pointer;transition:.2s;position:relative;overflow:hidden}
.result-card:hover{border-color:var(--orange);transform:translateX(3px)}
.sev-bar{position:absolute;top:0;left:0;width:3px;height:100%}
.rc-src{display:flex;align-items:center;gap:6px;font-size:9px;color:var(--text3);margin-bottom:4px;flex-wrap:wrap}
.rc-title{font-size:13px;font-weight:600;margin-bottom:3px;line-height:1.3}
.rc-desc{font-size:10px;color:var(--text3);margin-bottom:4px}
.rc-link{font-size:9px;margin-top:3px;padding:3px 8px;background:rgba(6,182,212,.06);border-radius:4px;display:inline-block}
.rc-link a{color:var(--cyan);word-break:break-all}
.kw{font-size:7px;color:var(--orange);background:rgba(249,115,22,.08);padding:1px 6px;border-radius:3px;display:inline-block;margin:1px}
.filter-bar{display:flex;gap:4px;overflow-x:auto;padding:6px 0 12px}
.filter-btn{padding:5px 14px;border-radius:16px;border:1px solid var(--border);background:var(--bg3);color:var(--text2);font-size:10px;cursor:pointer;transition:.2s;font-family:inherit;white-space:nowrap;flex-shrink:0}
.filter-btn.active{background:var(--grad);color:#fff;border-color:transparent}
input,select,textarea{background:var(--bg3);border:1px solid var(--border);border-radius:8px;padding:9px 14px;color:var(--text);font-size:12px;font-family:inherit;width:100%;outline:none;transition:.2s}
input:focus,select:focus,textarea:focus{border-color:var(--orange)}
.gallery{display:flex;gap:10px;overflow-x:auto;padding:8px 0 12px}
.gallery::-webkit-scrollbar{height:3px}
.g-card{min-width:220px;max-width:260px;flex-shrink:0;background:var(--card2);border:1px solid var(--border);border-radius:10px;overflow:hidden;cursor:pointer;transition:.3s}
.g-card:hover{border-color:var(--orange);transform:translateY(-4px);box-shadow:0 8px 24px rgba(249,115,22,.15)}
.g-card .img{width:100%;height:140px;overflow:hidden;background:var(--bg3);display:flex;align-items:center;justify-content:center;font-size:30px;color:var(--text3)}
.g-card .img img{width:100%;height:100%;object-fit:cover;transition:.4s}
.g-card:hover .img img{transform:scale(1.08)}
.g-card .body{padding:10px 12px}
.g-card .body .t{font-size:11px;font-weight:600;display:-webkit-box;-webkit-line-clamp:2;-webkit-box-orient:vertical;overflow:hidden}
.g-card .body .s{font-size:8px;color:var(--text3);margin-top:3px}
.sbadge{font-size:7px;padding:2px 7px;border-radius:4px;font-weight:700}
.sbadge.ci{background:rgba(16,185,129,.15);color:#10b981}
.sbadge.critical{background:rgba(239,68,68,.15);color:#ef4444;animation:pulse 2s infinite}
.social-card{display:flex;align-items:center;gap:10px;padding:10px;background:var(--card2);border:1px solid var(--border);border-radius:8px;margin-bottom:6px}
.social-card .ic{font-size:20px;width:36px;height:36px;display:flex;align-items:center;justify-content:center;background:var(--bg3);border-radius:8px;flex-shrink:0}
.social-card .nm{font-size:13px;font-weight:600}
.social-card .ur{font-size:8px;color:var(--text3);word-break:break-all}
.source-item{display:flex;align-items:center;gap:8px;padding:8px 10px;border-bottom:1px solid var(--border)}
.source-item .dot{width:8px;height:8px;border-radius:50%;flex-shrink:0}
.source-item .nm{flex:1;font-size:12px;font-weight:500}
.source-item .ct{font-size:9px;color:var(--text3)}
.source-item .del{cursor:pointer;color:var(--text3);font-size:11px;opacity:.4}.source-item .del:hover{opacity:1;color:#ef4444}
.preset-btn{font-size:9px;padding:4px 12px;border-radius:14px;border:1px solid var(--border);background:var(--bg3);color:var(--text2);cursor:pointer;transition:.2s;font-family:inherit}
.preset-btn.active{background:var(--grad);color:#fff;border-color:transparent}
.tag{display:inline-flex;align-items:center;gap:3px;font-size:9px;padding:2px 8px;background:rgba(249,115,22,.06);color:var(--orange);border-radius:5px;margin:2px;border:1px solid rgba(249,115,22,.1)}
.tag .rem{cursor:pointer;font-size:7px;opacity:.4}.tag .rem:hover{opacity:1;color:var(--red)}
.target-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(180px,1fr));gap:10px}
.target-card{background:var(--card2);border:1px solid var(--border);border-radius:10px;padding:14px;text-align:center;cursor:pointer;transition:.2s}
.target-card:hover{border-color:var(--orange);transform:translateY(-3px)}
.target-card .ic{font-size:26px;margin-bottom:6px}.target-card .nm{font-size:13px;font-weight:600}
.empty{padding:30px 20px;text-align:center;color:var(--text3)}
.empty .ic{font-size:40px;margin-bottom:10px}.empty h3{font-size:15px;color:var(--text2);margin-bottom:4px}
#toast{position:fixed;top:12px;right:12px;z-index:9999;display:flex;flex-direction:column;gap:6px;max-width:360px;pointer-events:none}
.toast{padding:10px 16px;border-radius:8px;font-size:12px;font-weight:500;transition:.3s;box-shadow:0 10px 30px rgba(0,0,0,.5);pointer-events:auto;animation:tin .3s}
.toast.s{background:rgba(6,95,70,.9);color:#a7f3d0;border:1px solid rgba(16,185,129,.3)}
.toast.e{background:rgba(127,29,29,.9);color:#fecaca;border:1px solid rgba(239,68,68,.3)}
.toast.w{background:rgba(113,63,18,.9);color:#fde68a;border:1px solid rgba(234,179,8,.3)}
.toast.i{background:rgba(30,58,95,.9);color:#bae6fd;border:1px solid rgba(6,182,212,.3)}
@keyframes tin{from{transform:translateX(100%);opacity:0}to{transform:translateX(0);opacity:1}}
#overlay{display:none;position:fixed;inset:0;z-index:9998;background:rgba(6,10,20,.9);backdrop-filter:blur(8px);justify-content:center;align-items:center;flex-direction:column;gap:12px}
#overlay.active{display:flex}#overlay .t{font-size:18px;font-weight:700}#overlay .s{font-size:11px;color:var(--text3)}
#overlay .bar{width:280px;height:4px;background:var(--bg3);border-radius:2px;overflow:hidden}
#overlay .bar div{height:100%;background:var(--grad);border-radius:2px;transition:.5s}
#detail{position:fixed;top:56px;right:0;width:460px;max-width:100%;height:calc(100vh-56px);background:var(--card);border-left:1px solid var(--border);z-index:98;transform:translateX(100%);transition:.35s;overflow-y:auto;padding:18px}
#detail.open{transform:translateX(0)}#detail .close{position:absolute;top:12px;right:12px;background:var(--bg3);border:none;color:var(--text3);width:28px;height:28px;border-radius:6px;cursor:pointer;display:flex;align-items:center;justify-content:center}
#detail .close:hover{color:var(--orange)}#detail .l{font-size:9px;color:var(--text3);text-transform:uppercase;margin-bottom:2px}#detail .v{font-size:13px;margin-bottom:10px;line-height:1.4}
#detail .img{width:100%;max-height:200px;object-fit:cover;border-radius:8px;margin-bottom:12px}
#progressBar{position:fixed;bottom:0;left:250px;right:0;height:3px;background:var(--bg3);z-index:9999}
#progressBar div{height:100%;background:var(--grad);transition:.5s;width:0%}
#progressInfo{position:fixed;bottom:6px;right:14px;z-index:9999;font-size:8px;color:var(--text3);background:rgba(12,19,34,.85);padding:3px 10px;border-radius:6px;display:none}
#progressInfo.show{display:flex;gap:4px}
.scan-log{background:var(--bg3);border-radius:8px;padding:10px;max-height:180px;overflow-y:auto;font-family:monospace;font-size:10px;line-height:1.6}
.chart-bar{display:flex;align-items:center;gap:6px;margin:3px 0;font-size:9px}
.chart-bar .l{width:70px;flex-shrink:0;color:var(--text3);overflow:hidden;white-space:nowrap;text-overflow:ellipsis}
.chart-bar .tr{flex:1;height:14px;background:var(--bg);border-radius:3px;overflow:hidden}
.chart-bar .tr .f{height:100%;border-radius:3px;transition:1s}
.chart-bar .va{width:26px;text-align:right;color:var(--orange);font-weight:600}
</style>
</head>
<body>

<div id="toast"></div>
<div id="overlay"><div class="t" id="olTitle">Analyse...</div><div class="bar"><div id="olProgress" style="width:0%"></div></div><div class="s" id="olSub">Patientez...</div></div>
<div id="progressBar"><div id="progressInner"></div></div>
<div id="progressInfo"><span id="piText">Initialisation</span><span id="piPct">0%</span></div>

<nav id="sidebar">
  <div class="brand">🕵️ CROGEND <span class="live"></span></div>
  <div class="nav-item active" onclick="go('dashboard')">📊 Accueil <span class="badge" id="bdDash">0</span></div>
  <div class="nav-item" onclick="go('scan')">🔍 Scan <span class="badge success" id="bdScan">Veille</span></div>
  <div class="nav-item" onclick="go('sources')">📡 Sources <span class="badge" id="bdSrc">0</span></div>
  <div class="nav-item" onclick="go('ci')">🇨🇮 Côte d'Ivoire <span class="badge danger" id="bdCI">0</span></div>
  <div class="nav-item" onclick="go('afrique')">🌍 Afrique <span class="badge" id="bdAf">0</span></div>
  <div class="nav-item" onclick="go('intl')">🌐 International <span class="badge" id="bdIntl">0</span></div>
  <div class="nav-item" onclick="go('keywords')">🏷️ Mots-clés</div>
  <div class="nav-item" onclick="go('targets')">🎯 Cibles</div>
  <div class="nav-item" onclick="go('social')">🌐 Réseaux</div>
  <div class="nav-item" onclick="go('search')">🔎 Recherche</div>
  <div class="nav-item" onclick="go('tracking')">📊 Stats</div>
  <div class="nav-item" onclick="go('alerts')">🔔 Alertes <span class="badge danger" id="bdAlert">0</span></div>
  <div class="nav-item" onclick="go('fav')">⭐ Favoris</div>
  <div class="nav-item" onclick="go('notes')">📝 Notes</div>
  <div class="nav-item" onclick="go('reports')">📄 Rapports</div>
  <div class="nav-item" onclick="go('settings')">⚙️ Paramètres</div>
</nav>

<header id="topbar">
  <div class="left"><button class="menu-btn" onclick="S.classList.toggle('open')">☰</button><span id="status">🟢 Prêt</span></div>
  <div class="right">
    <span style="font-size:9px;color:var(--text3)" id="hCount">0</span>
    <button class="btn btn-success btn-sm" id="scanBtn" onclick="toggleScan()">▶️ Lancer</button>
    <button class="btn btn-orange btn-sm" onclick="force()">⏳ Forcer</button>
  </div>
</header>

<div id="main">

<div class="page active" id="p-dashboard">
  <h1 class="page-title">📊 CROGEND Dashboard</h1>
  <div class="metrics" id="metrics">
    <div class="metric"><div class="val" style="background:var(--grad);-webkit-background-clip:text;-webkit-text-fill-color:transparent" id="mTotal">0</div><div class="lab">24h</div></div>
    <div class="metric" onclick="go('ci')"><div class="val" style="color:#10b981" id="mCI">0</div><div class="lab">🇨🇮 CI</div></div>
    <div class="metric" onclick="go('afrique')"><div class="val" style="color:#fbbf24" id="mAf">0</div><div class="lab">🌍 Afrique</div></div>
    <div class="metric" onclick="go('intl')"><div class="val" style="color:#3b82f6" id="mIntl">0</div><div class="lab">🌐 Monde</div></div>
    <div class="metric" onclick="go('alerts')"><div class="val" style="color:#ef4444" id="mCrit">0</div><div class="lab">🚨 Critique</div></div>
    <div class="metric"><div class="val" style="color:#a78bfa" id="mSrc">0</div><div class="lab">📡 Sources</div></div>
  </div>
  <div class="filter-bar" id="dashFilters">
    <button class="filter-btn active" onclick="df('all',this)">Tous</button>
    <button class="filter-btn" onclick="df('ci',this)">🇨🇮 CI</button>
    <button class="filter-btn" onclick="df('africa',this)">🌍 Afrique</button>
    <button class="filter-btn" onclick="df('world',this)">🌐 Monde</button>
    <button class="filter-btn" onclick="df('crit',this)">🔴 Critique</button>
    <span style="width:1px;height:20px;background:var(--border);margin:0 4px"></span>
    <button class="filter-btn" onclick="df('1h',this)">1h</button>
    <button class="filter-btn" onclick="df('3h',this)">3h</button>
    <button class="filter-btn" onclick="df('6h',this)">6h</button>
    <button class="filter-btn active" onclick="df('24h',this)">24h</button>
  </div>
  <div id="dashResults"></div>
  <div style="margin-top:16px"><div style="font-size:13px;font-weight:600;color:var(--text2);margin-bottom:8px">📸 Galerie</div><div class="gallery" id="gallery"></div></div>
  <div style="display:grid;grid-template-columns:1fr 1fr;gap:12px;margin-top:16px">
    <div class="card"><div class="card-title" style="font-size:12px;font-weight:600">📊 Activité</div><div id="activityChart"></div></div>
    <div class="card"><div class="card-title" style="font-size:12px;font-weight:600">📡 Top sources</div><div id="topSourcesDash"></div></div>
  </div>
</div>

<div class="page" id="p-scan">
  <h1 class="page-title">🔍 Scan</h1>
  <div class="card" style="border-color:rgba(249,115,22,.3)">
    <p class="text-sm" style="font-size:11px;color:var(--text3);margin-bottom:10px">Veille 24h/24 en arrière-plan. Navigation libre.</p>
    <div class="flex" style="display:flex;gap:8px;flex-wrap:wrap">
      <button class="btn btn-success" id="scanCtrl" onclick="toggleScan()">▶️ Démarrer</button>
      <button class="btn btn-orange" onclick="force()">⏳ Forcer</button>
      <button class="btn btn-sm btn-ghost" onclick="stopScan()">⏹️ Stop</button>
      <button class="btn btn-sm btn-ghost" onclick="test()">🧪 Test</button>
    </div>
    <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:10px">
      <span style="font-size:10px;color:var(--text3)">Intervalle:</span>
      <select id="scanIntv" style="width:60px" onchange="sv('intv',this.value)">
        <option value="1">1m</option><option value="3">3m</option><option value="5" selected>5m</option><option value="10">10m</option>
      </select>
      <span style="font-size:10px;color:var(--text3)">Cache:</span>
      <select id="scanCache" style="width:70px" onchange="sv('cache',this.value)">
        <option value="500">500</option><option value="1000">1k</option><option value="1500" selected>1.5k</option><option value="3000">3k</option>
      </select>
    </div>
    <div style="display:grid;grid-template-columns:repeat(3,1fr);gap:8px;margin-top:10px">
      <div style="background:var(--bg3);padding:8px;border-radius:6px;text-align:center"><div style="font-size:9px;color:var(--text3)">Cycles</div><div style="font-size:18px;font-weight:700" id="sCyc">0</div></div>
      <div style="background:var(--bg3);padding:8px;border-radius:6px;text-align:center"><div style="font-size:9px;color:var(--text3)">Dernier</div><div style="font-size:16px;font-weight:600" id="sLast">--:--</div></div>
      <div style="background:var(--bg3);padding:8px;border-radius:6px;text-align:center"><div style="font-size:9px;color:var(--text3)">Nouveaux</div><div style="font-size:18px;font-weight:700;color:var(--orange)" id="sNew">0</div></div>
    </div>
  </div>
  <div class="card"><div class="card-title" style="font-size:12px;font-weight:600">📋 Journal</div><div class="scan-log" id="log"></div></div>
</div>

<div class="page" id="p-sources">
  <h1 class="page-title">📡 Sources</h1>
  <div class="grid-2">
    <div class="card">
      <div style="font-size:12px;font-weight:600;margin-bottom:8px">➕ Ajouter</div>
      <div style="display:flex;gap:6px;margin-bottom:8px">
        <input id="srcN" placeholder="Nom" style="flex:1">
        <input id="srcU" placeholder="URL RSS" style="flex:2">
      </div>
      <button class="btn btn-orange w-full" onclick="addSrc()">➕ Ajouter</button>
    </div>
    <div class="card">
      <div style="font-size:12px;font-weight:600;margin-bottom:8px">📂 Présélections</div>
      <div style="display:flex;gap:4px;flex-wrap:wrap">
        <button class="preset-btn" onclick="pre('ci',this)">🇨🇮 CI</button>
        <button class="preset-btn" onclick="pre('gov',this)">🏛️ Gouvernement</button>
        <button class="preset-btn" onclick="pre('gnews',this)">📰 Google</button>
        <button class="preset-btn" onclick="pre('africa',this)">🌍 Afrique</button>
        <button class="preset-btn" onclick="pre('world',this)">🌐 International</button>
      </div>
      <div id="presetList" style="margin-top:8px"></div>
      <button class="btn btn-sm btn-blue" style="margin-top:8px;width:100%" onclick="loadAll()">📥 Charger tout</button>
    </div>
  </div>
  <div class="card">
    <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:8px">
      <span style="font-size:12px;font-weight:600">📡 Sources actives <span class="badge" id="srcCount">0</span></span>
      <button class="btn btn-xs btn-danger" onclick="rmAllSrc()">🗑️</button>
    </div>
    <div id="srcList"></div>
  </div>
</div>

<div class="page" id="p-ci">
  <h1 class="page-title">🇨🇮 Côte d'Ivoire</h1>
  <div class="filter-bar">
    <button class="filter-btn active" onclick="ciF('all',this)">Tous</button>
    <button class="filter-btn" onclick="ciF('securite',this)">🔒 Sécurité</button>
    <button class="filter-btn" onclick="ciF('politique',this)">🏛️ Politique</button>
    <button class="filter-btn" onclick="ciF('economie',this)">💰 Économie</button>
    <button class="filter-btn" onclick="ciF('justice',this)">⚖️ Justice</button>
  </div>
  <div id="ciRes"></div>
</div>

<div class="page" id="p-afrique"><h1 class="page-title">🌍 Afrique</h1><div id="afRes"></div></div>
<div class="page" id="p-intl"><h1 class="page-title">🌐 International</h1><div id="intlRes"></div></div>

<div class="page" id="p-keywords">
  <h1 class="page-title">🏷️ Mots-clés</h1>
  <div class="card">
    <div style="display:flex;justify-content:space-between;margin-bottom:8px">
      <span style="font-size:12px;font-weight:600">Surveillance <span class="badge" id="kwCount">0</span></span>
      <div style="display:flex;gap:4px">
        <button class="btn btn-xs btn-ghost" onclick="expKw()">📤</button>
        <button class="btn btn-xs btn-ghost" onclick="impKw()">📥</button>
      </div>
    </div>
    <div style="display:flex;gap:6px;margin-bottom:8px">
      <input id="kwIn" placeholder="Ajouter..." style="flex:1" onkeydown="if(event.key==='Enter')addKw()">
      <button class="btn btn-orange" onclick="addKw()">➕</button>
    </div>
    <div id="kwList" style="display:flex;gap:4px;flex-wrap:wrap"></div>
    <div style="display:flex;gap:6px;margin-top:8px">
      <button class="btn btn-sm btn-secondary" onclick="resetKw()">↩️ Défaut</button>
      <button class="btn btn-sm btn-ghost" onclick="clrKw()">🗑️</button>
    </div>
  </div>
</div>

<div class="page" id="p-targets">
  <h1 class="page-title">🎯 Cibles</h1>
  <div class="card">
    <div style="display:flex;gap:6px;">
      <input id="tgN" placeholder="Nom" style="flex:1">
      <input id="tgU" placeholder="URL" style="flex:1">
      <select id="tgT" style="width:100px"><option value="personne">Personne</option><option value="organisation">Organisation</option><option value="site">Site</option></select>
      <button class="btn btn-orange" onclick="addTg()">➕</button>
    </div>
  </div>
  <div class="card"><div style="font-size:12px;font-weight:600;margin-bottom:8px">📋 Listes <span class="badge" id="tgCount">0</span></div><div class="target-grid" id="tgGrid"></div></div>
</div>

<div class="page" id="p-social">
  <h1 class="page-title">🌐 Réseaux</h1>
  <div class="grid-2">
    <div class="card"><div style="font-size:12px;font-weight:600;margin-bottom:6px">📘 Facebook</div><div style="display:flex;gap:4px;margin-bottom:6px"><input id="fbU" placeholder="URL" style="flex:2"><input id="fbN" placeholder="Nom" style="flex:1"><button class="btn btn-sm btn-blue" onclick="addFB()">➕</button></div><div id="fbL"></div></div>
    <div class="card"><div style="font-size:12px;font-weight:600;margin-bottom:6px">🐦 Twitter</div><div style="display:flex;gap:4px;margin-bottom:6px"><input id="twU" placeholder="URL" style="flex:2"><input id="twN" placeholder="Nom" style="flex:1"><button class="btn btn-sm btn-blue" onclick="addTW()">➕</button></div><div id="twL"></div></div>
  </div>
  <div class="grid-2">
    <div class="card"><div style="font-size:12px;font-weight:600;margin-bottom:6px">✈️ Telegram</div><div style="display:flex;gap:4px;margin-bottom:6px"><input id="tgU" placeholder="URL" style="flex:2"><input id="tgN" placeholder="Nom" style="flex:1"><button class="btn btn-sm btn-blue" onclick="addTG()">➕</button></div><div id="tgL"></div></div>
    <div class="card"><div style="font-size:12px;font-weight:600;margin-bottom:6px">▶️ YouTube</div><div style="display:flex;gap:4px;margin-bottom:6px"><input id="ytU" placeholder="URL" style="flex:2"><input id="ytN" placeholder="Nom" style="flex:1"><button class="btn btn-sm btn-blue" onclick="addYT()">➕</button></div><div id="ytL"></div></div>
  </div>
</div>

<div class="page" id="p-search">
  <h1 class="page-title">🔎 Recherche</h1>
  <div class="card">
    <div style="display:flex;gap:6px">
      <input id="sq" placeholder="Rechercher..." style="flex:2" onkeydown="if(event.key==='Enter')searchW()">
      <select id="sl" style="width:70px"><option value="fr">FR</option><option value="en">EN</option></select>
      <input id="sc" placeholder="CI" style="width:40px;font-size:10px;padding:6px" value="CI">
      <button class="btn btn-orange" onclick="searchW()">🔍</button>
    </div>
    <div id="sCount" style="font-size:9px;color:var(--text3);margin-top:6px"></div>
    <div id="sRes" style="margin-top:8px"></div>
  </div>
</div>

<div class="page" id="p-tracking">
  <h1 class="page-title">📊 Stats</h1>
  <div class="grid-2">
    <div class="card"><div style="font-size:12px;font-weight:600;margin-bottom:8px">🏷️ Top mots-clés</div><div id="topKW"></div></div>
    <div class="card"><div style="font-size:12px;font-weight:600;margin-bottom:8px">📡 Top sources</div><div id="topSRC"></div></div>
  </div>
  <div class="card"><div style="font-size:12px;font-weight:600;margin-bottom:8px">📅 Activité</div><div id="actChart"></div></div>
</div>

<div class="page" id="p-alerts"><h1 class="page-title">🔔 Alertes</h1><div style="display:flex;justify-content:flex-end;margin-bottom:8px"><button class="btn btn-danger btn-sm" onclick="clrAl()">🗑️</button></div><div class="card"><div id="alList"></div></div></div>
<div class="page" id="p-fav"><h1 class="page-title">⭐ Favoris</h1><div style="display:flex;justify-content:flex-end;margin-bottom:8px"><button class="btn btn-danger btn-sm" onclick="clrFav()">🗑️</button></div><div class="card"><div id="favList"></div></div></div>

<div class="page" id="p-notes">
  <h1 class="page-title">📝 Notes</h1>
  <div class="card">
    <div style="display:flex;justify-content:space-between;margin-bottom:8px">
      <span style="font-size:12px;font-weight:600">Bloc-notes</span>
      <div style="display:flex;gap:4px">
        <button class="btn btn-success btn-sm" onclick="svNotes()">💾</button>
        <button class="btn btn-orange btn-sm" onclick="expNotes()">📤</button>
      </div>
    </div>
    <textarea id="ntArea" style="min-height:250px" placeholder="Notes de renseignement..."></textarea>
    <div style="font-size:9px;color:var(--text3);margin-top:4px">💡 Ctrl+S · Auto 30s</div>
  </div>
</div>

<div class="page" id="p-reports">
  <h1 class="page-title">📄 Rapports</h1>
  <div class="grid-2">
    <div class="card"><div style="font-size:12px;font-weight:600;margin-bottom:6px">📊 Rapport 24h</div><p style="font-size:10px;color:var(--text3);margin-bottom:8px">Rapport Word classé CI → Afrique → International</p><button class="btn btn-orange" onclick="genReport()">📄 Générer</button></div>
    <div class="card"><div style="font-size:12px;font-weight:600;margin-bottom:6px">💾 Configuration</div><p style="font-size:10px;color:var(--text3);margin-bottom:8px">Export/Import complet</p><div style="display:flex;gap:4px"><button class="btn btn-sm btn-secondary" onclick="expCfg()">📤</button><button class="btn btn-sm btn-ghost" onclick="impCfg()">📥</button></div></div>
  </div>
  <div class="card"><div style="font-size:12px;font-weight:600;margin-bottom:8px">📋 Historique</div><div id="repHist"></div></div>
</div>

<div class="page" id="p-settings">
  <h1 class="page-title">⚙️ Paramètres</h1>
  <div class="card">
    <div style="font-size:12px;font-weight:600;margin-bottom:10px">🔍 Scan</div>
    <div style="display:flex;flex-direction:column;gap:10px">
      <div><span style="font-size:10px;color:var(--text3)">Intervalle (min)</span><input id="setIntv" type="number" min="1" max="60" onchange="sv('intv',this.value)"></div>
      <div><span style="font-size:10px;color:var(--text3)">Cache max</span><input id="setCache" type="number" min="100" max="10000" onchange="sv('cache',this.value)"></div>
    </div>
  </div>
  <div class="card">
    <div style="font-size:12px;font-weight:600;margin-bottom:8px;color:var(--red)">🛡️ Zone dangereuse</div>
    <button class="btn btn-danger" onclick="nuke()">🗑️ Tout effacer</button>
    <p style="font-size:9px;color:var(--text3);margin-top:4px">⚠️ Supprime TOUTES les données</p>
  </div>
</div>

</div>

<div id="detail">
  <button class="close" onclick="closeD()">✕</button>
  <div id="dTitle" style="font-size:16px;font-weight:700;margin-bottom:12px">Détails</div>
  <div id="dBody"></div>
</div>

<script>
/* ===== CROGEND v10 — Moteur complet et stable ===== */

// STORAGE KEYS
const SK = { r:'cg_r', s:'cg_s', k:'cg_k', f:'cg_f', t:'cg_t', g:'cg_g', y:'cg_y', v:'cg_v', a:'cg_a', p:'cg_p', tg:'cg_tg' };

// CRITICAL KEYWORDS
const CK = ["meurtre","assassinat","attentat","coup d'état","putsch","guerre","massacre","enlèvement","terrorisme","attaque","explosion","bombe","fusillade","tuerie","génocide","crime de guerre","torture","exécution","émeute","insurrection","rébellion","milice","urgence","catastrophe","incendie","inondation","accident","mort","décès","victime","arrestation","scandale","corruption","détournement","évasion","prison","révolte","pillage"];

function getDefKws(){
  return ["côte d'ivoire","abidjan","sécurité","attaque","cyber","hack","piratage","vol","fraude","meurtre","assassinat","violence","conflit","guerre","manifestation","politique","élection","présidentiel","gouvernement","ministre","démission","crise","scandale","corruption","détournement","justice","tribunal","procès","prison","enlèvement","disparition","terrorisme","djihadiste","militaire","armée","police","gendarmerie","braquage","drogue","trafic","blanchiment","argent","banque","finance","économie","investissement","dette","fmi","banque mondiale","afrique","ua","cedeao","onu","interpol","france","chine","russie","usa","europe","immigration","réfugié","santé","épidémie","vaccin","hôpital","éducation","emploi","chômage","transport","accident","incendie","inondation","catastrophe","climat","environnement","énergie","pétrole","gaz","mine","or","cacao","agriculture","technologie","numérique","internet","réseau","mobile","données","confidentialité","surveillance","espionnage","renseignement","osint","dark web","bitcoin","cryptomonnaie","ransomware","malware","phishing","ddos","vulnérabilité","zero day","exploit","backdoor","fausse nouvelle","désinformation","propagande","deepfake","influence","haine","racisme","discrimination","viol","agression","trafic humain","esclavage","mineur","enfant","femme","droit","loi","réforme","constitution","référendum","vote","parlement","député","sénat","développement","infrastructure","projet","aide","coopération","partenariat","sommet","conférence","diplomatie","ambassade","sanction","embargo","paix","réconciliation","amnistie","déplacé","aide humanitaire","ong","société civile","médias","presse","journal","radio","télévision","réseau social","facebook","twitter","whatsapp","telegram","youtube","tiktok","instagram","intelligence artificielle","ia","gpt","chatgpt","big data","cloud","sécurité informatique","pentest","cyberattaque","cybercriminalité","cyberdéfense","hacktivisme","anonymous","trafic d'armes","crise humanitaire","famine","pauvreté","inégalité","ethnie","tribu","religion","islam","chrétien","conflit intercommunautaire","coup d'état","mutinerie","rébellion","bombe","explosion","fusillade","massacre","torture","exécution","liberté de la presse","censure","coupure internet","cyber espionnage","cyber guerre","investigation numérique","authentification","mfa","2fa","biométrie","mot de passe","pare-feu","firewall","siem","wazuh","nmap","metasploit","crogend","dggn","sdao","gendarmerie","renseignement opérationnel","alerte","urgence","opération","forces spéciales","anti-terroriste","croat","service de renseignement","sécurité nationale","défense","frontière","douane","immigration","visa","passeport","identité","contrôle","surveillance","caméra","drone","satellite","géolocalisation","gps","piste","enquête","témoin","indice","preuve","perquisition","coopération judiciaire","extradition","mandat d'arrêt"];
}

const PRESETS = {
  // 🇨🇮 CI — existant (inchangé)
  ci:[
    {n:"Abidjan.net",u:"https://news.abidjan.net/rss"},
    {n:"RTI Info",u:"https://www.rti.ci/rss"},
    {n:"FratMat",u:"https://www.fratmat.info/feed"},
    {n:"Koaci",u:"https://www.koaci.com/rss"},
    {n:"7Info",u:"https://7info.ci/feed"},
    {n:"AIP",u:"https://www.aip.ci/feed/"},
    {n:"Linfodrome",u:"https://www.linfodrome.com/feed"},
    {n:"Connection Ivoirienne",u:"https://www.connectionivoirienne.net/feed"},
    {n:"Yeclo",u:"https://www.yeclo.com/feed"},
    {n:"Afrikipresse",u:"https://www.afrikipresse.com/feed"},
    {n:"Ivoire Business",u:"https://www.ivoirebusiness.net/feed"},
    {n:"LeBabi",u:"https://www.lebabi.net/feed"},
    {n:"Alerte Info",u:"https://www.alerte-info.net/feed"},
    {n:"Gouv.ci",u:"https://www.gouv.ci/actualites?format=feed"},
    {n:"LeMatin CI",u:"https://www.lematin.ci/feed"},
    {n:"Infos Plus",u:"https://infosplusci.com/feed"},
    {n:"Studiotivi",u:"https://studiotivi.com/feed"},
    {n:"LeBledParle",u:"https://www.lebledparle.com/feed"},
    {n:"Le Patriote",u:"https://lepatriote.ci/rss/latest-posts"},
    {n:"L'Intelligent",u:"https://lintelligentdabidjan.ci/feed"},
    {n:"Ivorian.net",u:"https://ivorian.net/feed"}
  ],
  // 🏛️ Gouvernement CI — enrichi
  gov:[
    {n:"Portail Gouv",u:"https://www.gouv.ci/rss"},
    {n:"Primature",u:"https://www.primature.ci/feed"},
    {n:"Ass Nat",u:"https://www.assnat.ci/feed"},
    {n:"Sénat CI",u:"https://www.senat.ci/feed/"},
    {n:"Présidence CI",u:"https://www.presidence.ci/feed/"}
  ],
  // 📰 Google — enrichi
  gnews:[
    {n:"Google News CI",u:"https://news.google.com/rss?hl=fr&gl=CI&ceid=CI:fr"},
    {n:"Google Sécurité CI",u:"https://news.google.com/rss/search?q=s%C3%A9curit%C3%A9+C%C3%B4te+d%27Ivoire&hl=fr&gl=CI&ceid=CI:fr"},
    {n:"Google Politique CI",u:"https://news.google.com/rss/search?q=Politique+C%C3%B4te+d%27Ivoire&hl=fr&gl=CI&ceid=CI:fr"},
    {n:"Google Afrique",u:"https://news.google.com/rss/search?q=Afrique+s%C3%A9curit%C3%A9&hl=fr&gl=CI&ceid=CI:fr"},
    {n:"Google International",u:"https://news.google.com/rss/search?q=intelligence+s%C3%A9curit%C3%A9+terrorisme+cyber&hl=fr&gl=FR&ceid=FR:fr"}
  ],
  // 🌍 AFRIQUE — NOUVEAU
  africa:[
    {n:"Africanews",u:"https://fr.africanews.com/feed/rss?themes=news"},
    {n:"Jeune Afrique",u:"https://www.jeuneafrique.com/feed/"},
    {n:"RFI Afrique",u:"https://www.rfi.fr/fr/afrique/rss"},
    {n:"France24 Afrique",u:"https://www.france24.com/fr/afrique/rss"},
    {n:"Le Monde Afrique",u:"https://www.lemonde.fr/afrique/rss_full.xml"},
    {n:"BBC Afrique",u:"https://www.bbc.com/afrique/index.xml"},
    {n:"TV5 Monde Afrique",u:"https://information.tv5monde.com/afrique/rss"},
    {n:"Africa Intelligence",u:"https://www.africaintelligence.com/info/rss"},
    {n:"Reuters Africa",u:"https://www.reuters.com/world/africa/rss"},
    {n:"Al Jazeera Afrique",u:"https://www.aljazeera.com/xml/rss/all.xml"},
    {n:"The Africa Report",u:"https://www.theafricareport.com/feed/"},
    {n:"Bing Actu Afrique",u:"https://www.bing.com/news/search?q=afrique&qft=sortbydate%3d%221%22+interval%3d%227%22&form=YFNR&format=rss"},
    {n:"Le360 Afrique",u:"https://www.le360.ma/feed"},
    {n:"Actu Cameroun",u:"https://actucameroun.com/feed/"},
    {n:"Senego",u:"https://senego.com/feed/"},
    {n:"Koaci Afrique",u:"https://www.koaci.com/rss/category/afrique"}
  ],
  // 🌐 INTERNATIONAL — NOUVEAU
  world:[
    {n:"Le Monde",u:"https://www.lemonde.fr/rss/une.xml"},
    {n:"Le Figaro",u:"https://www.lefigaro.fr/rss/figaro_une.xml"},
    {n:"France24",u:"https://www.france24.com/fr/rss"},
    {n:"Euronews",u:"https://www.euronews.com/rss?culture=fr"},
    {n:"BBC News",u:"https://feeds.bbci.co.uk/news/rss.xml"},
    {n:"The Guardian",u:"https://www.theguardian.com/world/rss"},
    {n:"NY Times",u:"https://rss.nytimes.com/services/xml/rss/nyt/World.xml"},
    {n:"Reuters",u:"https://www.reuters.com/world/rss"},
    {n:"AP News",u:"https://feeds.ap.org/feeds/feed?q=world&format=rss"},
    {n:"CNN",u:"http://rss.cnn.com/rss/edition.rss"},
    {n:"Al Jazeera",u:"https://www.aljazeera.com/xml/rss/all.xml"},
    {n:"DW",u:"https://rss.dw.com/rdf/rss-en-world"},
    {n:"Le Point",u:"https://www.lepoint.fr/rss.xml"},
    {n:"Libération",u:"https://www.liberation.fr/feed/"},
    {n:"Courrier Intl",u:"https://www.courrierinternational.com/feed/all/rss.xml"}
  ]
};

// STATE
let S = { scan:false, timer:null, cycles:0, total:0, busy:false, last:null, lastN:0, dur:0, ok:0 };
let D = { r:[], s:[], k:[], fb:[], tw:[], tg:[], yt:[], v:[], a:[], p:[], tg:[] };
let DF = 'all', DTF = '24h', CF = 'all';

// HELPERS
function id(x){return document.getElementById(x)}
function get(x){try{return JSON.parse(localStorage.getItem(x)||'[]')}catch(e){return[]}}
function set(x,d){localStorage.setItem(x,JSON.stringify(d))}
function load(){
  Object.keys(SK).forEach(k=>{D[k]=get(SK[k])});
  if(!D.k||!D.k.length)D.k=getDefKws();
}
function save(){Object.keys(SK).forEach(k=>{set(SK[k],D[k]||[])})}
function esc(t){const d=document.createElement('div');d.textContent=t;return d.innerHTML}
function slp(ms){return new Promise(r=>setTimeout(r,ms))}
function r24(d){if(!d)return false;try{return(Date.now()-new Date(d).getTime())<86400000}catch(e){return false}}
function r24a(a){return a.filter(x=>r24(x.date))}
function img(d){if(!d)return null;const m=d.match(/<img[^>]+src=["']([^"']+)["']/i);if(m&&m[1]&&!m[1].includes('google'))return m[1];return null}
function sType(n){
  const s=(n||'').toLowerCase();
  const ciSources=['abidjan','rti','fratmat','koaci','7info','lebledparle','linfodrome','connection','yeclo','ivoire','lebabi','aip','alerte','afrik','gouv','primature','assnat','lematin','infos plus','studiotivi','patriote','intelligent','ivorian'];
  const afSources=['africanews','jeune afrique','rfi afrique','france24 afrique','le monde afrique','bbc afrique','tv5','africa intelligence','reuters africa','al jazeera','africa report','bing actu afrique','le360','actu cameroun','senego','koaci afrique'];
  if(ciSources.some(k=>s.includes(k))) return 'ci';
  if(afSources.some(k=>s.includes(k))) return 'africa';
  return 'world';
}
function sev(t,k){if(!t)return'low';const tl=t.toLowerCase();if(k&&k.some(x=>CK.includes(x)))return'critical';if(CK.some(x=>tl.includes(x)))return'critical';return'low'}

// TOAST
function to(m,t,d){
  t=t||'i';d=d||3000;
  const c=id('toast');if(!c)return;
  const e=document.createElement('div');
  e.className='toast '+{s:'s',e:'e',w:'w',i:'i',c:'e'}[t]||'i';
  e.innerHTML='<span>'+(t==='s'?'✅':t==='e'?'❌':t==='w'?'⚠️':'ℹ️')+'</span> '+esc(m);
  c.appendChild(e);
  setTimeout(()=>{e.style.opacity='0';setTimeout(()=>e.remove(),300)},d);
}

function log(m){
  const l=id('log');if(!l)return;
  const t=new Date().toLocaleTimeString('fr-FR');
  l.innerHTML+='<div>['+t+'] '+esc(m)+'</div>';l.scrollTop=l.scrollHeight;
}

function sts(t,s){
  const e=id('status');if(e)e.textContent=t||'🟢 Prêt';
}

function showL(t,p){
  const o=id('overlay');const ti=id('olTitle');const b=id('olProgress');const su=id('olSub');
  if(ti)ti.textContent=t||'Analyse...';
  if(o)o.classList.add('active');
  if(b&&p!==null){const x=Math.min(Math.max(p,0),100);b.style.width=x+'%';if(su)su.textContent=Math.round(x)+'%'}
  const pi=id('progressInner');if(pi)pi.style.width=(p||0)+'%';
  const pi2=id('progressInfo');if(pi2)pi2.classList.add('show');
  const pt=id('piText');if(pt)pt.textContent=t||'Analyse...';
  const pp=id('piPct');if(pp)pp.textContent=Math.round(p||0)+'%';
}

function hideL(){
  const o=id('overlay');if(o)o.classList.remove('active');
  const pi=id('progressInner');if(pi)pi.style.width='0%';
  const pi2=id('progressInfo');if(pi2)pi2.classList.remove('show');
}

function go(pg){
  document.querySelectorAll('.page').forEach(p=>p.classList.remove('active'));
  document.querySelectorAll('.nav-item').forEach(n=>n.classList.remove('active'));
  const p=id('p-'+pg);if(p)p.classList.add('active');
  const map={dashboard:'Accueil',scan:'Scan',sources:'Sources',ci:'Côte',afrique:'Afrique',intl:'International',keywords:'Mots-clés',targets:'Cibles',social:'Réseaux',search:'Recherche',tracking:'Stats',alerts:'Alertes',fav:'Favoris',notes:'Notes',reports:'Rapports',settings:'Paramètres'};
  const l=map[pg]||'';
  document.querySelectorAll('.nav-item').forEach(n=>{if(l&&n.textContent.includes(l))n.classList.add('active')});
  const s=id('sidebar');if(s&&s.classList.contains('open')&&window.innerWidth<=768)s.classList.remove('open');
  if(pg==='ci')renderCI();if(pg==='afrique')renderAf();if(pg==='intl')renderIntl();if(pg==='tracking')renderTrack();
  if(pg==='sources')renderSrc();if(pg==='reports')renderHist();up();
}

// PROXY
const PX=[
  {n:'rss2json',f:u=>`https://api.rss2json.com/v1/api.json?rss_url=${encodeURIComponent(u)}`},
  {n:'feed2json',f:u=>`https://www.toptal.com/developers/feed2json/convert?url=${encodeURIComponent(u)}`},
  {n:'rsstojson',f:u=>`https://rssjson.vercel.app/api?url=${encodeURIComponent(u)}`}
];

async function fetchRSS(u){
  for(const p of PX){
    try{
      const c=new AbortController();const t=setTimeout(()=>c.abort(),8000);
      const r=await fetch(p.f(u),{signal:c.signal});clearTimeout(t);
      if(!r.ok)continue;
      const j=await r.json();
      if(j&&j.items&&j.items.length)return{items:j.items,proxy:p.n};
    }catch(e){continue}
  }
  try{
    const c=new AbortController();const t=setTimeout(()=>c.abort(),10000);
    const r=await fetch(`https://api.allorigins.win/raw?url=${encodeURIComponent(u)}`,{signal:c.signal});clearTimeout(t);
    if(!r.ok)throw'fail';const txt=await r.text();
    const x=new DOMParser().parseFromString(txt,'text/xml');
    const items=x.querySelectorAll('item');
    if(items.length){const p=[];items.forEach(i=>{p.push({title:i.querySelector('title')?.textContent||'Sans titre',link:i.querySelector('link')?.textContent||'#',description:i.querySelector('description')?.textContent||'',pubDate:i.querySelector('pubDate')?.textContent||new Date().toISOString()})});if(p.length)return{items:p,proxy:'direct'}}
  }catch(e){}
  return null;
}

// ===== SCAN =====
async function scan(){
  if(S.busy){log('⏳ Déjà en cours');return}
  S.busy=true;const st=Date.now();
  const srcs=D.s.filter(s=>s.u&&s.u.startsWith('http'));
  const max=parseInt(localStorage.getItem('cg_cache')||'1500');
  const par=3;
  showL('🔍 '+srcs.length+' sources...',0);sts('🔴 Scan...',true);
  log('🚀 Cycle #'+(S.cycles+1)+' — '+srcs.length+' sources');
  let total=0,ok=0;
  for(let i=0;i<srcs.length;i+=par){
    const batch=srcs.slice(i,i+par);
    const res=await Promise.all(batch.map(async src=>{
      try{
        const r=await fetchRSS(src.u);
        if(!r||!r.items||!r.items.length)return 0;
        let f=0;
        for(const item of r.items){
          if(max>0&&D.r.length>=max)break;
          const txt=((item.title||'')+' '+(item.description||'')).toLowerCase();
          const m=D.k.filter(k=>txt.includes(k.toLowerCase()));
          if(m.length){
            const link=item.link||item.guid||item.url||'#';
            if(!D.r.some(x=>x.link===link&&link!=='#')){
              D.r.unshift({title:item.title||'Sans titre',link,description:(item.description||'').substring(0,600),source:src.n,date:item.pubDate||new Date().toISOString(),keywords:m.slice(0,8),proxy:r.proxy,severity:sev(txt,m),sourceType:sType(src.n)});
              f++;
            }
          }
        }
        if(f>0)ok++;return f;
      }catch(e){return 0}
    }));
    res.forEach(f=>total+=f);
    const pct=Math.min(((i+par)/srcs.length)*100,100);
    showL('📡 '+Math.min(i+par,srcs.length)+'/'+srcs.length,pct);
    await slp(20);
  }
  if(total>0){const m=parseInt(localStorage.getItem('cg_cache')||'1500');if(m>0&&D.r.length>m)D.r=D.r.slice(0,m);save()}
  const dur=((Date.now()-st)/1000).toFixed(1);
  S.cycles++;S.total+=total;S.last=new Date().toISOString();S.lastN=total;S.dur=dur;S.ok=ok;S.busy=false;
  hideL();up();log('✅ Cycle #'+S.cycles+' — '+total+' résultats en '+dur+'s ('+ok+'/'+srcs.length+' sources)');
  sts('🟢 '+(total?'+'+total:'OK'),S.scan);
  if(total>0){const c=D.r.slice(0,total).filter(x=>x.severity==='critical');if(c.length)addA(c.length+' alerte(s)','c',c.slice(0,3).map(x=>x.title).join(', '));to('✅ '+total+' résultat(s)','s')}
}

function force(){if(!S.busy)scan();else to('⚠️ En cours','w')}

async function test(){
  to('🧪 Test...','i');log('🧪 Test Google News');
  const r=await fetchRSS('https://news.google.com/rss?hl=fr&gl=CI&ceid=CI:fr');
  if(r&&r.items&&r.items.length){log('✅ '+r.items.length+' articles ('+r.proxy+')');to('✅ OK','s')}
  else{log('❌ Échec');to('❌ Test échoué','e')}
}

function toggleScan(){
  S.scan=!S.scan;
  ['scanBtn','scanCtrl'].forEach(x=>{const b=id(x);if(b)b.innerHTML=S.scan?'⏸ Pause':'▶️ Lancer'});
  const b2=id('bdScan');if(b2){b2.textContent=S.scan?'Actif':'Veille';b2.className='badge '+(S.scan?'success':'')}
  if(S.scan){
    if(S.timer)clearInterval(S.timer);
    const iv=(parseFloat(id('scanIntv')?.value||'5'))*60000;
    S.timer=setInterval(()=>{if(!S.busy&&S.scan)scan()},iv);
    if(!S.busy)scan();log('▶️ Auto activé');to('▶️ Veille activée','s');
  }else{if(S.timer){clearInterval(S.timer);S.timer=null}log('⏸ Désactivé');to('⏸ Veille désactivée','w')}
  up()
}

function stopScan(){if(S.scan)toggleScan();if(S.busy){S.busy=false;hideL()}}

// ===== UI =====
function up(){
  const r=r24a(D.r);const ci=r.filter(x=>x.sourceType==='ci');const crit=r.filter(x=>x.severity==='critical');
  const $=id;
  const safe=(e,v)=>{const x=$(e);if(x)x.textContent=v};
  safe('mTotal',r.length);safe('mCI',ci.length);safe('mAf',r.filter(x=>x.sourceType==='africa').length);safe('mIntl',r.filter(x=>x.sourceType==='world').length);safe('mCrit',crit.length);safe('mSrc',D.s.length);
  safe('bdDash',r.length);safe('bdSrc',D.s.length);safe('bdCI',ci.length);safe('bdAf',r.filter(x=>x.sourceType==='africa').length);safe('bdIntl',r.filter(x=>x.sourceType==='world').length);
  const ba=$('bdAlert');if(ba){ba.textContent=crit.length;ba.className='badge danger'}
  safe('hCount',r.length+' 24h');
  safe('sCyc',S.cycles);safe('sLast',S.last?new Date(S.last).toLocaleTimeString('fr-FR'):'--:--');safe('sNew',S.lastN);
  const sb=$('bdScan');if(sb){sb.textContent=S.scan?'Actif':'Veille';sb.className='badge '+(S.scan?'success':'')}
  renderDash();renderGal();renderKW();renderSrc();renderAl();
  const tc=id('tgCount');if(tc)tc.textContent=D.tg.length;
  const dv=id('dashResults');if(dv)renderDash();
  const pi=id('progressInner');
  if(pi&&!S.busy){pi.style.width='0%';const pi2=id('progressInfo');if(pi2)pi2.classList.remove('show')}
}

// ===== RENDER DASH =====
function renderDash(){
  const el=id('dashResults');if(!el)return;
  let r=r24a(D.r);
  if(DF==='ci')r=r.filter(x=>x.sourceType==='ci');
  else if(DF==='africa')r=r.filter(x=>x.sourceType==='africa');
  else if(DF==='world')r=r.filter(x=>x.sourceType==='world');
  else if(DF==='crit')r=r.filter(x=>x.severity==='critical');
  if(DTF==='1h')r=r.filter(x=>{try{return(Date.now()-new Date(x.date).getTime())<3600000}catch(e){return false}});
  else if(DTF==='3h')r=r.filter(x=>{try{return(Date.now()-new Date(x.date).getTime())<10800000}catch(e){return false}});
  else if(DTF==='6h')r=r.filter(x=>{try{return(Date.now()-new Date(x.date).getTime())<21600000}catch(e){return false}});
  if(!r.length){el.innerHTML='<div class="empty"><div class="ic">📊</div><h3>Aucun résultat</h3><p>Aucun résultat pour ces filtres</p></div>';return}
  el.innerHTML=r.slice(0,50).map((x,i)=>{
    const sevCls=x.severity==='critical'?' style="background:#ef4444"':'';
    return `<div class="result-card anim-fade" onclick="openD(${D.r.indexOf(x)})">
      <div class="sev-bar"${sevCls}></div>
      <div class="rc-src">
        <span class="sbadge ${x.sourceType==='ci'?'ci':x.severity==='critical'?'critical':'rss'}">${x.sourceType==='ci'?'🇨🇮 CI':x.severity==='critical'?'🔴 CRIT':'📰'}</span>
        <span>📡 ${esc(x.source)}</span>
        <span>${x.date?new Date(x.date).toLocaleTimeString('fr-FR'):''}</span>
        <span style="margin-left:auto;font-size:8px;color:var(--text3)">${x.proxy||''}</span>
      </div>
      <div class="rc-title">${esc(x.title||'')}</div>
      <div class="rc-desc">${esc((x.description||'').substring(0,200))}</div>
      ${x.keywords&&x.keywords.length?`<div>${x.keywords.slice(0,5).map(k=>`<span class="kw">#${esc(k)}</span>`).join('')}</div>`:''}
      <div class="rc-link"><a href="${x.link||'#'}" target="_blank" onclick="event.stopPropagation()">🔗 ${(x.link||'').substring(0,60)}</a></div>
    </div>`
  }).join('');
}

function df(f,btn){
  if(['all','ci','africa','world','crit'].includes(f))DF=f;
  else DTF=f;
  document.querySelectorAll('#dashFilters .filter-btn').forEach(b=>b.classList.remove('active'));
  if(btn)btn.classList.add('active');
  renderDash();
}

function renderCI(){
  const el=id('ciRes');if(!el)return;
  let r=D.r.filter(x=>x.sourceType==='ci'&&r24(x.date));
  if(CF!=='all'){const t=CF.toLowerCase();r=r.filter(x=>(x.title||'').toLowerCase().includes(t)||(x.description||'').toLowerCase().includes(t))}
  if(!r.length){el.innerHTML='<div class="empty"><div class="ic">🇨🇮</div><h3>Aucun résultat CI</h3></div>';return}
  el.innerHTML=r.slice(0,50).map((x,i)=>{
    return `<div class="result-card anim-fade" onclick="openD(${D.r.indexOf(x)})">
      <div class="rc-src"><span class="sbadge ci">🇨🇮 CI</span><span>📡 ${esc(x.source)}</span><span>${x.date?new Date(x.date).toLocaleTimeString('fr-FR'):''}</span></div>
      <div class="rc-title">${esc(x.title||'')}</div>
      <div class="rc-desc">${esc((x.description||'').substring(0,200))}</div>
      <div class="rc-link"><a href="${x.link||'#'}" target="_blank">🔗 ${(x.link||'').substring(0,60)}</a></div>
    </div>`
  }).join('');
}

function ciF(f,btn){
  CF=f;
  document.querySelectorAll('#p-ci .filter-btn').forEach(b=>b.classList.remove('active'));
  if(btn)btn.classList.add('active');
  renderCI();
}

function renderAf(){
  const el=id('afRes');if(!el)return;
  let r=D.r.filter(x=>x.sourceType==='africa'&&r24(x.date));
  if(!r.length){el.innerHTML='<div class="empty"><div class="ic">🌍</div><h3>Aucun résultat Afrique</h3></div>';return}
  el.innerHTML=r.slice(0,50).map(x=>{
    return `<div class="result-card anim-fade" onclick="openD(${D.r.indexOf(x)})">
      <div class="rc-src"><span class="sbadge rss">🌍 Afrique</span><span>📡 ${esc(x.source)}</span><span>${x.date?new Date(x.date).toLocaleTimeString('fr-FR'):''}</span></div>
      <div class="rc-title">${esc(x.title||'')}</div>
      <div class="rc-desc">${esc((x.description||'').substring(0,200))}</div>
    </div>`
  }).join('');
}

function renderIntl(){
  const el=id('intlRes');if(!el)return;
  let r=D.r.filter(x=>x.sourceType==='world'&&r24(x.date));
  if(!r.length){el.innerHTML='<div class="empty"><div class="ic">🌐</div><h3>Aucun résultat International</h3></div>';return}
  el.innerHTML=r.slice(0,50).map(x=>{
    return `<div class="result-card anim-fade" onclick="openD(${D.r.indexOf(x)})">
      <div class="rc-src"><span class="sbadge rss">🌐 International</span><span>📡 ${esc(x.source)}</span><span>${x.date?new Date(x.date).toLocaleTimeString('fr-FR'):''}</span></div>
      <div class="rc-title">${esc(x.title||'')}</div>
      <div class="rc-desc">${esc((x.description||'').substring(0,200))}</div>
    </div>`
  }).join('');
}

function renderGal(){
  const el=id('gallery');if(!el)return;
  const imgs=D.r.filter(x=>{const m=(x.description||'').match(/<img[^>]+src=["']([^"']+)["']/i);return m&&m[1]&&!m[1].includes('google')}).slice(0,12);
  if(!imgs.length){el.innerHTML='<div style="color:var(--text3);font-size:11px;padding:10px">Aucune image dans les résultats</div>';return}
  el.innerHTML=imgs.map(x=>{
    const src=(x.description||'').match(/<img[^>]+src=["']([^"']+)["']/i);
    const u=src?src[1]:null;
    return u?`<div class="g-card" onclick="openD(${D.r.indexOf(x)})">
      <div class="img"><img src="${esc(u)}" loading="lazy" onerror="this.parentElement.innerHTML='📷'" alt=""></div>
      <div class="body"><div class="t">${esc(x.title||'')}</div><div class="s">📡 ${esc(x.source)}</div></div>
    </div>`:'';
  }).join('');
}

// ===== DETAIL =====
function openD(idx){
  const x=D.r[idx];if(!x)return;
  const p=id('detail');const t=id('dTitle');const b=id('dBody');
  if(!p||!b)return;
  if(t)t.textContent=esc(x.title||'');
  const kws=(x.keywords||[]).map(k=>`<span class="kw">#${esc(k)}</span>`).join('');
  const im=(x.description||'').match(/<img[^>]+src=["']([^"']+)["']/i);
  const imHtml=im?`<img class="img" src="${esc(im[1])}" onerror="this.style.display='none'" alt="">`:'';
  const sevCls=x.severity==='critical'?'color:#ef4444':'color:var(--text3)';
  const isFav=D.p.includes(idx);
  b.innerHTML=`
    ${imHtml}
    <div class="l">Source</div><div class="v">📡 ${esc(x.source)} ${x.proxy?'· '+x.proxy:''}</div>
    <div class="l">Date</div><div class="v">${x.date?new Date(x.date).toLocaleString('fr-FR'):'Inconnue'}</div>
    <div class="l">Sévérité</div><div class="v" style="${sevCls}">${x.severity==='critical'?'🔴 CRITIQUE':'🟢 Normal'}</div>
    ${kws?`<div class="l">Mots-clés</div><div class="v">${kws}</div>`:''}
    <div class="l">Description</div><div class="v">${esc((x.description||'').substring(0,800))}</div>
    <div class="l">Lien</div><div class="v" style="word-break:break-all"><a href="${x.link||'#'}" target="_blank">${esc(x.link||'#')}</a></div>
    <div style="display:flex;gap:6px;margin-top:12px">
      <button class="btn btn-sm ${isFav?'btn-danger':'btn-secondary'}" onclick="togFav(${idx})">${isFav?'⭐ Retirer':'⭐ Favori'}</button>
      <button class="btn btn-sm btn-ghost" onclick="share(${idx})">📤 Partager</button>
    </div>`;
  p.classList.add('open');
}

function closeD(){const p=id('detail');if(p)p.classList.remove('open')}

function togFav(idx){
  const i=D.p.indexOf(idx);
  if(i>-1){D.p.splice(i,1);to('⭐ Retiré','w')}
  else{D.p.push(idx);to('⭐ Ajouté','s')}
  save();openD(idx);renderFav();up()
}

function share(idx){
  const x=D.r[idx];if(!x)return;
  if(navigator.share){navigator.share({title:x.title,text:x.description?.substring(0,200),url:x.link}).catch(()=>{})}
  else{navigator.clipboard.writeText(x.title+'\n'+x.link).then(()=>to('📋 Copié','s')).catch(()=>to('❌ Erreur','e'))}
}

// ===== SOURCES =====
function renderSrc(){
  const el=id('srcList');if(!el)return;
  const c=id('srcCount');if(c)c.textContent=D.s.length;
  if(!D.s.length){el.innerHTML='<div class="empty"><div class="ic">📡</div><h3>Aucune source</h3></div>';return}
  const sc={};D.r.forEach(r=>{sc[r.source]=(sc[r.source]||0)+1});
  el.innerHTML=D.s.map(s=>{
    const n=sc[s.n]||0;
    return `<div class="source-item">
      <span class="dot" style="background:${n>0?'#10b981':'var(--text3)'}"></span>
      <span class="nm">${esc(s.n)}</span>
      <span class="ct">${n>0?n+' résultats':'En attente'}</span>
      <span class="del" onclick="rmSrc('${s.u.replace(/'/g,"\\'")}')">✕</span>
    </div>`
  }).join('');
}

function addSrc(){
  const n=id('srcN').value.trim();const u=id('srcU').value.trim();
  if(!u){to('❌ URL requise','e');return}
  if(!u.startsWith('http')){to('❌ URL invalide','e');return}
  if(D.s.find(x=>x.u===u)){to('⚠️ Existe déjà','w');return}
  const nn=n||u.substring(0,30);
  D.s.push({n:nn,u:u,t:'rss'});
  save();renderSrc();id('srcN').value='';id('srcU').value='';to('✅ Source ajoutée','s')
}

function rmSrc(u){D.s=D.s.filter(s=>s.u!==u);save();renderSrc();to('🗑️ Supprimée','w')}
function rmAllSrc(){if(!confirm('⚠️ Supprimer TOUTES les sources ?'))return;D.s=[];save();renderSrc();to('🗑️ Tout vidé','w')}

function pre(k,btn){
  document.querySelectorAll('.preset-btn').forEach(b=>b.classList.remove('active'));
  if(btn)btn.classList.add('active');
  const el=id('presetList');if(!el)return;
  const items=PRESETS[k]||[];
  if(!items.length){el.innerHTML='<p style="font-size:10px;color:var(--text3)">Aucune</p>';return}
  el.innerHTML=items.map(s=>{
    const ex=D.s.find(x=>x.u===s.u);
    return `<div style="display:flex;justify-content:space-between;align-items:center;padding:4px 0;font-size:11px;border-bottom:1px solid var(--border)">
      <span>${esc(s.n)}</span>
      <button class="btn btn-xs ${ex?'btn-success':'btn-ghost'}" onclick="${ex?'':'addPre(\''+s.n.replace(/'/g,"\\'")+'\',\''+s.u.replace(/'/g,"\\'")+'\')'}">${ex?'✅':'➕'}</button>
    </div>`
  }).join('');
}

function addPre(n,u){
  if(D.s.find(x=>x.u===u)){to('⚠️ Déjà présent','w');return}
  D.s.push({n,u,t:'rss'});save();renderSrc();to('✅ '+n,'s');pre('ci',document.querySelector('.preset-btn'))
}

function loadAll(){
  let c=0;
  Object.values(PRESETS).forEach(cat=>cat.forEach(s=>{if(!D.s.find(x=>x.u===s.u)){D.s.push({n:s.n,u:s.u,t:'rss'});c++}}));
  if(c){save();renderSrc();to('📥 '+c+' chargées','s')}else to('Rien à ajouter','w')
}

// ===== KEYWORDS =====
function renderKW(){
  const el=id('kwList');if(!el)return;
  const c=id('kwCount');if(c)c.textContent=D.k.length;
  if(!D.k.length){el.innerHTML='<span style="font-size:10px;color:var(--text3)">Aucun mot-clé</span>';return}
  el.innerHTML=D.k.map(k=>`<span class="tag">${esc(k)} <span class="rem" onclick="rmKw('${k.replace(/'/g,"\\'")}')">✕</span></span>`).join('');
}

function addKw(){
  const v=id('kwIn').value.trim().toLowerCase();
  if(!v){to('❌ Vide','e');return}
  if(D.k.includes(v)){to('⚠️ Existe','w');return}
  D.k.push(v);save();renderKW();id('kwIn').value='';to('✅ Ajouté','s')
}

function rmKw(k){D.k=D.k.filter(x=>x!==k);save();renderKW()}
function resetKw(){D.k=getDefKws();save();renderKW();to('↩️ '+D.k.length+' mots-clés','s')}
function clrKw(){if(!confirm('Tout effacer ?'))return;D.k=[];save();renderKW();to('🗑️ Vidé','w')}
function expKw(){const b=new Blob([D.k.join('\n')],{type:'text/plain'});const u=URL.createObjectURL(b);const a=document.createElement('a');a.href=u;a.download='kws_crogend.txt';a.click();URL.revokeObjectURL(u);to('📤 Exportés','s')}
function impKw(){const i=document.createElement('input');i.type='file';i.accept='.txt';i.onchange=function(e){const f=e.target.files[0];if(!f)return;const r=new FileReader();r.onload=function(ev){const lines=ev.target.result.split('\n').map(l=>l.trim().toLowerCase()).filter(l=>l);let a=0;lines.forEach(l=>{if(!D.k.includes(l)){D.k.push(l);a++}});if(a){save();renderKW()};to('📥 '+a+' importés','s')};r.readAsText(f)};i.click()}

// ===== TARGETS =====
function renderTG(){
  const el=id('tgGrid');if(!el)return;
  const c=id('tgCount');if(c)c.textContent=D.tg.length;
  if(!D.tg.length){el.innerHTML='<div class="empty" style="grid-column:1/-1"><div class="ic">🎯</div><h3>Aucune cible</h3></div>';return}
  const icons={personne:'👤',organisation:'🏢',site:'🌐'};
  el.innerHTML=D.tg.map((t,i)=>`<div class="target-card" onclick="viewTg(${i})"><div class="ic">${icons[t.type]||'📌'}</div><div class="nm">${esc(t.name)}</div><div style="font-size:9px;color:var(--text3)">${t.type}${t.url?' · '+esc(t.url.substring(0,20)):''}</div></div>`).join('');
}

function addTg(){
  const n=id('tgN').value.trim();const u=id('tgU').value.trim();const tp=id('tgT').value;
  if(!n){to('❌ Nom requis','e');return}
  D.tg.push({name:n,url:u,type:tp,date:new Date().toISOString()});
  save();renderTG();id('tgN').value='';id('tgU').value='';to('🎯 Cible ajoutée','s')
}

function viewTg(i){const t=D.tg[i];if(t)alert('🎯 '+t.name+'\nType: '+t.type+(t.url?'\nURL: '+t.url:'')+'\nAjouté: '+new Date(t.date).toLocaleString('fr-FR'))}

// ===== SOCIAL =====
function renderSocial(){['fb','tw','tg','yt'].forEach(k=>{const el=id(k+'L');if(!el)return;const items=D[k]||[];if(!items.length){el.innerHTML='<div class="empty" style="padding:8px"><p style="font-size:10px">Aucun</p></div>';return}const icons={fb:'📘',tw:'🐦',tg:'✈️',yt:'▶️'};el.innerHTML=items.map(i=>`<div class="social-card"><div class="ic">${icons[k]||'🔗'}</div><div><div class="nm">${esc(i.name)}</div><div class="ur">${i.url.startsWith('http')?`<a href="${i.url}" target="_blank">${esc(i.url)}</a>`:esc(i.url)}</div></div></div>`).join('')})}
function addFB(){const u=id('fbU').value.trim();const n=id('fbN').value.trim()||'Facebook';if(!u)return;D.fb.push({url:u,name:n});save();id('fbU').value='';id('fbN').value='';renderSocial();to('📘 Ajouté','s')}
function addTW(){const u=id('twU').value.trim();const n=id('twN').value.trim()||'Twitter';if(!u)return;D.tw.push({url:u,name:n});save();id('twU').value='';id('twN').value='';renderSocial();to('🐦 Ajouté','s')}
function addTG(){const u=id('tgU').value.trim();const n=id('tgN').value.trim()||'Telegram';if(!u)return;D.tg.push({url:u,name:n});save();id('tgU').value='';id('tgN').value='';renderSocial();to('✈️ Ajouté','s')}
function addYT(){const u=id('ytU').value.trim();const n=id('ytN').value.trim()||'YouTube';if(!u)return;D.yt.push({url:u,name:n});save();id('ytU').value='';id('ytN').value='';renderSocial();to('▶️ Ajouté','s')}

// ===== ALERTS =====
function renderAl(){
  const el=id('alList');if(!el)return;
  const a=D.a||[];
  if(!a.length){el.innerHTML='<div class="empty"><div class="ic">🔔</div><h3>Aucune alerte</h3></div>';return}
  const c=id('bdAlert');const crit=a.filter(x=>x.s==='c').length;if(c){c.textContent=crit;c.className='badge danger'+(crit?'':'')}
  el.innerHTML=a.slice(0,50).map(x=>`<div class="result-card"><div class="rc-src"><span class="sbadge ${x.s==='c'?'critical':'rss'}">${x.s==='c'?'🔴 CRIT':'🟠 ALERTE'}</span><span>${x.d?new Date(x.d).toLocaleString('fr-FR'):''}</span></div><div class="rc-title">${esc(x.m)}</div>${x.t?`<div class="rc-desc">${esc(x.t)}</div>`:''}</div>`).join('');
}

function addA(m,s,t){if(!D.a)D.a=[];D.a.unshift({m,s:t||'',t,'d':new Date().toISOString()});if(D.a.length>100)D.a=D.a.slice(0,100);save();up()}
function clrAl(){if(!confirm('Effacer les alertes ?'))return;D.a=[];save();renderAl();to('🗑️ Vidé','w')}

// ===== FAVORIS =====
function renderFav(){
  const el=id('favList');if(!el)return;
  const f=D.p.map(i=>D.r[i]).filter(Boolean);
  if(!f.length){el.innerHTML='<div class="empty"><div class="ic">⭐</div><h3>Aucun favori</h3></div>';return}
  el.innerHTML=f.map(x=>`<div class="result-card" onclick="openD(${D.r.indexOf(x)})"><div class="rc-src"><span class="sbadge rss">⭐ Favori</span><span>📡 ${esc(x.source)}</span></div><div class="rc-title">${esc(x.title)}</div></div>`).join('');
}
function clrFav(){if(!confirm('Effacer favoris ?'))return;D.p=[];save();renderFav();to('🗑️ Vidé','w')}

// ===== RECHERCHE =====
async function searchW(){
  const q=id('sq').value.trim();const l=id('sl').value;const c=id('sc').value.trim().toUpperCase()||'CI';
  if(!q){to('❌ Recherche vide','e');return}
  showL('🔍 Recherche...',null);
  const u='https://news.google.com/rss/search?q='+encodeURIComponent(q)+'&hl='+l+'&gl='+c+'&ceid='+c+':'+l;
  try{
    const r=await fetchRSS(u);
    const el=id('sRes');const ct=id('sCount');
    if(r&&r.items&&r.items.length){
      if(ct)ct.textContent=r.items.length+' résultats ('+r.proxy+')';
      el.innerHTML=r.items.slice(0,30).map(item=>`<div class="result-card"><div class="rc-src"><span class="sbadge rss">📰 Google</span></div><div class="rc-title"><a href="${item.link||'#'}" target="_blank">${esc(item.title||'')}</a></div><div class="rc-desc">${esc((item.description||'').substring(0,250))}</div></div>`).join('')
    }else{el.innerHTML='<div class="empty"><div class="ic">🔍</div><h3>Aucun résultat</h3></div>';if(ct)ct.textContent='0'}
  }catch(e){id('sRes').innerHTML='<div class="empty"><div class="ic">❌</div><h3>Erreur réseau</h3></div>'}
  hideL()
}

// ===== TRACKING =====
function renderTrack(){
  const kwC={};D.r.forEach(r=>{if(r.keywords)r.keywords.forEach(k=>{kwC[k]=(kwC[k]||0)+1})});
  const topKW=Object.entries(kwC).sort((a,b)=>b[1]-a[1]).slice(0,20);
  const t1=id('topKW');if(t1)t1.innerHTML=topKW.length?topKW.map(([k,c])=>`<div class="chart-bar"><span class="l">#${esc(k)}</span><div class="tr"><div class="f" style="width:${(c/Math.max(...topKW.map(x=>x[1])))*100}%;background:var(--orange)"></div></div><span class="va">${c}</span></div>`).join(''):'<div style="color:var(--text3);font-size:10px;padding:10px">Aucune donnée</div>';
  
  const srcC={};D.r.forEach(r=>{srcC[r.source]=(srcC[r.source]||0)+1});
  const topSRC=Object.entries(srcC).sort((a,b)=>b[1]-a[1]).slice(0,20);
  const t2=id('topSRC');if(t2)t2.innerHTML=topSRC.length?topSRC.map(([s,c])=>`<div class="chart-bar"><span class="l">📡 ${esc(s)}</span><div class="tr"><div class="f" style="width:${(c/Math.max(...topSRC.map(x=>x[1])))*100}%;background:#10b981"></div></div><span class="va">${c}</span></div>`).join(''):'<div style="color:var(--text3);font-size:10px;padding:10px">Aucune donnée</div>';
  
  // Activité 14 jours
  const days={};D.r.forEach(r=>{if(r.date){try{const d=new Date(r.date).toLocaleDateString('fr-FR');days[d]=(days[d]||0)+1}catch(e){}}});
  const sDays=Object.entries(days).sort((a,b)=>new Date(a[0])-new Date(b[0])).slice(-14);
  const maxC=Math.max(...sDays.map(([_,c])=>c),1);
  const act=id('actChart');
  if(act)act.innerHTML=sDays.length?sDays.map(([d,c])=>`<div class="chart-bar"><span class="l" style="width:80px">${d}</span><div class="tr"><div class="f" style="width:${(c/maxC)*100}%;background:var(--grad)"></div></div><span class="va">${c}</span></div>`).join(''):'<div style="color:var(--text3);font-size:10px;padding:10px">Pas d\'activité</div>'
}

// ===== RAPPORT WORD =====
function genReport(){
  showL('📄 Rapport...',10);
  const now=new Date();
  const r=r24a(D.r);
  const ci=r.filter(x=>x.sourceType==='ci');
  const af=r.filter(x=>x.sourceType==='africa');
  const wd=r.filter(x=>x.sourceType==='world');
  const crit=r.filter(x=>x.severity==='critical');
  
  setTimeout(()=>{showL('📊 Analyse...',40)},300);
  setTimeout(()=>{
    showL('📝 Assemblage...',70);
    let html=`<html xmlns:o='urn:schemas-microsoft-com:office:office' xmlns:w='urn:schemas-microsoft-com:office:word' xmlns='http://www.w3.org/TR/REC-html40'>
    <head><meta charset="UTF-8"><style>
      body{font-family:Calibri,Arial,sans-serif;font-size:11pt;color:#1a1a1a;line-height:1.5;margin:40px}
      h1{color:#f97316;font-size:22pt;border-bottom:3px solid #f97316;padding-bottom:8px}
      h2{color:#1e3a5f;font-size:16pt;margin-top:24px;border-bottom:1px solid #ccc;padding-bottom:4px}
      .header{text-align:center;margin-bottom:30px}
      .header h1{font-size:26pt;border:none;color:#f97316}
      .meta{background:#f5f5f5;padding:12px;border-radius:6px;margin:16px 0;font-size:10pt}
      .item{border-left:3px solid #f97316;padding:8px 12px;margin:8px 0;background:#fafafa}
      .item .t{font-weight:600;font-size:11pt}
      .item .s{color:#666;font-size:9pt}
      .critical{border-left-color:#ef4444;background:#fef2f2}
      table{width:100%;border-collapse:collapse;margin:12px 0}
      td,th{border:1px solid #ddd;padding:6px 10px;font-size:10pt}
      th{background:#f97316;color:#fff}
      .footer{text-align:center;color:#999;font-size:9pt;margin-top:40px;border-top:1px solid #ddd;padding-top:12px}
    </style></head><body>
    <div class="header">
      <h1>🕵️ CROGEND — RAPPORT DE VEILLE 24H</h1>
      <div style="color:#666">Cellule de Renseignement Opérationnel</div>
      <p style="color:#666">${now.toLocaleDateString('fr-FR',{weekday:'long',year:'numeric',month:'long',day:'numeric'})} à ${now.toLocaleTimeString('fr-FR')}</p>
    </div>
    <div class="meta"><strong>📊 Synthèse :</strong> ${r.length} résultats · ${crit.length} alertes · ${D.s.length} sources · ${S.cycles} cycles</div>
    <h2>📊 Statistiques</h2>
    <table><tr><th>Métrique</th><th>Valeur</th></tr>
      <tr><td>🇨🇮 Côte d'Ivoire</td><td>${ci.length}</td></tr>
      <tr><td>🌍 Afrique</td><td>${af.length}</td></tr>
      <tr><td>🌐 International</td><td>${wd.length}</td></tr>
      <tr><td>🚨 Critiques</td><td>${crit.length}</td></tr>
      <tr><td>📡 Sources</td><td>${D.s.length}</td></tr>
      <tr><td>🔄 Cycles</td><td>${S.cycles}</td></tr>
    </table>
    <h2>🚨 ALERTES CRITIQUES</h2>
    ${crit.length?crit.slice(0,10).map(x=>'<div class="item critical"><div class="t">'+esc(x.title)+'</div><div class="s">📡 '+esc(x.source)+' · '+(x.date?new Date(x.date).toLocaleString('fr-FR'):'')+'</div></div>').join(''):'<p>Aucune</p>'}
    <h2>🇨🇮 CÔTE D'IVOIRE</h2>
    ${ci.length?ci.slice(0,30).map(x=>'<div class="item"><div class="t">'+esc(x.title)+'</div><div class="s">📡 '+esc(x.source)+' · '+(x.date?new Date(x.date).toLocaleString('fr-FR'):'')+'</div>'+(x.link&&x.link!=='#'?'<div style="color:#3b82f6;font-size:9pt">🔗 '+x.link+'</div>':'')+'</div>').join(''):'<p>Aucune</p>'}
    <h2>🌍 AFRIQUE</h2>
    ${af.length?af.slice(0,20).map(x=>'<div class="item"><div class="t">'+esc(x.title)+'</div><div class="s">📡 '+esc(x.source)+'</div></div>').join(''):'<p>Aucune</p>'}
    <h2>🌐 INTERNATIONAL</h2>
    ${wd.length?wd.slice(0,20).map(x=>'<div class="item"><div class="t">'+esc(x.title)+'</div><div class="s">📡 '+esc(x.source)+'</div></div>').join(''):'<p>Aucune</p>'}
    <div class="footer"><p>CROGEND OSINT v10 — ${now.toLocaleString('fr-FR')}</p><p><em>⚠️ Document confidentiel — usage interne</em></p></div>
    </body></html>`;
    
    setTimeout(()=>{
      const blob=new Blob([html],{type:'application/msword'});
      const url=URL.createObjectURL(blob);
      const a=document.createElement('a');
      a.href=url;
      a.download='RAPPORT_CROGEND_'+now.toISOString().slice(0,10)+'.doc';
      a.click();URL.revokeObjectURL(url);
      hideL();
      to('📄 Rapport généré','s');
      log('📄 Rapport Word — '+r.length+' résultats');
    },500);
  },600);
}

function renderHist(){
  const el=id('repHist');if(!el)return;
  const reps=D.v||[];
  if(!reps.length){el.innerHTML='<p style="font-size:10px;color:var(--text3)">Aucun rapport</p>';return}
  el.innerHTML=reps.slice(0,10).map(r=>`<div style="display:flex;justify-content:space-between;padding:6px 0;border-bottom:1px solid var(--border);font-size:10px"><span>📄 ${new Date(r.date).toLocaleString('fr-FR')}</span><span>${r.count} résultats</span></div>`).join('')
}

// ===== EXPORT/IMPORT =====
function expCfg(){
  const cfg={sources:D.s,keywords:D.k,fb:D.fb,tw:D.tw,tg:D.tg,yt:D.yt,favorites:D.p,targets:D.tg,date:new Date().toISOString()};
  const b=new Blob([JSON.stringify(cfg,null,2)],{type:'application/json'});
  const u=URL.createObjectURL(b);const a=document.createElement('a');a.href=u;a.download='config_crogend.json';a.click();URL.revokeObjectURL(u);to('💾 Exportée','s')
}

function impCfg(){
  const i=document.createElement('input');i.type='file';i.accept='.json';
  i.onchange=function(e){
    const f=e.target.files[0];if(!f)return;
    const r=new FileReader();r.onload=function(ev){try{const c=JSON.parse(ev.target.result);if(c.sources)D.s=c.sources;if(c.keywords)D.k=c.keywords;if(c.fb)D.fb=c.fb;if(c.tw)D.tw=c.tw;if(c.tg)D.tg=c.tg;if(c.yt)D.yt=c.yt;if(c.favorites)D.p=c.favorites;if(c.targets)D.tg=c.tg;save();up();renderTG();renderFav();to('📥 Importée','s')}catch(e){to('❌ Fichier invalide','e')}};r.readAsText(f)
  };i.click()
}

// ===== NOTES =====
function svNotes(){const ta=id('ntArea');if(ta&&ta.value)localStorage.setItem('cg_notes',ta.value);to('💾 Enregistré','s')}
function expNotes(){const ta=id('ntArea');if(!ta||!ta.value.trim()){to('❌ Notes vides','e');return}const b=new Blob([ta.value],{type:'text/plain'});const u=URL.createObjectURL(b);const a=document.createElement('a');a.href=u;a.download='notes_crogend_'+new Date().toISOString().slice(0,10)+'.txt';a.click();URL.revokeObjectURL(u);to('📤 Exportées','s')}

// ===== SETTINGS =====
function sv(k,v){
  const map={intv:'cg_intv',cache:'cg_cache'};
  localStorage.setItem(map[k]||k,v);
  if(k==='intv'&&S.scan){clearInterval(S.timer);const iv=parseFloat(v)*60000;S.timer=setInterval(()=>{if(!S.busy&&S.scan)scan()},iv)}
  to('✅ Sauvegardé','s')
}

function nuke(){
  if(!confirm('⚠️⚠️ TOUT EFFACER ?'))return;
  if(!confirm('⚠️⚠️ CONFIRMATION FINALE ?'))return;
  Object.values(SK).forEach(k=>localStorage.removeItem(k));
  ['cg_notes','cg_intv','cg_cache'].forEach(k=>localStorage.removeItem(k));
  Object.keys(D).forEach(k=>{if(Array.isArray(D[k]))D[k]=[]});
  D.k=getDefKws();
  D.s=Object.values(PRESETS).flat().slice(0,10).map(s=>({n:s.n,u:s.u,t:'rss'}));
  if(S.timer){clearInterval(S.timer);S.timer=null}
  S.scan=false;S.cycles=0;S.total=0;S.busy=false;S.last=null;S.lastN=0;S.dur=0;S.ok=0;
  save();up();renderTG();renderFav();log('🗑️ Tout effacé');to('🗑️ Réinitialisé','w')
}

// ===== INIT =====
document.addEventListener('DOMContentLoaded',function(){
  console.log('🕵️ CROGEND v10 PRO');
  load();
  if(!D.s.length){D.s=Object.values(PRESETS).flat().slice(0,15).map(s=>({n:s.n,u:s.u,t:'rss'}));save()}
  const nv=localStorage.getItem('cg_notes');const ta=id('ntArea');if(ta&&nv)ta.value=nv;
  const iv=localStorage.getItem('cg_intv');if(iv){const e=id('scanIntv');if(e)e.value=iv;const e2=id('setIntv');if(e2)e2.value=iv}
  const ca=localStorage.getItem('cg_cache');if(ca){const e=id('scanCache');if(e)e.value=ca;const e2=id('setCache');if(e2)e2.value=ca}
  renderSocial();renderTG();renderFav();up();
  log('🕵️ CROGEND v10 PRO initialisé');
  log('📡 '+D.s.length+' sources · '+D.k.length+' mots-clés');
  log('🇨🇮 Focus Côte d\'Ivoire');
  
  setInterval(()=>{const ta=id('ntArea');if(ta&&ta.value)localStorage.setItem('cg_notes',ta.value)},30000);
  
  if(D.r.length<50&&D.s.length){setTimeout(()=>{if(!S.busy)scan()},3000)}
  
  setTimeout(()=>to('🕵️ CROGEND prêt · '+r24a(D.r).length+' résultats','i',4000),800);
});

document.addEventListener('click',function(e){
  const p=id('detail');if(p&&p.classList.contains('open')&&!p.contains(e.target)&&!e.target.closest('.result-card')&&!e.target.closest('.g-card'))closeD()
});

document.addEventListener('keydown',function(e){
  if(e.key==='Escape'){closeD()}
  if(e.ctrlKey&&e.key==='s'){e.preventDefault();svNotes()}
});

console.log('🕵️ CROGEND v10 — Prêt');
</script>
</body>
</html>
