<!doctype html>
<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Carteirinhas Inteligente — Seguro & Salvo</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrious/4.0.2/qrious.min.js"></script>

<style>
:root{--primary:#0b5bd7;--muted:#6b7280;--bg:#f5f7fb}
*{box-sizing:border-box}
body{margin:0;font-family:Inter,system-ui,Arial,sans-serif;background:var(--bg);color:#111}
#loginScreen{display:flex;align-items:center;justify-content:center;height:100vh}
.login-box{background:#fff;padding:28px;border-radius:12px;box-shadow:0 8px 28px rgba(0,0,0,0.08);width:92%;max-width:380px;text-align:center}
.login-box h2{color:var(--primary);margin-bottom:14px}
.login-box input{width:100%;padding:10px;margin-bottom:10px;border-radius:8px;border:1px solid #e6e9ee}
.login-box button{width:100%;padding:10px;background:var(--primary);border:none;color:#fff;border-radius:8px;cursor:pointer}
#errorMsg{color:#b91c1c;font-size:0.9rem;margin-bottom:8px}
#appScreen{display:none;height:100vh;overflow:auto}
header{background:var(--primary);color:#fff;padding:12px 16px;display:flex;align-items:center;justify-content:space-between}
.container{max-width:1200px;margin:18px auto;padding:16px}
.controls{display:flex;gap:12px;flex-wrap:wrap;align-items:center;margin-bottom:12px}
.control{background:#fff;padding:10px;border-radius:8px;box-shadow:0 2px 8px rgba(12,24,40,0.06)}
.small{font-size:0.85rem;color:var(--muted)}
.btn{background:var(--primary);color:#fff;padding:8px 10px;border-radius:8px;border:0;cursor:pointer}
.cards-grid{display:flex;flex-wrap:wrap;gap:12px;justify-content:center;margin-top:14px}

/* Carteirinha com fundo creme */
.card {
  width: 8.6cm;
  height: 5.4cm;
  position: relative;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 6px 18px rgba(2,6,23,0.12);
  background: linear-gradient(135deg, #f5f0e6, #e8dfd3);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  padding: 6px;
}

.card .foto {
  width: 2.5cm;
  height: 3cm;
  border-radius: 6px;
  object-fit: cover;
  border: 2px solid #fff;
  background: #e5e7eb;
}

.card .logo-bottom {
  position: absolute;
  top: 8px;
  left: 8px;
  width: 30%;
  max-height: 30%;
  object-fit: contain;
  opacity: 0.95;
}

.card .content {
  text-align: center;
  width: 100%;
}

.card .content .nome {
  font-weight: 700;
  font-size: 13px;
  color: #a00000; /* vermelho escuro */
  margin-top: 4px;
}

.card .content .nascimento {
  font-size: 12px;
  color: #6b4e0a; /* tom marrom */
  font-weight: 700;
  margin-top: 3px;
}

.card .content .meta {
  font-size: 11px;
  color: #770000; /* vermelho suave */
  margin-top: 3px;
  line-height: 1.2;
}

.card .content .meta b {
  color: #b22222; /* vermelho forte no código */
}

.qr {
  position: absolute;
  bottom: 6px;
  right: 6px;
  width: 1.8cm;
  height: 1.8cm;
  background: #fff;
  border-radius: 4px;
  padding: 2px;
}

.select-photo-btn {
  margin-top: 4px;
  background: #b22222; /* botão vermelho */
  color: #fff;
  border: none;
  padding: 6px 10px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 12px;
  transition: background 0.3s ease;
}
.select-photo-btn:hover {
  background: #8b0000; /* vermelho mais escuro no hover */
}

@media print {
  .controls,
  .select-photo-btn,
  .header-actions {
    display: none;
  }
  .card {
    break-inside: avoid-page;
  }
}
</style>
</head>
<body>

<!-- LOGIN -->
<div id="loginScreen">
  <div class="login-box">
    <h2>Entrar no Sistema</h2>
    <div id="errorMsg"></div>
    <input id="loginUser" type="text" placeholder="Usuário">
    <input id="loginPass" type="password" placeholder="Senha">
    <button id="loginBtn">Entrar</button>
  </div>
</div>

<!-- APP -->
<div id="appScreen">
  <header>
    <h1>Carteirinhas Inteligente — válido até 31/07/2026 </h1>
    <button id="logoutBtn">Sair</button>
  </header>

  <div class="container">
    <div class="controls">
      <div class="control">
        <label class="small">Upload (CSV ou XLSX)</label><br>
        <input id="fileInput" type="file" accept=".csv,.xlsx,.xls">
      </div>
      <div class="control">
        <label class="small">Corporação</label><br>
        <input id="corpName" class="input" placeholder="Ex: Banda Municipal"><br><br>
        <label class="small">Logotipo</label><br>
        <input id="corpLogoBottom" type="file" accept="image/*">
      </div>
      <div class="control">
        <button id="generateBtn" class="btn">Gerar Carteirinhas</button>
        <button id="printBtn" class="btn" style="background:#10b981">Imprimir / PDF</button>
      </div>
    </div>
    <div id="cards" class="cards-grid"></div>
  </div>
</div>

<script>
// Login
loginBtn.onclick = () => {
  const u = loginUser.value.trim();
  const p = loginPass.value.trim();
  if (u === "FMBF" && p === "2143") {
    loginScreen.style.display = "none";
    appScreen.style.display = "block";
  } else errorMsg.textContent = "Usuário ou senha incorretos.";
};
logoutBtn.onclick = () => {
  loginScreen.style.display = "flex";
  appScreen.style.display = "none";
};

// Utilitários
function stripCpf(v){
  const dig = String(v||"").replace(/\D/g,"");
  return dig.length===11 ? dig.replace(/(\d{3})(\d{3})(\d{3})(\d{2})/, "$1.$2.$3-$4") : "";
}

// Função para formatar data no formato dd/mm/yyyy
function formatDateBR(d){
  if(!(d instanceof Date) || isNaN(d)) return null;
  const day = String(d.getDate()).padStart(2,"0");
  const month = String(d.getMonth()+1).padStart(2,"0");
  const year = d.getFullYear();
  return `${day}/${month}/${year}`;
}

// Função que tenta converter valor qualquer em Date
function tryParseDate(value){
  if(!value) return null;

  if(value instanceof Date) return value;

  if(typeof value === "number"){
    // Excel serial date -> JS date
    const date = new Date((value - 25569) * 86400 * 1000);
    if(!isNaN(date)) return date;
  }

  if(typeof value === "string"){
    const s = value.trim();

    // Detecta formato dd/mm/yyyy
    const ddmmyyyy = /^(\d{2})\/(\d{2})\/(\d{4})$/;
    const yyyymmdd = /^(\d{4})-(\d{2})-(\d{2})$/;

    let match;

    if(match = s.match(ddmmyyyy)){
      const [_, day, month, year] = match;
      return new Date(`${year}-${month}-${day}`);
    } else if(match = s.match(yyyymmdd)){
      return new Date(s);
    } else {
      // Tenta converter direto
      const d = new Date(s);
      if(!isNaN(d)) return d;
    }
  }

  return null; // não conseguiu converter
}

// Detecção automática de campos
function detectarCampos(row){
  let nome="", cpf="", nascRaw=null, nascDate=null;

  const keys = Object.keys(row);

  // Procura campo de nascimento pelo nome da coluna
  for(let k of keys){
    const val = row[k];
    const lowerKey = k.toLowerCase();

    if(nascRaw === null && (lowerKey.includes("nasc") || lowerKey.includes("data") || lowerKey.includes("birth") || lowerKey.includes("dt"))){
      nascRaw = val;
      nascDate = tryParseDate(val);
    }
  }

  // Se não achou pela coluna, tenta no conteúdo
  if(nascRaw === null){
    for(const val of Object.values(row)){
      const s = String(val).trim();
      if(nascRaw === null && (/\d{2}\/\d{2}\/\d{4}/.test(s) || /\d{4}-\d{2}-\d{2}/.test(s) || /^\d{8}$/.test(s))){
        nascRaw = val;
        nascDate = tryParseDate(val);
      }
    }
  }

  // Nome (2+ palavras)
  for(const val of Object.values(row)){
    const s = String(val).trim();
    if(!nome && /[a-zA-Z\u00C0-\u017F]/.test(s) && s.split(" ").length>=2)
      nome = s;
  }

  // CPF
  for(const val of Object.values(row)){
    const s = String(val).trim();
    const onlyDigits = s.replace(/\D/g,"");
    if(!cpf && onlyDigits.length===11){
      cpf = stripCpf(s);
      break;
    }
  }

  return {nome, cpf, nascRaw, nascDate};
}

// Leitura dos arquivos
let rows=[],logo=null;
fileInput.onchange = async e=>{
  const f = e.target.files[0]; if(!f) return;
  if(f.name.endsWith(".csv")){
    const text = await f.text();
    const parsed = Papa.parse(text,{header:true,skipEmptyLines:true});
    rows = parsed.data;
  } else {
    const buf = await f.arrayBuffer();
    const wb = XLSX.read(buf,{type:"array"});
    const ws = wb.Sheets[wb.SheetNames[0]];
    rows = XLSX.utils.sheet_to_json(ws,{defval:""});
  }
  alert("Arquivo carregado! Clique em 'Gerar Carteirinhas'.");
};

corpLogoBottom.onchange = e=>{
  const f=e.target.files[0]; if(!f) return;
  const r=new FileReader();
  r.onload=ev=>logo=ev.target.result;
  r.readAsDataURL(f);
};

// Geração das carteirinhas
function gerar(){
  cards.innerHTML="";
  const corp = corpName.value.trim()||"Organização";
  rows.forEach((r,i)=>{
    const {nome,cpf,nascRaw,nascDate} = detectarCampos(r);
    if(!nome) return;
    const codigo = corp.substring(0,4).toUpperCase()+"-"+String(i+1).padStart(4,"0");
    const card=document.createElement("div"); card.className="card";
    if(logo){const img=document.createElement("img");img.src=logo;img.className="logo-bottom";card.appendChild(img);}
    const foto=document.createElement("img");foto.className="foto";card.appendChild(foto);
    const btn=document.createElement("button");btn.textContent="Selecionar Foto";btn.className="select-photo-btn";
    btn.onclick=()=>{const inp=document.createElement("input");inp.type="file";inp.accept="image/*";inp.onchange=ev=>{
      const fl=ev.target.files[0];if(fl){const rr=new FileReader();rr.onload=x=>foto.src=x.target.result;rr.readAsDataURL(fl);}};inp.click();};
    card.appendChild(btn);
    const d=document.createElement("div");
    d.className="content";
    const nascimentoExibir = formatDateBR(nascDate) || nascRaw || "-";
    d.innerHTML=`<div class='nome'>${nome}</div>
    <div class='nascimento'>Nascimento: ${nascimentoExibir}</div>
    <div class='meta'>CPF: ${cpf||"-"}<br>${corp}<br><b>${codigo}</b></div>`;
    card.appendChild(d);
    const qr=document.createElement("canvas");qr.className="qr";
    new QRious({
      element:qr,
      value:`${nome}\nCPF: ${cpf}\nNascimento: ${nascimentoExibir}\n${corp}\n${codigo}`,
      size:90
    });
    card.appendChild(qr);
    cards.appendChild(card);
  });
}

generateBtn.onclick=gerar;
printBtn.onclick=()=>window.print();
</script>
</body>
</html>
