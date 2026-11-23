# Data card

## Resumen
Quejas de consumidores en texto libre con su categoría de producto, usadas para entrenar y evaluar un clasificador de tema (7 clases). Se usan como caso análogo a las PQRS ciudadanas.

## Procedencia
- **Fuente original:** Consumer Complaint Database, Consumer Financial Protection Bureau (CFPB).
  https://www.consumerfinance.gov/data-research/consumer-complaints/
- **Copia utilizada:** versión en Parquet en Hugging Face, `Mouwiya/cfpb-consumer-complaints` (67 archivos Parquet).
  https://huggingface.co/datasets/Mouwiya/cfpb-consumer-complaints
- **Licencia:** el CFPB indica que los datos publicados son de libre uso, análisis y desarrollo. Verificar el texto vigente en su sitio antes de redistribuir.
- **Fecha de descarga:** 23 de noviembre de 2026.
- **Idioma:** inglés.
- **Rango de fechas de recepción en la muestra:** marzo de 2015 a junio de 2026.
- **Redistribución:** este repositorio no incluye los datos. El notebook los descarga desde Hugging Face.

## Columnas usadas

| Columna | Uso |
|---|---|
| `complaint_id` | Identificador |
| `date_received` | Fecha de recepción (solo exploración) |
| `product` | Base de la etiqueta de tema (se agrupa en 7 temas) |
| `issue` | Solo exploración |
| `consumer_complaint_narrative` | Texto de entrada del modelo |

Las demás columnas del dataset original (empresa, estado, código postal, respuesta de la empresa, etc.) no se usan.

## Construcción de la muestra
1. Se recorrieron los archivos Parquet en orden aleatorio (semilla 42).
2. De cada archivo se conservaron solo las filas con narrativa no vacía, hasta 1.500 por archivo.
3. Se detuvo la lectura al reunir 61.482 registros (42 de los 67 archivos).

Muchas filas del dataset original no tienen narrativa (el consumidor decide si la publica), por eso se filtra por texto no vacío.

## Etiqueta de tema
Los 21 valores de `product` se agruparon en 7 temas:

| Tema | Productos CFPB agrupados |
|---|---|
| Reportes de crédito | Credit reporting or other personal consumer reports; Credit reporting, credit repair services, or other personal consumer reports; Credit reporting |
| Cobro de deudas | Debt collection; Debt or credit management |
| Cuentas bancarias | Checking or savings account; Bank account or service |
| Tarjetas | Credit card; Credit card or prepaid card; Prepaid card |
| Hipotecas | Mortgage |
| Préstamos | Student loan; Vehicle loan or lease; Payday loan, title loan, personal loan, or advance loan; Payday loan, title loan, or personal loan; Consumer Loan; Payday loan |
| Transferencias de dinero | Money transfer, virtual currency, or money service; Money transfers; Virtual currency |

Se descartó "Other financial service" (10 filas).

La etiqueta es el producto que el consumidor seleccionó al presentar la queja; no fue revisada por los autores.

## Limpieza
- Eliminación de marcadores de anonimización del CFPB (por ejemplo, `XXXX`, `XX/XX/XXXX`), correos, teléfonos y montos monetarios (estos últimos reemplazados por un token genérico).
- Normalización de espacios.
- Se descartaron textos de menos de 10 palabras.
- Se eliminaron duplicados exactos (sin distinguir mayúsculas), **antes** de dividir en particiones.

Resultado: **48.823** registros.

## Particiones
Estratificadas por tema, aleatorias (no temporales), semilla 42.

| Partición | Registros |
|---|---|
| Entrenamiento | 34.176 |
| Validación | 7.323 |
| Prueba | 7.324 |

Distribución en entrenamiento: Reportes de crédito 16.190; Cobro de deudas 5.714; Tarjetas 3.448; Cuentas bancarias 3.160; Préstamos 2.379; Hipotecas 1.739; Transferencias de dinero 1.546.

## Etiquetas de urgencia (débiles)
El dataset no trae urgencia. Se generaron etiquetas por reglas de palabras clave en inglés (Alta, Media, Baja). **No son verdad real**: no se validaron con anotadores humanos ni se calculó acuerdo entre anotadores. En el conjunto de entrenamiento quedaron en Alta 39,5 %, Baja 33,1 % y Media 27,3 %. Solo se usan en la demo; no se entrena ni se evalúa ningún modelo con ellas.

## Limitaciones conocidas
- La muestra se tomó por archivos y no necesariamente conserva la distribución de clases de la base completa.
- No se eliminaron casi duplicados (solo duplicados exactos).
- La partición es aleatoria, no temporal; los cambios en las categorías del CFPB a lo largo de los años pueden influir en los resultados.
- No se auditó la anonimización de las narrativas más allá de la limpieza descrita.
- Una queja puede tocar varios temas, pero solo tiene una etiqueta.
- El contenido es de dominio financiero y en inglés, por lo que no representa solicitudes ciudadanas en español.
- Los textos provienen de personas reales: no se deben usar para identificar ni perfilar individuos.
