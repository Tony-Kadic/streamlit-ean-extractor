import streamlit as st
import pandas as pd

def process_file(uploaded_file):
    try:
        # Detectar el formato del archivo y leerlo
        if uploaded_file.name.endswith('.csv'):
            df = pd.read_csv(uploaded_file, delimiter=";", dtype=str, encoding="latin1")
        elif uploaded_file.name.endswith('.xlsx'):
            df = pd.read_excel(uploaded_file, dtype=str)
        else:
            st.error("Formato de archivo no soportado. Usa CSV o Excel.")
            return None

        # Buscar la columna que contiene los códigos EAN
        ean_column = None
        for col in df.columns:
            if "código" in col.lower():
                ean_column = col
                break

        if ean_column is None:
            st.error("No se encontró una columna con códigos EAN en el archivo.")
            return None

        # Extraer códigos EAN y limpiar datos
        ean_codes = df[ean_column].dropna().astype(str).tolist()
        ean_codes = [code.strip() for code in ean_codes if code.strip().isdigit() and 12 <= len(code) <= 14]

        if not ean_codes:
            st.error("No se encontraron códigos EAN válidos.")
            return None

        # Generar la secuencia
        sequence = '{productos:,,,"' + '";"'.join(ean_codes) + '",,,,,p.codigo_eurowin,,,}'
        return sequence
    except Exception as e:
        st.error(f"Error procesando el archivo: {e}")
        return None

# Interfaz de Streamlit
st.title("Generador de Secuencia de EAN para Promociones")
st.write("Sube un archivo CSV o Excel con los códigos EAN para generar la secuencia.")

# Cargar archivo
uploaded_file = st.file_uploader("Selecciona un archivo", type=["csv", "xlsx"])

if uploaded_file is not None:
    sequence = process_file(uploaded_file)
    if sequence:
        st.success("Secuencia generada con éxito.")
        st.code(sequence, language="text")
        st.button("Copiar al portapapeles", on_click=lambda: st.write("Usa Ctrl+C para copiar la secuencia."))
