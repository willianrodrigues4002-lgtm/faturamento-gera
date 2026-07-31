@@ -0,0 +1,262 @@
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Registro diário — faturamento</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@500;700&family=IBM+Plex+Sans:wght@400;500&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
<style>
  :root{
    --ink: #14161B;
    --panel: #1D2027;
    --panel-2: #262A33;
    --line: #33373F;
    --text: #EDEEF0;
    --text-dim: #8B909C;
    --teal: #3FA796;
    --teal-dim: #234E47;
    --gold: #E8B34B;
    --danger: #E0665B;
  }
  *{ box-sizing:border-box; }
  body{ margin:0; background:var(--ink); color:var(--text); font-family:'IBM Plex Sans', sans-serif; min-height:100vh; }
  .wrap{ max-width: 1040px; margin:0 auto; padding: 40px 24px 80px; }

  header{ border-bottom:1px solid var(--line); padding-bottom:20px; margin-bottom:28px; }
  h1{ font-family:'Space Grotesk', sans-serif; font-weight:700; font-size:28px; margin:0; letter-spacing:-0.5px; }
  h1 span{ color:var(--teal); }
  header p{ font-family:'JetBrains Mono', monospace; font-size:12px; color:var(--text-dim); margin:6px 0 0;
    text-transform:uppercase; letter-spacing:0.08em; }

  .panel{ background:var(--panel); border:1px solid var(--line); border-radius:12px; padding:20px; margin-bottom:24px; }
  .panel h2{ font-family:'Space Grotesk', sans-serif; font-size:16px; margin:0 0 16px; }

  .form-row{ display:grid; grid-template-columns: repeat(auto-fit, minmax(120px, 1fr)); gap:10px; align-items:end; }
  .form-row button{ grid-column: 1 / -1; }
  @media (min-width: 900px){ .form-row button{ grid-column: auto; } }
  .field label{ display:block; font-size:11px; color:var(--text-dim); margin-bottom:6px;
    font-family:'JetBrains Mono', monospace; text-transform:uppercase; letter-spacing:0.05em; }
  .field input{ width:100%; padding:10px 12px; border-radius:8px; border:1px solid var(--line);
    background:var(--ink); color:var(--text); font-size:14px; }
  .field input:focus{ outline:none; border-color:var(--teal); }

  button{ font-family:'IBM Plex Sans', sans-serif; font-size:14px; font-weight:500; padding:10px 16px;
    border-radius:8px; border:1px solid var(--line); background:var(--panel-2); color:var(--text);
    cursor:pointer; }
  button:hover{ border-color:var(--text-dim); }
  .btn-primary{ background:var(--teal-dim); border-color:var(--teal); color:#C9F2EC; white-space:nowrap; }

  .error-msg{ font-size:12px; color:var(--danger); margin:10px 0 0; display:none; }

  .cards{ display:grid; grid-template-columns:repeat(auto-fit, minmax(160px, 1fr)); gap:12px; margin-bottom:24px; }
  .metric{ background:var(--panel); border:1px solid var(--line); border-radius:12px; padding:14px 16px; }
  .metric .label{ font-size:11px; color:var(--text-dim); font-family:'JetBrains Mono', monospace;
    text-transform:uppercase; letter-spacing:0.05em; margin-bottom:6px; }
  .metric .value{ font-size:22px; font-weight:500; font-family:'Space Grotesk', sans-serif; }
  .metric.accent .value{ color: var(--gold); }

  table{ width:100%; border-collapse:collapse; font-size:13px; }
  th, td{ text-align:left; padding:10px 8px; border-bottom:1px solid var(--line); }
  th{ font-family:'JetBrains Mono', monospace; font-size:11px; color:var(--text-dim);
    text-transform:uppercase; letter-spacing:0.05em; font-weight:400; }
  td.num{ font-family:'JetBrains Mono', monospace; }
  tr:last-child td{ border-bottom:none; }
  .del-btn{ padding:4px 8px; font-size:12px; color:var(--danger); border-color:var(--line); }
  .empty{ color:var(--text-dim); font-size:13px; font-family:'JetBrains Mono', monospace; padding:20px 0; text-align:center; }
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1>registro<span>_</span>diário</h1>
    <p>faturamento por funcionário</p>
  </header>

  <div class="panel">
    <h2>novo registro</h2>
    <div class="form-row">
      <div class="field">
        <label>funcionário</label>
        <input type="text" id="f-name" placeholder="seu nome">
      </div>
      <div class="field">
        <label>data</label>
        <input type="date" id="f-date">
      </div>
      <div class="field">
        <label>tipo de caminhão</label>
        <input type="text" id="f-truck" placeholder="ex: baú, carreta">
      </div>
      <div class="field">
        <label>placa</label>
        <input type="text" id="f-plate" placeholder="ABC1D23">
      </div>
      <div class="field">
        <label>cargas</label>
        <input type="number" id="f-loads" placeholder="0" min="0">
      </div>
      <div class="field">
        <label>paletes</label>
        <input type="number" id="f-pallets" placeholder="0" min="0">
      </div>
      <div class="field">
        <label>lotes</label>
        <input type="number" id="f-batches" placeholder="0" min="0">
      </div>
      <button class="btn-primary" onclick="addEntry()">registrar</button>
    </div>
    <p class="error-msg" id="form-error">preencha nome, paletes e lotes antes de registrar.</p>
  </div>

  <div class="panel">
    <h2>médias por dia</h2>
    <div class="cards" id="summary-cards"></div>
    <table id="summary-table">
      <thead>
        <tr><th>funcionário</th><th>dias registrados</th><th>total cargas</th><th>total paletes</th><th>total lotes</th><th>média cargas/dia</th><th>média paletes/dia</th><th>média lotes/dia</th></tr>
      </thead>
      <tbody id="summary-body"></tbody>
    </table>
  </div>

  <div class="panel">
    <h2>histórico de registros</h2>
    <table>
      <thead>
        <tr><th>data</th><th>funcionário</th><th>caminhão</th><th>placa</th><th>cargas</th><th>paletes</th><th>lotes</th><th></th></tr>
      </thead>
      <tbody id="log-body"></tbody>
    </table>
  </div>
</div>

<script>
function loadEntries(){
  const raw = localStorage.getItem('registro_entries');
  return raw ? JSON.parse(raw) : [];
}
function saveEntries(entries){ localStorage.setItem('registro_entries', JSON.stringify(entries)); }

function fmtMoney(v){
  return 'R$ ' + v.toLocaleString('pt-BR', {minimumFractionDigits:2, maximumFractionDigits:2});
}
function fmtNum(v){
  return v.toLocaleString('pt-BR', {minimumFractionDigits:0, maximumFractionDigits:1});
}

document.getElementById('f-date').valueAsDate = new Date();

function addEntry(){
  const name = document.getElementById('f-name').value.trim();
  const date = document.getElementById('f-date').value;
  const truck = document.getElementById('f-truck').value.trim();
  const plate = document.getElementById('f-plate').value.trim();
  const loads = parseFloat(document.getElementById('f-loads').value);
  const pallets = parseFloat(document.getElementById('f-pallets').value);
  const batches = parseFloat(document.getElementById('f-batches').value);
  const errorEl = document.getElementById('form-error');
  if(!name || !date || isNaN(pallets) || isNaN(batches)){
    errorEl.style.display = 'block';
    return;
  }
  errorEl.style.display = 'none';
  const entries = loadEntries();
  entries.push({ id: crypto.randomUUID(), name, date, truck, plate, loads: isNaN(loads) ? 0 : loads, pallets, batches });
  saveEntries(entries);
  document.getElementById('f-truck').value = '';
  document.getElementById('f-plate').value = '';
  document.getElementById('f-loads').value = '';
  document.getElementById('f-pallets').value = '';
  document.getElementById('f-batches').value = '';
  render();
}

function deleteEntry(id){
  const entries = loadEntries().filter(e => e.id !== id);
  saveEntries(entries);
  render();
}

function render(){
  const entries = loadEntries();

  // log
  const logBody = document.getElementById('log-body');
  if(entries.length === 0){
    logBody.innerHTML = '<tr><td colspan="8" class="empty">nenhum registro ainda</td></tr>';
  } else {
    const sorted = [...entries].sort((a,b) => b.date.localeCompare(a.date));
    logBody.innerHTML = sorted.map(e =>
      '<tr><td>' + e.date.split('-').reverse().join('/') + '</td>' +
      '<td>' + escapeHtml(e.name) + '</td>' +
      '<td>' + escapeHtml(e.truck || '—') + '</td>' +
      '<td>' + escapeHtml(e.plate || '—') + '</td>' +
      '<td class="num">' + fmtNum(e.loads || 0) + '</td>' +
      '<td class="num">' + fmtNum(e.pallets) + '</td>' +
      '<td class="num">' + fmtNum(e.batches) + '</td>' +
      '<td><button class="del-btn" onclick="deleteEntry(\'' + e.id + '\')">remover</button></td></tr>'
    ).join('');
  }

  // group by name+date (daily totals per person)
  const byPersonDay = {};
  entries.forEach(e => {
    const key = e.name.toLowerCase() + '|' + e.date;
    if(!byPersonDay[key]) byPersonDay[key] = { name: e.name, date: e.date, pallets: 0, batches: 0, loads: 0 };
    byPersonDay[key].pallets += e.pallets;
    byPersonDay[key].batches += e.batches;
    byPersonDay[key].loads += (e.loads || 0);
  });
  const dailyRows = Object.values(byPersonDay);

  // per employee summary
  const byPerson = {};
  dailyRows.forEach(r => {
    const key = r.name.toLowerCase();
    if(!byPerson[key]) byPerson[key] = { name: r.name, days: 0, pallets: 0, batches: 0, loads: 0 };
    byPerson[key].days += 1;
    byPerson[key].pallets += r.pallets;
    byPerson[key].batches += r.batches;
    byPerson[key].loads += r.loads;
  });
  const summaryBody = document.getElementById('summary-body');
  const people = Object.values(byPerson).sort((a,b) => b.value - a.value);
  if(people.length === 0){
    summaryBody.innerHTML = '<tr><td colspan="8" class="empty">sem dados suficientes ainda</td></tr>';
  } else {
    summaryBody.innerHTML = people.map(p =>
      '<tr><td>' + escapeHtml(p.name) + '</td>' +
      '<td class="num">' + p.days + '</td>' +
      '<td class="num">' + fmtNum(p.loads) + '</td>' +
      '<td class="num">' + fmtNum(p.pallets) + '</td>' +
      '<td class="num">' + fmtNum(p.batches) + '</td>' +
      '<td class="num">' + fmtNum(p.loads / p.days) + '</td>' +
      '<td class="num">' + fmtNum(p.pallets / p.days) + '</td>' +
      '<td class="num">' + fmtNum(p.batches / p.days) + '</td></tr>'
    ).join('');
  }

  // overall: average per day across the whole team (distinct days)
  const distinctDays = new Set(entries.map(e => e.date));
  const totalPallets = entries.reduce((s,e) => s + e.pallets, 0);
  const totalBatches = entries.reduce((s,e) => s + e.batches, 0);
  const numDays = distinctDays.size || 1;

  document.getElementById('summary-cards').innerHTML =
    '<div class="metric"><div class="label">dias registrados</div><div class="value">' + distinctDays.size + '</div></div>' +
    '<div class="metric"><div class="label">total lotes</div><div class="value">' + fmtNum(totalBatches) + '</div></div>' +
    '<div class="metric accent"><div class="label">média geral / dia (paletes)</div><div class="value">' + fmtNum(totalPallets / numDays) + '</div></div>' +
    '<div class="metric accent"><div class="label">média geral / dia (lotes)</div><div class="value">' + fmtNum(totalBatches / numDays) + '</div></div>';
}

function escapeHtml(s){
  const d = document.createElement('div');
  d.textContent = s || '';
  return d.innerHTML;
}

render();
</script>
</body>
</html>
