<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>OLHO DE DEUS | DHL CONTROL TOWER</title>
    <script src="https://cdn.plot.ly/plotly-2.27.0.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@700&family=Roboto:wght@400;900&display=swap" rel="stylesheet">
    
    <style>
        /* ESTILO COMANDO MILITAR DARK */
        body { background-color: #000; color: #fff; font-family: 'Roboto', sans-serif; margin: 0; padding: 15px; overflow: hidden; }
        
        /* HEADER */
        header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #333; padding-bottom: 15px; margin-bottom: 15px; }
        .logo { font-family: 'Orbitron', sans-serif; font-size: 26px; color: #ffcc00; letter-spacing: 2px; text-shadow: 0 0 10px #ffcc00; }
        .status-badge { background: #111; border: 1px solid #333; padding: 5px 15px; font-size: 12px; color: #888; border-radius: 4px; }
        .blink { animation: blinker 1.5s linear infinite; color: red; font-weight: bold; }
        @keyframes blinker { 50% { opacity: 0; } }

        /* LAYOUT PRINCIPAL */
        .main-grid { display: grid; grid-template-columns: 65% 35%; gap: 20px; height: 85vh; }
        
        /* ÁREA DO MAPA */
        .map-box { background: #0a0a0a; border: 1px solid #333; border-radius: 8px; position: relative; }
        
        /* ÁREA DE COMANDO (DIREITA) */
        .command-box { display: flex; flex-direction: column; gap: 10px; overflow-y: auto; padding-right: 5px; }

        /* CARTÕES DE ILHA (HUD) */
        .hud-card { 
            background: #111; border-left: 5px solid #444; 
            padding: 15px; margin-bottom: 10px; border-radius: 4px; 
            box-shadow: 0 4px 6px rgba(0,0,0,0.5); transition: all 0.3s;
        }
        
        .hud-header { display: flex; justify-content: space-between; font-family: 'Orbitron'; font-size: 18px; margin-bottom: 8px; }
        .hud-metrics { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 5px; font-size: 12px; color: #aaa; text-align: center; }
        .hud-val { font-size: 16px; font-weight: bold; color: #fff; display: block; }
        
        /* BARRA DE PROGRESSO (TEMPO RESTANTE) */
        .progress-bg { height: 6px; background: #333; margin-top: 10px; border-radius: 3px; }
        .progress-bar { height: 100%; transition: width 1s; border-radius: 3px; }

        /* ALERTAS CRÍTICOS */
        .critico { border-color: #ff0000; background: #1a0000; animation: shake 0.5s; }
        .alerta-acao { 
            background: #ff0000; color: #fff; text-align: center; 
            font-weight: bold; padding: 5px; margin-top: 10px; border-radius: 3px; 
            font-size: 14px; letter-spacing: 1px;
        }

        .ok { border-color: #00ff00; }
        .atencao { border-color: #ffff00; }

        @keyframes shake { 0% { transform: translateX(0); } 25% { transform: translateX(5px); } 75% { transform: translateX(-5px); } 100% { transform: translateX(0); } }

    </style>
</head>
<body>

    <header>
        <div class="logo">👁️ OLHO DE DEUS <span style="font-size:12px; color:#666;">v4.0 LIVE</span></div>
        <div class="status-badge"><span class="blink">●</span> CONECTADO À PLANILHA</div>
    </header>

    <div class="main-grid">
        <div class="map-box">
            <div id="tactical-map" style="width:100%; height:100%;"></div>
            <div style="position:absolute; bottom:10px; left:10px; font-size:11px; color:#555;">
                Tamanho da Bolha = Volume Atual | Cor = Risco de Atraso
            </div>
        </div>

        <div class="command-box" id="hud-container">
            <div style="text-align:center; padding:50px; color:#666;">
                Aguardando sincronização com a planilha...
            </div>
        </div>
    </div>

    <script>
        // ====================================================================
        // 🔴 PASSO CRUCIAL: COLE O LINK CSV DA SUA PLANILHA AQUI EMBAIXO:
        const SHEET_URL = 'COLE_SEU_LINK_AQUI';
        // Exemplo: 'https://docs.google.com/spreadsheets/d/e/2PAC.../pub?output=csv'
        // ====================================================================

        // Atualiza a cada 3 segundos (Tempo Real)
        setInterval(() => {
            if(!SHEET_URL.includes('COLE')) loadOperations();
        }, 3000);

        function loadOperations() {
            Papa.parse(SHEET_URL, {
                download: true, header: true, dynamicTyping: true,
                complete: (results) => renderTacticalView(results.data),
                error: (err) => console.log("Erro leitura:", err)
            });
        }

        function renderTacticalView(data) {
            const container = document.getElementById('hud-container');
            container.innerHTML = ""; 

            let mapX=[], mapY=[], mapSize=[], mapColor=[], mapText=[];
            
            // Ordena para mostrar os críticos primeiro
            let processedData = data.filter(r => r.Setor && r.Volume).map(row => {
                // 1. CÁLCULO DE POTÊNCIA (Quantos pacotes a equipe mata por minuto)
                let powerPerMinute = row.Equipe * row.Velocidade; 
                if(powerPerMinute === 0) powerPerMinute = 1; // Evita divisão por zero

                // 2. TEMPO RESTANTE (Realidade)
                let minutesLeft = row.Volume / powerPerMinute;

                // 3. DEFINE STATUS
                let status = "NORMAL";
                if (minutesLeft > row.Meta) status = "CRITICO";
                else if (minutesLeft < (row.Meta * 0.5)) status = "FOLGADO";

                return { ...row, minutesLeft, status, powerPerMinute };
            });

            // Ordena: Críticos no topo
            processedData.sort((a, b) => (a.status === 'CRITICO' ? -1 : 1));

            processedData.forEach(row => {
                let cssClass = "ok";
                let colorHex = "#00ff00";
                let actionMsg = "";
                let progressColor = "#00ff00";

                // LÓGICA DO PROFETA (O que fazer?)
                if (row.status === "CRITICO") {
                    cssClass = "critico";
                    colorHex = "#ff0000";
                    progressColor = "#ff0000";
                    
                    // Cálculo: Quantas pessoas faltam?
                    let neededSpeed = row.Volume / row.Meta; // Velocidade necessária para bater a meta
                    let peopleNeeded = Math.ceil(neededSpeed / row.Velocidade);
                    let add = peopleNeeded - row.Equipe;
                    
                    actionMsg = `🚨 ADICIONAR +${add} PESSOAS AGORA`;
                } else if (row.status === "FOLGADO") {
                    cssClass = "atencao"; // Amarelo
                    colorHex = "#ffff00";
                    progressColor = "#ffff00";
                    actionMsg = "📉 PODE REDUZIR EQUIPE";
                }

                // DADOS PARA O MAPA
                mapX.push(row.X); mapY.push(row.Y);
                mapSize.push(Math.min(row.Volume/10, 80) + 10);
                mapColor.push(colorHex);
                mapText.push(`<b>${row.Setor}</b><br>Vol: ${row.Volume}<br>Restam: ${row.minutesLeft.toFixed(0)}m`);

                // CRIA O CARD DO HUD
                let card = `
                <div class="hud-card ${cssClass}">
                    <div class="hud-header">
                        <span>${row.Setor}</span>
                        <span style="color:${colorHex}">${row.minutesLeft.toFixed(0)} min</span>
                    </div>
                    
                    <div class="hud-metrics">
                        <div><span class="hud-val">${row.Volume}</span>PCTS</div>
                        <div><span class="hud-val">${row.Equipe}</span>PESSOAS</div>
                        <div><span class="hud-val">${row.Velocidade}</span>VEL/MIN</div>
                    </div>

                    ${actionMsg ? `<div class="alerta-acao">${actionMsg}</div>` : ''}

                    <div class="progress-bg">
                        <div class="progress-bar" style="width: ${Math.min((row.minutesLeft/row.Meta)*100, 100)}%; background:${progressColor}"></div>
                    </div>
                    <div style="text-align:right; font-size:10px; color:#666; margin-top:5px;">META: ${row.Meta} min</div>
                </div>`;
                
                container.innerHTML += card;
            });

            updateTacticalMap(mapX, mapY, mapSize, mapColor, mapText);
        }

        function updateTacticalMap(x, y, s, c, t) {
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
            Plotly.react('tactical-map', data, layout, {displayModeBar: false});
        }
        
        // Check inicial
        if(SHEET_URL.includes('COLE')) {
            document.getElementById('hud-container').innerHTML = "<h3 style='text-align:center; color:red'>ERRO: Cole o link CSV da sua planilha no código!</h3>";
        } else {
            loadOperations();
        }
    </script>
</body>
</html>
