<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>OLHO DE DEUS | DHL PERFORMANCE 290</title>
    <script src="https://cdn.plot.ly/plotly-2.27.0.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@700&family=Roboto:wght@400;900&display=swap" rel="stylesheet">
    
    <style>
        /* VISUAL CYBERPUNK DARK */
        body { background-color: #050505; color: #fff; font-family: 'Roboto', sans-serif; margin: 0; padding: 10px; overflow: hidden; }
        
        /* HEADER */
        header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #00e5ff; padding-bottom: 10px; margin-bottom: 15px; }
        .brand { font-family: 'Orbitron', sans-serif; font-size: 24px; color: #00e5ff; letter-spacing: 2px; text-shadow: 0 0 10px #00e5ff; }
        .meta-tag { background: #111; border: 1px solid #00e5ff; color: #00e5ff; padding: 5px 15px; border-radius: 4px; font-weight: bold; font-size: 14px; }

        /* GRID */
        .grid { display: grid; grid-template-columns: 2fr 1fr; gap: 15px; height: 88vh; }
        
        /* MAPA */
        .map-container { background: #0a0a0a; border: 1px solid #333; border-radius: 8px; position: relative; }
        
        /* PAINEL LATERAL */
        .side-panel { display: flex; flex-direction: column; gap: 10px; overflow-y: auto; padding-right: 5px; }

        /* CARTÃO DA ILHA */
        .card { 
            background: #111; border-left: 5px solid #333; padding: 15px; 
            border-radius: 4px; position: relative; transition: all 0.3s;
        }
        .card-header { display: flex; justify-content: space-between; font-family: 'Orbitron'; font-size: 16px; margin-bottom: 10px; }
        
        .metrics-row { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 10px; font-size: 12px; color: #aaa; }
        .big-number { font-size: 18px; font-weight: 900; color: #fff; display: block; }
        
        /* ALERTAS */
        .alert-box { 
            text-align: center; font-weight: bold; padding: 8px; 
            margin-top: 10px; border-radius: 3px; font-size: 13px; letter-spacing: 1px;
            animation: pulse 2s infinite;
        }
        @keyframes pulse { 0% { opacity: 1; } 50% { opacity: 0.8; } 100% { opacity: 1; } }

        /* CORES DE STATUS */
        .st-critico { border-color: #ff0000; background: #1a0505; }
        .badge-critico { background: #ff0000; color: #fff; }
        
        .st-ok { border-color: #00ff00; }
        .badge-ok { background: #006600; color: #ccffcc; }
        
        .st-warn { border-color: #ffff00; }
        .badge-warn { background: #999900; color: #fff; }

        /* BARRA DE PROGRESSO */
        .prog-track { height: 4px; background: #333; margin-top: 10px; }
        .prog-fill { height: 100%; width: 50%; transition: width 1s; }

    </style>
</head>
<body>

    <header>
        <div class="brand">⚡ OLHO DE DEUS <span style="font-size:12px; opacity:0.7">v5.0</span></div>
        <div class="meta-tag">META PADRÃO: 290 UPH</div>
    </header>

    <div class="grid">
        <div class="map-container">
            <div id="warehouse-map" style="width:100%; height:100%;"></div>
        </div>

        <div class="side-panel" id="tactical-feed">
            <div style="text-align:center; padding:50px; color:#444;">
                Conectando ao Satélite...
            </div>
        </div>
    </div>

    <script>
        // ====================================================================
        // 🔴 COLE O LINK CSV DA PLANILHA AQUI:
        const SHEET_URL = 'COLE_SEU_LINK_AQUI'; 
        // ====================================================================

        // CONFIGURAÇÃO DE PERFORMANCE
        const UPH_META = 290; // Unidades Por Hora por Pessoa
        const UPM_META = UPH_META / 60; // Unidades Por Minuto (4.83)

        // Loop de atualização (3 segundos)
        setInterval(() => {
            if(!SHEET_URL.includes('COLE')) loadData();
        }, 3000);

        function loadData() {
            Papa.parse(SHEET_URL, {
                download: true, header: true, dynamicTyping: true,
                complete: (results) => renderBoard(results.data)
            });
        }

        function renderBoard(data) {
            const container = document.getElementById('tactical-feed');
            container.innerHTML = "";

            let mapX=[], mapY=[], mapSize=[], mapColor=[], mapText=[];
            
            // Processamento dos Dados
            let sectors = data.filter(r => r.Setor && r.Volume).map(row => {
                
                // 1. CAPACIDADE TEÓRICA (O que a equipe DEVERIA estar fazendo)
                let capacidadeRealMinuto = row.Equipe * UPM_META; 
                
                // 2. TEMPO NECESSÁRIO (Minutos para zerar fila trabalhando a 290 UPH)
                let minutosParaZerar = 0;
                if(capacidadeRealMinuto > 0) {
                    minutosParaZerar = row.Volume / capacidadeRealMinuto;
                } else {
                    minutosParaZerar = 999; // Sem equipe = Tempo infinito
                }

                // 3. ANÁLISE DO PROFETA (Meta vs Realidade)
                let status = "NORMAL";
                
                // Se o tempo para zerar for maior que o tempo limite (Meta da planilha)
                if (minutosParaZerar > row.Meta) status = "CRITICO";
                else if (minutosParaZerar < (row.Meta * 0.6)) status = "FOLGA";

                return { ...row, minutosParaZerar, status, capacidadeRealMinuto };
            });

            // Ordena: Críticos primeiro
            sectors.sort((a, b) => (a.status === 'CRITICO' ? -1 : 1));

            sectors.forEach(row => {
                let css = "st-ok";
                let color = "#00ff00";
                let alertMsg = "";
                let alertClass = "";

                if (row.status === "CRITICO") {
                    css = "st-critico";
                    color = "#ff0000";
                    
                    // CÁLCULO DE REFORÇO NECESSÁRIO
                    // Quantos pacotes/minuto eu preciso para bater a meta?
                    let velocidadeNecessaria = row.Volume / row.Meta;
                    // Quantas pessoas batendo 290 UPH eu preciso?
                    let pessoasNecessarias = Math.ceil(velocidadeNecessaria / UPM_META);
                    let gap = pessoasNecessarias - row.Equipe;

                    if (gap > 0) {
                        alertMsg = `🚨 ADICIONAR +${gap} PESSOAS`;
                        alertClass = "badge-critico";
                    } else {
                        // Se não falta gente, mas tá atrasado -> A EQUIPE TÁ LENTA
                        alertMsg = `⚡ COBRAR RITMO (BAIXA PRODUTIVIDADE)`;
                        alertClass = "badge-critico";
                        // Truque visual: pinta de laranja pra diferenciar de falta de gente
                        color = "#ff9900"; 
                        css = "st-warn";
                    }

                } else if (row.status === "FOLGA") {
                    css = "st-warn";
                    color = "#ffff00";
                    alertMsg = "📉 EXCESSO DE EQUIPE (REDUZIR)";
                    alertClass = "badge-warn";
                } else {
                    alertMsg = "✅ OPERAÇÃO ESTÁVEL";
                    alertClass = "badge-ok";
                }

                // MAPA
                mapX.push(row.X); mapY.push(row.Y);
                mapSize.push(Math.min(row.Volume/20, 90));
                mapColor.push(color);
                mapText.push(`<b>${row.Setor}</b><br>Vol: ${row.Volume}<br>Tempo: ${row.minutosParaZerar.toFixed(0)}m`);

                // CARD
                let html = `
                <div class="card ${css}">
                    <div class="card-header">
                        <span>${row.Setor}</span>
                        <span style="color:${color}">${row.minutosParaZerar.toFixed(0)} min</span>
                    </div>
                    
                    <div class="metrics-row">
                        <div>
                            <span>VOLUME</span>
                            <span class="big-number">${row.Volume}</span>
                        </div>
                        <div style="text-align:right;">
                            <span>EQUIPE</span>
                            <span class="big-number">${row.Equipe}</span>
                        </div>
                    </div>
                    
                    <div class="alert-box ${alertClass}">
                        ${alertMsg}
                    </div>

                    <div class="prog-track">
                        <div class="prog-fill" style="width:${Math.min((row.minutosParaZerar/row.Meta)*100, 100)}%; background:${color}"></div>
                    </div>
                    <div style="text-align:right; font-size:10px; color:#666; margin-top:4px;">META TEMPO: ${row.Meta} min</div>
                </div>`;

                container.innerHTML += html;
            });

            updateMap(mapX, mapY, mapSize, mapColor, mapText);
        }

        function updateMap(x, y, s, c, t) {
            var data = [{
                x: x, y: y, text: t,
                mode: 'markers+text', textposition: 'top center',
                marker: { size: s, color: c, opacity: 0.8, line: {color: '#fff', width: 1} },
                type: 'scatter', hoverinfo: 'text'
            }];
            var layout = {
                paper_bgcolor: 'rgba(0,0,0,0)', plot_bgcolor: 'rgba(0,0,0,0)',
                showlegend: false, xaxis: {visible:false, range:[0,11]}, yaxis: {visible:false, range:[0,6]},
                margin: {l:0,r:0,t:0,b:0}, font:{color:'#fff'}
            };
            Plotly.react('warehouse-map', data, layout, {displayModeBar: false});
        }

        if(SHEET_URL.includes('COLE')) {
            document.getElementById('tactical-feed').innerHTML = "<h2 style='text-align:center; color:red; margin-top:50px;'>⚠️ COLE O LINK DA PLANILHA NO CÓDIGO</h2>";
        } else {
            loadData();
        }
    </script>
</body>
</html>
