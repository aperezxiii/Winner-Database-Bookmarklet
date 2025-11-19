(() => {
  // ============================
  // EOY HOF — Console UI Export
  // Endpoint: /hofapi/api/MemberProfile/membersearch  (POST)
  // SORT: RegionName A→Z → RegionAwardYear DESC → CompanyName A→Z
  // ============================

  // ---- Tunables ----
  const YEAR_START = 1986;  // earliest year to include when "All years" is checked
  const CONCURRENCY = 6;    // number of parallel requests
  const TIMEOUT_MS  = 20000;
  const RETRIES     = 2;

  // ---- Endpoint + helpers ----
  const endpoint = "/hofapi/api/MemberProfile/membersearch";
  const headers = {
    "accept": "application/json, text/plain, */*",
    "content-type": "application/json;charset=UTF-8"
  };
  const esc = (s="") => `"${String(s).replace(/"/g,'""')}"`;
  const withTimeout = (p, ms) => {
    if (!ms) return p;
    let t; const to = new Promise((_,rej)=>t=setTimeout(()=>rej(new Error("timeout")), ms));
    return Promise.race([p.finally(()=>clearTimeout(t)), to]);
  };
  async function fetchMemberSearch(regionId, year, signal){
    const body = { companyName:"", lastName:"", regionId:String(regionId), awardYear:String(year) };
    const res = await withTimeout(fetch(endpoint, {
      method: "POST", headers, body: JSON.stringify(body), credentials: "include", signal
    }), TIMEOUT_MS);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const j = await res.json();
    return Array.isArray(j) ? j : (Array.isArray(j?.data) ? j.data : []);
  }
  async function fetchWithRetry(fn, args, retries=RETRIES){
    let a=0;
    while(true){
      try { return await fn(...args); }
      catch(e){ if(++a>retries) throw e; await new Promise(r=>setTimeout(r, 250*a)); }
    }
  }

  // ---- Region lists ----
  const US_REGIONS = [
    {id:16,name:"Bay Area"},{id:117,name:"East Central"},{id:20,name:"Florida"},
    {id:12,name:"Greater Los Angeles"},{id:3,name:"Greater Philadelphia"},
    {id:122,name:"Gulf South"},{id:118,name:"Heartland"},
    {id:85,name:"Michigan and Northwest Ohio"},{id:29,name:"Mid-Atlantic"},
    {id:14,name:"Midwest"},{id:6,name:"Mountain West"},{id:11,name:"New England"},
    {id:25,name:"New Jersey"},{id:13,name:"New York"},{id:120,name:"Pacific Southwest"},
    {id:2,name:"Southeast"},{id:123,name:"Southwest"},
    {id:24,name:"Central Plains"},{id:17,name:"Central South"}
  ];
  const US_ID_SET = new Set(US_REGIONS.map(r=>String(r.id)));
  const WORLD_REGIONS = [
    {id:30,name:"Canada"},
    {id:31,name:"World-Australia"},{id:32,name:"World-Belgium"},{id:33,name:"World-Brazil"},
    {id:34,name:"World-Canada"},{id:36,name:"World-Czech Republic"},{id:37,name:"World-Denmark"},
    {id:38,name:"World-Finland"},{id:39,name:"World-France"},{id:40,name:"World-Germany"},
    {id:41,name:"World-Hungary"},{id:42,name:"World-India"},{id:43,name:"World-Indonesia"},
    {id:44,name:"World-Ireland"},{id:45,name:"World-Israel"},{id:46,name:"World-Italy"},
    {id:47,name:"World-Japan"},{id:48,name:"World-Luxembourg"},{id:49,name:"World-Malaysia"},
    {id:50,name:"World-Netherlands"},{id:51,name:"World-New Zealand"},{id:52,name:"World-Norway"},
    {id:53,name:"World-Philippines"},{id:54,name:"World-Poland"},{id:55,name:"World-Russia"},
    {id:56,name:"World-Singapore"},{id:57,name:"World-South Africa"},{id:58,name:"World-Spain"},
    {id:59,name:"World-Sweden"},{id:60,name:"World-Switzerland"},{id:61,name:"World-Taiwan"},
    {id:62,name:"World-Turkey"},{id:63,name:"World-United Kingdom"},
    {id:65,name:"World-Canada Pacific"},{id:66,name:"World-Canada Prairies"},
    {id:67,name:"World-Canada Québec"},{id:68,name:"World-Canada Atlantic"},
    {id:69,name:"World-Canada Ontario"},{id:70,name:"World-Austria"},
    {id:71,name:"World-China - Mainland"},{id:72,name:"World-China - Hong Kong/Macau"},
    {id:73,name:"World-Greece"},{id:74,name:"World-Portugal"},{id:75,name:"World-Slovak Republic"},
    {id:76,name:"World-Ukraine"},{id:77,name:"World-Chile"},{id:78,name:"World-Kazakhstan"},
    {id:79,name:"World-Korea"},{id:80,name:"World-Mozambique"},{id:81,name:"World-Middle East"},
    {id:82,name:"World-Colombia"},{id:83,name:"World-Estonia"},{id:84,name:"World-Liechtenstein"},
    {id:86,name:"World-Argentina"},{id:87,name:"World-Jordan"},{id:88,name:"Mexico"},
    {id:91,name:"World-Kenya"},{id:92,name:"World-Nigeria"},
    {id:96,name:"World-Serbia"},{id:98,name:"World-Uganda"},{id:99,name:"World-Uruguay"},
    {id:102,name:"World-Belarus"},{id:104,name:"World-Croatia"},{id:107,name:"World-Peru"},
    {id:108,name:"World-Romania"},{id:111,name:"World-South Korea"},
    {id:113,name:"World-Trinidad and Tobago/ Caribbean"},
    {id:114,name:"World-Malta"},{id:115,name:"World-Mexico"},
    {id:119,name:"World-Canada Quebec"}
  ];

  // ---- CSV fields ----
  const FIELD_MAP = [
    ["firstName","FirstName"],["lastName","LastName"],
    ["companyName","CompanyName"],["companyUrl","CompanyURL"],
    ["companyDescription","CompanyDescription"],
    ["regionName","RegionName"],["regionAwardYear","RegionAwardYear"],
    ["regionAwardName","RegionAwardName"],
    ["nationalAwardName","NationalAwardName"],["nationalAwardYear","NationalAwardYear"],
    ["worldAwardName","WorldAwardName"],["worldAwardYear","WorldAwardYear"]
  ];
  const toCSV = rows => {
    const head = FIELD_MAP.map(([,L])=>L).join(",");
    const body = rows.map(r => FIELD_MAP.map(([k]) => esc(r[k] ?? "")).join(",")).join("\n");
    return head + "\n" + body;
  };

  // ---- Modal (Shadow DOM) ----
  const rootId = "eoyhof_modal_root";
  const existing = document.getElementById(rootId); if (existing) existing.remove();
  const host = document.createElement("div"); host.id = rootId; document.body.appendChild(host);
  const sh = host.attachShadow({ mode: "open" });

  const css = document.createElement("style");
  css.textContent = `
  .b{position:fixed;inset:0;background:rgba(0,0,0,.35);display:flex;align-items:center;justify-content:center;z-index:2147483647}
  .c{background:#fff;min-width:720px;max-width:95vw;border-radius:12px;box-shadow:0 10px 30px rgba(0,0,0,.2);font:14px/1.45 system-ui,Segoe UI,Roboto,Arial,sans-serif}
  .h{padding:14px 16px;border-bottom:1px solid #eee;font-weight:700}
  .bd{padding:16px}
  .grid2{display:grid;grid-template-columns:1fr 1fr;gap:14px}
  .lbl{font-size:12px;color:#444;margin-bottom:6px;display:block}
  .list{height:260px;overflow:auto;border:1px solid #ddd;border-radius:8px;padding:8px;background:#fafafa}
  .item{display:flex;align-items:center;gap:8px;margin:4px 0}
  .s{width:100%;padding:8px;border:1px solid #ccc;border-radius:8px;margin:6px 0}
  input[type=text]{width:100%;padding:8px;border:1px solid #ccc;border-radius:8px}
  .stack{display:flex;gap:12px;align-items:center;flex-wrap:wrap}
  .acts{display:flex;gap:10px;justify-content:space-between;padding:12px 16px;border-top:1px solid #eee}
  .btn{padding:8px 12px;border-radius:8px;border:1px solid #ccc;background:#f7f7f7;cursor:pointer}
  .btn.p{background:#0b5cff;color:#fff;border-color:#0b5cff}
  .btn.warn{background:#fff5f0;border-color:#ffb199;color:#c04a00}
  .btn.ghost{background:#f2f5ff;border-color:#cfd8ff;color:#2f47c7}
  .badge{display:inline-block;background:#eef4ff;border:1px solid #cfd8ff;color:#2f47c7;border-radius:999px;padding:2px 8px;font-size:12px}
  .status{font-size:12px;color:#333;margin-top:8px;min-height:18px}
  .progress{margin-top:10px;border:1px solid #ddd;border-radius:8px;overflow:hidden;height:12px;background:#f2f2f2}
  .bar{height:100%;width:0%;background:#0b5cff;transition:width .2s}
  .muted{font-size:12px;color:#666}
  .dis{opacity:.6;pointer-events:none}
  `;
  sh.appendChild(css);

  const wrap = document.createElement("div");
  wrap.className = "b";
  wrap.innerHTML = `
    <div class="c">
      <div class="h">EOY Hall of Fame Export</div>
      <div class="bd">
        <div class="grid2">
          <div>
            <label class="lbl">US Regions</label>
            <div class="stack">
              <label class="muted"><input type="checkbox" id="selAll"> Select all</label>
              <span class="badge" id="usCount">0 selected</span>
            </div>
            <input class="s" id="q" placeholder="Search regions…">
            <div class="list" id="lst"></div>
            <label class="muted"><input type="checkbox" id="rest"> Include “Rest of EOY World”</label>
          </div>
          <div>
            <label class="lbl">Years</label>
            <div class="stack">
              <label class="muted"><input type="checkbox" id="allYears" checked> All years</label>
              <span class="badge" id="yrInfo"></span>
            </div>
            <input id="yrs" type="text" placeholder='Comma-separated: e.g. "2024,2023" (disabled if All years)' disabled>
            <div class="help muted">Sort is: RegionName A→Z, then RegionAwardYear (desc), then CompanyName A→Z.</div>
            <div class="status" id="st"></div>
            <div class="progress"><div class="bar" id="bar"></div></div>
          </div>
        </div>
      </div>
      <div class="acts">
        <div class="stack">
          <button class="btn ghost" id="reset">Reset selections</button>
        </div>
        <div class="stack">
          <button class="btn" id="close">Close</button>
          <button class="btn warn" id="stop">Stop</button>
          <button class="btn p" id="run">Run</button>
        </div>
      </div>
    </div>`;
  sh.appendChild(wrap);

  // ---- UI refs ----
  const $ = id => sh.getElementById(id);
  const lst=$("lst"), q=$("q"), selAll=$("selAll"), usBadge=$("usCount");
  const restCb=$("rest"), allYrs=$("allYears"), yrsInp=$("yrs"), yrInfo=$("yrInfo");
  const st=$("st"), bar=$("bar"), runBtn=$("run"), stopBtn=$("stop"), closeB=$("close"), resetB=$("reset");

  // ---- Render US list
  function renderUS(filter=""){
    const f=(filter||"").toLowerCase(); lst.innerHTML="";
    US_REGIONS.filter(r => (r.name+" "+r.id).toLowerCase().includes(f)).forEach(r=>{
      const row=document.createElement("label"); row.className="item";
      row.innerHTML=`<input type="checkbox" value="${r.id}"> <span>${r.name} <span class="muted">(ID ${r.id})</span></span>`;
      lst.appendChild(row);
    });
    const saved=(localStorage.getItem("EOYHOF_US_IDS")||"").split(",").map(s=>s.trim()).filter(Boolean).map(Number);
    saved.forEach(v => { const cb = lst.querySelector(`input[value="${v}"]`); if (cb) cb.checked = true; });
    updateUSBadge();
  }
  function updateUSBadge(){ usBadge.textContent = `${lst.querySelectorAll('input[type="checkbox"]:checked').length} selected`; }
  q.addEventListener("input",()=>renderUS(q.value));
  selAll.addEventListener("change",()=>{ lst.querySelectorAll('input[type="checkbox"]').forEach(cb=>cb.checked=selAll.checked); updateUSBadge(); });
  lst.addEventListener("change",updateUSBadge);

  // ---- Years UI
  allYrs.addEventListener("change",()=>{
    yrsInp.disabled = allYrs.checked;
    if (allYrs.checked) {
      const y0 = new Date().getFullYear();
      yrInfo.textContent = `Years: ${YEAR_START}–${y0} (${y0 - YEAR_START + 1})`;
    } else {
      yrInfo.textContent = "";
      yrsInp.focus();
    }
  });
  (function initYearBadge(){ const y0=new Date().getFullYear(); yrInfo.textContent = `Years: ${YEAR_START}–${y0} (${y0 - YEAR_START + 1})`; })();

  // ---- Controls
  function setStatus(txt,pct=null,spin=true){
    st.innerHTML = `${spin?'<span style="display:inline-block;width:12px;height:12px;border:2px solid #ccc;border-top-color:#0b5cff;border-radius:50%;animation:sp .8s linear infinite;vertical-align:-2px;margin-right:6px"></span>':""}${txt}`;
    if (pct!=null) bar.style.width = Math.max(0,Math.min(100,pct))+"%";
  }
  resetB.onclick=()=>{ localStorage.removeItem("EOYHOF_US_IDS"); localStorage.removeItem("EOYHOF_AWARD_YEARS");
    selAll.checked=false; restCb.checked=false; allYrs.checked=true; yrsInp.value=""; yrsInp.disabled=true; initYearBadge();
    renderUS(""); setStatus("Reset.",0,false); bar.style.width="0%"; };
  closeB.onclick=()=>host.remove();

  let aborter=null;
  stopBtn.onclick=()=>{ if(aborter) aborter.abort(); setStatus("Stopped.",null,false); runBtn.classList.remove("dis"); };

  renderUS();

  // ---- Runner
  runBtn.onclick = async () => {
    const selectedUS = Array.from(lst.querySelectorAll('input[type="checkbox"]:checked')).map(cb=>Number(cb.value));
    const includeWorld = restCb.checked;
    if (!selectedUS.length && !includeWorld) { alert("Select at least one US region or include Rest of World."); return; }

    // Years list
    const yearsRaw = (yrsInp.value||"").trim();
    let YEARS = [];
    if (allYrs.checked || !yearsRaw) {
      const y0 = new Date().getFullYear(); for (let y=y0; y>=YEAR_START; y--) YEARS.push(String(y));
    } else {
      YEARS = yearsRaw.split(",").map(s=>s.trim()).filter(Boolean).map(String);
    }

    // Save prefs
    localStorage.setItem("EOYHOF_US_IDS", selectedUS.join(","));
    localStorage.setItem("EOYHOF_AWARD_YEARS", yearsRaw);

    // Lock UI
    runBtn.classList.add("dis"); closeB.classList.add("dis"); resetB.classList.add("dis");
    q.disabled=true; selAll.disabled=true; restCb.disabled=true; allYrs.disabled=true; yrsInp.disabled=true;

    try{
      const nameMap = new Map(US_REGIONS.map(r=>[r.id,r.name]));
      WORLD_REGIONS.forEach(r=>nameMap.set(r.id,r.name));
      let targets = [...selectedUS];
      if (includeWorld) targets = targets.concat(WORLD_REGIONS.map(r=>r.id));

      const tasks = []; for (const rid of targets) for (const y of YEARS) tasks.push([rid,y]);
      const total = tasks.length; let done=0, active=0, idx=0, totalRows=0;

      const rowsOut = []; const seen = new Set();
      const keyOf = r => `${r.profileId ?? ""}|${(r.firstName||"").toLowerCase()}|${(r.lastName||"").toLowerCase()}|${(r.companyName||"").toLowerCase()}|${r.regionId ?? r.regionID}|${r.regionAwardYear ?? r.awardYear ?? ""}`;

      aborter = new AbortController();
      setStatus(`Running ${total} requests…`, 10);

      await new Promise((resolve)=>{
        const pump = () => {
          while (active < CONCURRENCY && idx < tasks.length && !aborter.signal.aborted) {
            const [rid, y] = tasks[idx++]; active++;
            const rname = nameMap.get(rid) || `Region_${rid}`;
            const pct = 10 + Math.floor((done/Math.max(1,total))*90);
            setStatus(`Fetching: ${rname} • Year ${y} • ${done}/${total} • Rows: ${totalRows}`, pct, true);

            fetchWithRetry(fetchMemberSearch, [rid, y, aborter.signal], RETRIES)
              .then(list=>{
                const filtered = Array.isArray(list) ? list.filter(x=>String(x.regionId ?? x.regionID)===String(rid)) : [];
                for (const row of filtered) { const k=keyOf(row); if (!seen.has(k)) { seen.add(k); rowsOut.push(row); totalRows++; } }
              })
              .catch(()=>{})
              .finally(()=>{ active--; done++; if (done>=total && active===0) resolve(); else pump(); });
          }
          if (aborter.signal.aborted) resolve();
        };
        pump();
      });

      if (!rowsOut.length) { setStatus("No rows found.", 100, false); alert("No rows found."); return; }

      // === SORT: RegionName A→Z → RegionAwardYear DESC → CompanyName A→Z ===
      rowsOut.sort((a,b)=>{
        const ar = (a.regionName || "").localeCompare(b.regionName || "", undefined, {sensitivity:"base"});
        if (ar !== 0) return ar;
        const ay = Number(a.regionAwardYear ?? a.awardYear ?? 0);
        const by = Number(b.regionAwardYear ?? b.awardYear ?? 0);
        if (ay !== by) return by - ay;            // newest → oldest
        return (a.companyName||"").localeCompare(b.companyName||"", undefined, {sensitivity:"base"});
      });

      // Filename scope
      const usOnly = !includeWorld && targets.every(id=>US_ID_SET.has(String(id)));
      let scopeTag = usOnly
        ? (selectedUS.length===1 ? (US_REGIONS.find(z=>z.id===selectedUS[0])?.name || `Region_${selectedUS[0]}`) : `US_MULTI_${selectedUS.length}`)
        : "US_plus_RestOfWorld";
      scopeTag = scopeTag.replace(/\s+/g,"_").replace(/[^\w\-]/g,"");

      const yearsTag = (allYrs.checked || !yearsRaw) ? "ALL" : YEARS.join("-");
      const fileName = `eoy_hof_${scopeTag}_${yearsTag}.csv`;

      setStatus(`Packaging CSV… ${rowsOut.length} rows`, 100, true);
      const a = document.createElement("a");
      a.href = URL.createObjectURL(new Blob([toCSV(rowsOut)],{type:"text/csv;charset=utf-8;"}));
      a.download = fileName;
      a.click();
      setStatus(`Done! Exported ${rowsOut.length} rows → ${fileName}`, 100, false);
    } catch(e){
      console.error(e);
      setStatus("Error — see console for details.", 100, false);
      alert("Something went wrong. See console for details.");
    } finally {
      runBtn.classList.remove("dis"); closeB.classList.remove("dis"); resetB.classList.remove("dis");
      q.disabled=false; selAll.disabled=false; restCb.disabled=false; allYrs.disabled=false;
      if (!allYrs.checked) yrsInp.disabled=false;
      aborter=null;
    }
  };
})();
