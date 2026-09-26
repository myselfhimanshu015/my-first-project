<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Distributed Object Store — Simulator</title>
<style>
:root{
  --bg:#0b0f14; --panel:#131a22; --panel2:#0f151c; --border:#22303c;
  --text:#e6edf3; --sub:#8fa3b3; --accent:#4fd1c5; --accent2:#63b3ed;
  --ok:#4ade80; --warn:#facc15; --bad:#f87171; --dead:#5b6774;
  --mono:'SF Mono',Consolas,'Courier New',monospace;
}
@media (prefers-color-scheme: light){
  :root:not([data-theme="dark"]){
    --bg:#f4f7f9; --panel:#ffffff; --panel2:#eef2f5; --border:#dbe3e9; --text:#152028; --sub:#5b6b78;
  }
}
:root[data-theme="dark"]{ --bg:#0b0f14; }
*{box-sizing:border-box;}
html{scroll-padding-top:env(safe-area-inset-top,0px);}
body{margin:0;background:var(--bg);color:var(--text);font-family:-apple-system,Segoe UI,Roboto,Helvetica,Arial,sans-serif;
  padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px);}
header{padding:16px 18px 8px;border-bottom:1px solid var(--border);display:flex;justify-content:space-between;align-items:flex-start;gap:10px;flex-wrap:wrap}
header h1{margin:0;font-size:1.15rem}
header p{margin:4px 0 0;color:var(--sub);font-size:.82rem}
.wrap{max-width:1200px;margin:0 auto;padding:14px;display:grid;gap:14px;grid-template-columns:1fr}
@media(min-width:980px){.wrap{grid-template-columns:2fr 1fr}}
.panel{background:var(--panel);border:1px solid var(--border);border-radius:12px;padding:14px;min-width:0}
.panel h2{margin:0 0 10px;font-size:.92rem;color:var(--accent2)}
.row{display:flex;gap:8px;flex-wrap:wrap;align-items:center;margin-bottom:10px}
input,select,button{font:inherit;background:var(--panel2);color:var(--text);border:1px solid var(--border);border-radius:8px;padding:7px 10px;}
input{min-width:110px;flex:1}
button{cursor:pointer;font-weight:600;transition:.15s}
button:hover{border-color:var(--accent)}
button.primary{background:var(--accent);color:#04201c;border-color:var(--accent)}
button.danger{background:transparent;color:var(--bad);border-color:var(--bad)}
button.warnbtn{background:transparent;color:var(--warn);border-color:var(--warn)}
button.ghost{background:transparent;color:var(--sub)}
button:disabled{opacity:.4;cursor:not-allowed}
.nodes{display:grid;grid-template-columns:repeat(auto-fill,minmax(148px,1fr));gap:10px}
.node{border:1px solid var(--border);border-radius:10px;padding:8px;background:var(--panel2);font-size:.78rem}
.node.dead{opacity:.55}
.dot{width:9px;height:9px;border-radius:50%;display:inline-block;margin-right:5px}
.node .id{font-weight:700;font-family:var(--mono)}
.node .stat{color:var(--sub);margin:4px 0}
.node .btns{display:flex;gap:4px;margin-top:6px;flex-wrap:wrap}
.node .btns button{padding:3px 6px;font-size:.72rem;flex:1}
table{width:100%;border-collapse:collapse;font-size:.8rem}
th,td{padding:6px 8px;text-align:left;border-bottom:1px solid var(--border);word-break:break-word}
th{color:var(--sub);font-weight:600}
.replicas{display:flex;gap:4px;flex-wrap:wrap}
.rchip{font-family:var(--mono);font-size:.68rem;padding:2px 5px;border-radius:5px;border:1px solid var(--border)}
.log{background:var(--panel2);border:1px solid var(--border);border-radius:8px;padding:8px;height:240px;overflow-y:auto;font-family:var(--mono);font-size:.72rem;line-height:1.5}
.log div{border-bottom:1px dashed var(--border);padding:2px 0}
.tag{font-weight:700}
.t-ok{color:var(--ok)} .t-warn{color:var(--warn)} .t-bad{color:var(--bad)} .t-info{color:var(--accent2)}
.small{color:var(--sub);font-size:.74rem}
.metrics{display:grid;grid-template-columns:repeat(3,1fr);gap:8px;margin-bottom:12px}
.metric{background:var(--panel2);border:1px solid var(--border);border-radius:8px;padding:8px;text-align:center}
.metric b{display:block;font-size:1.15rem}
.metric span{color:var(--sub);font-size:.68rem}
.tableWrap{overflow-x:auto}
footer{text-align:center;color:var(--sub);font-size:.7rem;padding:18px}
.savepill{font-size:.7rem;color:var(--ok);background:rgba(74,222,128,.1);border:1px solid var(--ok);border-radius:20px;padding:3px 9px;white-space:nowrap}
</style>
</head>
<body>
<header>
  <div>
    <h1>🗄️ Distributed Object Store — live fault-tolerance simulator</h1>
    <p>N storage nodes · replication factor R · checksums · failure injection · background repair &amp; rebalancing.<br>State auto-saves to your browser (localStorage) and survives a page refresh.</p>
  </div>
  <span class="savepill" id="saveStatus">saved</span>
</header>

<div class="wrap">
  <div style="display:flex;flex-direction:column;gap:14px;min-width:0">

    <div class="panel">
      <h2>Storage nodes</h2>
      <div class="row">
        <button id="addNode">+ Add node</button>
        <button id="killRandom" class="danger">Kill random node</button>
        <button id="reviveAll">Revive all</button>
        <label class="small" style="display:flex;gap:6px;align-items:center;margin-left:auto">
          <input type="checkbox" id="autoRepair" checked style="min-width:auto;width:16px;height:16px"> Auto-repair loop
        </label>
      </div>
      <div class="nodes" id="nodeGrid"></div>
    </div>

    <div class="panel">
      <h2>Objects</h2>
      <div class="row">
        <input id="objKey" placeholder="key (e.g. invoice-2026.pdf)">
        <input id="objData" placeholder="data / payload">
        <select id="objR" style="min-width:70px">
          <option value="1">R=1</option><option value="2">R=2</option><option value="3" selected>R=3</option><option value="4">R=4</option><option value="5">R=5</option>
        </select>
        <button class="primary" id="putBtn">PUT</button>
      </div>
      <div class="tableWrap">
      <table>
        <thead><tr><th>Key</th><th>R</th><th>Replicas (node:status)</th><th>Actions</th></tr></thead>
        <tbody id="objTable"></tbody>
      </table>
      </div>
      <p class="small" id="emptyMsg">No objects yet — write one above.</p>
    </div>

  </div>

  <div style="display:flex;flex-direction:column;gap:14px;min-width:0">
    <div class="panel">
      <h2>Cluster health</h2>
      <div class="metrics">
        <div class="metric"><b id="mNodes">0</b><span>alive / total nodes</span></div>
        <div class="metric"><b id="mObjs">0</b><span>objects</span></div>
        <div class="metric"><b id="mUnder">0</b><span>under-replicated</span></div>
      </div>
      <div class="row">
        <button id="repairNow">Run repair cycle now</button>
        <button id="rebalance">Rebalance</button>
        <button id="resetAll" class="ghost">Reset cluster &amp; storage</button>
      </div>
    </div>

    <div class="panel">
      <h2>Event log</h2>
      <div class="log" id="logBox"></div>
    </div>

    <div class="panel">
      <h2>How it works</h2>
      <p class="small">
        PUT hashes the key to pick R distinct alive nodes and stores a replica + checksum on each.
        GET reads replicas in placement order, verifies the checksum, and falls through to the next
        healthy replica on corruption or node death. A background loop scans every object, counts
        healthy replicas, and heals any object below R by copying from a surviving replica onto a
        new alive node. "Rebalance" moves replicas off overloaded nodes without changing durability.
        Everything here — nodes, objects, and the log — is persisted to <code>localStorage</code>,
        so refreshing this page restores your cluster exactly as you left it.
      </p>
    </div>
  </div>
</div>

<footer>Client-side simulation for demonstrating replication, integrity verification, and repair algorithms. Data lives only in this browser.</footer>

<script>
const STORAGE_KEY = 'dostore_sim_v1';
const state = { nodes: [], objects: new Map(), nextNodeId: 1, log: [] };

function checksum(str){
  let h = 0x811c9dc5;
  for(let i=0;i<str.length;i++){ h ^= str.charCodeAt(i); h = Math.imul(h, 16777619); }
  return (h >>> 0).toString(16).padStart(8,'0');
}
function hashKey(key){
  let h = 0;
  for(let i=0;i<key.length;i++){ h = (h*31 + key.charCodeAt(i)) >>> 0; }
  return h;
}

// ---------- persistence ----------
function saveState(){
  try{
    const data = {
      nextNodeId: state.nextNodeId,
      log: state.log.slice(0,200),
      nodes: state.nodes.map(n=>({id:n.id, alive:n.alive, chunks:[...n.chunks.entries()]})),
      objects: [...state.objects.entries()]
    };
    localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
    flashSaved();
  }catch(e){ console.error('save failed', e); }
}
function flashSaved(){
  const el = document.getElementById('saveStatus');
  el.textContent = 'saved'; el.style.opacity = '1';
  clearTimeout(flashSaved._t);
  flashSaved._t = setTimeout(()=>{ el.style.opacity='.55'; }, 900);
}
function loadState(){
  try{
    const raw = localStorage.getItem(STORAGE_KEY);
    if(!raw) return false;
    const data = JSON.parse(raw);
    state.nextNodeId = data.nextNodeId || 1;
    state.log = data.log || [];
    state.nodes = (data.nodes||[]).map(n=>({id:n.id, alive:n.alive, chunks:new Map(n.chunks)}));
    state.objects = new Map(data.objects||[]);
    return state.nodes.length>0;
  }catch(e){ console.error('load failed', e); return false; }
}

function log(msg, cls){
  const t = new Date().toLocaleTimeString();
  state.log.unshift(`<div><span class="small">${t}</span> <span class="tag t-${cls||'info'}">${msg}</span></div>`);
  if(state.log.length>200) state.log.pop();
  document.getElementById('logBox').innerHTML = state.log.join('');
}
function aliveNodes(){ return state.nodes.filter(n=>n.alive); }

function addNode(silent){
  const n = { id:'N'+state.nextNodeId++, alive:true, chunks:new Map() };
  state.nodes.push(n);
  if(!silent) log(`Node <b>${n.id}</b> joined cluster`, 'ok');
  render();
}
function killNode(id){
  const n = state.nodes.find(x=>x.id===id); if(!n) return;
  n.alive=false; log(`Node <b>${id}</b> went DOWN`, 'bad'); render();
}
function reviveNode(id){
  const n = state.nodes.find(x=>x.id===id); if(!n) return;
  n.alive=true; log(`Node <b>${id}</b> recovered / rejoined`, 'ok'); render();
}
function corruptNode(id){
  const n = state.nodes.find(x=>x.id===id); if(!n || n.chunks.size===0) return;
  const keys = [...n.chunks.keys()];
  const victim = keys[Math.floor(Math.random()*keys.length)];
  const c = n.chunks.get(victim);
  c.data = c.data + '#';
  log(`Bit-rot injected on <b>${id}</b> (chunk ${victim})`, 'warn');
  render();
}
function pickNodes(key, r){
  const alive = aliveNodes();
  if(alive.length===0) return [];
  const h = hashKey(key);
  const arr = alive.slice().sort((a,b)=>((hashKey(a.id+h)) - (hashKey(b.id+h))));
  return arr.slice(0, Math.min(r, arr.length));
}
function putObject(key, data, r){
  const nodes = pickNodes(key, r);
  if(nodes.length===0){ log('PUT failed: no alive nodes', 'bad'); return; }
  const replicas = nodes.map(n=>{ n.chunks.set(key, {data, checksum: checksum(data)}); return {node:n.id}; });
  state.objects.set(key, {data, r:Number(r), replicas});
  if(nodes.length < r) log(`PUT <b>${key}</b>: only ${nodes.length}/${r} replicas placed (insufficient alive nodes)`, 'warn');
  else log(`PUT <b>${key}</b> → replicated to ${nodes.map(n=>n.id).join(', ')}`, 'ok');
  render();
}
function replicaStatus(nodeId, key){
  const n = state.nodes.find(x=>x.id===nodeId);
  if(!n) return 'missing';
  if(!n.alive) return 'down';
  const c = n.chunks.get(key);
  if(!c) return 'missing';
  return c.checksum === checksum(c.data) ? 'ok' : 'corrupt';
}
function getObject(key){
  const obj = state.objects.get(key);
  if(!obj){ log(`GET <b>${key}</b>: not found`, 'bad'); return; }
  for(const rep of obj.replicas){
    const status = replicaStatus(rep.node, key);
    if(status==='ok'){ log(`GET <b>${key}</b> ← served from ${rep.node} (verified)`, 'ok'); return; }
    if(status==='corrupt') log(`GET <b>${key}</b>: replica on ${rep.node} FAILED checksum, trying next`, 'bad');
    if(status==='down') log(`GET <b>${key}</b>: replica on ${rep.node} unreachable, trying next`, 'warn');
    if(status==='missing') log(`GET <b>${key}</b>: replica missing on ${rep.node}, trying next`, 'warn');
  }
  log(`GET <b>${key}</b>: ALL replicas unavailable — object currently unreadable`, 'bad');
}
function repairCycle(){
  let healed = 0;
  for(const [key, obj] of state.objects){
    const healthyReps = obj.replicas.filter(r=>replicaStatus(r.node,key)==='ok');
    if(healthyReps.length===0 || healthyReps.length >= obj.r) continue;
    const sourceNode = state.nodes.find(n=>n.id===healthyReps[0].node);
    const goodData = sourceNode.chunks.get(key).data;
    const holderIds = new Set(obj.replicas.map(r=>r.node));
    const candidates = aliveNodes().filter(n=>!holderIds.has(n.id) || replicaStatus(n.id,key)!=='ok');
    for(const target of candidates){
      if(healthyReps.length + healed >= obj.r) break;
      target.chunks.set(key, {data: goodData, checksum: checksum(goodData)});
      if(!obj.replicas.find(r=>r.node===target.id)) obj.replicas.push({node: target.id});
      log(`Repaired <b>${key}</b> → new/healed replica on ${target.id}`, 'ok');
      healed++; break;
    }
  }
  log(healed===0 ? 'Repair cycle: cluster fully healthy, nothing to do' : `Repair cycle complete: ${healed} replica(s) healed`, healed===0?'info':'ok');
  render();
}
function rebalance(){
  const alive = aliveNodes();
  if(alive.length<2) return;
  const avg = alive.reduce((s,n)=>s+n.chunks.size,0)/alive.length;
  let moved=0;
  for(const n of alive){
    if(n.chunks.size > avg+1){
      const target = alive.slice().sort((a,b)=>a.chunks.size-b.chunks.size)[0];
      const [key] = n.chunks.keys();
      if(key===undefined || target.id===n.id) continue;
      const c = n.chunks.get(key);
      n.chunks.delete(key); target.chunks.set(key, c);
      const obj = state.objects.get(key);
      if(obj){ const r = obj.replicas.find(r=>r.node===n.id); if(r) r.node = target.id; }
      moved++;
    }
  }
  log(moved ? `Rebalance moved ${moved} chunk(s) to even out load` : 'Rebalance: load already even', moved?'ok':'info');
  render();
}
function resetAll(){
  if(!confirm('Reset the entire cluster and clear saved storage?')) return;
  localStorage.removeItem(STORAGE_KEY);
  state.nodes=[]; state.objects=new Map(); state.nextNodeId=1; state.log=[];
  seedDefault();
  render();
}
function seedDefault(){
  for(let i=0;i<5;i++) addNode(true);
  putObject('welcome.txt', 'hello-distributed-world', 3);
  log('Cluster initialized with 5 nodes', 'info');
}

function render(){
  const grid = document.getElementById('nodeGrid');
  grid.innerHTML = state.nodes.map(n=>{
    const color = n.alive ? 'var(--ok)' : 'var(--dead)';
    return `<div class="node ${n.alive?'':'dead'}">
      <div class="id"><span class="dot" style="background:${color}"></span>${n.id}</div>
      <div class="stat">${n.alive?'alive':'DOWN'} · ${n.chunks.size} chunk(s)</div>
      <div class="btns">
        ${n.alive
          ? `<button onclick="killNode('${n.id}')" class="danger">Kill</button><button onclick="corruptNode('${n.id}')" class="warnbtn">Corrupt</button>`
          : `<button onclick="reviveNode('${n.id}')" class="primary">Revive</button>`}
      </div>
    </div>`;
  }).join('') || '<p class="small">No nodes yet.</p>';

  const tbody = document.getElementById('objTable');
  const rows = [...state.objects.entries()].map(([key,obj])=>{
    const chips = obj.replicas.map(r=>{
      const s = replicaStatus(r.node,key);
      const color = s==='ok'?'var(--ok)':s==='corrupt'?'var(--bad)':s==='down'?'var(--dead)':'var(--warn)';
      return `<span class="rchip" style="border-color:${color};color:${color}">${r.node}:${s}</span>`;
    }).join('');
    return `<tr><td><code>${key}</code></td><td>${obj.r}</td><td><div class="replicas">${chips}</div></td>
      <td><button onclick="getObject('${key}')">GET</button></td></tr>`;
  });
  tbody.innerHTML = rows.join('');
  document.getElementById('emptyMsg').style.display = rows.length? 'none':'block';

  document.getElementById('mNodes').textContent = `${aliveNodes().length}/${state.nodes.length}`;
  document.getElementById('mObjs').textContent = state.objects.size;
  let under = 0;
  for(const [key,obj] of state.objects){
    const ok = obj.replicas.filter(r=>replicaStatus(r.node,key)==='ok').length;
    if(ok < obj.r) under++;
  }
  document.getElementById('mUnder').textContent = under;
  document.getElementById('logBox').innerHTML = state.log.join('');

  saveState();
}

// wire up controls
document.getElementById('addNode').onclick = ()=>addNode(false);
document.getElementById('killRandom').onclick = ()=>{ const a=aliveNodes(); if(a.length) killNode(a[Math.floor(Math.random()*a.length)].id); };
document.getElementById('reviveAll').onclick = ()=>{ state.nodes.forEach(n=>n.alive=true); log('All nodes revived','ok'); render(); };
document.getElementById('putBtn').onclick = ()=>{
  const k = document.getElementById('objKey').value.trim();
  const d = document.getElementById('objData').value.trim() || ('payload-'+Math.random().toString(36).slice(2,8));
  const r = document.getElementById('objR').value;
  if(!k){ log('PUT failed: key required', 'bad'); return; }
  putObject(k,d,r);
  document.getElementById('objKey').value=''; document.getElementById('objData').value='';
};
document.getElementById('repairNow').onclick = repairCycle;
document.getElementById('rebalance').onclick = rebalance;
document.getElementById('resetAll').onclick = resetAll;

// boot: restore from localStorage, else seed a fresh demo cluster
if(!loadState()) seedDefault();
setInterval(()=>{ if(document.getElementById('autoRepair').checked) repairCycle(); }, 6000);
render();
</script>
</body>
</html>
