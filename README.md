<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>OLHO DE DEUS | DHL CONTROL TOWER</title>
    <script src="https://cdn.plot.ly/plotly-2.27.0.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@700&family=Roboto:wght@300;400;900&display=swap" rel="stylesheet">
    
    <style>
        /* ESTILO PROFISSIONAL (Glassmorphism) */
        :root {
            --primary: #ffcc00; /* Amarelo DHL */
            --bg-dark: #0f0f13;
            --glass: rgba(255, 255, 255, 0.05);
            --border: rgba(255, 255, 255, 0.1);
        }

        body { 
            background-color: var(--bg-dark); 
            background-image: radial-gradient(circle at 50% 50%, #1a1a2e 0%, #000 100%);
            color: #fff; 
            font-family: 'Roboto', sans-serif; 
            margin: 0; 
            padding: 20px; 
            height: 100vh;
            overflow: hidden;
            box-sizing: border-box;
        }
        
        /* HEADER COM INPUT */
        header { 
            display: flex; 
            justify-content: space-between; 
            align-items: center; 
            background: var(--glass);
            backdrop-filter: blur(10px);
            border: 1px solid var(--border);
            padding: 15px 25px; 
            border-radius: 12px;
            margin-bottom: 20px; 
            box-shadow: 0 4px 30px rgba(0, 0, 0, 0.3);
        }

        .brand { 
            font-family: 'Orbitron', sans-serif; 
            font-size: 24px; 
            color: var(--primary); 
            letter-spacing: 2px;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        /* ÁREA DE CONEXÃO (NOVIDADE) */
        .connection-area {
            display: flex;
            gap: 10px;
            background: rgba(0,0,0,0.3);
            padding: 5px;
            border-radius: 8px;
            border: 1px solid var(--border);
        }

        #sheet-input {
            background: transparent;
            border: none;
            color: #fff;
            padding: 8px 15px;
            width: 300px;
            font-size: 14px;
            outline: none;
        }

        #connect-btn {
            background: var(--primary);
            color: #000;
            border: none;
            padding: 8px 20px;
            border-radius: 6px;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.2s;
        }
        #connect-btn:hover { background: #e6b800; transform: scale(1.05); }

        /* GRID PRINCIPAL */
        .main-grid { 
            display: grid; 
            grid-template-columns: 70% 30%; 
            gap: 20px; 
            height: calc(100vh - 120px); 
        }
        
        /* BLOCOS */
        .panel { 
            background: var(--glass);
            border: 1px solid var(--border);
            border-radius: 12px; 
            position: relative; 
            overflow: hidden;
            backdrop-filter: blur(5px);
        }
        
        /* MAPA */
        #tactical-map { width: 100%; height: 100%; }

        /* PAINEL LATERAL (HUD) */
        .hud-container { 
            padding: 15px; 
            overflow-y: auto; 
            display: flex; 
            flex-direction: column; 
            gap: 12px; 
        }

        /* CARTÕES DAS ILHAS */
        .island-card { 
            background: rgba(0, 0, 0, 0.4); 
            border-left: 4px solid #444; 
            padding: 15px; 
            border-radius: 6px; 
            transition: transform 0.2s;
        }
        .island-card:hover { transform: translateX(5px); background: rgba(255,255,255,0.03); }
        
        .card-top { display: flex; justify-content: space-between; margin-bottom: 10px; font-family: 'Orbitron'; font-size: 16px; }
        
        .metrics { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; font-size: 12px; color: #aaa; margin-bottom: 10px; }
        .metric-val { font-size: 18px; color: #fff; font-weight: bold; }
        
        /* ALERTAS VISUAIS */
        .action-alert { 
            background: linear-gradient(90deg, #300, #500); 
            border: 1px solid #ff0000; 
            color: #ffcccc; 
            padding: 8px; 
            text-align: center; 
            font-weight: bold; 
            font-size: 13px; 
            border-radius: 4px;
            animation: pulse 2s infinite;
        }
        @keyframes pulse { 0% { box-shadow: 0 0 0 0 rgba(255, 0, 0, 0.4); } 70% { box-shadow: 0 0 0 10px rgba(255, 0, 0, 0); } 100% { box-shadow: 0 0 0 0 rgba(255, 0, 0, 0); } }

        /* STATUS CORES */
        .st-critico { border-color: #ff3333; }
        .st-ok { border-color: #00ff00; }
        .st-atencao { border-color: #ffff00; }

        /* BARRA DE PROGRESSO */
        .progress-bar { height: 4px; background: #333; border-radius: 2px; overflow: hidden; }
        .fill { height: 100%; width: 50%; transition: width 1s ease-in-out; }

        /* SCROLLBAR PERSONALIZADA */
        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-track { background: #000; }
        ::-webkit-scrollbar-thumb { background: #333; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #555; }

        /* MENSAGEM INICIAL */
        .empty-state {
            display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%; color: #666; text-align: center;
        }
        .empty-icon { font-size: 48px; margin-bottom: 15px; opacity: 0.5; }

    </style>
</head>
<body>

    <header>
        <div class="brand">
            <span style="font-size: 28px;">👁️</span> 
            <div>
                OLHO DE DEUS <br>
                <span style="font-size: 10px; color: #888; letter-spacing: 1px;">SISTEMA TÁTICO LOGÍSTICO</span>
            </div>
        </div>

        <div class="connection-area">
            <input type="text" id="sheet-input" placeholder="Cole o Link CSV da Planilha aqui...">
            <button id="connect-btn" onclick="connectSheet()">CONECTAR</button>
        </div>
    </header>

    <div class="main-grid">
        <div class="panel">
            <div id="tactical-map"></div>
            <div id="loading-map" class="empty-state">
                <div class="empty-icon">🗺️</div>
                <div>Aguardando dados...</div>
            </div>
        </div>

        <div class="panel hud-container" id="hud-feed">
            <div class="empty-state">
                <div class="empty-icon">📡</div>
                <div>Cole o link acima para iniciar<br>o monitoramento em tempo real.</div>
            </div>
        </div>
    </div>

    <script>
        // CONFIGURAÇÃO DE META
        const META_UPH = 290; // Meta padrão de produtividade
        const META_UPM = META_UPH / 60; // Unidades por minuto (4.83)

        let updateTimer = null;
        let currentSheetUrl = "";

        // Tenta carregar link salvo anteriormente
        window.onload = function() {
            const savedUrl = localStorage.getItem('dhl_sheet_url');
            if(savedUrl) {
                document.getElementById('sheet-input').value = savedUrl;
                connectSheet(); // Conecta automático se já tiver salvo
            }
        }

        function connectSheet() {
            const url = document.getElementById('sheet-input').value.trim();
            
            if (!url) {
                alert("Por favor, cole o link CSV da planilha!");
                return;
            }
            if (!url.includes("output=csv")) {
                alert("Atenção: O link deve terminar com 'output=csv'. Verifique se você publicou a planilha corretamente.");
            }

            currentSheetUrl = url;
            localStorage.setItem('dhl_sheet_url', url); // Salva no navegador
            
            // Muda botão visualmente
            const btn = document.getElementById('connect-btn');
            btn.innerText = "CONECTADO";
            btn.style.background = "#00ff00";

            // Inicia leitura
            fetchData();
            
            // Limpa timer anterior se houver e inicia novo
            if (updateTimer) clearInterval(updateTimer);
            updateTimer = setInterval(fetchData, 3000); // Atualiza a cada 3s
        }

        function fetchData() {
            Papa.parse(currentSheetUrl, {
                download: true,
                header: true,
                dynamicTyping: true,
                complete: (results) => renderDashboard(results.data),
                error: (err) => console.log("Erro de conexão:", err)
            });
        }

        function renderDashboard(data) {
            const container = document.getElementById('hud-feed');
            const mapContainer = document.getElementById('tactical-map');
            const loadingMap = document.getElementById('loading-map');

            // Limpa estados vazios na primeira carga
            if (loadingMap) loadingMap.style.display = 'none';
            container.innerHTML = "";

            let mapX=[], mapY=[], mapSize=[], mapColor=[], mapText=[];
            
            // Processa dados e ordena por criticidade
            let processed = data.filter(r => r.Setor && r.Volume).map(row => {
                // Cálculo Matemático
                let capMinuto = row.Equipe * META_UPM;
                let tempoRestante = capMinuto > 0 ? (row.Volume / capMinuto) : 999;
                
                let status = "NORMAL";
                if (tempoRestante > row.Meta) status = "CRITICO";
                else if (tempoRestante < row.Meta * 0.6) status = "FOLGA";

                return { ...row, tempoRestante, status };
            });

            processed.sort((a, b) => (a.status === 'CRITICO' ? -1 : 1));

            // Renderiza Cards e Prepara Mapa
            processed.forEach(row => {
                let css = "st-ok";
                let color = "#00ff00";
                let msg = "";

                if (row.status === "CRITICO") {
                    css = "st-critico";
                    color = "#ff3333";
                    
                    // Cálculo de Reforço
                    let velNecessaria = row.Volume / row.Meta;
                    let pessoasNecessarias = Math.ceil(velNecessaria / META_UPM);
                    let gap = pessoasNecessarias - row.Equipe;

                    if (gap > 0) msg = `🚨 ADICIONAR +${gap} PESSOAS`;
                    else msg = `⚡ COBRAR RITMO DA EQUIPE`;
                } 
                else if (row.status === "FOLGA") {
                    css = "st-atencao";
                    color = "#ffff00";
                    msg = "📉 REDUZIR EQUIPE";
                }

                // Dados Mapa
                mapX.push(row.X); mapY.push(row.Y);
                mapSize.push(Math.min(row.Volume/20, 90) + 10);
                mapColor.push(color);
                mapText.push(`<b>${row.Setor}</b><br>Vol: ${row.Volume}<br>${row.tempoRestante.toFixed(0)} min`);

                // HTML do Card
                let html = `
                <div class="island-card ${css}">
                    <div class="card-top">
                        <span>${row.Setor}</span>
                        <span style="color:${color}">${row.tempoRestante.toFixed(0)} min</span>
                    </div>
                    <div class="metrics">
                        <div>VOLUME <br><span class="metric-val">${row.Volume}</span></div>
                        <div style="text-align:right">EQUIPE <br><span class="metric-val">${row.Equipe}</span></div>
                    </div>
                    
                    ${msg ? `<div class="action-alert" style="border-color:${color}; color:${color === '#ffff00' ? '#ffff00' : '#ffcccc'}; background:rgba(0,0,0,0.5)">${msg}</div>` : ''}

                    <div style="margin-top:10px;">
                        <div class="progress-bar">
                            <div class="fill" style="width:${Math.min((row.tempoRestante/row.Meta)*100, 100)}%; background:${color}"></div>
                        </div>
                        <div style="text-align:right; font-size:10px; color:#666; margin-top:3px">META: ${row.Meta} min</div>
                    </div>
                </div>`;
                
                container.innerHTML += html;
            });

            // Atualiza Mapa
            var trace = {
                x: mapX, y: mapY, text: mapText,
                mode: 'markers+text', textposition: 'top center',
                marker: { size: mapSize, color: mapColor, opacity: 0.8, line: {color: '#fff', width: 1} },
                type: 'scatter', hoverinfo: 'text'
            };
            var layout = {
                paper_bgcolor: 'rgba(0,0,0,0)', plot_bgcolor: 'rgba(0,0,0,0)',
                showlegend: false, xaxis: {visible:false, range:[0,11]}, yaxis: {visible:false, range:[0,6]},
                margin: {l:0,r:0,t:0,b:0}, font:{color:'#fff'}
            };
            Plotly.react('tactical-map', [trace], layout, {displayModeBar: false});
        }
    </script>
</body>
</html>
