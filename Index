import streamlit as st
import pandas as pd
import plotly.express as px

# --- CONFIGURAÇÃO DA PÁGINA ---
st.set_page_config(layout="wide", page_title="God's Eye - DHL Prophet", page_icon="👁️")

# Estilo CSS para visual profissional
st.markdown("""
<style>
    .big-font { font-size:20px !important; }
    .action-box { background-color: #440000; padding: 20px; border-radius: 10px; border-left: 5px solid #ff0000; }
</style>
""", unsafe_allow_html=True)

st.title("👁️ GOD'S EYE: Orquestrador Logístico em Tempo Real")

# --- 1. FONTE DE DADOS (SIMULAÇÃO DA PLANILHA GOOGLE) ---
# Aqui simulamos o que o script leria do Google Sheets dos coordenadores
data = {
    'Ilha': ['Recebimento A', 'Recebimento B', 'Triagem 1', 'Triagem 2', 'Expedição Z'],
    'Volume_Pacotes': [200, 3500, 150, 4500, 800],   # Carga atual
    'Colaboradores': [5, 4, 6, 3, 4],                # Staff atual
    'Capacidade_Pessoa': [100, 100, 120, 120, 150],  # Produtividade média/hora
    'Posicao_X': [1, 1, 3, 3, 5],                    # Layout físico (Eixo X)
    'Posicao_Y': [3, 1, 3, 1, 2]                     # Layout físico (Eixo Y)
}
df = pd.DataFrame(data)

# --- 2. O CÉREBRO (CÁLCULOS PREDITIVOS) ---
# Capacidade Total da Ilha = Pessoas * Velocidade Individual
df['Capacidade_Total'] = df['Colaboradores'] * df['Capacidade_Pessoa']

# Horas para Zerar a Fila (Backlog)
df['Horas_Para_Zerar'] = df['Volume_Pacotes'] / df['Capacidade_Total']

# Definição de Status (Lógica do Semáforo)
def definir_status(horas):
    if horas > 4: return 'CRÍTICO (Gargalo)'  # Vai explodir (Vermelho)
    if horas < 1: return 'OCIOSO (Sobra)'     # Gente parada (Verde)
    return 'ESTÁVEL'                          # Normal (Azul)

df['Status'] = df['Horas_Para_Zerar'].apply(definir_status)

# --- 3. A VISUALIZAÇÃO (DASHBOARD) ---
col_mapa, col_acao = st.columns([3, 2])

with col_mapa:
    st.subheader("📍 Mapa de Calor da Operação")
    # Gráfico de Bolhas Interativo
    fig = px.scatter(df, x="Posicao_X", y="Posicao_Y",
                     size="Horas_Para_Zerar",  # Tamanho da bolha = Tamanho do Problema
                     color="Status",
                     hover_name="Ilha",
                     text="Ilha",
                     color_discrete_map={
                         "CRÍTICO (Gargalo)": "#FF0000", # Vermelho
                         "ESTÁVEL": "#00CCFF",           # Azul
                         "OCIOSO (Sobra)": "#00FF00"     # Verde
                     },
                     size_max=80,
                     template="plotly_dark")
    
    fig.update_layout(showlegend=True, height=500, xaxis_visible=False, yaxis_visible=False)
    fig.update_traces(textposition='top center')
    st.plotly_chart(fig, use_container_width=True)

with col_acao:
    st.subheader("🔮 O Profeta Sugere:")
    st.markdown("---")
    
    # Lógica de Sugestão de Movimentação (Algoritmo de Balanceamento)
    criticos = df[df['Status'] == 'CRÍTICO (Gargalo)']
    ociosos = df[df['Status'] == 'OCIOSO (Sobra)']
    
    sugestao_existente = False
    
    if not criticos.empty and not ociosos.empty:
        for i, row_c in criticos.iterrows():
            # Procura uma ilha ociosa para ajudar
            if not ociosos.empty:
                row_o = ociosos.iloc[0] # Pega a primeira ilha ociosa disponível
                
                # Cálculo simples de quantos mover
                pessoas_mover = 2 # Simplificação para o MVP
                
                st.markdown(f"""
                <div class="action-box">
                    <h3>🚨 AÇÃO IMEDIATA NECESSÁRIA</h3>
                    <p>Para evitar atraso na ilha <b>{row_c['Ilha']}</b>:</p>
                    <h2>➡️ Mover {pessoas_mover} Pessoas</h2>
                    <p>DE: <b style="color:#00FF00">{row_o['Ilha']}</b> (Está Ociosa)</p>
                    <p>PARA: <b style="color:#FF0000">{row_c['Ilha']}</b> (Está Crítica)</p>
                </div>
                """, unsafe_allow_html=True)
                sugestao_existente = True
                
    if not sugestao_existente:
        if not criticos.empty:
            st.warning("⚠️ Há gargalos, mas nenhuma ilha está ociosa! Considere Hora Extra.")
        else:
            st.success("✅ Operação perfeitamente balanceada. Bom trabalho.")

# --- 4. DETALHE DOS DADOS ---
st.markdown("### 📋 Dados em Tempo Real (Espelho da Planilha)")
st.dataframe(df.style.applymap(
    lambda v: 'color: red; font-weight: bold;' if v == 'CRÍTICO (Gargalo)' else 
              ('color: green; font-weight: bold;' if v == 'OCIOSO (Sobra)' else None), 
    subset=['Status']
))
