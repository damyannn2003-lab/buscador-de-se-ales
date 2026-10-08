import streamlit as st

import pandas as pd

import numpy as np

import yfinance as yf

import plotly.graph_objects as go

# Configuración de la página

st.set_page_config(

    page_title="Buscador de Señales de Opciones",

    page_icon="📈",

    layout="wide"

)

# Estilo y Recordatorio Emocional (Regla fundamental del seminario)

st.title("🎯 Buscador Automático de Señales para Opciones")

st.markdown("""

> **🧘‍♂️ REVISIÓN EMOCIONAL:** *Es muy importante trabajar las emociones y la mente para que los resultados sean los correctos. No te apresures y espera al momento correcto para abrir una operación por el cumplimiento de una estrategia.*

""")

# Diccionario ampliado con Siglas y Nombre Completo de los activos

ASSETS_DICT = {

    "SPY": "SPDR S&P 500 ETF Trust",

    "QQQ": "Invesco QQQ Trust (Nasdaq 100)",

    "DIA": "SPDR Dow Jones Industrial Average ETF",

    "GLD": "SPDR Gold Shares",

    "SLV": "iShares Silver Trust",

    "USO": "United States Oil Fund",

    "AAPL": "Apple Inc.",

    "META": "Meta Platforms, Inc.",

    "NFLX": "Netflix, Inc.",

    "AMZN": "Amazon.com, Inc.",

    "NVDA": "NVIDIA Corporation",

    "TSLA": "Tesla, Inc.",

    "MRNA": "Moderna, Inc.",

    "CVX": "Chevron Corporation"

}

st.sidebar.header("⚙️ Configuración del Escáner")

selected_tickers = st.sidebar.multiselect(

    "Activos a monitorizar:",

    options=list(ASSETS_DICT.keys()),

    default=list(ASSETS_DICT.keys()),

    format_func=lambda x: f"{x} - {ASSETS_DICT[x]}"

)

# Función para descargar datos y calcular indicadores

@st.cache_data(ttl=3600)

def analyze_market(tickers):

    signals = []

    market_data = {}

    

    for ticker in tickers:

        try:

            # Descargar datos diarios y horarios

            data_daily = yf.download(ticker, period="1y", interval="1d", progress=False)

            data_hourly = yf.download(ticker, period="1mo", interval="60m", progress=False)

            

            if data_daily.empty or data_hourly.empty:

                continue

                

            # Aplanar columnas si vienen en MultiIndex

            if isinstance(data_daily.columns, pd.MultiIndex):

                data_daily.columns = data_daily.columns.get_level_values(0)

            if isinstance(data_hourly.columns, pd.MultiIndex):

                data_hourly.columns = data_hourly.columns.get_level_values(0)

            # Cálculos Diarios

            data_daily['SMA_100'] = data_daily['Close'].rolling(window=100).mean()

            data_daily['SMA_200'] = data_daily['Close'].rolling(window=200).mean()

            

            # Cálculos Horarios

            data_hourly['SMA_20'] = data_hourly['Close'].rolling(window=20).mean()

            data_hourly['SMA_40'] = data_hourly['Close'].rolling(window=40).mean()

            

            # Guardar dataframes para gráficos posteriores

            market_data[ticker] = {

                "daily": data_daily,

                "hourly": data_hourly

            }

            

            # Últimos valores

            current_price = float(data_daily['Close'].iloc[-1])

            sma100_d = float(data_daily['SMA_100'].iloc[-1])

            sma200_d = float(data_daily['SMA_200'].iloc[-1])

            

            sma20_h = float(data_hourly['SMA_20'].iloc[-1])

            sma40_h = float(data_hourly['SMA_40'].iloc[-1])

            

            # --- EVALUACIÓN DE ESTRATEGIAS ---

            

            # 1. Estrategia de Piso Fuerte (Diario: SMA 100 > SMA 200, precio cerca de SMA 200)

            piso_fuerte_activa = False

            distancia_sma200 = abs(current_price - sma200_d) / sma200_d

            if sma100_d > sma200_d and distancia_sma200 < 0.04: 

                piso_fuerte_activa = True

                

            # 2. Estrategia Promedio Móvil de 40 (Horario: SMA 20 sobre SMA 40 y toque reciente)

            pm_40_activa = False

            if sma20_h > sma40_h:

                recent_lows = data_hourly['Low'].iloc[-5:].min()

                recent_sma40 = data_hourly['SMA_40'].iloc[-5:].min()

                if abs(recent_lows - recent_sma40) / recent_sma40 < 0.02:

                    pm_40_activa = True

            # Guardar resultados si cumple alguna estrategia

            if piso_fuerte_activa or pm_40_activa:

                estrategia_encontrada = []

                if piso_fuerte_activa:

                    estrategia_encontrada.append("🔥 Piso Fuerte (Diario)")

                if pm_40_activa:

                    estrategia_encontrada.append("⚡ Promedio Móvil 40 (1h)")

                

                signals.append({

                    "Ticker": ticker,

                    "Nombre": ASSETS_DICT.get(ticker, ticker),

                    "Precio Actual": round(current_price, 2),

                    "Estrategia Detectada": " + ".join(estrategia_encontrada),

                    "Tendencia Base": "Alcista (SMA 100 > 200)" if sma100_d > sma200_d else "Revisar"

                })

                

        except Exception as e:

            print(f"Error procesando {ticker}: {e}")

            

    return pd.DataFrame(signals), market_data

# Botón para ejecutar el escaneo en vivo

if st.button("🚀 Escanear Mercado en Vivo", type="primary"):

    with st.spinner("Analizando acciones, nombres y gráficos... Por favor espere."):

        df_results, market_data = analyze_market(selected_tickers)

        

        st.subheader("📊 Resultados y Señales Activas")

        if not df_results.empty:

            st.success(f"¡Se encontraron {len(df_results)} oportunidades cumpliendo las reglas!")

            

            # Mostrar tabla detallada con nombres completos

            st.dataframe(df_results, use_container_width=True)

            

            st.markdown("---")

            st.subheader("📉 Gráficos de Cotización de las Señales Detectadas")

            

            # Renderizar un gráfico interactivo para cada activo con señal

            for idx, row in df_results.iterrows():

                ticker = row["Ticker"]

                nombre = row["Nombre"]

                

                st.markdown(f"### {ticker} - {nombre} ({row['Estrategia Detectada']})")

                

                df_d = market_data[ticker]["daily"].tail(180) # Últimos 6 meses diarios

                

                fig = go.Figure()

                # Vela diaria

                fig.add_trace(go.Candlestick(

                    x=df_d.index,

                    open=df_d['Open'],

                    high=df_d['High'],

                    low=df_d['Low'],

                    close=df_d['Close'],

                    name='Precio Diario'

                ))

                # Medias móviles

                fig.add_trace(go.Scatter(x=df_d.index, y=df_d['SMA_100'], line=dict(color='orange', width=1.5), name='SMA 100'))

                fig.add_trace(go.Scatter(x=df_d.index, y=df_d['SMA_200'], line=dict(color='blue', width=1.5), name='SMA 200'))

                

                fig.update_layout(

                    title=f"Gráfico Diario - {nombre}",

                    xaxis_title="Fecha",

                    yaxis_title="Precio (USD)",

                    template="plotly_dark",

                    height=450,

                    margin=dict(l=20, r=20, t=40, b=20)

                )

                

                st.plotly_chart(fig, use_container_width=True)

            

            st.markdown("### 📝 Instrucciones de Entrada para Opciones:")

            st.info("""

            * **Plazo sugerido para opciones:** Darles entre **7 y 14 días de plazo** como se indica en el seminario.

            * **Confirmación:** Asegúrate de validar que exista la **vela verde de ruptura** antes de confirmar la compra.

            """)

        else:

            st.warning("No se detectaron activos cumpliendo estrictamente los parámetros exactos en este preciso instante. La paciencia es parte del trading exitoso.")

else:

    st.info("Presiona el botón **'Escanear Mercado en Vivo'** para analizar automáticamente las acciones con nombres completos y gráficos interactivos.")
