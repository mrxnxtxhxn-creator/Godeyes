<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>DHL TRIAGEM | MONITOR DE PRODUÇÃO</title>
    <!-- Leitura de CSV e Excel -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700;900&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --vermelho-dhl: #d40511; 
            --amarelo-dhl: #ffcc00; 
            --verde-ok: #00ff00;
            --azul-oracle: #00e5ff;
            --fundo-mapa: #0d1117;
            --painel-bg: rgba(15, 15, 20, 0.95);
            --grid-line: rgba(255, 204, 0, 0.08);
        }

        body { 
            background-color: #000; 
            color: #fff; 
            font-family: 'Roboto', sans-serif; 
            margin: 0; padding: 0;
            height: 100vh; 
            overflow: hidden; 
            display: flex;
            flex-direction: column; 
        }

        /* HEADER */
        header { 
            position: relative; 
            z-index: 1000;
            flex-shrink: 0; 
            display: flex; justify-content: space-between; align-items: center; 
            background: #111; border-bottom: 2px solid var(--amarelo-dhl);
            padding: 0 20px; height: 60px; box-sizing: border-box;
        }

        .brand { font-size: 24px; font-weight: 900; color: var(--amarelo-dhl); font-style: italic; display: flex; align-items: center; gap: 10px; }
        .brand span { color: #fff; font-weight: 300; font-size: 14px; font-style: normal; text-transform: uppercase;}
        
        .controls { display: flex; gap: 10px; align-items: center; }
        
        .btn {
            background: #333; color: #fff; padding: 8px 12px; border: 1px solid #555;
            cursor: pointer; font-size: 11px; border-radius: 4px; font-weight: bold; text-transform: uppercase;
            display: flex; align-items: center; justify-content: center; gap: 5px; white-space: nowrap;
        }
        .btn:hover { background: #555; border-color: #fff; }
        .btn-connect { background: var(--amarelo-dhl); color: #000; border: none; }
        .btn-excel { background: #1D6F42; color: #fff; border: none; } 
        .btn-excel:hover { background: #155e35; }
        
        /* BOTÃO DO ORÁCULO */
        .btn-oracle { 
            background: linear-gradient(45deg, #005f73, #0a9396); 
            border: 1px solid var(--azul-oracle); color: #fff; 
            box-shadow: 0 0 10px rgba(0, 229, 255, 0.3);
        }
        .btn-oracle:hover { background: #00e5ff; color: #000; box-shadow: 0 0 20px var(--azul-oracle); }
        
        .btn-icon { width: 32px; padding: 0; font-size: 16px; }
        
        #sheet-input { background: #222; border: 1px solid #444; color: #fff; padding: 8px; width: 180px; border-radius: 4px; }

        /* ÁREA PRINCIPAL (MAPA) */
        #map-container {
            position: relative;
            flex-grow: 1; 
            width: 100%;
            overflow: auto; 
            background-color: var(--fundo-mapa);
            background-image: 
                linear-gradient(var(--grid-line) 1px, transparent 1px),
                linear-gradient(90deg, var(--grid-line) 1px, transparent 1px);
            background-size: 50px 50px;
        }

        #map-content { 
            position: relative; 
            min-width: 100%; 
            min-height: 100%; 
            display: inline-block; 
        }

        #floor-plan { display: block; opacity: 0.5; display: none; }

        /* MENSAGEM DE BOAS-VINDAS (EMPTY STATE) */
        #empty-state {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            text-align: center; color: #444; font-family: 'Roboto'; pointer-events: none;
        }
        #empty-state h1 { font-size: 30px; margin: 0; color: #666; }
        #empty-state p { font-size: 14px; margin-top: 10px; }

        /* CARD DA ILHA */
        .zone-card {
            position: absolute; transform: translate(-50%, -50%);
            background: rgba(10, 10, 10, 0.9); border: 2px solid #444; border-radius: 6px;
            min-width: 140px; box-shadow: 0 10px 20px rgba(0,0,0,0.6);
            transition: all 0.5s ease; z-index: 10;
            display: flex; flex-direction: column; overflow: hidden;
        }

        .status-critico { 
            border-color: var(--vermelho-dhl); 
            box-shadow: 0 0 25px rgba(212, 5, 17, 0.5);
            animation: pulse-border 1s infinite alternate; z-index: 100;
        }
        .status-critico .zone-header { background: var(--vermelho-dhl); color: white; }
        
        .status-ok { border-color: var(--verde-ok); }
        .status-ok .zone-header { background: #003300; color: var(--verde-ok); border-bottom: 1px solid #004400; }
        
        .status-folga { border-color: var(--amarelo-dhl); }
        .status-folga .zone-header { background: #332b00; color: var(--amarelo-dhl); border-bottom: 1px solid #443a00; }

        @keyframes pulse-border { 
            0% { transform: translate(-50%, -50%) scale(1); box-shadow: 0 0 10px var(--vermelho-dhl); } 
            100% { transform: translate(-50%, -50%) scale(1.05); box-shadow: 0 0 30px var(--vermelho-dhl); } 
        }

        .zone-header { font-size: 14px; font-weight: 900; text-transform: uppercase; text-align: center; padding: 6px; white-space: nowrap; }
        .card-body { padding: 8px; display: flex; flex-direction: column; gap: 4px; }
        .metric-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 5px; text-align: center; }
        .metric-item { display: flex; flex-direction: column; }
        .metric-lbl { font-size: 8px; color: #aaa; margin-bottom: 2px; text-transform: uppercase; }
        .metric-val { font-size: 16px; font-weight: bold; color: #fff; }
        .action-box { margin-top: 5px; padding: 4px; border-radius: 3px; text-align: center; font-size: 11px; font-weight: 900; text-transform: uppercase; }
        .act-add { background: var(--vermelho-dhl); color: #fff; }
        .act-remove { background: var(--amarelo-dhl); color: #000; }
        .act-ok { background: #222; color: #555; border: 1px solid #333; }
        .vol-bar-bg { height: 4px; background: #333; border-radius: 2px; overflow: hidden; margin-top: 5px; }
        .vol-bar-fill { height: 100%; transition: width 0.5s; }

        /* SIDEBAR (HUD) */
        #hud-sidebar {
            position: absolute; top: 10px; right: 20px; bottom: 20px; width: 300px;
            background: var(--painel-bg); border: 1px solid #444; border-radius: 8px;
            display: flex; flex-direction: column; backdrop-filter: blur(10px); z-index: 900;
            transition: transform 0.3s ease-in-out; transform: translateX(0);
            max-height: calc(100vh - 80px);
        }
        #hud-sidebar.hidden { transform: translateX(120%); }

        .hud-header-container { display: flex; justify-content: space-between; align-items: center; padding: 15px; border-bottom: 1px solid #444; }
        .hud-title { font-weight: bold; color: #ccc; font-size: 14px; }
        .btn-close-hud { cursor: pointer; color: #888; font-size: 18px; }
        
        .hud-list { flex-grow: 1; overflow-y: auto; padding: 10px; }
        .list-row { display: flex; justify-content: space-between; align-items: center; padding: 10px; border-bottom: 1px solid #333; font-size: 12px; }
        .status-led { width: 8px; height: 8px; border-radius: 50%; display: inline-block; margin-right: 8px; }
        .ilha-nome { font-weight: bold; font-size: 13px; color: #fff; }
        .rua-nome { font-size: 10px; color: #888; display: block; }

        /* LOG DE EVENTOS */
        .hud-log {
            height: 150px; border-top: 1px solid #444; background: rgba(0,0,0,0.3);
            padding: 10px; overflow-y: auto; font-family: monospace; font-size: 10px;
            flex-shrink: 0;
        }
        .log-title { color: #888; font-weight: bold; margin-bottom: 5px; display: block; }
        .log-item { margin-bottom: 3px; color: #aaa; border-bottom: 1px solid #222; padding-bottom: 2px;}
        .log-time { color: var(--amarelo-dhl); margin-right: 5px; }
        .log-crit { color: var(--vermelho-dhl); }

        /* MODAIS */
        .modal-overlay {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.85); z-index: 2000; align-items: center; justify-content: center;
        }
        .modal-content {
            background: #1a1a1a; padding: 25px; border-radius: 8px; border: 1px solid #444;
            width: 500px; color: #fff; box-shadow: 0 0 30px rgba(0,0,0,0.8);
        }
        .modal-content h3 { color: var(--amarelo-dhl); margin-top: 0; display: flex; align-items: center; gap: 10px;}
        .modal-content table { width: 100%; border-collapse: collapse; font-size: 12px; margin-top: 10px; }
        .modal-content th { text-align: left; border-bottom: 1px solid #555; color: #aaa; padding: 5px; }
        .modal-content td { border-bottom: 1px solid #333; padding: 5px; }
        
        /* ESTILOS DO ORÁCULO */
        .oracle-result { background: #0f1c24; border: 1px solid var(--azul-oracle); padding: 15px; border-radius: 5px; margin-top: 15px; }
        .oracle-header { color: var(--azul-oracle); font-weight: bold; margin-bottom: 10px; text-transform: uppercase; font-size: 14px; display: flex; justify-content: space-between; }
        .move-card { 
            display: flex; justify-content: space-between; align-items: center; 
            background: rgba(0, 229, 255, 0.1); border-left: 3px solid var(--azul-oracle);
            padding: 8px; margin-bottom: 5px; font-size: 13px;
        }
        .move-arrow { color: var(--azul-oracle); font-weight: bold; padding: 0 10px; }
        .move-highlight { color: #fff; font-weight: bold; }

        .input-group { margin-bottom: 15px; }
        .input-group label { display: block; color: #aaa; font-size: 12px; margin-bottom: 5px; }
        .input-group input { width: 100%; background: #222; border: 1px solid #444; color: #fff; padding: 8px; box-sizing: border-box; }
        .hint { font-size: 10px; color: #666; margin-top: 3px; }

        ::-webkit-scrollbar { width: 12px; height: 12px; }
        ::-webkit-scrollbar-track { background: #111; }
        ::-webkit-scrollbar-thumb { background: var(--amarelo-dhl); border-radius: 6px; border: 2px solid #111; }
        ::-webkit-scrollbar-thumb:hover { background: #ffd700; }

    </style>
</head>
<body>

    <header>
        <div class="brand">
            <div style="width:10px; height:10px; background:red; border-radius:50%; animation:pulse-border 1s infinite"></div>
            DHL <span>MONITOR DE TRIAGEM</span>
        </div>
        <div class="controls">
            <!-- BOTÃO GÊNIO -->
            <button class="btn btn-oracle" onclick="abrirOraculo()">🔮 O ORÁCULO</button>
            <div style="height:20px; width:1px; background:#444; margin:0 5px;"></div>

            <button class="btn btn-icon" onclick="toggleSidebar()" title="Painel Lateral">📋</button>
            <button class="btn btn-icon" onclick="toggleConfig()" title="Configurações de Automação">⚙️</button>
            <button class="btn btn-icon" onclick="toggleHelp()" title="Ajuda">?</button>
            
            <div style="height:20px; width:1px; background:#444; margin:0 5px;"></div>

            <label class="btn btn-excel">
                📂 ABRIR EXCEL
                <input type="file" id="excel-upload" accept=".xlsx, .xls, .csv" style="display:none" onchange="lerExcel(this)">
            </label>

            <input type="text" id="sheet-input" placeholder="Cole link CSV Google...">
            <button class="btn btn-connect" onclick="conectarPlanilha()">CONECTAR</button>
            
            <label class="btn">
                📤 FOTO
                <input type="file" accept="image/*" style="display:none" onchange="carregarFundo(this)">
            </label>
        </div>
    </header>

    <div id="map-container">
        <div id="map-content">
            <img id="floor-plan" src="" alt="">
            <div id="zones-layer"></div>
            
            <div id="empty-state">
                <h1>SISTEMA PRONTO</h1>
                <p>Conecte uma planilha ou carregue um Excel para iniciar.</p>
            </div>
        </div>
        <div style="position: fixed; bottom: 10px; left: 20px; font-size: 11px; color: #666; font-family: monospace; z-index: 100;">
            X,Y = COORDENADAS | META: 290 UPH
        </div>
    </div>

    <!-- HUD LATERAL -->
    <div id="hud-sidebar">
        <div class="hud-header-container">
            <div class="hud-title">STATUS GERAL (<span id="total-ilhas">0</span> Ilhas)</div>
            <div class="btn-close-hud" onclick="toggleSidebar()">✕</div>
        </div>
        <div class="hud-list" id="hud-list">
            <div style="text-align:center; padding:20px; color:#666;">Aguardando dados...</div>
        </div>
        <div class="hud-log">
            <span class="log-title">LOG DE EVENTOS</span>
            <div id="log-container"></div>
        </div>
    </div>

    <!-- MODAL ORÁCULO -->
    <div id="oracle-modal" class="modal-overlay" onclick="abrirOraculo()">
        <div class="modal-content" onclick="event.stopPropagation()">
            <h3>🔮 O ORÁCULO DE ALOCAÇÃO</h3>
            <p style="font-size:12px; color:#aaa">Cálculo matemático para distribuição perfeita da equipe atual.</p>
            
            <div id="oracle-content">
                <p style="color:#666; text-align:center">Carregue dados primeiro para usar o oráculo.</p>
            </div>

            <br>
            <button class="btn btn-connect" style="width:100%" onclick="abrirOraculo()">FECHAR</button>
        </div>
    </div>

    <!-- MODAL AJUDA -->
    <div id="help-modal" class="modal-overlay" onclick="toggleHelp()">
        <div class="modal-content" onclick="event.stopPropagation()">
            <h3>📝 FORMATO DA PLANILHA</h3>
            <p>O sistema aceita Excel (.xlsx) ou Google Sheets com estas colunas (não importa a ordem):</p>
            <table>
                <tr><th>Coluna</th><th>Exemplo</th></tr>
                <tr><td>Setor</td><td>ILHA A</td></tr>
                <tr><td>Rua</td><td>RUA 3</td></tr>
                <tr><td>Volume</td><td>500</td></tr>
                <tr><td>Equipe</td><td>2</td></tr>
                <tr><td>Meta</td><td>60</td></tr>
                <tr><td>X</td><td>50</td></tr>
                <tr><td>Y</td><td>50</td></tr>
            </table>
            <br>
            <button class="btn btn-connect" style="width:100%" onclick="toggleHelp()">FECHAR</button>
        </div>
    </div>

    <!-- MODAL CONFIGURAÇÃO (AUTOMAÇÃO) -->
    <div id="config-modal" class="modal-overlay" onclick="toggleConfig()">
        <div class="modal-content" onclick="event.stopPropagation()">
            <h3>⚙️ AUTOMAÇÃO & WEBHOOKS</h3>
            <div class="input-group">
                <label>URL do Webhook (n8n / Zapier)</label>
                <input type="text" id="webhook-url" placeholder="https://seu-n8n.com/webhook/...">
                <div class="hint">O sistema enviará um POST JSON quando uma ilha entrar em estado CRÍTICO.</div>
            </div>
            <div class="input-group">
                <label>Configurações de Alerta</label>
                <div style="display:flex; gap:10px; align-items:center; margin-top:5px;">
                    <input type="checkbox" id="chk-audio" checked style="width:auto;"> <span style="font-size:12px">Alertas de Voz</span>
                    <input type="checkbox" id="chk-notif" checked style="width:auto;"> <span style="font-size:12px">Notificações Navegador</span>
                </div>
            </div>
            <button class="btn btn-connect" style="width:100%" onclick="salvarConfig()">SALVAR CONFIGURAÇÃO</button>
        </div>
    </div>

    <script>
        const META_UPH = 290; 
        const META_UPM = META_UPH / 60; 

        // GLOBAL DATA STORE
        let currentData = [];

        let sheetUrl = "";
        let webhookUrl = "";
        let updateInterval = null;
        let lastVoiceTime = 0;
        let lastWebhookTime = {};

        window.onload = () => {
            const savedSheet = localStorage.getItem('dhl_sheet_v9');
            const savedWebhook = localStorage.getItem('dhl_webhook_v9');
            
            if(savedSheet) {
                document.getElementById('sheet-input').value = savedSheet;
                conectarPlanilha();
            }

            if(savedWebhook) {
                document.getElementById('webhook-url').value = savedWebhook;
                webhookUrl = savedWebhook;
            }

            if (Notification.permission !== "granted") Notification.requestPermission();
        }

        // --- UI ---
        function toggleSidebar() { document.getElementById('hud-sidebar').classList.toggle('hidden'); }
        function toggleHelp() { toggleModal('help-modal'); }
        function toggleConfig() { toggleModal('config-modal'); }
        
        function toggleModal(id) {
            const el = document.getElementById(id);
            el.style.display = el.style.display === 'flex' ? 'none' : 'flex';
        }

        function abrirOraculo() { 
            const modal = document.getElementById('oracle-modal');
            if (modal.style.display !== 'flex') {
                calcularOraculo();
                modal.style.display = 'flex';
            } else {
                modal.style.display = 'none';
            }
        }

        function salvarConfig() {
            const url = document.getElementById('webhook-url').value.trim();
            webhookUrl = url;
            localStorage.setItem('dhl_webhook_v9', url);
            toggleConfig();
            alert("Configurações salvas!");
        }

        // --- LÓGICA DO ORÁCULO ---
        function calcularOraculo() {
            const content = document.getElementById('oracle-content');
            
            if (!currentData || currentData.length === 0) {
                content.innerHTML = "Carregue dados primeiro.";
                return;
            }

            // 1. Totais
            let totalVolume = 0;
            let totalEquipe = 0;
            let ilhas = [];

            currentData.forEach(row => {
                let vol = row.Volume || 0;
                let equipe = row.Equipe || 0;
                if (vol > 0 || equipe > 0) {
                    totalVolume += vol;
                    totalEquipe += equipe;
                    ilhas.push({ ...row, Volume: vol, Equipe: equipe });
                }
            });

            if (totalEquipe === 0) {
                content.innerHTML = "Sem equipe disponível para alocar.";
                return;
            }

            // 2. Cálculo
            let ilhasComSobra = [];
            let ilhasComFalta = [];

            ilhas.forEach(ilha => {
                let share = ilha.Volume / totalVolume;
                let ideal = Math.round(share * totalEquipe);
                if (ideal < 1 && ilha.Volume > 100) ideal = 1;
                
                let diff = ilha.Equipe - ideal;
                
                if (diff > 0) ilhasComSobra.push({ nome: ilha.Setor, sobra: diff });
                if (diff < 0) ilhasComFalta.push({ nome: ilha.Setor, falta: Math.abs(diff) });
            });

            // 3. Matchmaker
            let htmlMovimentos = "";
            
            ilhasComFalta.forEach(falta => {
                for (let i = 0; i < ilhasComSobra.length; i++) {
                    let doador = ilhasComSobra[i];
                    if (doador.sobra > 0 && falta.falta > 0) {
                        let transferir = Math.min(doador.sobra, falta.falta);
                        
                        htmlMovimentos += `
                        <div class="move-card">
                            <span>Mover <b class="move-highlight">${transferir}</b> logs</span>
                            <span>${doador.nome} <span class="move-arrow">➜</span> ${falta.nome}</span>
                        </div>`;
                        
                        doador.sobra -= transferir;
                        falta.falta -= transferir;
                    }
                }
            });

            if (htmlMovimentos === "") {
                content.innerHTML = `<div style="text-align:center; color:#00ff00; font-weight:bold; padding:20px;">✅ EQUIPE JÁ ESTÁ OTIMIZADA!</div>`;
            } else {
                content.innerHTML = `
                    <div style="font-size:12px; margin-bottom:10px;">
                        Volume Total: <b>${totalVolume}</b> | Equipe Total: <b>${totalEquipe}</b>
                    </div>
                    <div class="oracle-header">SUGESTÕES DE MOVIMENTAÇÃO</div>
                    ${htmlMovimentos}
                `;
            }
        }

        // --- LEITOR DE EXCEL LOCAL ---
        function lerExcel(input) {
            const file = input.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = function(e) {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, {type: 'array'});
                const firstSheetName = workbook.SheetNames[0];
                const worksheet = workbook.Sheets[firstSheetName];
                const jsonData = XLSX.utils.sheet_to_json(worksheet);
                currentData = normalizarDados(jsonData);
                if(updateInterval) clearInterval(updateInterval);
                renderizar(currentData);
                document.getElementById('empty-state').style.display = 'none';
            };
            reader.readAsArrayBuffer(file);
        }

        function normalizarDados(dados) {
            return dados.map(row => {
                const getVal = (keys) => {
                    for (let k of Object.keys(row)) {
                        if (keys.includes(k.toLowerCase().trim())) return row[k];
                    }
                    return undefined;
                };
                return {
                    Setor: getVal(['setor', 'ilha', 'area', 'nome']) || "Desconhecido",
                    Rua: getVal(['rua', 'corredor', 'local']) || "",
                    Volume: parseInt(getVal(['volume', 'pacotes', 'qtd', 'pendente'])) || 0,
                    Equipe: parseInt(getVal(['equipe', 'logs', 'pessoas', 'headcount'])) || 0,
                    Meta: parseInt(getVal(['meta', 'tempo', 'sla'])) || 60,
                    X: parseFloat(getVal(['x', 'posx', 'coluna'])),
                    Y: parseFloat(getVal(['y', 'posy', 'linha']))
                };
            });
        }

        // --- CONEXÃO GOOGLE SHEETS ---
        function conectarPlanilha() {
            const input = document.getElementById('sheet-input').value.trim();
            if(!input) return alert("Cole o link CSV!");
            sheetUrl = input;
            localStorage.setItem('dhl_sheet_v9', input);
            fetchData();
            if(updateInterval) clearInterval(updateInterval);
            updateInterval = setInterval(fetchData, 3000);
        }

        function fetchData() {
            Papa.parse(sheetUrl, {
                download: true, header: true, dynamicTyping: true,
                complete: (res) => {
                    currentData = normalizarDados(res.data);
                    renderizar(currentData);
                    document.getElementById('empty-state').style.display = 'none';
                },
                error: (err) => console.log(err)
            });
        }

        function carregarFundo(input) {
            if (input.files && input.files[0]) {
                var reader = new FileReader();
                reader.onload = function (e) {
                    const img = document.getElementById('floor-plan');
                    const mapContent = document.getElementById('map-content');
                    img.src = e.target.result;
                    img.style.display = 'block';
                    mapContent.style.width = "fit-content";
                    mapContent.style.height = "fit-content";
                };
                reader.readAsDataURL(input.files[0]);
            }
        }

        // --- RENDERIZAÇÃO ---
        function renderizar(data) {
            const layer = document.getElementById('zones-layer');
            const list = document.getElementById('hud-list');
            layer.innerHTML = "";
            list.innerHTML = "";

            let processed = data.filter(r => r.Setor).map(row => {
                let vol = row.Volume || 0; 
                let tempoAlvoSeguranca = (row.Meta || 60) * 0.9;
                let velocidadeNecessaria = vol / tempoAlvoSeguranca;
                let pessoasIdeais = Math.ceil(velocidadeNecessaria / META_UPM);
                if (vol > 0 && pessoasIdeais < 1) pessoasIdeais = 1;
                if (vol === 0) pessoasIdeais = 0;

                let gap = pessoasIdeais - (row.Equipe || 0);
                
                let capReal = (row.Equipe || 0) * META_UPM;
                let tempoFila = capReal > 0 ? vol / capReal : 999;

                let status = "OK";
                if (gap > 0) status = "CRITICO";
                else if (gap < 0) status = "FOLGA";

                return { ...row, Volume: vol, tempoFila, status, gap };
            }).sort((a,b) => a.status === 'CRITICO' ? -1 : 1);

            document.getElementById('total-ilhas').innerText = processed.length;

            processed.forEach(row => {
                let cssClass = "";
                let actionHtml = "";
                let color = "#fff";
                let barColor = "#333";
                let ruaText = row.Rua ? row.Rua : ""; 

                if (row.status === "CRITICO") {
                    cssClass = "status-critico";
                    color = "#d40511";
                    barColor = "#d40511";
                    let add = Math.abs(row.gap);
                    actionHtml = `<div class="action-box act-add">🚨 +${add} LOGS NA ${ruaText}</div>`;
                    triggerAutomation(row, add);

                } else if (row.status === "FOLGA") {
                    cssClass = "status-folga";
                    color = "#ffcc00";
                    barColor = "#ffcc00";
                    let remove = Math.abs(row.gap);
                    if ((row.Equipe - remove) < 1 && row.Volume > 0) remove = row.Equipe - 1;

                    if (remove > 0) {
                        actionHtml = `<div class="action-box act-remove">🔻 LIBERAR ${remove} LOGS</div>`;
                    } else {
                        cssClass = "status-ok"; 
                        color = "#00ff00";
                        barColor = "#00ff00";
                        actionHtml = `<div class="action-box act-ok">ESTÁVEL</div>`;
                    }

                } else {
                    cssClass = "status-ok";
                    color = "#00ff00";
                    barColor = "#00ff00";
                    actionHtml = `<div class="action-box act-ok">ESTÁVEL</div>`;
                }

                // Card Mapa
                let card = document.createElement('div');
                card.className = `zone-card ${cssClass}`;
                let posX = row.X !== undefined ? row.X : 50;
                let posY = row.Y !== undefined ? row.Y : 50;
                card.style.left = posX + "%";
                card.style.top = posY + "%";
                let percentBar = Math.min((row.tempoFila / (row.Meta || 60)) * 100, 100);

                card.innerHTML = `
                    <div class="zone-header">${row.Setor}</div>
                    <div class="card-body">
                        <div style="font-size:10px; color:#aaa; text-align:center; margin-top:-5px;">${ruaText}</div>
                        <div class="metric-grid">
                            <div class="metric-item"><span class="metric-lbl">VOLUME</span><span class="metric-val">${row.Volume}</span></div>
                            <div class="metric-item"><span class="metric-lbl">FILA (MIN)</span><span class="metric-val" style="color:${color}">${row.tempoFila.toFixed(0)}</span></div>
                        </div>
                        <div class="vol-bar-bg"><div class="vol-bar-fill" style="width:${percentBar}%; background:${barColor}"></div></div>
                        ${actionHtml}
                    </div>`;
                layer.appendChild(card);

                // Item Lista
                list.innerHTML += `
                    <div class="list-row">
                        <div style="display:flex; align-items:center;">
                            <div class="status-led" style="background:${color}"></div>
                            <div><div class="ilha-nome">${row.Setor}</div><span class="rua-nome">${ruaText}</span></div>
                        </div>
                        <div style="text-align:right">
                            <div style="font-weight:bold">${row.Volume}</div>
                            <div style="font-size:10px; color:${color}">${row.tempoFila.toFixed(0)}m</div>
                        </div>
                    </div>`;
            });
        }

        // --- AUTOMAÇÃO ---
        function triggerAutomation(row, gap) {
            const now = Date.now();
            const lastTime = lastWebhookTime[row.Setor] || 0;
            
            if (now - lastTime > 300000) {
                const logDiv = document.getElementById('log-container');
                const timeStr = new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
                logDiv.innerHTML = `<div class="log-item"><span class="log-time">${timeStr}</span> <span class="log-crit">${row.Setor}</span> CRÍTICO! Vol: ${row.Volume}</div>` + logDiv.innerHTML;

                if (document.getElementById('chk-notif').checked && Notification.permission === "granted") {
                    new Notification(`ALERTA DHL: ${row.Setor}`, {
                        body: `Volume Crítico: ${row.Volume}. Adicionar +${gap > 0 ? gap : 1} Logs.`,
                        icon: "https://upload.wikimedia.org/wikipedia/commons/b/b9/DHL_Logo.svg"
                    });
                }

                if (webhookUrl) {
                    const payload = {
                        setor: row.Setor,
                        rua: row.Rua,
                        volume: row.Volume,
                        equipe: row.Equipe,
                        acao: gap > 0 ? `Adicionar ${gap} logs` : "Cobrar Ritmo",
                        timestamp: new Date().toISOString()
                    };
                    
                    fetch(webhookUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    }).catch(e => console.error("Erro webhook", e));
                }
                lastWebhookTime[row.Setor] = now;
            }
        }

        function falar(texto) {
            if ('speechSynthesis' in window) {
                const u = new SpeechSynthesisUtterance(texto);
                u.lang = 'pt-BR'; u.rate = 1.2;
                window.speechSynthesis.speak(u);
            }
        }
    </script>
</body>
</html>
