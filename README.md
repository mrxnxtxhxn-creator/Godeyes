<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>OLHO DE DEUS | ULTIMATE COMMAND CENTER</title>
    <script src="https://cdn.plot.ly/plotly-2.27.0.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@700&family=Roboto:wght@300;400;900&display=swap" rel="stylesheet">
    
    <style>
        /* --- ESTÉTICA GLOBAL (CYBERPUNK TÁTICO) --- */
        :root {
            --primary: #ffcc00; /* Amarelo DHL */
            --danger: #ff3333;
            --success: #00ff00;
            --bg-dark: #0a0a0e;
            --glass: rgba(20, 20, 30, 0.7);
            --border: rgba(255, 204, 0, 0.2);
        }

        body { 
            background-color: var(--bg-dark); 
            background-image: 
                linear-gradient(rgba(10, 10, 16, 0.9), rgba(10, 10, 16, 0.9)), 
                url('https://www.transparenttextures.com/patterns/cubes.png'); /* Textura de fundo */
            color: #fff; 
            font-family: 'Roboto', sans-serif; 
            margin: 0; padding: 0; 
            height: 100vh; overflow: hidden; 
        }
        
        /* CONTAINER PRINCIPAL */
        #app-container {
            display: flex; flex-direction: column; height: 100vh; padding: 15px; box-sizing: border-box; transition: padding 0.5s;
        }

        /* --- CABEÇALHO --- */
        header { 
            display: flex; justify-content: space-between; align-items: center; 
            background: var(--glass); backdrop-filter: blur(10px); 
            border: 1px solid var(--border); border-radius: 8px; 
            padding: 10px 20px; margin-bottom: 15px; flex-shrink: 0;
            box-shadow: 0 0 20px rgba(0,0,0,0.5);
            transition: margin-top 0.5s;
        }

        .brand-group { display: flex; align-items: center; gap: 15px; }
        .brand-icon { font-size: 36px; filter: drop-shadow(0 0 8px var(--primary)); }
        .brand-title { font-family: 'Orbitron'; font-size: 22px; color: var(--primary); line-height: 1; }
        .brand-sub { font-size: 10px; letter-spacing: 3px; color: #888; text-transform: uppercase; }

        /* ÁREA DE CONEXÃO (Bloco Compacto) */
        .connect-box { display: flex; flex-direction: column; gap: 5px; width: 250px; }
        #sheet-input { 
            background: rgba(0,0,0,0.5); border: 1px solid #444; color: #fff; 
            padding: 5px 10px; font-size: 11px; text-align: center; border-radius: 4px; 
        }
        .btn-row { display: flex; gap: 5px; }
        .btn { 
            flex: 1; border: none; padding: 6px; font-weight: bold; font-size: 11px; 
            cursor: pointer; border-radius: 3px; font-family: 'Orbitron'; transition: 0.2s; 
        }
        #btn-connect { background: var(--primary); color: #000; }
        #btn-tv { background: #333; color: #fff; border: 1px solid #555; }
        #btn-tv:hover { background: #fff; color: #000; }

        /* --- GRID PRINCIPAL --- */
        .main-grid { 
            display: grid; grid-template-columns: 2fr 1fr; gap: 15px; 
            flex-grow: 1; overflow: hidden; margin-bottom: 35px; /* Espaço para o Ticker */
        }

        /* PAINÉIS */
        .panel { 
            background: var(--glass); border: 1px solid #333; border-radius: 8px; 
            position: relative; overflow: hidden; display: flex; flex-direction: column;
        }

        /* --- MAPA TÁTICO --- */
        #tactical-map { flex-grow: 1; width: 100%; }
        .map-bg { 
            /* Grid Tático de Fundo no Mapa */
            background-image: 
                linear-gradient(rgba(255,255,255,0.03) 1px, transparent 1px),
                linear-gradient(90deg, rgba(255,255,255,0.03) 1px, transparent 1px);
            background-size: 50px 50px;
        }

        /* --- HUD (LISTA LATERAL) --- */
        .hud-scroll { overflow-y: auto; padding: 10px; display: flex; flex-direction: column; gap: 8px; }
        
        /* CARTÕES */
        .card { 
            background: rgba(0,0,0,0.6); border-left: 4px solid #555; padding: 10px; 
            border-radius: 4px; position: relative; 
        }
        .card-top { display: flex; justify-content: space-between; font-family: 'Orbitron'; font-size: 14px; margin-bottom: 5px; }
        .card-stats { display: flex; justify-content: space-between; font-size: 11px; color: #aaa; }
        .val { color: #fff; font-weight: bold; font-size: 14px; }
        
        /* BARRAS E ALERTAS */
        .progress-track { height: 4px; background: #222; margin-top: 8px; border-radius: 2px; }
        .progress-fill { height: 100%; transition: width 1s; }
        
        .alert-badge { 
            margin-top: 8px; padding: 6px; text-align: center; font-weight: bold; 
            font-size: 12px; border-radius: 3px; animation: pulse 1s infinite;
            border: 1px solid;
        }

        /* --- TICKER (RODAPÉ) --- */
        .ticker-wrap {
            position: fixed; bottom: 0; left: 0; width: 100%; height: 35px; 
            background: #000; border-top: 2px solid var(--primary); 
            display: flex; align-items: center; overflow: hidden; z-index: 100;
        }
        .ticker-move { display: inline-block; white-space: nowrap; animation: ticker 30s linear infinite; }
        .ticker-item { display: inline-block; padding: 0 50px; font-family: 'Orbitron'; font-size: 14px; color: var(--primary); }
        
        @keyframes ticker { 0% { transform: translateX(100vw); } 100% { transform: translateX(-100%); } }
        @keyframes pulse { 0% { opacity: 1; } 50% { opacity: 0.6; } 100% { opacity: 1; } }

        /* CORES DE STATUS */
        .st-crit { border-color: var(--danger); }
        .bg-crit { background: var(--danger); color: #fff; }
        .st-ok { border-color: var(--success); }
        .st-warn { border-color: #ffcc00; }
        .bg-warn { background: #ffcc00; color: #000; }

        /* MODO FULLSCREEN */
        body.tv-mode #app-container { padding: 0; }
        body.tv-mode header { margin-top: -100px; } /* Esconde Header */
        body.tv-mode .main-grid { height: 100vh; border-radius: 0; margin-bottom: 35px; }
        body.tv-mode .panel { border-radius: 0; border: none; }

        /* Empty State */
        .empty-state { height: 100%; display: flex; flex-direction: column; align-items: center; justify-content: center; color: #555; }

    </style>
</head>
<body>

    <div id="app-container">
        <header>
            <div class="brand-group">
                <div class="brand-icon">👁️</div>
                <div>
                    <div class="brand-title">OLHO DE DEUS</div>
                    <div class="brand-sub">SISTEMA INTEGRADO V8.0</div>
                </div>
            </div>

            <div class="connect-box">
                <input type="text" id="sheet-input" placeholder="Cole o Link CSV aqui...">
                <div class="btn-row">
                    <button class="btn" id="btn-connect" onclick="connectSystem()">CONECTAR</button>
                    <button class="btn" id="btn-tv" onclick="toggleTV()">📺 TV MODE</button>
                </div>
            </div>
        </header>

        <div class="main-grid">
            <div class="panel map-bg">
                <div id="tactical-map"></div>
                <div id="map-loader" class="empty-state">
                    <div style="font-size:40px">🗺️</div>
                    <p>Aguardando Dados Táticos...</p>
                </div>
            </div>

            <div class="panel">
                <div class="hud-scroll" id="hud-feed">
                    <div class="empty-state">
                        <div style="font-size:40px">📡</div>
                        <p>Sistema Offline</p>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div class="ticker-wrap">
        <div class="ticker-move" id="ticker-content">
            <span class="ticker-item">SISTEMA INICIADO... AGUARDANDO SINCRONIZAÇÃO...</span>
        </div>
    </div>

    <script>
        // CONFIGURAÇÕES GLOBAIS
        const META_UPH = 290; // Unidades Por Hora (Meta)
        const META_UPM = META_UPH / 60; // Unidades Por Minuto (4.83)
        const REFRESH_RATE = 3000; // 3 Segundos

        let timer = null;
        let currentUrl = "";
        
        // SISTEMA DE ÁUDIO (Sirene)
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        
        function beep(urgency) {
            if (audioCtx.state === 'suspended') audioCtx.resume();
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            
            if (urgency === 'high') {
                // Som de Erro Grave (Sawtooth)
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(440, audioCtx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(220, audioCtx.currentTime + 0.3);
                gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.3);
            }
        }

        // CARREGAR URL SALVA
        window.onload = () => {
            const saved = localStorage.getItem('dhl_sheet_url');
            if(saved) {
                document.getElementById('sheet-input').value = saved;
                connectSystem();
            }
        };

        // MODO TV (FULLSCREEN)
        function toggleTV() {
            document.body.classList.toggle('tv-mode');
            // Redimensiona o mapa após a transição
            setTimeout(() => { window.dispatchEvent(new Event('resize')); }, 500);
        }

        // CONEXÃO
        function connectSystem() {
            const url = document.getElementById('sheet-input').value.trim();
            if(!url) return alert("Por favor, cole o link!");
            
            currentUrl = url;
            localStorage.setItem('dhl_sheet_url', url);
            
            const btn = document.getElementById('btn-connect');
            btn.innerText = "ONLINE";
            btn.style.background = "#00ff00";
            
            // Ativa o áudio no primeiro clique do usuário
            if (audioCtx.state === 'suspended') audioCtx.resume();

            fetchData();
            if(timer) clearInterval(timer);
            timer = setInterval(fetchData, REFRESH_RATE);
        }

        function fetchData() {
            Papa.parse(currentUrl, {
                download: true, header: true, dynamicTyping: true,
                complete: (res) => render(res.data),
                error: (e) => console.log("Erro:", e)
            });
        }

        function render(data) {
            const hud = document.getElementById('hud-feed');
            const mapLoader = document.getElementById('map-loader');
            if(mapLoader) mapLoader.style.display = 'none';
            hud.innerHTML = "";

            let mapX=[], mapY=[], mapSize=[], mapColor=[], mapText=[];
            let totalVol = 0;
            let totalPpl = 0;
            let critCount = 0;
            let alertSoundNeeded = false;

            // PROCESSAMENTO LÓGICO
            let sectors = data.filter(r => r.Setor && r.Volume).map(row => {
                let capacity = row.Equipe * META_UPM;
                let minutesLeft = capacity > 0 ? (row.Volume / capacity) : 999;
                
                let status = "NORMAL";
                if(minutesLeft > row.Meta) status = "CRITICO";
                else if(minutesLeft < row.Meta * 0.6) status = "FOLGA";

                totalVol += row.Volume;
                totalPpl += row.Equipe;

                return { ...row, minutesLeft, status };
            });

            // ORDENAR: Críticos primeiro
            sectors.sort((a,b) => (a.status === 'CRITICO' ? -1 : 1));

            // RENDERIZAR
            sectors.forEach(row => {
                let color = "#00ff00"; // Verde
                let css = "st-ok";
                let msg = "";
                let alertClass = "";

                if(row.status === "CRITICO") {
                    color = "#ff3333"; // Vermelho
                    css = "st-crit";
                    critCount++;
                    alertSoundNeeded = true;

                    // Lógica de Solução
                    let speedNeeded = row.Volume / row.Meta;
                    let peopleNeeded = Math.ceil(speedNeeded / META_UPM);
                    let gap = peopleNeeded - row.Equipe;

                    if(gap > 0) {
                        msg = `🚨 ADD +${gap} PESSOAS`;
                        alertClass = "bg-crit";
                    } else {
                        msg = `⚡ COBRAR RITMO`;
                        alertClass = "bg-crit";
                    }
                } 
                else if(row.status === "FOLGA") {
                    color = "#ffcc00"; // Amarelo
                    css = "st-warn";
                    msg = "📉 REDUZIR EQUIPE";
                    alertClass = "bg-warn";
                }

                // DADOS DO MAPA
                mapX.push(row.X); mapY.push(row.Y);
                mapSize.push(Math.min(row.Volume/25, 90) + 15);
                mapColor.push(color);
                mapText.push(`<b>${row.Setor}</b><br>Vol: ${row.Volume}<br>${row.minutesLeft.toFixed(0)} min`);

                // CARD HUD
                hud.innerHTML += `
                <div class="card ${css}">
                    <div class="card-top">
                        <span>${row.Setor}</span>
                        <span style="color:${color}">${row.minutesLeft.toFixed(0)}m</span>
                    </div>
                    <div class="card-stats">
                        <div>VOL: <span class="val">${row.Volume}</span></div>
                        <div>EQP: <span class="val">${row.Equipe}</span></div>
                    </div>
                    ${msg ? `<div class="alert-badge ${alertClass}">${msg}</div>` : ''}
                    <div class="progress-track">
                        <div class="progress-fill" style="width:${Math.min((row.minutesLeft/row.Meta)*100, 100)}%; background:${color}"></div>
                    </div>
                </div>`;
            });

            // UPDATE MAPA
            Plotly.react('tactical-map', [{
                x: mapX, y: mapY, text: mapText,
                mode: 'markers+text', textposition: 'top center',
                marker: { size: mapSize, color: mapColor, opacity: 0.8, line: {color: '#fff', width: 1} },
                type: 'scatter', hoverinfo: 'text'
            }], {
                paper_bgcolor: 'rgba(0,0,0,0)', plot_bgcolor: 'rgba(0,0,0,0)',
                showlegend: false, xaxis: {visible:false, range:[0,11]}, yaxis: {visible:false, range:[0,6]},
                margin: {l:0,r:0,t:0,b:0}, font:{color:'#fff'}
            }, {displayModeBar: false});

            // UPDATE TICKER
            updateTicker(totalVol, totalPpl, critCount);

            // TOCAR SOM SE NECESSÁRIO
            if(alertSoundNeeded) beep('high');
        }

        function updateTicker(vol, ppl, crit) {
            const el = document.getElementById('ticker-content');
            let statusText = crit > 0 ? `<span style="color:red">⚠️ ALERTA: ${crit} SETORES CRÍTICOS</span>` : `<span style="color:#00ff00">✅ OPERAÇÃO ESTÁVEL</span>`;
            
            let html = `
                <span class="ticker-item">VOLUME TOTAL: ${vol.toLocaleString()}</span>
                <span class="ticker-item">HEADCOUNT: ${ppl}</span>
                <span class="ticker-item">${statusText}</span>
                <span class="ticker-item">META PADRÃO: ${META_UPH} UPH</span>
                <span class="ticker-item">DHL CONTROL TOWER V8.0</span>
            `;
            // Atualiza apenas se mudar para evitar "pulo" na animação
            if(!el.innerHTML.includes(vol.toLocaleString()) || crit > 0) el.innerHTML = html + html; 
        }
    </script>
</body>
</html>
  
