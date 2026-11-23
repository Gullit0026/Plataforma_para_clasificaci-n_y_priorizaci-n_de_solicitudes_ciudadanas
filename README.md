# Clasificación y priorización de solicitudes ciudadanas con NLP

Proyecto final del curso de **Big Data** (Universidad Autónoma de Occidente, 2026).
Docente: PhD (c). Julián René Muñoz Burbano.
Autor: Julian Esteban Salamanca Angelone

Prototipo que recibe una queja en texto libre, predice su **tema** con un modelo DistilBERT, estima su **urgencia** con reglas, la asigna a una **dependencia** mediante una tabla fija y calcula una **fecha límite de atención (SLA)**. Se compara contra un baseline TF-IDF + Regresión Logística.

> **Alcance.** Es un prototipo académico, no un sistema productivo. Usa como caso análogo las quejas de consumidores del CFPB (en inglés), no solicitudes ciudadanas en español. Ver [Limitaciones](#limitaciones).

## Resultados

Conjunto de prueba: 7.324 quejas, 7 temas. Una sola corrida, semilla 42.

| Modelo | Macro-F1 | Accuracy |
|---|---|---|
| TF-IDF + Regresión Logística (baseline) | 0,7964 | 0,8284 |
| DistilBERT (fine-tuning, 2 épocas) | 0,8130 | 0,8479 |

- Latencia de inferencia del modelo en GPU Tesla T4: p50 6,7 ms, p95 11,3 ms (200 quejas; no incluye tokenización ni red).
- La mejora de DistilBERT es moderada y no se evaluó su significancia estadística.
- Detalle por clase, matrices de confusión y análisis de errores: ver el artículo en `docs/`.

## Contenido del repositorio

```
.
├── Plataforma_para_clasificación_y_priorización_de_solicitudes_ciudadanas.ipynb                  # Código completo (Colab): datos, baseline, DistilBERT, urgencia/SLA y demo
├── README.md
├── DATA_CARD.md                                                                                  # Procedencia, procesamiento y limitaciones de los datos
├── requirements.txt
└── docs/
    ├── Articulo_Clasificacion_Solicitudes.docx
    ├── cm_baseline.png                                                                           # Matriz de confusión del baseline
    ├── cm_distilbert.png                                                                         # Matriz de confusión de DistilBERT
    ├── demo_1.png                                                                                # Capturas de la demo
    └── demo_2.png
```

El modelo entrenado (unos 250 MB) **no está en el repositorio**. Descarga: https://colab.research.google.com/drive/1Hk8h4c6AwrkhcfmtCOlXNKFRCpSXVAk2

## Cómo reproducir

Se desarrolló en **Google Colab con GPU Tesla T4**. Se recomienda ejecutarlo allí.

1. Abre `notebook.ipynb` en Colab (Archivo → Subir cuaderno).
2. Activa la GPU: Entorno de ejecución → Cambiar tipo de entorno de ejecución → T4.
3. Instala las dependencias (primera celda, o `pip install -r requirements.txt`).
4. Ejecuta las celdas en orden:

| Celda | Qué hace |
|---|---|
| 1 | Instala librerías |
| 2 | Inspecciona las columnas del dataset (streaming) |
| 3 | Verifica la GPU |
| 4 | Descarga una muestra con narrativa desde Hugging Face y la guarda en Google Drive |
| 5 | Exploración rápida (productos, longitud de texto, duplicados) |
| 6 | Agrupa temas, limpia texto, deduplica y divide 70/15/15 (estratificado) |
| 7 | Entrena y evalúa el baseline |
| 8 | Fine-tuning de DistilBERT y guardado del modelo en Drive |
| 9 | Evaluación en prueba, comparación y latencia |
| 10 | Reglas de urgencia, tabla de dependencias y SLA |
| 11 | Demo interactiva con Gradio |

Los datos y artefactos intermedios se guardan en Google Drive (`muestra_cfpb.parquet`, `splits/`, `models/v1.0/`). Para reproducirlo hace falta montar Drive en la celda 4.

**Semilla:** 42 en el muestreo, las particiones y el entrenamiento. Los resultados pueden variar ligeramente según la versión de las librerías y el hardware.

## Datos

CFPB Consumer Complaint Database (versión Parquet en Hugging Face, `Mouwiya/cfpb-consumer-complaints`). Detalles, licencia y limitaciones en [`DATA_CARD.md`](DATA_CARD.md). La descarga es automática desde el notebook.

## Demo

La celda 11 lanza una interfaz Gradio con un enlace público temporal (`gradio.live`) que solo funciona mientras la celda se ejecuta. Devuelve tema, confianza, las 3 clases más probables, urgencia, dependencia destino y fecha límite de SLA. Muestra un aviso de revisión humana si la confianza es menor a 0,6 y rechaza textos de menos de 5 palabras. Escribe las solicitudes **en inglés**: el modelo se entrenó con ese idioma.

Capturas en `docs/demo_1.png` y `docs/demo_2.png`.

## Limitaciones

- **Idioma y dominio:** inglés y quejas financieras de consumidores, no PQRS ciudadanas.
- **Urgencia:** etiquetas débiles por reglas de palabras clave, sin validación humana.
- **Dependencias y SLA:** supuestos del prototipo, no provienen de datos reales. No se midió tiempo de atención ni cumplimiento de SLA.
- **Muestreo y particiones:** la muestra se tomó por archivos y no conserva necesariamente la distribución de la base completa; la partición es aleatoria, no temporal; no se eliminaron casi duplicados.
- **Validez estadística:** una sola corrida y una sola semilla, sin intervalos de confianza.
- **Fuera de alcance:** no incluye backend REST, base de datos, autenticación, pruebas automatizadas, Docker ni CI.
- **Explicabilidad:** solo se muestran confianza y probabilidades; no hay SHAP ni LIME.

## Fuentes y licencia

- Datos: Consumer Financial Protection Bureau, https://www.consumerfinance.gov/data-research/consumer-complaints/ (el CFPB indica que los datos publicados son de libre uso; verificar el texto vigente).
- Modelo base: `distilbert-base-uncased` (Hugging Face).
