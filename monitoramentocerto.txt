import streamlit as st
import numpy as np
import pandas as pd
import time
import matplotlib.pyplot as plt

# Configuração da página
st.set_page_config(page_title="Monitor de Sensores", layout="wide")
st.title("📊 Monitoramento de Sensores com Gráficos e Alertas")

# --- Limites configuráveis ---
st.sidebar.header("⚙️ Limites de Operação por Sensor")
limites = {
    "Temperatura": {
        "min": st.sidebar.number_input("Temperatura - mínimo (°C)", value=15.0),
        "max": st.sidebar.number_input("Temperatura - máximo (°C)", value=45.0)
    },
    "Carga Móvel": {
        "min": st.sidebar.number_input("Carga Móvel - mínimo (kN)", value=0.0),
        "max": st.sidebar.number_input("Carga Móvel - máximo (kN)", value=450.0)
    },
    "Carga Distribuída": {
        "min": st.sidebar.number_input("Carga Distribuída - mínimo (kN/m²)", value=0.0),
        "max": st.sidebar.number_input("Carga Distribuída - máximo (kN/m²)", value=5.0)
    },
    "Reação de Apoio": {
        "min": st.sidebar.number_input("Reação de Apoio - mínimo (kN)", value=315.0),
        "max": st.sidebar.number_input("Reação de Apoio - máximo (kN)", value=365.0)
    },
}

# --- Inicialização ---
if "dados" not in st.session_state:
    st.session_state.dados = {
        "Temperatura": np.random.uniform(20, 40),
        "Carga Móvel": np.random.uniform(100, 400),
        "Carga Distribuída": np.random.uniform(1, 4),
        "Reação de Apoio": np.random.uniform(320, 360)
    }

if "historico" not in st.session_state:
    st.session_state.historico = pd.DataFrame(
        columns=["Tempo", "Temperatura", "Carga Móvel", "Carga Distribuída", "Reação de Apoio"]
    )

# --- Função de atualização ---
def atualizar_valores():
    for sensor in st.session_state.dados:
        variacao = np.random.uniform(-10, 10)
        st.session_state.dados[sensor] = round(st.session_state.dados[sensor] + variacao, 2)

# --- Área principal ---
placeholder = st.empty()
tempo = 0

while True:
    atualizar_valores()
    tempo += 1

    # Atualiza histórico
    novo_dado = {"Tempo": tempo}
    novo_dado.update(st.session_state.dados)
    st.session_state.historico = pd.concat(
        [st.session_state.historico, pd.DataFrame([novo_dado])],
        ignore_index=True
    )

    # Mantém apenas os últimos 50 pontos
    if len(st.session_state.historico) > 50:
        st.session_state.historico = st.session_state.historico.iloc[-50:]

    with placeholder.container():
        st.subheader("Leituras Atuais")
        cols = st.columns(4)
        alerta_geral = False

        for i, (sensor, valor) in enumerate(st.session_state.dados.items()):
            lim_min = limites[sensor]["min"]
            lim_max = limites[sensor]["max"]

            if valor < lim_min:
                status = f"⚠️ Abaixo ({valor})"
                alerta_geral = True
            elif valor > lim_max:
                status = f"🚨 Acima ({valor})"
                alerta_geral = True
            else:
                status = f"✅ Normal ({valor})"

            cols[i].markdown(f"### {sensor}")
            progresso = max(0.0, min((valor - lim_min) / (lim_max - lim_min), 1.0))
            cols[i].progress(progresso)
            cols[i].write(f"**Status:** {status}")
            cols[i].write(f"**Limites:** {lim_min} - {lim_max}")

        if alerta_geral:
            st.error("⚠️ ALERTA: Um ou mais sensores estão fora dos limites definidos!")
            # Som automático, sem exibir player
            st.markdown(
                """
                <audio autoplay style="display:none">
                    <source src="https://actions.google.com/sounds/v1/alarms/alarm_clock.ogg" type="audio/ogg">
                </audio>
                """,
                unsafe_allow_html=True
            )

        # --- Gráficos ---
        st.subheader("📊 Histórico dos Sensores")
        fig, ax = plt.subplots(2, 2, figsize=(10, 6))
        sensores = ["Temperatura", "Carga Móvel", "Carga Distribuída", "Reação de Apoio"]

        for i, sensor in enumerate(sensores):
            linha = i // 2
            coluna = i % 2
            ax[linha, coluna].plot(
                st.session_state.historico["Tempo"],
                st.session_state.historico[sensor],
                label=sensor
            )
            ax[linha, coluna].set_title(sensor)
            ax[linha, coluna].grid(True)
            ax[linha, coluna].legend()

        st.pyplot(fig)

    time.sleep(1)
