<!DOCTYPE html>
<html lang="fr" dir="ltr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Gestion Prod V16 - Delete Mode</title>
    <style>
        :root {
            --primary: #2c3e50;
            --secondary: #3498db;
            --bg: #f0f2f5;
            --white: #ffffff;
            --green: #27ae60;
            --red: #e74c3c;
            --orange: #f39c12;
            --text: #333;
        }

        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin: 0; background-color: var(--bg); color: var(--text); -webkit-tap-highlight-color: transparent; padding-bottom: 80px; }

        /* Status Bar */
        #status-bar {
            background-color: #7f8c8d; color: white;
            text-align: center; font-size: 0.8rem; padding: 4px;
            position: sticky; top: 0; z-index: 200;
        }

        /* --- NAVIGATION --- */
        nav { background-color: var(--primary); padding: 10px 15px; box-shadow: 0 2px 5px rgba(0,0,0,0.2); position: sticky; top: 25px; z-index: 100; }
        .nav-container { display: flex; justify-content: space-between; align-items: center; max-width: 1000px; margin: 0 auto; }
        nav h1 { margin: 0; font-size: 1.1rem; color: white; white-space: nowrap; }
        
        .nav-links { display: flex; gap: 5px; overflow-x: auto; -webkit-overflow-scrolling: touch; padding-bottom: 5px; margin-top: 5px; align-items: center; }
        .nav-links::-webkit-scrollbar { display: none; }
        
        .nav-btn {
            background: rgba(255,255,255,0.15); border: none; color: #ecf0f1; cursor: pointer; padding: 8px 12px; 
            border-radius: 20px; font-size: 0.85rem; position: relative; transition: 0.3s; white-space: nowrap; flex-shrink: 0;
            display: none; 
        }
        .nav-btn.visible { display: block; }
        .nav-btn:hover, .nav-btn.active { background: var(--secondary); color: white; }

        .btn-login { background: var(--green) !important; color: white !important; display: block; }
        .btn-logout { background: var(--red) !important; color: white !important; }

        .badge {
            position: absolute; top: -3px; right: -3px;
            color: white; font-size: 0.65rem; font-weight: bold;
            padding: 2px 5px; border-radius: 50%;
            border: 2px solid var(--primary); display: none;
        }
        .badge-red { background-color: var(--red); }
        .badge-orange { background-color: var(--orange); }

        /* --- LAYOUT --- */
        .container { max-width: 1000px; margin: 15px auto; padding: 0 10px; }
        .section { display: none; animation: fadeIn 0.3s; }
        .section.active { display: block; }
        @keyframes fadeIn { from{opacity:0; transform:translateY(5px);} to{opacity:1; transform:translateY(0);} }

        .card { background: var(--white); padding: 20px; border-radius: 12px; box-shadow: 0 2px 8px rgba(0,0,0,0.08); margin-bottom: 15px; }
        h2 { border-bottom: 3px solid var(--secondary); padding-bottom: 8px; margin-top: 0; color: var(--primary); font-size: 1.3rem; }
        h3 { margin-top:0; color: #555; font-size: 1.1rem; }
        
        .form-row { display: flex; gap: 15px; margin-bottom: 15px; }
        .col { flex: 1; }
        
        @media (max-width: 768px) {
            .nav-container { flex-direction: column; align-items: flex-start; }
            .nav-links { width: 100%; margin-top: 10px; }
            .form-row { flex-direction: column; gap: 10px; }
            .btn { width: 100%; padding: 15px; font-size: 1.1rem; }
            .check-item input[type="checkbox"] { transform: scale(1.3); }
        }

        label { display: block; font-weight: 600; margin-bottom: 5px; color: #555; font-size: 0.9rem; }
        select, input { width: 100%; padding: 12px; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; font-size: 16px; background: #fff; }
        
        .btn { padding: 12px 20px; border: none; border-radius: 8px; cursor: pointer; color: white; font-weight: bold; font-size: 1rem; transition: 0.2s; }
        .btn:active { transform: scale(0.98); }
        .btn-blue { background-color: var(--secondary); }
        .btn-green { background-color: var(--green); }
        .btn-red { background-color: var(--red); }
        .btn-orange { background-color: var(--orange); }
        .btn-purple { background-color: #8e44ad; }
        .btn-sm { padding: 8px 15px; font-size: 0.85rem; width: auto; }

        /* Checklist */
        .checklist-box { background: #fafafa; border: 1px solid #eee; padding: 10px; border-radius: 8px; max-height: 300px; overflow-y: auto; }
        .check-item { display: flex; align-items: center; padding: 12px 5px; border-bottom: 1px solid #eee; background: white; margin-bottom: 5px; border-radius: 4px; }
        .check-item input[type="checkbox"] { width: 22px; height: 22px; margin-right: 15px; }
        .check-item input[type="number"] { width: 130px; margin-left: auto; text-align: center; font-size: 1.1rem; font-weight: bold; }

        /* Table & Misc */
        .table-wrap { overflow-x: auto; }
        table { width: 100%; border-collapse: collapse; min-width: 350px; }
        th, td { border: 1px solid #ddd; padding: 10px; text-align: left; font-size: 0.9rem; vertical-align: middle; }
        th { background: #eee; white-space: nowrap; }
        .warning-box { background-color: #fff3cd; color: #856404; border: 1px solid #ffeeba; border-radius: 6px; padding: 10px; margin: 10px 0; display: flex; align-items: center; gap: 10px; }
        .info-pill { font-size:0.8rem; background:#eef2f7; padding:4px 8px; border-radius:4px; display:inline-block; margin-right:5px; border:1px solid #cbd5e0;}

        /* Delete Mode Toolbar */
        #delete-toolbar {
            background-color: #e74c3c; color: white; padding: 15px; border-radius: 8px; margin-bottom: 15px;
            display: none; animation: fadeIn 0.3s;
        }
        #delete-toolbar h3 { color: white; border-bottom: 1px solid rgba(255,255,255,0.3); margin-bottom: 10px; }
        .del-checkbox { width: 25px; height: 25px; margin-right: 15px; cursor: pointer; }

        /* Calc & Login */
        .fab-calc { position: fixed; bottom: 20px; right: 20px; background-color: var(--primary); color: white; width: 60px; height: 60px; border-radius: 50%; display: flex; align-items: center; justify-content: center; box-shadow: 0 4px 15px rgba(0,0,0,0.3); cursor: pointer; z-index: 999; font-size: 24px; }
        .calc-modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.6); z-index: 2000; justify-content: center; align-items: center; }
        .calc-body { background: #2c3e50; padding: 20px; border-radius: 15px; width: 300px; }
        .calc-display { background: #ecf0f1; padding: 15px; font-size: 1.5rem; text-align: right; border-radius: 8px; margin-bottom: 15px; color: #333; font-family: monospace; min-height: 30px; }
        .calc-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
        .calc-btn { padding: 15px; font-size: 1.2rem; border: none; border-radius: 8px; cursor: pointer; background: #34495e; color: white; }
        .calc-btn.op { background: var(--secondary); }
        .calc-btn.eq { background: var(--green); grid-column: span 2; }
        .calc-btn.clr { background: var(--red); }
        .close-calc { text-align: center; margin-top: 10px; color: #bdc3c7; cursor: pointer; text-decoration: underline; }
        
        #loginOverlay { display: flex; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); z-index: 3000; justify-content: center; align-items: center; }
        .login-box { background: white; padding: 30px; border-radius: 10px; width: 90%; max-width: 350px; text-align: center; box-shadow: 0 10px 25px rgba(0,0,0,0.5); }
        .login-box input { margin-bottom: 15px; }
        
        @media print {
            nav, .fab-calc, .no-print, .calc-modal, #loginOverlay, #status-bar, #delete-toolbar { display: none !important; }
            .section { display: block !important; }
            #home, #settings, #operations, #validation { display: none !important; }
            #history { display: block !important; }
            body { background: white; }
        }
    </style>
</head>
<body>

<div id="status-bar">Connexion au serveur...</div>

<nav>
    <div class="nav-container">
        <h1>Gestion V16 (Online)</h1>
        <div class="nav-links">
            <button id="btnHome" class="nav-btn protected" onclick="nav('home')">Produire</button>
            <button id="btnVal" class="nav-btn protected" onclick="nav('validation')">
                Validations <span id="badgeVal" class="badge badge-orange">0</span>
            </button>
            <button id="btnOps" class="nav-btn protected" onclick="nav('operations')">
                En Cours <span id="badgeOps" class="badge badge-red">0</span>
            </button>
            <button id="btnConfig" class="nav-btn protected" onclick="nav('settings')">Config</button>
            <button id="btnHist" class="nav-btn visible active" onclick="nav('history')">Historique</button>
            <button id="btnLogin" class="nav-btn visible btn-login" onclick="showLogin()">Connexion 🔒</button>
            <button id="btnLogout" class="nav-btn protected btn-logout" onclick="logout()">Déconnexion</button>
        </div>
    </div>
</nav>

<div id="loginOverlay" style="display:none;">
    <div class="login-box">
        <h2>Authentification</h2>
        <input type="text" id="loginUser" placeholder="Nom Utilisateur">
        <input type="password" id="loginPass" placeholder="Code Secret">
        <button class="btn btn-blue" style="width:100%" onclick="attemptLogin()">Entrer</button>
        <br><br>
        <button class="btn btn-red btn-sm" onclick="closeLogin()">Annuler</button>
    </div>
</div>

<div class="fab-calc protected" style="display:none;" onclick="openCalc()" title="Calculatrice">
    <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="4" y="2" width="16" height="20" rx="2"/><line x1="8" y1="6" x2="16" y2="6"/><line x1="16" y1="14" x2="16" y2="14"/><line x1="16" y1="18" x2="16" y2="18"/><line x1="12" y1="14" x2="12" y2="14"/><line x1="12" y1="18" x2="12" y2="18"/><line x1="8" y1="14" x2="8" y2="14"/><line x1="8" y1="18" x2="8" y2="18"/></svg>
</div>

<div id="calcModal" class="calc-modal">
    <div class="calc-body">
        <div id="calcDisplay" class="calc-display">0</div>
        <div class="calc-grid">
            <button class="calc-btn clr" onclick="calcClear()">C</button>
            <button class="calc-btn op" onclick="calcInput('/')">÷</button>
            <button class="calc-btn op" onclick="calcInput('*')">×</button>
            <button class="calc-btn" onclick="calcDel()">←</button>
            <button class="calc-btn" onclick="calcInput('7')">7</button>
            <button class="calc-btn" onclick="calcInput('8')">8</button>
            <button class="calc-btn" onclick="calcInput('9')">9</button>
            <button class="calc-btn op" onclick="calcInput('-')">-</button>
            <button class="calc-btn" onclick="calcInput('4')">4</button>
            <button class="calc-btn" onclick="calcInput('5')">5</button>
            <button class="calc-btn" onclick="calcInput('6')">6</button>
            <button class="calc-btn op" onclick="calcInput('+')">+</button>
            <button class="calc-btn" onclick="calcInput('1')">1</button>
            <button class="calc-btn" onclick="calcInput('2')">2</button>
            <button class="calc-btn" onclick="calcInput('3')">3</button>
            <button class="calc-btn" onclick="calcInput('0')">0</button>
            <button class="calc-btn" onclick="calcInput('.')">.</button>
            <button class="calc-btn eq" onclick="calcResult()">=</button>
        </div>
        <div class="close-calc" onclick="closeCalc()">Fermer</div>
    </div>
</div>

<div class="container">

    <!-- SETTINGS (PROTECTED) -->
    <div id="settings" class="section">
        <div class="card">
            <h2>Paramètres (Cloud)</h2>
            <div class="form-row">
                <div class="col">
                    <h3>1. Chefs d'équipe</h3>
                    <div style="display:flex; gap:5px;">
                        <input type="text" id="setChefName" placeholder="Nom du Chef">
                        <button class="btn btn-purple" style="width:auto;" onclick="addChef()">+</button>
                    </div>
                    <ul id="listChefs" style="padding-left:20px; margin-top:10px;"></ul>
                </div>
            </div>
            <hr style="border:0; border-top:1px solid #eee;">
            <div class="form-row">
                <div class="col">
                    <h3>2. Lignes</h3>
                    <div style="display:flex; gap:5px;">
                        <input type="text" id="setLineName" placeholder="Ligne A">
                        <button class="btn btn-blue" style="width:auto;" onclick="addLine()">+</button>
                    </div>
                    <ul id="listLines" style="padding-left:20px; margin-top:10px;"></ul>
                </div>
            </div>
            <div class="form-row">
                <div class="col">
                    <h3>3. Produits</h3>
                    <div style="display:flex; flex-direction:column; gap:5px;">
                        <select id="setProdLine"><option value="">-- Choisir Ligne --</option></select>
                        <input type="text" id="setProdName" placeholder="Nom Produit">
                        <input type="text" id="setProdMsg" placeholder="Message de Rappel">
                        <button class="btn btn-blue" style="width:100%;" onclick="addProduct()">Ajouter Produit</button>
                    </div>
                    <ul id="listProducts" style="padding-left:20px; margin-top:10px;"></ul>
                </div>
            </div>
            <hr style="border:0; border-top:1px solid #eee;">
            <h3>4. Articles (Matière)</h3>
            <div class="form-row">
                <div class="col"><select id="setArtLine"></select></div>
            </div>
            <div class="form-row">
                <div class="col"><input type="text" id="setArtName" placeholder="Nom Article"></div>
                <div class="col"><select id="setArtUnit"><option value="Kg">Kg</option><option value="L">L</option><option value="Pcs">Pcs</option><option value="m">Mètre</option></select></div>
            </div>
            <button class="btn btn-green" style="width:100%" onclick="addArticle()">Enregistrer Article</button>
            <div style="margin-top:10px; max-height:200px; overflow-y:auto; border:1px solid #eee; border-radius:8px; padding:10px;">
                <ul id="listArticles" style="list-style:none; padding:0; margin:0;"></ul>
            </div>

            <!-- DANGER ZONE -->
            <hr style="border:0; border-top:1px solid #eee; margin:30px 0;">
            <h3>Zone de Danger</h3>
            <div style="background:#fff5f5; border:1px solid #fc8181; padding:15px; border-radius:8px;">
                <p style="color:#c53030; margin-top:0;">Suppression des opérations archivées.</p>
                <button class="btn btn-red" style="width:100%" onclick="goToDeleteMode()">Supprimer des Opérations</button>
            </div>
        </div>
    </div>

    <!-- HOME -->
    <div id="home" class="section">
        <div class="card">
            <h2>Démarrer Production</h2>
            <div class="form-row">
                <div class="col"><label>Ligne</label><select id="runLine" onchange="onLineChange()"></select></div>
            </div>
            <div class="form-row">
                <div class="col"><label>Produit</label><select id="runProd"></select></div>
                <div class="col"><label>SAP</label><input type="text" id="runSap" placeholder="Ex: 123456"></div>
            </div>
            <div class="form-row">
                <div class="col"><label>Chef Départ</label><select id="runChefStart"></select></div>
            </div>
            <div id="checkArea" style="display:none; margin-top:15px; border-top:1px solid #eee; padding-top:10px;">
                <label>Matériaux & Quantité :</label>
                <div class="checklist-box" id="checklist"></div>
                <br>
                <button class="btn btn-orange" style="width:100%" onclick="saveRun()">Enregistrer pour Validation</button>
            </div>
            <p id="msgNoArt" style="display:none; color:var(--red); text-align:center;">Aucun article.</p>
        </div>
    </div>

    <!-- VALIDATION -->
    <div id="validation" class="section">
        <h2>À Valider</h2>
        <div id="valList"></div>
    </div>

    <!-- OPERATIONS -->
    <div id="operations" class="section">
        <h2>En Cours</h2>
        <div id="opsList"></div>
    </div>

    <!-- HISTORY -->
    <div id="history" class="section active">
        <div class="card no-print">
            
            <!-- DELETE TOOLBAR -->
            <div id="delete-toolbar">
                <h3>Mode Suppression</h3>
                <div style="display:flex; flex-direction:column; gap:10px;">
                    <button class="btn" style="background:white; color:#e74c3c; font-weight:bold;" onclick="deleteSelected()">Supprimer Sélection</button>
                    <button class="btn" style="background:rgba(255,255,255,0.2); border:1px solid white;" onclick="deleteAllFiltered()">TOUT Supprimer (Filtre actuel)</button>
                    <button class="btn btn-blue" onclick="exitDeleteMode()">Annuler / Retour</button>
                </div>
            </div>

            <div style="display:flex; flex-direction:column; gap:10px;">
                <div style="display:flex; justify-content:space-between; align-items:center;">
                    <h2>Historique</h2>
                    <!-- Reset button removed -->
                </div>
                <input type="date" id="histDate" onchange="renderHistory()">
            </div>
        </div>
        <div id="histList"></div>
    </div>

</div>

<!-- FIREBASE SDK -->
<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
    import { getDatabase, ref, set, onValue } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-database.js";

    const firebaseConfig = {
      apiKey: "AIzaSyCIoKMtGK9rjkpm7liNPXbLfgvvuCrSJiU",
      authDomain: "project-7795305292120649521.firebaseapp.com",
      projectId: "project-7795305292120649521",
      storageBucket: "project-7795305292120649521.firebasestorage.app",
      messagingSenderId: "508930440956",
      appId: "1:508930440956:web:7662197319a96bd57a6179",
      measurementId: "G-6SZ6B7F89E",
      databaseURL: "https://project-7795305292120649521-default-rtdb.firebaseio.com"
    };

    const app = initializeApp(firebaseConfig);
    const dbRT = getDatabase(app);
    const dbRef = ref(dbRT, 'prodAppV13');

    window.db = { lines: [], products: [], articles: [], chefs: [], operations: [] };

    document.getElementById('status-bar').innerText = "Connexion...";

    onValue(dbRef, (snapshot) => {
        const data = snapshot.val();
        if (data) {
            window.db = {
                lines: data.lines || [],
                products: data.products || [],
                articles: data.articles || [],
                chefs: data.chefs || [],
                operations: data.operations || []
            };
            window.updateBadge();
            const activeSection = document.querySelector('.section.active').id;
            if(window.isAuthenticated || activeSection === 'history') {
                if(activeSection === 'home') window.initHome();
                if(activeSection === 'validation') window.renderValidation();
                if(activeSection === 'operations') window.renderOps();
                if(activeSection === 'history') window.renderHistory();
                if(activeSection === 'settings') window.renderSettings();
            }
            document.getElementById('status-bar').style.backgroundColor = "#27ae60";
            document.getElementById('status-bar').innerText = "Connecté (Synchronisé)";
        } else {
            document.getElementById('status-bar').innerText = "Connecté (Base Vide)";
            document.getElementById('status-bar').style.backgroundColor = "#f39c12";
        }
    }, (error) => {
        console.error(error);
        document.getElementById('status-bar').style.backgroundColor = "#e74c3c";
        document.getElementById('status-bar').innerText = "Erreur de connexion";
    });

    window.saveToFirebase = function() {
        set(dbRef, window.db).catch(err => alert("Erreur sauvegarde: " + err));
    }
</script>

<!-- APP LOGIC -->
<script>
    let isAuthenticated = false;
    const AUTH_USER = "Riad";
    const AUTH_PASS = "1988";
    window.isAuthenticated = false;

    // DELETE MODE STATE
    let isDeleteMode = false;

    function showLogin() { document.getElementById('loginOverlay').style.display = 'flex'; }
    function closeLogin() { document.getElementById('loginOverlay').style.display = 'none'; }
    
    function attemptLogin() {
        if(document.getElementById('loginUser').value === AUTH_USER && document.getElementById('loginPass').value === AUTH_PASS) {
            isAuthenticated = true; window.isAuthenticated = true; toggleAuthUI(); closeLogin();
            document.getElementById('loginUser').value = ''; document.getElementById('loginPass').value = '';
            nav('home');
        } else { alert("Incorrect !"); }
    }

    function logout() { isAuthenticated = false; window.isAuthenticated = false; toggleAuthUI(); nav('history'); }

    function toggleAuthUI() {
        const pBtns = document.querySelectorAll('.protected');
        const lBtn = document.getElementById('btnLogin');
        const cBtn = document.querySelector('.fab-calc');
        if(isAuthenticated) {
            pBtns.forEach(el => { el.style.display = 'block'; el.classList.add('visible'); });
            lBtn.style.display = 'none'; cBtn.style.display = 'flex';
        } else {
            pBtns.forEach(el => { el.style.display = 'none'; el.classList.remove('visible'); });
            lBtn.style.display = 'block'; cBtn.style.display = 'none';
        }
    }

    function getId() { return Date.now(); }
    function save() { if(window.saveToFirebase) window.saveToFirebase(); }

    function nav(id) {
        if(!isAuthenticated && id !== 'history') { alert("Veuillez vous connecter."); return; }
        
        // Disable delete mode when changing tabs
        if(isDeleteMode) exitDeleteMode();

        document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
        
        const btnMap = { 'home': 'btnHome', 'validation': 'btnVal', 'operations': 'btnOps', 'history': 'btnHist', 'settings': 'btnConfig' };
        if(document.getElementById(btnMap[id])) document.getElementById(btnMap[id]).classList.add('active');

        if(id === 'home') initHome();
        if(id === 'validation') renderValidation();
        if(id === 'operations') renderOps();
        if(id === 'history') renderHistory();
        if(id === 'settings') renderSettings();
    }

    window.updateBadge = function() {
        if(!window.db) return;
        const valCount = window.db.operations.filter(o => o.status === 'draft').length;
        const opsCount = window.db.operations.filter(o => o.status === 'pending').length;
        document.getElementById('badgeVal').innerText = valCount;
        document.getElementById('badgeVal').style.display = valCount > 0 ? 'inline-block' : 'none';
        document.getElementById('badgeOps').innerText = opsCount;
        document.getElementById('badgeOps').style.display = opsCount > 0 ? 'inline-block' : 'none';
    }

    // --- CALC ---
    let calcExp = "";
    function openCalc() { document.getElementById('calcModal').style.display = 'flex'; }
    function closeCalc() { document.getElementById('calcModal').style.display = 'none'; }
    function calcInput(v) { calcExp += v; document.getElementById('calcDisplay').innerText = calcExp; }
    function calcClear() { calcExp = ""; document.getElementById('calcDisplay').innerText = "0"; }
    function calcDel() { calcExp = calcExp.slice(0, -1); document.getElementById('calcDisplay').innerText = calcExp || "0"; }
    function calcResult() { try { const res = new Function('return ' + calcExp)(); document.getElementById('calcDisplay').innerText = res; calcExp = String(res); } catch(e) { document.getElementById('calcDisplay').innerText = "Erreur"; calcExp = ""; } }

    // --- SETTINGS ---
    window.renderSettings = function() {
        const db = window.db;
        document.getElementById('listChefs').innerHTML = db.chefs.map((c,i) => `<li>${c} <span onclick="del('chefs',${i})" style="color:red;cursor:pointer;">✕</span></li>`).join('');
        document.getElementById('listLines').innerHTML = db.lines.map((l,i) => `<li>${l.name} <span onclick="del('lines',${i})" style="color:red;cursor:pointer;">✕</span></li>`).join('');
        
        const lOpts = '<option value="">-- Ligne --</option>' + db.lines.map(l => `<option value="${l.id}">${l.name}</option>`).join('');
        document.getElementById('setArtLine').innerHTML = lOpts;
        document.getElementById('setProdLine').innerHTML = lOpts;
        
        document.getElementById('listProducts').innerHTML = db.products.map((p,i) => {
            const lName = db.lines.find(l=>l.id == p.lineId)?.name || '??';
            const msgTxt = p.message ? `<br><small style="color:#f39c12">Msg: ${p.message}</small>` : '';
            return `<li style="border-bottom:1px solid #eee; padding:5px 0;"><b>${p.name}</b> <small>(${lName})</small>${msgTxt}<span onclick="del('products',${i})" style="color:red;float:right;cursor:pointer;">✕</span></li>`
        }).join('');

        document.getElementById('listArticles').innerHTML = db.articles.map((a,i) => {
            const lName = db.lines.find(l=>l.id == a.lineId)?.name || '?';
            return `<li style="padding:5px 0; border-bottom:1px solid #eee;"><b>${a.name}</b> (${a.unit}) <small>${lName}</small> <span onclick="del('articles',${i})" style="color:red;float:right;cursor:pointer;">✕</span></li>`;
        }).join('');
    }

    function addLine() { const v = document.getElementById('setLineName').value; if(v) { window.db.lines.push({id:getId(), name:v}); save(); renderSettings(); document.getElementById('setLineName').value=''; } }
    function addChef() { const v = document.getElementById('setChefName').value; if(v) { window.db.chefs.push(v); save(); renderSettings(); document.getElementById('setChefName').value=''; } }
    function addProduct() { 
        const n = document.getElementById('setProdName').value, l = document.getElementById('setProdLine').value, m = document.getElementById('setProdMsg').value; 
        if(n && l) { window.db.products.push({id:getId(), name:n, lineId:l, message:m}); save(); renderSettings(); document.getElementById('setProdName').value=''; document.getElementById('setProdMsg').value=''; } 
    }
    function addArticle() {
        const l = document.getElementById('setArtLine').value, n = document.getElementById('setArtName').value, u = document.getElementById('setArtUnit').value;
        if(l && n) { window.db.articles.push({id:getId(), lineId:l, name:n, unit:u}); save(); renderSettings(); document.getElementById('setArtName').value=''; }
    }
    function del(k, i) { if(confirm('Supprimer ?')) { window.db[k].splice(i,1); save(); renderSettings(); } }

    // --- HOME ---
    window.initHome = function() {
        const db = window.db;
        const ls = document.getElementById('runLine'), cs = document.getElementById('runChefStart');
        ls.innerHTML = '<option value="">-- Ligne --</option>' + db.lines.map(l=>`<option value="${l.id}">${l.name}</option>`).join('');
        cs.innerHTML = '<option value="">-- Chef --</option>' + db.chefs.map(c=>`<option value="${c}">${c}</option>`).join('');
        document.getElementById('runProd').innerHTML = '<option value="">-- Produit --</option>';
        document.getElementById('checkArea').style.display='none';
    }

    function onLineChange() {
        const lid = document.getElementById('runLine').value;
        const ps = document.getElementById('runProd');
        ps.innerHTML = '<option value="">-- Produit --</option>';
        if(!lid) { document.getElementById('checkArea').style.display='none'; return; }
        const prods = window.db.products.filter(p => p.lineId == lid);
        if(prods.length > 0) ps.innerHTML += prods.map(p => `<option value="${p.id}">${p.name}</option>`).join('');
        loadChecklist();
    }

    function loadChecklist() {
        const lid = document.getElementById('runLine').value;
        const box = document.getElementById('checklist'), area = document.getElementById('checkArea');
        box.innerHTML = '';
        if(!lid) { area.style.display='none'; return; }
        const arts = window.db.articles.filter(a => a.lineId == lid);
        if(arts.length === 0) { area.style.display='none'; return; }
        area.style.display='block';
        arts.forEach(a => {
            box.innerHTML += `<div class="check-item"><input type="checkbox" class="cb-art" data-id="${a.id}" onchange="toggleInp(this)"><div style="flex:1; padding:0 10px;">${a.name} <small>(${a.unit})</small></div><input type="number" step="0.01" id="q-${a.id}" disabled placeholder="0"></div>`;
        });
    }
    function toggleInp(cb) { const inp = document.getElementById('q-'+cb.dataset.id); inp.disabled = !cb.checked; if(!cb.checked) inp.value=''; else inp.focus(); }
    
    function saveRun() {
        const db = window.db;
        const lid = document.getElementById('runLine').value, pid = document.getElementById('runProd').value, sap = document.getElementById('runSap').value, chef = document.getElementById('runChefStart').value;
        if(!lid || !pid || !chef) return alert('Champs requis manquants.');
        const cbs = document.querySelectorAll('.cb-art:checked');
        if(cbs.length===0) return alert('Au moins un article requis.');
        
        const pObj = db.products.find(p => p.id == pid);
        let items = [], err = false;
        cbs.forEach(cb => {
            const val = parseFloat(document.getElementById('q-'+cb.dataset.id).value);
            if(!val || val<=0) err=true;
            const art = db.articles.find(a=>a.id==cb.dataset.id);
            items.push({id:cb.dataset.id, name:art.name, unit:art.unit, qtyOut:val, qtyRem:0, qtyCon:0});
        });
        if(err) return alert('Quantités invalides.');
        
        db.operations.push({
            id: getId(), date: new Date().toISOString(),
            line: db.lines.find(l=>l.id==lid).name,
            product: pObj.name, prodMsg: (pObj.message||""), sapCode: sap,
            startChef: chef, endChef: "", items: items, status: 'draft'
        });
        save();
        alert('Validation !'); nav('validation');
    }

    // --- VALIDATION ---
    window.renderValidation = function() {
        const div = document.getElementById('valList'); div.innerHTML = '';
        const list = window.db.operations.filter(o=>o.status==='draft').sort((a,b)=>b.id-a.id);
        if(list.length===0) return div.innerHTML = '<p style="text-align:center;">Vide.</p>';

        list.forEach(op => {
            const alertHtml = op.prodMsg ? `<div class="warning-box">⚠️ Rappel: ${op.prodMsg}</div>` : '';
            const rows = op.items.map((it, idx) => `<tr><td>${it.name}</td><td><input type="number" class="val-inp-${op.id}" data-idx="${idx}" value="${it.qtyOut}" style="width:130px; text-align:center; border:1px solid orange; background:#fffdf0; font-size:1.1rem; font-weight:bold;"></td></tr>`).join('');
            
            div.innerHTML += `
                <div class="card" style="border-left:5px solid orange;">
                    <h3>${op.product} <small>(${op.line})</small></h3>
                    <div style="margin-bottom:5px;"><span class="info-pill">SAP: ${op.sapCode}</span> <span class="info-pill">Chef: ${op.startChef}</span></div>
                    ${alertHtml}
                    <div class="table-wrap"><table><thead><tr><th>Art</th><th>Sortie</th></tr></thead><tbody>${rows}</tbody></table></div>
                    <br><div style="display:flex;gap:10px;"><button class="btn btn-red btn-sm" style="flex:1" onclick="drop(${op.id}, 'validation')">Supprimer</button><button class="btn btn-green btn-sm" style="flex:2" onclick="confirmDraft(${op.id})">Valider</button></div>
                </div>
            `;
        });
    }

    function confirmDraft(id) {
        const op = window.db.operations.find(o=>o.id===id);
        const inps = document.querySelectorAll(`.val-inp-${id}`);
        inps.forEach(inp => { const v = parseFloat(inp.value); if(v>0) op.items[inp.dataset.idx].qtyOut = v; });
        op.status = 'pending'; save(); nav('operations');
    }

    // --- OPERATIONS ---
    window.renderOps = function() {
        const div = document.getElementById('opsList'); div.innerHTML = '';
        const list = window.db.operations.filter(o=>o.status==='pending').sort((a,b)=>b.id-a.id);
        if(list.length===0) return div.innerHTML = '<p style="text-align:center;">Vide.</p>';
        const cOpts = '<option value="">-- Chef Fin --</option>' + window.db.chefs.map(c=>`<option value="${c}">${c}</option>`).join('');

        list.forEach(op => {
            const rows = op.items.map((it,idx) => `<tr><td>${it.name}</td><td style="text-align:center;background:#f9f9f9;font-size:1.1rem;"><b>${it.qtyOut}</b></td><td><input type="number" class="rem-${op.id}" data-idx="${idx}" placeholder="Reste" style="width:130px; text-align:center; font-size:1.1rem; font-weight:bold;"></td></tr>`).join('');
            
            div.innerHTML += `
                <div class="card" style="border-left:5px solid #3498db;">
                    <h3>${op.product} <small>(${op.line})</small></h3>
                    <div class="table-wrap"><table><thead><tr><th>Art</th><th>Sort</th><th>Ret</th></tr></thead><tbody>${rows}</tbody></table></div>
                    <div style="margin-top:10px; background:#f8f9fa; padding:10px;">
                        <select id="endChef-${op.id}" style="margin-bottom:10px;">${cOpts}</select>
                        <div style="display:flex;justify-content:flex-end;gap:10px;"><button class="btn btn-red btn-sm" onclick="drop(${op.id}, 'operations')">Annuler</button><button class="btn btn-green" onclick="finish(${op.id})">Clôturer</button></div>
                    </div>
                </div>
            `;
        });
    }

    function drop(id, view) { if(confirm('Supprimer ?')) { window.db.operations=window.db.operations.filter(o=>o.id!==id); save(); if(view==='validation') renderValidation(); else renderOps(); } }

    function finish(id) {
        const endC = document.getElementById(`endChef-${id}`).value;
        if(!endC) return alert("Chef Fin requis.");
        const op = window.db.operations.find(o=>o.id===id);
        const rems = document.querySelectorAll('.rem-'+id);
        op.items.forEach((item, idx) => {
            const r = parseFloat(rems[idx].value)||0;
            item.qtyRem = r;
            item.qtyCon = parseFloat((item.qtyOut - r).toFixed(2));
            if(item.qtyCon<0) item.qtyCon=0;
        });
        op.endChef = endC; op.status = 'completed'; save();
        printTick(op); renderOps();
    }

    // --- HISTORY DELETION LOGIC ---
    window.goToDeleteMode = function() {
        nav('history');
        isDeleteMode = true;
        document.getElementById('delete-toolbar').style.display = 'block';
        renderHistory(); // Re-render with checkboxes
    }

    window.exitDeleteMode = function() {
        isDeleteMode = false;
        document.getElementById('delete-toolbar').style.display = 'none';
        renderHistory(); // Re-render normal view
    }

    window.deleteSelected = function() {
        const checkboxes = document.querySelectorAll('.del-checkbox:checked');
        if(checkboxes.length === 0) return alert("Sélectionnez au moins une opération.");
        
        const code = prompt("Code de sécurité pour suppression :");
        if(code !== "1988") return alert("Code incorrect !");

        const idsToDelete = Array.from(checkboxes).map(cb => parseInt(cb.value));
        window.db.operations = window.db.operations.filter(op => !idsToDelete.includes(op.id));
        save();
        exitDeleteMode();
        alert("Opérations supprimées.");
    }

    window.deleteAllFiltered = function() {
        const code = prompt("ATTENTION: Cela va supprimer TOUTES les opérations affichées.\nCode de sécurité :");
        if(code !== "1988") return alert("Code incorrect !");

        // Logic matches renderHistory logic to know what is currently shown
        const dIn = document.getElementById('histDate').value;
        let list = window.db.operations.filter(o => o.status === 'completed');
        
        // Items to KEEP are those NOT in the current filter
        let itemsToKeep = [];
        let itemsToDeleteIDs = [];

        if(dIn) {
            // Delete only matching date
            itemsToKeep = window.db.operations.filter(o => o.status !== 'completed' || !o.date.startsWith(dIn));
        } else {
            // No date filter = delete last 10 (as displayed) or ALL completed?
            // "TOUT Supprimer" implies all visible. Default view is last 10.
            // But usually "Delete All" means "Clear History".
            // Let's implement: Delete currently visible items.
            list.sort((a,b) => new Date(b.date) - new Date(a.date));
            const visibleSlice = list.slice(0, 10);
            const visibleIDs = visibleSlice.map(o => o.id);
            
            itemsToKeep = window.db.operations.filter(o => !visibleIDs.includes(o.id));
        }

        window.db.operations = itemsToKeep;
        save();
        exitDeleteMode();
        alert("Suppression effectuée.");
    }

    // --- HISTORY RENDERING ---
    window.renderHistory = function() {
        const div = document.getElementById('histList'), dIn = document.getElementById('histDate').value;
        div.innerHTML = '';
        if(!window.db || !window.db.operations) return;
        let list = window.db.operations.filter(o => o.status === 'completed');
        list.sort((a,b) => new Date(b.date) - new Date(a.date));
        if(dIn) list = list.filter(o => o.date.startsWith(dIn)); else list = list.slice(0, 10);
        
        list.forEach(op => {
            const rows = op.items.map(it => `<tr><td>${it.name}</td><td>${it.qtyOut}</td><td>${it.qtyRem}</td><td><b>${it.qtyCon}</b></td></tr>`).join('');
            
            // Checkbox logic
            const checkHtml = isDeleteMode ? `<input type="checkbox" class="del-checkbox" value="${op.id}">` : '';
            const headerClass = isDeleteMode ? 'hist-header delete-mode' : 'hist-header';

            div.innerHTML += `
                <div class="hist-item">
                    <div class="${headerClass}" style="display:flex; align-items:center; padding:15px; cursor:pointer; background:white;">
                        ${checkHtml}
                        <div style="flex:1" onclick="document.getElementById('d-${op.id}').classList.toggle('open')">
                            <div><strong>${op.product}</strong><br><small>${op.line}</small></div>
                        </div>
                        <span onclick="document.getElementById('d-${op.id}').classList.toggle('open')">▼</span>
                    </div>
                    <div id="d-${op.id}" class="hist-details">
                        <p>SAP: ${op.sapCode} | Chefs: ${op.startChef} > ${op.endChef}</p>
                        <div class="table-wrap"><table><thead><tr><th>Art</th><th>Sort</th><th>Ret</th><th>Conso</th></tr></thead><tbody>${rows}</tbody></table></div>
                        <br><button class="btn btn-blue btn-sm no-print" onclick="printTickId(${op.id})">Imprimer</button>
                    </div>
                </div>`;
        });
    }

    function printTickId(id) { printTick(window.db.operations.find(o=>o.id===id)); }
    function printTick(op) {
        const w = window.open('','','width=800,height=600');
        const rows = op.items.map(i=>`<tr><td>${i.name}</td><td align="center">${i.unit}</td><td align="center">${i.qtyOut}</td><td align="center">${i.qtyRem}</td><td align="center"><b>${i.qtyCon}</b></td></tr>`).join('');
        w.document.write(`<html><head><title>Bon</title><style>body{font-family:sans-serif} table{width:100%;border-collapse:collapse;} th,td{border:1px solid #000;padding:5px} .box{border:1px solid #000;padding:10px;margin-bottom:10px}</style></head><body><center><h2>BON</h2></center><div class="box">Date: ${new Date(op.date).toLocaleString('fr-FR')}<br>Ligne: ${op.line}<br>Produit: ${op.product}<br>SAP: ${op.sapCode}</div><div class="box">Départ: ${op.startChef}<br>Fin: ${op.endChef}</div><table><thead><tr><th>Art</th><th>Un</th><th>Sor</th><th>Ret</th><th>Con</th></tr></thead><tbody>${rows}</tbody></table></body></html>`);
        w.document.close(); w.print();
    }

    // Init
    isAuthenticated = false; toggleAuthUI(); nav('history');
</script>

</body>
</html>
