<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Worldwide Tariff Tracker — MVP</title>
  <meta name="description" content="Track headline tariff rates by country/product over time. Single-file MVP with filters, timeline, change-highlighting, and ad slots." />
  <style>
    :root{--bg:#0b0f17;--card:#121826;--muted:#9aa4b2;--text:#e6edf7;--accent:#7dd3fc;--accent2:#a78bfa}
    *{box-sizing:border-box}
    html,body{height:100%}
    body{margin:0;font-family:ui-sans-serif,system-ui,-apple-system,Segoe UI,Roboto;background:radial-gradient(1200px 900px at 20% -10%,#192338 0%,#0b0f17 60%);color:var(--text)}
    a{color:var(--accent)}
    .wrap{max-width:1150px;margin:0 auto;padding:18px}
    .grid{display:grid;gap:16px}
    .two{grid-template-columns:1fr}
    @media(min-width:1024px){.two{grid-template-columns:320px 1fr}}
    .card{background:linear-gradient(180deg,#121826,#0f1626);border:1px solid #1f2937;border-radius:16px;padding:16px;box-shadow:0 10px 40px rgba(0,0,0,.25)}
    .title{font-size:clamp(22px,3.6vw,34px);margin:0 0 6px}
    .muted{color:var(--muted)}
    .row{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
    .pill{display:inline-block;padding:6px 10px;border-radius:999px;background:#0f172a;border:1px solid #243042;color:#b6c2d0;font-size:12px}
    input,select{width:100%;padding:10px;border-radius:10px;border:1px solid #243042;background:#0a1220;color:#cfe3ff}
    table{width:100%;border-collapse:separate;border-spacing:0}
    th,td{padding:10px 12px;border-bottom:1px solid #243042}
    th{position:sticky;top:0;background:#0f1626;z-index:1}
    .tag{font-size:12px;padding:4px 8px;border-radius:999px;border:1px solid #28405d;background:#0b1322;color:#9cc4ff}
    .up{color:#92f6a0}
    .down{color:#f6a092}
    .btn{display:inline-flex;gap:8px;align-items:center;padding:10px 12px;border-radius:10px;border:1px solid #334155;background:#0f172a;color:#e6edf7;cursor:pointer}
    .btn:hover{border-color:#465872}
    .banner{height:90px;display:flex;align-items:center;justify-content:center;border:1px dashed #334155;border-radius:12px;background:#0b1220;color:#7aa2d2}
    .small{height:250px}
    .footer{margin:20px 0;text-align:center;color:#8aa2c2;font-size:13px}
    .legend{display:flex;gap:10px;align-items:center}
  </style>
</head>
<body>
  <div class="wrap">
    <div class="banner card" id="topAd">🔧 Top Banner Ad Slot — paste AdSense/Adsterra/Propeller script here</div>

    <header class="card" style="margin-top:16px">
      <div class="row" style="justify-content:space-between;align-items:flex-start">
        <div>
          <span class="pill">MVP • Client-side only</span>
          <h1 class="title">Worldwide Tariff Tracker</h1>
          <div class="muted">Filter by country / product, slide through time, and see how headline tariff rates changed. (Demo data below — swap with your CSV/JSON.)</div>
        </div>
        <div class="legend">
          <span class="tag">Rate = ad‑valorem %</span>
          <span class="tag">Change: <span class="up">▲ up</span> / <span class="down">▼ down</span></span>
        </div>
      </div>
    </header>

    <main class="grid two" style="margin-top:16px">
      <aside class="card">
        <h3 style="margin-top:0">Filters</h3>
        <label>Country</label>
        <select id="countrySel"></select>
        <label style="margin-top:10px;display:block">Product category</label>
        <select id="catSel"></select>
        <label style="margin-top:10px;display:block">Effective date</label>
        <input id="dateRange" type="range" min="0" max="0" step="1" />
        <div class="row" style="justify-content:space-between">
          <div class="muted">Selected:</div>
          <div id="dateOut" class="pill">—</div>
        </div>
        <div class="row" style="margin-top:10px">
          <button class="btn" onclick="resetFilters()">Reset</button>
          <button class="btn" onclick="exportCSV()">Export CSV</button>
        </div>
        <div class="banner small" id="sideAd" style="margin-top:12px">🔧 Sidebar Ad Slot</div>
      </aside>

      <section class="card">
        <div class="row" style="justify-content:space-between;align-items:center">
          <h3 style="margin:6px 0">Results</h3>
          <div class="muted">Rows: <span id="rowCount">0</span></div>
        </div>
        <div style="max-height:520px;overflow:auto;border:1px solid #223049;border-radius:12px">
          <table>
            <thead>
              <tr>
                <th style="text-align:left">Country</th>
                <th style="text-align:left">Product</th>
                <th style="text-align:right">Rate %</th>
                <th style="text-align:left">Basis</th>
                <th style="text-align:left">Policy</th>
                <th style="text-align:left">Effective</th>
                <th style="text-align:left">Prev → Curr</th>
              </tr>
            </thead>
            <tbody id="tbody"></tbody>
          </table>
        </div>
      </section>
    </main>

    <div class="footer">© <span id="yr"></span> Tariff Tracker (MVP). Replace demo data with your live feed. Ads: paste your scripts into the banner divs.</div>
  </div>

  <script>
    // ===== Demo dataset (replace with live JSON fetched from your backend/Sheet) =====
    // Each record = a snapshot for a date; tracker computes prev→curr change for display
    const demoData = [
      { country:"China", product:"Consumer electronics", rate:10, basis:"Ad valorem", policy:"Section 301 (example)", effective:"2025-08-01" },
      { country:"China", product:"Consumer electronics", rate:15, basis:"Ad valorem", policy:"Section 301 (revised)", effective:"2025-08-20" },
      { country:"Mexico", product:"Steel & aluminum", rate:5, basis:"Ad valorem", policy:"National security (example)", effective:"2025-07-15" },
      { country:"Mexico", product:"Steel & aluminum", rate:8, basis:"Ad valorem", policy:"Updated measure", effective:"2025-08-10" },
      { country:"EU", product:"EVs & batteries", rate:12.5, basis:"Ad valorem", policy:"Countervailing (example)", effective:"2025-06-12" },
      { country:"EU", product:"EVs & batteries", rate:12.5, basis:"Ad valorem", policy:"No change", effective:"2025-08-10" },
      { country:"India", product:"Apparel & textiles", rate:7, basis:"Ad valorem", policy:"MFN baseline (example)", effective:"2025-07-01" },
      { country:"India", product:"Apparel & textiles", rate:6, basis:"Ad valorem", policy:"Adjustment", effective:"2025-08-05" }
    ];

    const state = { data: demoData, filtered: [], dates: [], countries: [], cats: [], idx: 0 };

    function unique(arr, key){ return [...new Set(arr.map(x=>x[key]))]; }

    function init(){
      document.getElementById('yr').textContent = new Date().getFullYear();
      // Build distinct lists
      state.countries = unique(state.data, 'country').sort();
      state.cats = unique(state.data, 'product').sort();
      state.dates = [...new Set(state.data.map(x=>x.effective))].sort();
      // Populate filters
      fillSelect('countrySel', ['All', ...state.countries]);
      fillSelect('catSel', ['All', ...state.cats]);
      const r = document.getElementById('dateRange');
      r.min = 0; r.max = state.dates.length-1; r.value = state.dates.length-1;
      document.getElementById('dateOut').textContent = state.dates[state.dates.length-1];
      // First render
      applyFilters();
      // Range listener
      r.addEventListener('input', (e)=>{ state.idx = parseInt(e.target.value,10); document.getElementById('dateOut').textContent = state.dates[state.idx]; applyFilters(); });
      document.getElementById('countrySel').addEventListener('change', applyFilters);
      document.getElementById('catSel').addEventListener('change', applyFilters);
    }

    function fillSelect(id, items){
      const el = document.getElementById(id); el.innerHTML = '';
      items.forEach(v=>{ const o=document.createElement('option'); o.value=v; o.textContent=v; el.appendChild(o); });
    }

    function applyFilters(){
      const cutoff = state.dates[state.idx];
      const c = document.getElementById('countrySel').value;
      const cat = document.getElementById('catSel').value;
      // Keep only records with date <= cutoff
      const upto = state.data.filter(x=>x.effective<=cutoff);
      // Reduce to latest snapshot per (country, product)
      const map = new Map();
      upto.sort((a,b)=>a.effective.localeCompare(b.effective));
      upto.forEach(rec=>{ const key = rec.country+'|'+rec.product; map.set(key, rec); });
      let arr = [...map.values()];
      // Optional filters
      if(c && c!=='All') arr = arr.filter(x=>x.country===c);
      if(cat && cat!=='All') arr = arr.filter(x=>x.product===cat);
      // Compute prev value for change arrow
      arr.forEach(rec=>{ rec.prev = prevRate(rec.country, rec.product, rec.effective); });
      state.filtered = arr;
      renderTable();
    }

    function prevRate(country, product, effective){
      const before = state.data.filter(x=>x.country===country && x.product===product && x.effective<effective).sort((a,b)=>b.effective.localeCompare(a.effective));
      return before[0]?.rate ?? null;
    }

    function renderTable(){
      const tb = document.getElementById('tbody'); tb.innerHTML='';
      state.filtered.forEach(rec=>{
        const tr = document.createElement('tr');
        const chg = (rec.prev==null) ? '' : (rec.rate>rec.prev?`<span class="up">▲</span> ${rec.prev}→${rec.rate}`:(rec.rate<rec.prev?`<span class="down">▼</span> ${rec.prev}→${rec.rate}`:`${rec.prev}→${rec.rate}`));
        tr.innerHTML = `
          <td>${rec.country}</td>
          <td>${rec.product}</td>
          <td style="text-align:right">${rec.rate.toFixed(2)}</td>
          <td>${rec.basis}</td>
          <td>${rec.policy}</td>
          <td>${rec.effective}</td>
          <td>${chg}</td>`;
        tb.appendChild(tr);
      });
      document.getElementById('rowCount').textContent = state.filtered.length;
    }

    function resetFilters(){
      document.getElementById('countrySel').value='All';
      document.getElementById('catSel').value='All';
      document.getElementById('dateRange').value=state.dates.length-1;
      document.getElementById('dateOut').textContent=state.dates[state.dates.length-1];
      state.idx=state.dates.length-1; applyFilters();
    }

    function exportCSV(){
      const header = ['country','product','rate','basis','policy','effective'];
      const rows = state.filtered.map(r=>[r.country,r.product,r.rate,r.basis,r.policy,r.effective]);
      const csv = [header.join(','),...rows.map(r=>r.map(v=>`"${String(v).replaceAll('"','""')}"`).join(','))].join('\n');
      const blob = new Blob([csv],{type:'text/csv'});
      const a = document.createElement('a'); a.href=URL.createObjectURL(blob); a.download='tariffs_export.csv'; a.click();
    }

    // ===== Replace these banners with real ad scripts =====
    // document.getElementById('topAd').innerHTML = "<script>/* your ad code */<\\/script>";

    init();
  </script>
</body>
</html>
