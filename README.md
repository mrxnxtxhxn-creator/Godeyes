<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GOD'S EYE | DHL Real-Time</title>
    <script src="https://cdn.plot.ly/plotly-2.27.0.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=Roboto:wght@300;400;700&display=swap" rel="stylesheet">
    
    <style>
        /* ESTILO DARK MODE */
        body { background-color: #0e1117; color: #e0e0e0; font-family: 'Roboto', sans-serif; padding: 20px; }
        h1, h2, h3 { font-family: 'Orbitron', sans-serif; margin: 0; }
        
        /* Cabeçalho */
        header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #333; padding-bottom: 15px; margin-bottom: 20px; }
        .logo { color: #ffcc00; font-size: 24px; letter-spacing: 2px; }
        .live-indicator { color: #ff0000; font-weight: bold; display: flex; align-items: center; gap: 5px; }
        .dot { height: 10px; width: 10px; background-color: #f00; border-radius: 50%; display: inline-block; animation: blink 1.5s infinite; }
        @keyframes blink { 50% { opacity: 0; } }

        /* Grid */
        .dashboard-grid { display: grid; grid-template-columns: 2fr 1fr; gap: 20px; }
        .card { background-color: #161b22; border: 1px solid #30363d; border-radius: 8px; padding: 20px; box-shadow: 0 4px 6px rgba(0,0,0,0.3); }

        /* Sugestões */
        .action-box { background-color: #2a0e0e; border-left: 5px solid #ff4444; padding: 15px; margin-top: 15px; border-radius: 4px; }
        .move-from { color: #00ff00; font-weight: bold; }
        .move-to { color: #ff4444; font-weight: bold; }
        .count { font-size: 1.5em; color: white; }
        .ok-box { background-color: #0e2a15; border-left: 5px solid #00ff00; padding: 15px; color: #aaffaa; }

        /* Tabela */
        table { width: 100%; border-collapse: collapse; margin-top: 20px; font-size: 0.9em; }
        th, td { padding: 12px; text-align: left; border-bottom: 1px solid #333; }
        th { background-color: #21262d; color: #8b949e; }
        .status-critico { color: #ff4444; font-weight: bold; }
        .status-ocioso { color: #00ff00; font-weight: bold; }
        .status-estavel { color: #58a6ff; }
        
        #lastUpdate { font-size: 0.8em; color: #666; }

        @media (max-width: 768px) { .dashboard-grid { grid-template-columns: 1fr; } }
    </style>
</head>
<body>

    <header>
        <div class="logo">👁️ GOD'S EYE <span style="font-size:0.6em; color:#888;">| REAL-TIME</span></div>
        <div class="live-indicator"><span class="dot"></span> AO VIVO</div>
    </header>

    <div class="dashboard-grid">
        <div class="card">
            <h2>📍 MAPA DA OPERAÇÃO</h2>
            <div id="warehouseMap" style="width:100%; height:450px;"></div>
            <div id="lastUpdate">Aguardando dados...</div>
        </div>

        <div class="card">
            <h2>🔮 O PROFETA SUGERE</h2>
            <div id="prophetSuggestions">Carregando inteligência...</div>
        </div>
    </div>

    <div class="card" style="margin-top: 20px;">
        <h3>📋 DADOS DA PLANILHA</h3>
        <table id="dataTable">
            <thead>
                <tr>
                    <th>Ilha</th><th>Volume</th><th>Equipe</th><th>Horas Restantes</th><th>Status</th>
                </tr>
            </thead>
            <tbody></tbody>
        </table>
    </div>

    <script>
        // ======================================================================
        // 🚨 COLE SEU LINK DA PLANILHA (CSV) AQUI EMBAIXO:
        // ======================================================================
        const SHEET_URL = 'COLE_SEU_LINK_DO_PASSO_2_AQUI'; 
        // Exemplo: 'https://docs.google.com/spreadsheets/d/e/2PACX-.../pub?output=csv'
        
        // Configuração de Intervalo (Atualiza a cada 30 segundos)
        const UPDATE_INTERVAL_MS = 30000; 

        function fetchData() {
            if (SHEET_URL.includes('COLE_SEU_LINK')) {
                alert("Você esqueceu de colar o link da planilha no código!");
                return;
            }

            console.log("Buscando atualizações...");
            
            Papa.parse(SHEET_URL, {
                download: true,
                header: true,
                dynamicTyping: true, // Converte números automaticamente
                complete: function(results) {
                    processData(results.data);
                },
                error: function(err) {
                    console.error("Erro ao ler planilha:", err);
                }
            });
        }

        function processData(rawData) {
            let criticalNodes = [];
            let idleNodes = [];
            let plotDataX = [], plotDataY = [], plotSize = [], plotColor = [], plotText = [];
            
            const tableBody = document.querySelector("#dataTable tbody");
            const suggestionsDiv = document.getElementById("prophetSuggestions");
            
            tableBody.innerHTML = ""; // Limpa tabela antiga

            rawData.forEach(row => {
                // Validação básica para ignorar linhas vazias
                if(!row.Ilha || !row.Volume) return;

                // Cálculos Matemáticos
                let capacidadeTotal = row.Equipe * row.Capacidade;
                let horasParaZerar = (row.Volume / capacidadeTotal).toFixed(1);
                
                // Lógica de Status
                let status = "ESTÁVEL";
                let statusClass = "status-estavel";
                let bubbleColor = "#58a6ff"; 

                if (horasParaZerar > 4) {
                    status = "CRÍTICO (Gargalo)";
                    statusClass = "status-critico";
                    bubbleColor = "#ff4444";
                    criticalNodes.push(row);
                } else if (horasParaZerar < 1) {
                    status = "OCIOSO (Sobra)";
                    statusClass = "status-ocioso";
                    bubbleColor = "#00ff00";
                    idleNodes.push(row);
                }

                // Dados para o Mapa
                plotDataX.push(row.X);
                plotDataY.push(row.Y);
                plotSize.push(horasParaZerar * 15);
                plotColor.push(bubbleColor);
                plotText.push(`<b>${row.Ilha}</b><br>Fila: ${horasParaZerar}h`);

                // Preenche Tabela
                let tr = `<tr>
                    <td>${row.Ilha}</td><td>${row.Volume}</td><td>${row.Equipe}</td>
                    <td>${horasParaZerar}h</td><td class="${statusClass}">${status}</td>
                </tr>`;
                tableBody.innerHTML += tr;
            });

            updateMap(plotDataX, plotDataY, plotText, plotSize, plotColor);
            updateProphet(criticalNodes, idleNodes);
            
            // Atualiza hora
            let now = new Date();
            document.getElementById("lastUpdate").innerText = "Última atualização: " + now.toLocaleTimeString();
        }

        function updateMap(x, y, text, size, color) {
            var trace1 = {
                x: x, y: y, text: text,
                mode: 'markers+text', textposition: 'top center',
                marker: { size: size, color: color, opacity: 0.8, line: { color: 'white', width: 1 } },
                type: 'scatter', hoverinfo: 'text'
            };

            var layout = {
                paper_bgcolor: 'rgba(0,0,0,0)', plot_bgcolor: 'rgba(0,0,0,0)',
                showlegend: false,
                xaxis: { showgrid: false, zeroline: false, showticklabels: false, range: [0, 8] },
                yaxis: { showgrid: false, zeroline: false, showticklabels: false, range: [0, 6] },
                margin: { l: 20, r: 20, b: 20, t: 20 }, font: { color: '#ffffff' }
            };

            Plotly.newPlot('warehouseMap', [trace1], layout, {displayModeBar: false});
        }

        function updateProphet(criticos, ociosos) {
            const div = document.getElementById("prophetSuggestions");
            div.innerHTML = "";

            if (criticos.length > 0 && ociosos.length > 0) {
                criticos.forEach(crit => {
                    let idle = ociosos[0]; 
                    let card = `
                        <div class="action-box">
                            <div style="font-weight:bold; color:#ff9999">🚨 AÇÃO RECOMENDADA</div>
                            <p>Salvar ilha <b>${crit.Ilha}</b>:</p>
                            <div style="display:flex; justify-content:space-between;">
                                <div>MOVER <span class="count">2</span> Pessoas</div>
                                <div style="text-align:right;">
                                    DE: <span class="move-from">${idle.Ilha}</span><br>
                                    PARA: <span class="move-to">${crit.Ilha}</span>
                                </div>
                            </div>
                        </div>`;
                    div.innerHTML += card;
                });
            } else if (criticos.length === 0) {
                div.innerHTML = `<div class="ok-box">✅ Operação Fluindo!</div>`;
            } else {
                div.innerHTML = `<div class="action-box" style="border-color:orange;">⚠️ Faltam recursos! Sem ilhas ociosas.</div>`;
            }
        }

        // Inicia o sistema
        fetchData();
        setInterval(fetchData, UPDATE_INTERVAL_MS); // Roda a cada 30s
    </script>
</body>
</html>
