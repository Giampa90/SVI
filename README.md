<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>SOGV PRO - Controllo Ispettivo</title>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.25/jspdf.plugin.autotable.min.js"></script>

    <style>
        :root {
            --navy: #002366; --steel: #4A5568; --light-bg: #F8FAFC;
            --card-bg: #FFFFFF; --danger: #E53E3E; --success: #38A169;
            --warning: #DD6B20; --border: #CBD5E0; --gold: #B7791F;
        }

        body {
            font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
            background-color: var(--light-bg); margin: 0; padding-bottom: 90px; color: #1A202C;
            -webkit-tap-highlight-color: transparent;
        }

        header {
            background: var(--navy); color: white; padding: 18px; text-align: center;
            border-bottom: 4px solid var(--gold); font-weight: 800; text-transform: uppercase;
            letter-spacing: 1.5px; box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }

        /* Progress Bar Turno */
        .progress-container { background: #E2E8F0; height: 10px; width: 100%; position: sticky; top: 0; z-index: 10; }
        #progress-bar { background: linear-gradient(90deg, var(--success), #48BB78); height: 100%; width: 0%; transition: width 0.6s cubic-bezier(0.4, 0, 0.2, 1); }

        .container { padding: 15px; max-width: 650px; margin: auto; }

        .card {
            background: var(--card-bg); border-radius: 10px; padding: 18px;
            margin-bottom: 15px; border: 1px solid var(--border);
            box-shadow: 0 2px 5px rgba(0,0,0,0.04);
        }

        h2 {
            font-size: 0.85rem; margin: 0 0 15px 0; color: var(--navy);
            border-left: 5px solid var(--gold); padding-left: 12px;
            text-transform: uppercase; letter-spacing: 0.5px;
            display: flex; justify-content: space-between; align-items: center;
        }

        label { font-size: 0.7rem; color: var(--steel); font-weight: 800; text-transform: uppercase; display: block; margin-bottom: 6px; }

        input, select, textarea {
            width: 100%; padding: 14px; margin-bottom: 14px;
            border: 2px solid #EDF2F7; border-radius: 8px; font-size: 16px;
            background: #F7FAFC; box-sizing: border-box; transition: 0.2s;
        }

        input:focus, select:focus { border-color: var(--navy); outline: none; background: #FFF; }

        .btn {
            border: none; border-radius: 8px; padding: 18px;
            font-weight: 800; cursor: pointer; width: 100%;
            text-transform: uppercase; transition: all 0.2s; letter-spacing: 1px;
        }

        .btn-save { background: var(--navy); color: white; box-shadow: 0 4px 0 #00153D; }
        .btn-save:active { transform: translateY(3px); box-shadow: 0 1px 0 #00153D; }
        .btn-pdf { background: var(--gold); color: white; margin-top: 15px; }
        .btn-danger-outline { background: transparent; border: 1px solid var(--danger); color: var(--danger); font-size: 0.7rem; padding: 10px; }

        /* Log Style */
        .log-entry {
            background: white; padding: 15px; margin-bottom: 10px;
            border-radius: 8px; border-left: 6px solid var(--navy);
            box-shadow: 0 1px 3px rgba(0,0,0,0.08); font-size: 0.9rem;
        }
        .log-entry.alert { border-left-color: var(--danger); background: #FFF5F5; }

        /* Navbar */
        .nav-bar {
            position: fixed; bottom: 0; left: 0; right: 0;
            background: var(--navy); display: flex; height: 75px; z-index: 1000;
            padding-bottom: env(safe-area-inset-bottom);
        }
        .nav-item {
            flex: 1; display: flex; flex-direction: column;
            align-items: center; justify-content: center; color: #A0AEC0;
            text-decoration: none; font-size: 0.7rem; font-weight: bold;
        }
        .nav-item.active { color: white; background: rgba(255,255,255,0.1); border-top: 3px solid var(--gold); }
        .nav-item span.icon { font-size: 1.5rem; margin-bottom: 3px; }

        .page { display: none; }
        .page.active { display: block; animation: slideIn 0.3s ease-out; }
        @keyframes slideIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        .status-badge { padding: 4px 8px; border-radius: 6px; font-size: 0.65rem; font-weight: 900; }
        .badge-miss { background: #FED7D7; color: #822727; }
        .badge-ok { background: #C6F6D5; color: #22543D; }
    </style>
</head>
<body>

<header>VIGILANZA ISPETTIVA SOGV</header>
<div class="progress-container"><div id="progress-bar"></div></div>

<div class="container">

    <div id="page-service" class="page active">
        <div class="card">
            <h2>REGISTRAZIONE</h2>
            <input type="hidden" id="edit-log-idx" value="-1">

            <label>Seleziona Sito</label>
            <select id="sel-client" onchange="updateStepUI()"></select>

            <div id="step-info" style="display:none; background:#EBF8FF; padding:12px; border-radius:8px; margin-bottom:15px; font-size:0.85rem; border: 1px solid #BEE3F8;">
                📝 Passaggi Turno: <b id="cur-s">0</b> / <b id="tot-s">0</b>
            </div>

            <label>Orario Ispezione</label>
            <input type="time" id="ins-time">

            <label>Esito Operativo</label>
            <input type="text" id="ins-note" placeholder="ES: REGOLARE" list="note-suggest">
            <datalist id="note-suggest">
                <option value="REGOLARE">
                <option value="CANCELLO CHIUSO">
                <option value="ANOMALIA SEGNALATA">
                <option value="RONDA PERIMETRALE">
            </datalist>

            <button class="btn btn-save" id="btn-log-save" onclick="handleRegistration()">REGISTRA ISPEZIONE</button>
        </div>

        <h3 style="font-size:0.75rem; color:var(--steel); margin:20px 5px 10px; font-weight: 900; text-transform: uppercase;">Cronologia Recente</h3>
        <div id="log-list"></div>
    </div>

    <div id="page-clients" class="page">
        <div class="card">
            <h2>CONFIGURAZIONE SITI</h2>
            <input type="hidden" id="edit-idx" value="-1">
            <label>Nome Cliente / Sito</label>
            <input type="text" id="c-name" placeholder="Es: Banca Roma">
            <label>Indirizzo Operativo</label>
            <input type="text" id="c-addr" placeholder="Via del Corso 1">
            <label>Numero Passaggi Richiesti</label>
            <input type="number" id="c-steps" value="1" min="1">

            <button class="btn btn-save" id="btn-c-save" onclick="saveClient()">AGGIUNGI IN ANAGRAFICA</button>
        </div>
        <div id="client-list"></div>
    </div>

    <div id="page-report" class="page">
        <div class="card">
            <h2>DETTAGLI SERVIZIO</h2>
            <label>Operatore (Matricola)</label>
            <input type="text" id="op-id" placeholder="Es. GPG 452">
            <label>Mezzo di Servizio</label>
            <input type="text" id="op-vehicle" placeholder="Es. Fiat Panda - AA000BB">
            <label>Check-up Mezzo / Note</label>
            <textarea id="op-notes" rows="3" placeholder="Note su stato veicolo o comunicazioni fine turno..."></textarea>
        </div>

        <div class="card">
            <h2>RIEPILOGO COPERTURA</h2>
            <div id="report-summary"></div>
            <button class="btn btn-pdf" onclick="generatePDF()">GENERA REPORT PDF</button>
            <div style="margin-top: 25px; border-top: 1px solid #eee; padding-top: 15px;">
                <button class="btn btn-danger-outline" onclick="clearAllData()">RESET TOTALE (PULISCI APP)</button>
            </div>
        </div>
    </div>

</div>

<nav class="nav-bar">
    <a href="javascript:void(0)" class="nav-item active" onclick="switchPage('page-service', this)"><span class="icon">🛡️</span>SERVIZIO</a>
    <a href="javascript:void(0)" class="nav-item" onclick="switchPage('page-clients', this)"><span class="icon">🏢</span>SITI</a>
    <a href="javascript:void(0)" class="nav-item" onclick="switchPage('page-report', this)"><span class="icon">📊</span>REPORT</a>
</nav>

<script>
    let clients = JSON.parse(localStorage.getItem('sogv_clients')) || [];
    let log = JSON.parse(localStorage.getItem('sogv_log')) || [];

    // --- NAVIGAZIONE ---
    function switchPage(id, el) {
        document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
        el.classList.add('active');
        if(id === 'page-report') updateReportSummary();
        updateProgressBar();
    }

    // --- LOGICA SERVIZIO ---
    function updateStepUI() {
        const name = document.getElementById('sel-client').value;
        const box = document.getElementById('step-info');
        if(!name) { box.style.display = 'none'; return; }

        const client = clients.find(c => c.name === name);
        if(!client) { box.style.display = 'none'; return; }
        
        const count = log.filter(l => l.client === name).length;
        box.style.display = 'block';
        document.getElementById('cur-s').innerText = count;
        document.getElementById('tot-s').innerText = client.steps;
    }

    function handleRegistration() {
        const client = document.getElementById('sel-client').value;
        const time = document.getElementById('ins-time').value;
        const note = document.getElementById('ins-note').value.toUpperCase() || "REGOLARE";
        const logIdx = document.getElementById('edit-log-idx').value;
        const todayDate = new Date().toLocaleDateString('it-IT');

        if(!client || !time) {
            alert("⚠️ Seleziona un sito e inserisci l'orario!");
            return;
        }

        if(logIdx === "-1") {
            // Controllo duplicati: verifica se l'operatore è già entrato in questo sito oggi
            const alreadyRegistered = log.some(l => l.client === client && l.date === todayDate);
            if (alreadyRegistered) {
                const confirmDouble = confirm(`⚠️ Attenzione: Risulta che sei GESTITO / ENTRATO nel sito "${client}" in data odierna. Vuoi registrare un ulteriore passaggio?`);
                if (!confirmDouble) return; // Interrompe la registrazione se l'utente clicca Annulla
            }

            // Inserimento nuovo Log
            const newEntry = {
                client, time, note,
                date: todayDate,
                id: Date.now()
            };
            log.unshift(newEntry);
        } else {
            // Modifica Log Esistente
            log[logIdx].client = client;
            log[logIdx].time = time;
            log[logIdx].note = note;
            
            document.getElementById('edit-log-idx').value = "-1";
            document.getElementById('btn-log-save').innerText = "REGISTRA ISPEZIONE";
        }

        localStorage.setItem('sogv_log', JSON.stringify(log));

        document.getElementById('ins-note').value = "";
        document.getElementById('ins-time').value = new Date().toTimeString().slice(0,5);
        render();
        updateStepUI();
        updateProgressBar();
    }

    function editLog(i) {
        document.getElementById('sel-client').value = log[i].client;
        document.getElementById('ins-time').value = log[i].time;
        document.getElementById('ins-note').value = log[i].note;
        document.getElementById('edit-log-idx').value = i;
        document.getElementById('btn-log-save').innerText = "AGGIORNA ISPEZIONE";
        updateStepUI();
        window.scrollTo({top: 0, behavior: 'smooth'});
    }

    function deleteLog(i) {
        if(confirm("Eliminare questa registrazione dalla cronologia?")) {
            log.splice(i, 1);
            localStorage.setItem('sogv_log', JSON.stringify(log));
            render();
            updateStepUI();
            updateProgressBar();
        }
    }

    // --- GESTIONE DATABASE SITI ---
    function saveClient() {
        const name = document.getElementById('c-name').value;
        const addr = document.getElementById('c-addr').value;
        const steps = parseInt(document.getElementById('c-steps').value) || 1;
        const idx = document.getElementById('edit-idx').value;

        if(!name) return;

        if(idx === "-1") {
            clients.push({name, addr, steps});
        } else {
            clients[idx] = {name, addr, steps};
            document.getElementById('edit-idx').value = "-1";
            document.getElementById('btn-c-save').innerText = "AGGIUNGI IN ANAGRAFICA";
        }

        localStorage.setItem('sogv_clients', JSON.stringify(clients));
        document.getElementById('c-name').value = '';
        document.getElementById('c-addr').value = '';
        document.getElementById('c-steps').value = '1';
        render();
    }

    function editClient(i) {
        document.getElementById('c-name').value = clients[i].name;
        document.getElementById('c-addr').value = clients[i].addr;
        document.getElementById('c-steps').value = clients[i].steps;
        document.getElementById('edit-idx').value = i;
        document.getElementById('btn-c-save').innerText = "AGGIORNA SITO";
        switchPage('page-clients', document.querySelectorAll('.nav-item')[1]);
    }

    function deleteClient(i) {
        if(confirm("Eliminare definitivamente questo sito?")) {
            clients.splice(i, 1);
            localStorage.setItem('sogv_clients', JSON.stringify(clients));
            render();
        }
    }

    // --- UI HELPERS ---
    function updateProgressBar() {
        if(clients.length === 0) {
            document.getElementById('progress-bar').style.width = "0%";
            return;
        }
        let totalNeeded = clients.reduce((a, b) => a + (parseInt(b.steps) || 0), 0);
        let completed = 0;
        clients.forEach(c => {
            let done = log.filter(l => l.client === c.name).length;
            completed += Math.min(done, c.steps);
        });
        let perc = totalNeeded > 0 ? (completed / totalNeeded) * 100 : 0;
        document.getElementById('progress-bar').style.width = perc + "%";
    }

    function updateReportSummary() {
        let html = "";
        clients.forEach(c => {
            let done = log.filter(l => l.client === c.name).length;
            let isOk = done >= c.steps;
            html += `<div style="margin-bottom:12px; font-size:0.85rem; display:flex; justify-content:space-between; align-items:center; border-bottom:1px solid #f0f0f0; padding-bottom:8px;">
                <span><b>${c.name}</b> (${done}/${c.steps})</span>
                <span class="status-badge ${isOk ? 'badge-ok' : 'badge-miss'}">${isOk ? 'OK' : 'INCOMPLETO'}</span>
            </div>`;
        });
        document.getElementById('report-summary').innerHTML = html || "Nessun sito configurato.";
    }

    function render() {
        // Dropdown Siti
        const sel = document.getElementById('sel-client');
        const currentVal = sel.value;
        sel.innerHTML = '<option value="">-- SELEZIONA OBIETTIVO --</option>';
        clients.sort((a,b) => a.name.localeCompare(b.name)).forEach(c => {
            sel.innerHTML += `<option value="${c.name}" ${currentVal === c.name ? 'selected' : ''}>${c.name}</option>`;
        });

        // Lista Log
        const logDiv = document.getElementById('log-list');
        logDiv.innerHTML = "";
        log.slice(0, 15).forEach((l, i) => {
            logDiv.innerHTML += `
                <div class="log-entry ${l.note !== 'REGOLARE' ? 'alert' : ''}" style="display:flex; justify-content:space-between; align-items:center;">
                    <div style="flex-grow: 1;">
                        <div style="display:flex; justify-content:space-between; margin-bottom:5px; padding-right: 10px;">
                            <b>${l.time}</b> <span>${l.client}</span>
                        </div>
                        <div style="font-weight: bold; color: var(--navy)">ESITO: ${l.note}</div>
                    </div>
                    <div style="display:flex; gap: 10px;">
                        <button onclick="editLog(${i})" style="border:none; background:none; font-size:1.2rem; cursor:pointer;">✏️</button>
                        <button onclick="deleteLog(${i})" style="border:none; background:none; font-size:1.2rem; cursor:pointer;">🗑️</button>
                    </div>
                </div>`;
        });

        // Lista Database Siti
        const cList = document.getElementById('client-list');
        cList.innerHTML = "";
        clients.forEach((c, i) => {
            cList.innerHTML += `
                <div class="card" style="display:flex; justify-content:space-between; align-items:center;">
                    <div style="font-size:0.85rem">
                        <b>${c.name}</b><br>
                        <small style="color:var(--steel)">${c.addr} | <b>Turno: ${c.steps}</b></small>
                    </div>
                    <div>
                        <button onclick="editClient(${i})" style="border:none; background:none; font-size:1.2rem; cursor:pointer;">✏️</button>
                        <button onclick="deleteClient(${i})" style="border:none; background:none; font-size:1.2rem; cursor:pointer;">🗑️</button>
                    </div>
                </div>`;
        });
    }

    function clearAllData() {
        if(confirm("ATTENZIONE: Verranno eliminati tutti i siti e i log. Sei sicuro?")) {
            localStorage.clear();
            location.reload();
        }
    }

    async function generatePDF() {
        const { jsPDF } = window.jspdf;
        const doc = new jsPDF();
        const op = document.getElementById('op-id').value || "N.D.";

        // Header PDF
        doc.setFillColor(0, 35, 102); doc.rect(0, 0, 210, 40, 'F');
        doc.setTextColor(255, 255, 255); doc.setFontSize(20);
        doc.text("RAPPORTO ATTIVITA' ISPETTIVA", 14, 25);

        doc.setTextColor(0); doc.setFontSize(10);
        doc.setFont(undefined, 'bold');
        doc.text(`OPERATORE: ${op.toUpperCase()}`, 14, 50);
        doc.text(`VEICOLO: ${document.getElementById('op-vehicle').value.toUpperCase()}`, 14, 56);
        doc.text(`DATA RAPPORTO: ${new Date().toLocaleDateString()}`, 140, 50);

        // Tabella PDF
        doc.autoTable({
            startY: 65,
            head: [['ORA', 'CLIENTE / SITO', 'ESITO DI SERVIZIO']],
            body: log.map(l => [l.time, l.client, l.note]),
            headStyles: { fillColor: [0, 35, 102] }
        });

        const finalY = doc.lastAutoTable.finalY;
        doc.text("NOTE DI SERVIZIO E MEZZO:", 14, finalY + 15);
        doc.setFont(undefined, 'normal');
        doc.setFontSize(9);
        const notes = document.getElementById('op-notes').value || "Nessuna nota aggiuntiva.";
        doc.text(notes, 14, finalY + 22, { maxWidth: 180 });

        doc.save(`Rapporto_SOGV_${new Date().toISOString().slice(0,10)}.pdf`);
    }

    // Inizializzazione Orario
    document.getElementById('ins-time').value = new Date().toTimeString().slice(0,5);
    render();
    updateProgressBar();
</script>

</body>
</html>
