# Base de Conhecimento — Aprendizados do AmbuCheck

> Referencia para criacao de novos aplicativos web.
> Extraido do projeto AmbuCheck (SPA com Firebase, dark theme, multi-tenant).

---

## 1. ARQUITETURA

### 1.1 SPA (Single Page Application) sem framework

**Padrao usado:**
```
index.html (unico arquivo)
├── <script type="module">  → Firebase SDK (ESM imports)
├── <style>                 → CSS completo com variaveis
├── HTML                    → Todas as paginas como <div class="page">
└── <script>                → Logica JS vanilla
```

**Vantagens comprovadas:**
- Deploy instantaneo (1 arquivo, GitHub Pages, Firebase Hosting)
- Zero build tools, zero dependencias locais
- Funciona offline com localStorage

**Problemas encontrados:**
- Arquivo ficou com 2200+ linhas — dificulta manutencao
- Codigo duplicado (auto-login apareceu 2x)
- Sem separacao de responsabilidades

**Recomendacao para novos apps:**
- Ate ~800 linhas: arquivo unico OK
- 800-2000 linhas: separar em `style.css` + `app.js` + `index.html`
- 2000+ linhas: considerar modulos ES6 (`auth.js`, `checklist.js`, `stock.js`)

### 1.2 Navegacao SPA

```javascript
// Padrao: paginas como divs, mostrar/esconder com classe
.page { display: none }
.page.active { display: block }

// Paineis dentro da pagina app
.panel { display: none }
.panel.active { display: block }

// Funcao de navegacao
function showPanel(id) {
  document.querySelectorAll('.panel').forEach(p => p.classList.remove('active'));
  document.getElementById('panel-' + id).classList.add('active');
}
```

### 1.3 Firebase — Setup Client-Side

```javascript
// Padrao: importar via CDN ESM, expor para script nao-modular
import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js';
import { getAuth, signInWithEmailAndPassword, ... } from '.../firebase-auth.js';
import { getFirestore, doc, getDoc, setDoc, ... } from '.../firebase-firestore.js';

const app = initializeApp(firebaseConfig);
window._fbAuth = getAuth(app);
window._fbDb = getFirestore(app);
window._fbFns = { signInWithEmailAndPassword, doc, getDoc, setDoc, ... };
window._fbReady = true;
window.dispatchEvent(new Event('firebase-ready'));
```

**Licao aprendida:** Nunca confiar que Firebase estara pronto no momento do uso.
Sempre checar `window._fbReady` antes de chamar funcoes Firebase.

---

## 2. DESIGN SYSTEM (Dark Theme)

### 2.1 Paleta de Cores (CSS Variables)

```css
:root {
  /* Cor primaria (identidade) */
  --primary: #E02020;
  --primary-dark: #B01010;
  --primary-light: #FF4040;

  /* Backgrounds (do mais escuro ao mais claro) */
  --dark:   #080A0F;   /* body background */
  --dark2:  #0F1219;   /* cards, sidebar */
  --dark3:  #161B26;   /* headers de card, inputs bg */
  --dark4:  #1E2536;   /* hover states */

  /* Textos e bordas */
  --border: #232B3E;
  --text:   #F0F2F8;   /* texto principal */
  --muted:  #8892A4;   /* texto secundario */
  --muted2: #50596A;   /* placeholders, separadores */

  /* Semanticas */
  --green:  #22C55E;   /* sucesso, OK */
  --green2: #15803D;   /* botoes verdes */
  --yellow: #F5A623;   /* alerta, pendente */
  --blue:   #2563EB;   /* informacao */
  --purple: #7C3AED;   /* especial, premium */
}
```

### 2.2 Tipografia

```css
/* Titulos: fonte display bold */
font-family: 'Syne', sans-serif;
font-weight: 700 | 800;

/* Corpo: fonte legivel */
font-family: 'DM Sans', sans-serif;
font-weight: 300 | 400 | 500 | 600;

/* Google Fonts import */
@import url('https://fonts.googleapis.com/css2?family=Syne:wght@700;800&family=DM+Sans:wght@300;400;500;600&display=swap');
```

### 2.3 Componentes Reutilizaveis

#### Card
```css
.card {
  background: var(--dark2);
  border: 1px solid var(--border);
  border-radius: 12px;
  margin-bottom: 16px;
  overflow: hidden;
}
.card-header {
  background: var(--dark3);
  padding: 12px 18px;
  display: flex;
  align-items: center;
  gap: 9px;
  border-bottom: 1px solid var(--border);
}
.card-body {
  padding: 16px 18px;
  display: flex;
  flex-direction: column;
  gap: 13px;
}
```

#### Input
```css
.input {
  width: 100%;
  background: #0A0C12;
  border: 1px solid var(--border);
  border-radius: 7px;
  color: var(--text);
  font-family: 'DM Sans', sans-serif;
  font-size: 0.87rem;
  padding: 9px 12px;
  outline: none;
  transition: border-color 0.2s;
}
.input:focus { border-color: var(--primary) }
.input::placeholder { color: var(--muted2) }
```

#### Badge
```css
.badge {
  display: inline-block;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 0.68rem;
  font-weight: 700;
}
.badge-ok    { background: rgba(34,197,94,0.12);  color: var(--green);  border: 1px solid rgba(34,197,94,0.2) }
.badge-error { background: rgba(224,32,32,0.12);  color: #FF7070;      border: 1px solid rgba(224,32,32,0.2) }
.badge-info  { background: rgba(37,99,235,0.12);  color: var(--blue);   border: 1px solid rgba(37,99,235,0.2) }
.badge-warn  { background: rgba(245,166,35,0.12); color: var(--yellow); border: 1px solid rgba(245,166,35,0.2) }
```

#### Botao
```css
.btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 9px 18px;
  border-radius: 8px;
  font-family: 'DM Sans', sans-serif;
  font-size: 0.85rem;
  font-weight: 600;
  cursor: pointer;
  border: none;
  transition: all 0.2s;
}
.btn-primary { background: var(--primary); color: white }
.btn-ghost   { background: transparent; border: 1px solid var(--border); color: var(--muted) }
.btn-danger  { background: rgba(224,32,32,0.1); color: #FF7070; border: 1px solid rgba(224,32,32,0.2) }
```

#### Stat Card (Dashboard)
```css
.stat {
  background: var(--dark2);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 16px;
  position: relative;
  overflow: hidden;
}
.stat::before {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0;
  height: 2px;
  background: var(--primary); /* cor muda por tipo */
}
```

#### Toast (Notificacao)
```css
.toast {
  position: fixed;
  bottom: 24px; right: 24px;
  background: var(--dark3);
  border: 1px solid var(--border);
  border-radius: 10px;
  padding: 12px 18px;
  box-shadow: 0 8px 28px rgba(0,0,0,0.5);
  z-index: 500;
  display: none;
}
.toast.show {
  display: flex;
  animation: slideIn 0.22s ease;
}
```

#### Modal
```css
.modal-overlay {
  display: none;
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.8);
  z-index: 300;
  align-items: center;
  justify-content: center;
  backdrop-filter: blur(3px);
}
.modal-overlay.show { display: flex }
.modal-box {
  background: var(--dark3);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 28px;
  width: 90%; max-width: 460px;
  max-height: 90vh;
  overflow-y: auto;
  animation: popIn 0.22s cubic-bezier(.175,.885,.32,1.275);
}
```

### 2.4 Layout Responsivo

```css
/* Desktop: sidebar fixa + conteudo a direita */
.app-wrap { display: flex; min-height: 100vh }
.sidebar  { width: 232px; position: fixed; top: 0; bottom: 0; left: 0 }
.app-main { margin-left: 232px; flex: 1 }

/* Mobile: sidebar esconde, bottom nav aparece */
@media (max-width: 768px) {
  .sidebar    { transform: translateX(-100%); transition: transform 0.25s }
  .sidebar.open { transform: translateX(0) }
  .app-main   { margin-left: 0; padding-bottom: 70px }
  .bottom-nav { display: flex; position: fixed; bottom: 0; left: 0; right: 0; height: 60px }
}
@media (min-width: 769px) {
  .bottom-nav { display: none }
}
```

### 2.5 Grids Responsivos
```css
.grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 12px }
.grid-3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 12px }
.stats  { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px }

@media (max-width: 768px) {
  .grid-2, .grid-3 { grid-template-columns: 1fr }
  .stats { grid-template-columns: 1fr 1fr }
}
```

---

## 3. AUTENTICACAO & MULTI-TENANT

### 3.1 Estrutura Multi-Empresa

```
companies (array no localStorage, collection no Firestore)
├── company_1
│   ├── users[]        → todos os usuarios desta empresa
│   ├── ambs[]         → ambulancias
│   ├── checklists[]   → registros
│   ├── stock[]        → estoque
│   ├── config{}       → nome, RT principal, URL drive
│   └── ...
├── company_2
│   └── ...
```

**Padrao de acesso isolado por empresa:**
```javascript
const cid = () => currentUser.companyId;
const getUsers = () => DB.getC(cid(), 'users') || [];
const setUsers = (v) => DB.setC(cid(), 'users', v);
// Cada empresa so acessa seus proprios dados
```

### 3.2 Roles (Papeis)

```javascript
const ROLES = {
  superadmin:    { label: 'Super Admin',  pill: 'rp-super',  access: ['*'] },
  Administrador: { label: 'Admin',        pill: 'rp-admin',  access: ['dashboard','checklist','aprovacoes','alteracoes','estoque','usuarios','ambulancias','config'] },
  RT:            { label: 'Responsavel Tecnico', pill: 'rp-rt', access: ['dashboard','checklist','aprovacoes','alteracoes','estoque','usuarios','ambulancias','config'] },
  Condutor:      { label: 'Condutor',     pill: 'rp-func',   access: ['dashboard','checklist','sugerir','alteracoes','estoque'] },
  Tecnico:       { label: 'Tecnico',      pill: 'rp-func',   access: ['dashboard','checklist','sugerir','alteracoes','estoque'] },
};
```

**Licao:** No AmbuCheck, o controle de acesso era apenas visual (escondia menu).
Em novos apps, validar role tambem no backend/Firestore Rules.

### 3.3 Fluxo de Auth

```
1. Tentar Firebase Auth (signInWithEmailAndPassword)
2. Se falhar → fallback localStorage (busca em todas as empresas)
3. loginOk() → salva sessao SEM senha
4. Auto-login: checa sessao salva ao carregar
```

### 3.4 Planos SaaS

```javascript
const PLANS = [
  { id: 'basico',  name: 'Basico',  price: 49,  maxAmbs: 2,        desc: 'ate 2 ambulancias' },
  { id: 'padrao',  name: 'Padrao',  price: 129, maxAmbs: 6,        desc: 'ate 6 ambulancias' },
  { id: 'premium', name: 'Premium', price: 249, maxAmbs: Infinity,  desc: 'ilimitado' },
];
```

---

## 4. PERSISTENCIA DE DADOS

### 4.1 Padrao: localStorage + Firebase Sync

```javascript
const DB = {
  get(key) {
    try {
      const raw = localStorage.getItem('prefix_' + key);
      return raw ? JSON.parse(raw) : null;
    } catch { return null }
  },
  set(key, value) {
    try {
      localStorage.setItem('prefix_' + key, JSON.stringify(value));
    } catch (e) {
      if (e.name === 'QuotaExceededError') {
        alert('Armazenamento local cheio');
      }
    }
  },
  // Versao com sync Firestore
  setC(companyId, key, value) {
    this.set('co_' + companyId + '_' + key, value);
    // Background sync
    if (window._fbReady) {
      setDoc(doc(db, 'companies', companyId, key, 'data'),
        { value: JSON.stringify(value), updated: Date.now() },
        { merge: true }
      ).catch(e => console.warn('Sync:', e));
    }
  }
};
```

**Licao:** localStorage tem limite de ~5-10MB. Para apps com muitos dados (200+ itens de estoque x multiplas empresas), priorizar Firestore como fonte principal.

### 4.2 Padrao CRUD

```javascript
// GET (com fallback)
const getItems = () => DB.getC(companyId, 'items') || [];

// CREATE
function addItem(item) {
  const items = getItems();
  items.unshift({ id: generateId(), ...item, criado: new Date().toLocaleString('pt-BR') });
  setItems(items);
}

// UPDATE
function updateItem(id, changes) {
  const items = getItems();
  const idx = items.findIndex(i => i.id === id);
  if (idx >= 0) items[idx] = { ...items[idx], ...changes };
  setItems(items);
}

// DELETE
function removeItem(id) {
  setItems(getItems().filter(i => i.id !== id));
}
```

### 4.3 IDs Unicos

```javascript
const generateId = () =>
  Date.now().toString(36).toUpperCase() +
  Math.random().toString(36).slice(2, 4).toUpperCase();
// Resultado: "M5X2HKAB"
```

---

## 5. PADROES DE UI/UX

### 5.1 Formulario com Radio Buttons Estilizados

```html
<label class="radio-btn">
  <input type="radio" name="grupo" value="opcao1" onchange="update()">
  <span>Opcao 1</span>
</label>
```
```css
.radio-btn input { display: none }
.radio-btn span {
  display: inline-block;
  padding: 6px 13px;
  border-radius: 6px;
  border: 1px solid var(--border);
  background: #0A0C12;
  color: var(--muted);
  cursor: pointer;
  transition: all 0.15s;
}
.radio-btn input:checked + span {
  background: var(--primary);
  border-color: var(--primary);
  color: white;
}
```

### 5.2 Barra de Progresso

```html
<div class="progress-row">
  <span class="progress-label">Progresso</span>
  <div class="progress-bar">
    <div class="progress-fill" id="fill" style="width: 0%"></div>
  </div>
  <span class="progress-count" id="count">0/0</span>
</div>
```

### 5.3 Tabela com Acoes

```html
<div class="table-wrap">
  <table>
    <thead><tr><th>Nome</th><th>Status</th><th>Acoes</th></tr></thead>
    <tbody id="tbody"></tbody>
  </table>
</div>
```
```javascript
function renderTable(data) {
  const tbody = document.getElementById('tbody');
  if (!data.length) {
    tbody.innerHTML = '<tr><td colspan="3" class="empty">Nenhum registro.</td></tr>';
    return;
  }
  tbody.innerHTML = data.map(item => `<tr>
    <td>${esc(item.name)}</td>
    <td><span class="badge badge-ok">${esc(item.status)}</span></td>
    <td><button class="btn btn-danger btn-sm" onclick="remove('${esc(item.id)}')">Remover</button></td>
  </tr>`).join('');
}
```

### 5.4 Fluxo de Aprovacao

```
Funcionario sugere item
    ↓
Sugestao fica "pendente"
    ↓
RT/Admin vê na tela de aprovacoes (badge com contagem)
    ↓
Abre modal → Aprovar / Rejeitar (com observacao)
    ↓
Se aprovado → item adicionado automaticamente ao checklist e/ou estoque
```

### 5.5 Controle de Estoque com Alerta

```javascript
// Cada item tem: qty (atual) e min (minimo)
const lowStock = stock.filter(item => item.qty <= item.min);

// Exibir alerta se houver itens baixos
if (lowStock.length) {
  showAlert(`${lowStock.length} item(ns) com estoque baixo`);
}

// Grid de estoque com botoes +/-
function changeQty(id, delta) {
  const item = stock.find(i => i.id === id);
  item.qty = Math.max(0, item.qty + delta);
  saveStock(stock);
  renderStock();
}
```

---

## 6. SEGURANCA — LICOES APRENDIDAS

### 6.1 OBRIGATORIO em todo app

```javascript
// 1. Sanitizar TODA saida HTML (previne XSS)
function esc(s) {
  if (s == null) return '';
  return String(s)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;');
}

// 2. Anti-spam em formularios
const locks = {};
function lockSubmit(key, ms) {
  if (locks[key]) return false;
  locks[key] = true;
  setTimeout(() => { locks[key] = false }, ms || 2500);
  return true;
}

// 3. Validar URLs externas
function isValidUrl(url, allowedDomain) {
  try {
    const u = new URL(url);
    return u.protocol === 'https:' && u.hostname.endsWith(allowedDomain);
  } catch { return false }
}

// 4. Hash de senhas (nunca armazenar texto plano)
async function hashPassword(password) {
  const enc = new TextEncoder().encode(password);
  const buf = await crypto.subtle.digest('SHA-256', enc);
  return Array.from(new Uint8Array(buf)).map(b => b.toString(16).padStart(2, '0')).join('');
}

// 5. Sessao segura (nunca armazenar senha)
function saveSession(user) {
  const safe = { id: user.id, email: user.email, nome: user.nome, role: user.role };
  localStorage.setItem('session', JSON.stringify(safe));
}
```

### 6.2 Erros que foram cometidos (nao repetir)

| Erro | Consequencia | Correcao |
|------|-------------|----------|
| Senha do admin no codigo-fonte | Qualquer pessoa ve a senha | Usar hash SHA-256 |
| innerHTML com dados do usuario | XSS — roubo de sessao | Sempre usar `esc()` |
| Senha salva no localStorage | Exposta pelo DevTools | Salvar apenas id/email/role |
| Auto-login duplicado | Executa 2x, potencial conflito | Remover duplicatas |
| Sem rate-limiting | Spam de registros | lockSubmit() em todo formulario |
| URL do Drive sem validacao | Redirecionamento malicioso | Whitelist de dominio |
| Sem validacao de email | Registros com "aaa" como email | Regex de validacao |
| Sem limites de tamanho | Payload gigante no storage | maxLength em campos |

### 6.3 Firestore Security Rules (modelo)

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Usuarios so leem seus proprios dados
    match /users/{userId} {
      allow read, write: if request.auth.uid == userId;
    }
    // Dados da empresa: apenas membros
    match /companies/{companyId}/{document=**} {
      allow read, write: if request.auth != null
        && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.companyId == companyId;
    }
  }
}
```

---

## 7. PADROES DE DADOS (SCHEMAS)

### 7.1 Usuario
```json
{
  "id": "u_M5X2HK",
  "nome": "Leonardo",
  "email": "leo@empresa.com",
  "role": "Administrador",
  "companyId": "co_ABC123",
  "criado": "16/04/2026, 14:30:00"
}
```

### 7.2 Empresa
```json
{
  "id": "co_ABC123",
  "nome": "SAMU Regional",
  "plano": "Padrao",
  "criado": "16/04/2026, 10:00:00"
}
```

### 7.3 Item de Estoque
```json
{
  "id": "epinefrina",
  "name": "Epinefrina 1mg/ml",
  "cat": "Medicamento",
  "qty": 20,
  "min": 3,
  "unit": "amp"
}
```

### 7.4 Checklist
```json
{
  "id": "CHK-M5X2HK",
  "dt": "16/04/2026, 07:15:00",
  "condutor": "Carlos",
  "amb": "SA-01 (JBV 5I90)",
  "turno": "07:00 MANHA",
  "combustivel": "COMPLETO",
  "m_agua": "NORMAL",
  "m_oleo": "NORMAL",
  "hodometro": "45740",
  "cil01": "150 kgf",
  "pneus": "SIM",
  "intercorrencias": ""
}
```

### 7.5 Sugestao (Workflow de Aprovacao)
```json
{
  "id": "SUG-M5X2HK",
  "nome": "Desfibrilador",
  "categoria": "Equipamento",
  "tipo": "checklist",
  "justificativa": "Necessario para atendimento cardiaco",
  "solicitante": "Carlos",
  "status": "pendente | aprovado | rejeitado",
  "obsRT": "Aprovado conforme protocolo",
  "dtSolicitacao": "16/04/2026",
  "dtDecisao": "16/04/2026"
}
```

### 7.6 Alteracao/Manutencao
```json
{
  "id": "ALT-M5X2HK",
  "amb": "SA-01 (JBV 5I90)",
  "tipo": "Mecanica | Eletrica | Equip. Medico | Carroceria | Acidente",
  "grav": "Leve | Moderada | Grave",
  "desc": "Freio traseiro com folga",
  "resp": "Oficina Central",
  "status": "Pendente | Em Andamento | Resolvido",
  "custo": "350.00"
}
```

---

## 8. INTEGRACOES

### 8.1 Google Apps Script (Exportar para Drive)

```javascript
async function sendToDrive(payload) {
  const url = config.gas_url;
  if (!url || !isValidUrl(url, 'script.google.com')) return;
  try {
    await fetch(url, {
      method: 'POST',
      body: JSON.stringify(payload),
      headers: { 'Content-Type': 'application/json' }
    });
  } catch (e) {
    console.warn('Erro Drive:', e.message);
  }
}
```

### 8.2 PWA (Progressive Web App)

```html
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="NomeDoApp">
<meta name="theme-color" content="#E02020">
```

---

## 9. CHECKLIST PARA NOVOS APPS

### Antes de comecar
- [ ] Definir roles e permissoes
- [ ] Definir se e multi-tenant (varias empresas)
- [ ] Escolher persistencia (localStorage, Firebase, Supabase, etc)
- [ ] Definir paleta de cores (adaptar as variaveis CSS acima)

### Estrutura minima
- [ ] Tela de login/registro
- [ ] Dashboard com stats
- [ ] Sidebar + topbar (ou bottom nav mobile)
- [ ] Sistema de toast/notificacoes
- [ ] Modais para acoes

### Seguranca (obrigatorio)
- [ ] Funcao `esc()` para sanitizar HTML
- [ ] `lockSubmit()` em todos os formularios
- [ ] Validacao de email com regex
- [ ] Limites de tamanho em todos os campos
- [ ] Sessao sem dados sensiveis
- [ ] Senhas nunca em texto plano
- [ ] URLs externas validadas por whitelist
- [ ] Firestore Rules configuradas

### Qualidade
- [ ] Nenhum codigo duplicado
- [ ] Tratamento de erros visivel ao usuario
- [ ] Funciona offline (localStorage)
- [ ] Responsivo (mobile + desktop)
- [ ] Dados sanitizados antes de exibir

---

## 10. SNIPPETS PRONTOS PARA COPIAR

### Helper: ID unico
```javascript
const uid = () => Date.now().toString(36).toUpperCase() + Math.random().toString(36).slice(2,4).toUpperCase();
```

### Helper: Data/Hora formatada (pt-BR)
```javascript
const nowStr  = () => new Date().toLocaleString('pt-BR');
const nowDate = () => new Date().toISOString().slice(0,10);
const nowTime = () => new Date().toTimeString().slice(0,5);
```

### Helper: Query selectors curtos
```javascript
const qs  = s => document.querySelector(s);
const qid = id => document.getElementById(id);
const gR  = name => { const e = qs(`input[name="${name}"]:checked`); return e ? e.value : '' };
```

### Helper: Toast
```javascript
function showToast(msg, duration = 3200) {
  const t = document.getElementById('toast');
  document.getElementById('toast-msg').textContent = msg;
  t.classList.add('show');
  clearTimeout(t._t);
  t._t = setTimeout(() => t.classList.remove('show'), duration);
}
```

### Helper: Modal
```javascript
function openModal(id)  { document.getElementById(id).classList.add('show') }
function closeModal(id) { document.getElementById(id).classList.remove('show') }
```
