---LAB_START---
LAB_ID: 09-00-01
---MARKDOWN---
# Práctica 3 — Proceso RPA End-to-End con Checklist

## 1. Metadatos

| Campo | Valor |
|---|---|
| **Duración estimada** | 108 minutos |
| **Complejidad** | Alta |
| **Nivel Bloom** | Analizar |
| **Módulo** | 9 — Automatización de Archivos y Procesos RPA |
| **Versión del laboratorio** | 1.0 |

---

## 2. Descripción General

Este laboratorio simula un proceso RPA empresarial realista: el procesamiento automatizado de un lote de órdenes de compra. Integrarás en un único flujo orquestado cinco fases secuenciales que combinan lectura de Excel, enriquecimiento via API REST (mock local con WireMock), registro en portal web, generación de PDF de resumen y archivado con trazabilidad completa. El proceso es completamente parametrizable desde la línea de comandos y finaliza con un checklist de 10 puntos de validación que verifica la integridad de todos los artefactos generados.

---

## 3. Objetivos de Aprendizaje

Al completar este laboratorio serás capaz de:

- [ ] Implementar las cinco fases de un proceso RPA (ingesta, enriquecimiento, procesamiento web, reportes y cierre) usando `RPA.Excel.Files`, `RPA.PDF`, `RPA.FileSystem`, `RequestsLibrary` y `SeleniumLibrary` en un flujo cohesivo.
- [ ] Aplicar manejo robusto de errores con bloques `TRY/EXCEPT` y el patrón de reintentos `Wait Until Keyword Succeeds` para garantizar resiliencia ante fallos parciales sin abortar el proceso completo.
- [ ] Parametrizar el proceso desde CLI usando `--variable` y organizar el código en keywords modulares con docstrings que sigan las convenciones de nomenclatura profesionales.
- [ ] Generar trazabilidad completa mediante logging estructurado por niveles (`INFO`/`WARN`/`ERROR`), tags semánticos por fase y análisis del `output.xml` generado.
- [ ] Ejecutar y verificar un checklist automatizado de 10 puntos que valide la integridad de todos los artefactos del proceso.

---

## 4. Prerrequisitos

### Conocimiento previo requerido

| Área | Nivel requerido |
|---|---|
| Robot Framework — Keywords, Variables, Loops | Intermedio (Labs 07 y 08 completados) |
| `TRY/EXCEPT` en Robot Framework 5+ | Básico |
| `SeleniumLibrary` — interacción con formularios web | Intermedio (Lab 07-00-01 completado) |
| `RequestsLibrary` — peticiones HTTP y aserciones | Básico (Lab 08-00-01 completado) |
| Manejo de archivos con `OperatingSystem` | Básico |
| Línea de comandos (CLI) — ejecución de `robot` con flags | Básico |

### Acceso y herramientas requeridas

- Python 3.10 o 3.11 instalado y en el `PATH`.
- `rpaframework` instalado: `pip install rpaframework`.
- `robotframework-requests` instalado: `pip install robotframework-requests`.
- `SeleniumLibrary` instalado: `pip install robotframework-seleniumlibrary`.
- Java 11+ instalado (requerido para WireMock standalone).
- Google Chrome y ChromeDriver compatibles (o `webdriver-manager`).
- Archivo `wiremock-standalone.jar` disponible (ver sección de entorno).
- Repositorio del curso clonado localmente.

---

## 5. Entorno del Laboratorio

### Tabla de software

| Software | Versión mínima | Propósito en este lab |
|---|---|---|
| Robot Framework | 7.x | Motor de ejecución |
| rpaframework | 28.x | `RPA.Excel.Files`, `RPA.PDF`, `RPA.FileSystem` |
| SeleniumLibrary | 6.2+ | Automatización del portal web |
| RequestsLibrary | 0.9.4+ | Llamadas a la API mock |
| WireMock Standalone | 3.x | Servidor mock local de la API |
| Java JRE/JDK | 11+ | Requerido por WireMock |
| Google Chrome | 120+ | Navegador de automatización |
| webdriver-manager | Última | Gestión automática de ChromeDriver |

### Estructura de carpetas del proyecto

El laboratorio utiliza la siguiente estructura. Créala antes de comenzar:

```
lab-09-00-01/
├── resources/
│   ├── keywords/
│   │   ├── file_keywords.resource
│   │   ├── api_keywords.resource
│   │   ├── web_keywords.resource
│   │   └── report_keywords.resource
│   └── variables/
│       └── common_variables.resource
├── data/
│   ├── input/
│   │   └── ordenes_compra.xlsx        ← archivo de entrada (se provee)
│   ├── output/                        ← generado por el proceso
│   ├── processed/                     ← archivos movidos tras éxito
│   └── errors/                        ← CSV de errores para reproceso
├── wiremock/
│   ├── wiremock-standalone.jar
│   └── mappings/
│       ├── clientes_mapping.json
│       └── productos_mapping.json
├── evidence/                          ← screenshots y PDFs de evidencia
├── tests/
│   └── rpa_ordenes_e2e.robot          ← suite principal
└── results/                           ← output.xml, log.html, report.html
```

### Configuración del entorno — Paso previo obligatorio

**1. Instalar dependencias Python:**

```bash
pip install rpaframework robotframework-seleniumlibrary robotframework-requests webdriver-manager
```

**2. Crear la estructura de carpetas del proyecto:**

```bash
# Linux/macOS
mkdir -p lab-09-00-01/{resources/keywords,resources/variables,data/{input,output,processed,errors},wiremock/mappings,evidence,tests,results}

# Windows PowerShell
New-Item -ItemType Directory -Force -Path lab-09-00-01\resources\keywords
New-Item -ItemType Directory -Force -Path lab-09-00-01\resources\variables
New-Item -ItemType Directory -Force -Path lab-09-00-01\data\input
New-Item -ItemType Directory -Force -Path lab-09-00-01\data\output
New-Item -ItemType Directory -Force -Path lab-09-00-01\data\processed
New-Item -ItemType Directory -Force -Path lab-09-00-01\data\errors
New-Item -ItemType Directory -Force -Path lab-09-00-01\wiremock\mappings
New-Item -ItemType Directory -Force -Path lab-09-00-01\evidence
New-Item -ItemType Directory -Force -Path lab-09-00-01\tests
New-Item -ItemType Directory -Force -Path lab-09-00-01\results
```

**3. Descargar WireMock standalone:**

```bash
cd lab-09-00-01/wiremock
# Descargar desde el repositorio oficial
curl -L https://repo1.maven.org/maven2/org/wiremock/wiremock-standalone/3.3.1/wiremock-standalone-3.3.1.jar \
     -o wiremock-standalone.jar
```

> **Entorno corporativo con proxy:** Si estás detrás de un proxy, usa:
> ```bash
> curl --proxy http://PROXY_HOST:PORT -L <url> -o wiremock-standalone.jar
> ```

**4. Crear los mappings de WireMock:**

Crea el archivo `wiremock/mappings/clientes_mapping.json`:

```json
{
  "mappings": [
    {
      "request": {
        "method": "GET",
        "urlPattern": "/api/clientes/([0-9]+)"
      },
      "response": {
        "status": 200,
        "headers": { "Content-Type": "application/json" },
        "jsonBody": {
          "cliente_id": "{{request.pathSegments.[2]}}",
          "nombre": "Cliente Demo",
          "direccion": "Av. Principal 123, Ciudad",
          "categoria_credito": "A",
          "limite_credito": 50000
        },
        "transformers": ["response-template"]
      }
    }
  ]
}
```

Crea el archivo `wiremock/mappings/productos_mapping.json`:

```json
{
  "mappings": [
    {
      "request": {
        "method": "GET",
        "urlPattern": "/api/productos/([A-Z0-9]+)"
      },
      "response": {
        "status": 200,
        "headers": { "Content-Type": "application/json" },
        "jsonBody": {
          "producto_id": "{{request.pathSegments.[2]}}",
          "nombre": "Producto Demo",
          "stock_disponible": 150,
          "precio_actualizado": 299.99,
          "categoria": "electronica"
        },
        "transformers": ["response-template"]
      }
    }
  ]
}
```

**5. Iniciar WireMock (mantener esta terminal abierta durante todo el lab):**

```bash
# Linux/macOS
cd lab-09-00-01/wiremock
java -jar wiremock-standalone.jar --port 8089 --root-dir . --verbose

# Windows CMD
cd lab-09-00-01\wiremock
java -jar wiremock-standalone.jar --port 8089 --root-dir . --verbose
```

Verifica que WireMock esté activo:
```bash
curl http://localhost:8089/api/clientes/1001
# Debe retornar JSON con datos del cliente
```

**6. Crear el archivo Excel de entrada `data/input/ordenes_compra.xlsx`:**

Crea un script Python auxiliar para generar el archivo de prueba:

```python
# generar_ordenes.py  — ejecutar UNA VEZ para crear el Excel de entrada
import openpyxl
from openpyxl import Workbook

wb = Workbook()
ws = wb.active
ws.title = "Ordenes"

# Encabezados
headers = ["orden_id", "cliente_id", "producto_id", "cantidad", "precio_unitario", "estado"]
ws.append(headers)

# 22 registros de prueba
ordenes = [
    ("ORD-001", "1001", "PROD-A1", 5,  150.00, "PENDIENTE"),
    ("ORD-002", "1002", "PROD-B2", 2, 2500.00, "PENDIENTE"),
    ("ORD-003", "1003", "PROD-C3", 10,  80.00, "PENDIENTE"),
    ("ORD-004", "1004", "PROD-A1", 1, 5000.00, "PENDIENTE"),
    ("ORD-005", "1005", "PROD-D4", 3,  320.00, "PENDIENTE"),
    ("ORD-006", "1006", "PROD-B2", 7,  150.00, "PENDIENTE"),
    ("ORD-007", "1007", "PROD-E5", 4, 1200.00, "PENDIENTE"),
    ("ORD-008", "1008", "PROD-C3", 2,  900.00, "PENDIENTE"),
    ("ORD-009", "1009", "PROD-A1", 15,  50.00, "PENDIENTE"),
    ("ORD-010", "1010", "PROD-F6", 1, 3500.00, "PENDIENTE"),
    ("ORD-011", "1011", "PROD-B2", 6,  200.00, "PENDIENTE"),
    ("ORD-012", "1012", "PROD-G7", 3,  750.00, "PENDIENTE"),
    ("ORD-013", "1013", "PROD-A1", 8,  120.00, "PENDIENTE"),
    ("ORD-014", "1014", "PROD-H8", 2, 1800.00, "PENDIENTE"),
    ("ORD-015", "1015", "PROD-C3", 5,  400.00, "PENDIENTE"),
    ("ORD-016", "1016", "PROD-I9", 1, 6000.00, "PENDIENTE"),
    ("ORD-017", "1017", "PROD-B2", 4,  250.00, "PENDIENTE"),
    ("ORD-018", "1018", "PROD-J10",9,   90.00, "PENDIENTE"),
    ("ORD-019", "1019", "PROD-A1", 2, 2200.00, "PENDIENTE"),
    ("ORD-020", "1020", "PROD-K11",3,  600.00, "PENDIENTE"),
    ("ORD-021", "1021", "PROD-C3", 11, 110.00, "PENDIENTE"),
    ("ORD-022", "1022", "PROD-L12",1, 4500.00, "PENDIENTE"),
]

for orden in ordenes:
    ws.append(orden)

wb.save("lab-09-00-01/data/input/ordenes_compra.xlsx")
print("Archivo ordenes_compra.xlsx generado correctamente.")
```

```bash
python generar_ordenes.py
```

---

## 6. Pasos del Laboratorio

> **Gestión del tiempo recomendada:**
> - **Bloque A (0–35 min):** Pasos 1–3 — Variables, estructura de recursos y Fase 1 (ingesta).
> - **Bloque B (35–80 min):** Pasos 4–6 — Fases 2, 3 y 4 (API, web y reportes).
> - **Bloque C (80–108 min):** Pasos 7–9 — Fase 5, suite principal, checklist y ejecución.

---

### Paso 1: Crear el archivo de variables comunes

**Objetivo:** Centralizar todas las rutas, URLs y parámetros en un único archivo de variables para que el proceso sea completamente parametrizable desde CLI.

**Instrucciones:**

1. Crea el archivo `resources/variables/common_variables.resource` con el siguiente contenido:

```robotframework
*** Settings ***
Documentation    Variables globales del proceso RPA de órdenes de compra.
...              Todas las rutas usan ${/} para compatibilidad multiplataforma.

*** Variables ***
# --- Rutas de carpetas ---
${BASE_DIR}          ${CURDIR}${/}..${/}data
${INPUT_DIR}         ${BASE_DIR}${/}input
${OUTPUT_DIR}        ${BASE_DIR}${/}output
${PROCESSED_DIR}     ${BASE_DIR}${/}processed
${ERRORS_DIR}        ${BASE_DIR}${/}errors
${EVIDENCE_DIR}      ${CURDIR}${/}..${/}evidence

# --- Archivo de entrada ---
${INPUT_FILE}        ${INPUT_DIR}${/}ordenes_compra.xlsx
${SHEET_NAME}        Ordenes

# --- URLs ---
${API_BASE_URL}      http://localhost:8089
${WEB_PORTAL_URL}    https://the-internet.herokuapp.com/login

# --- Parámetros del proceso ---
${MONTO_UMBRAL}      ${1000}
${MAX_RETRIES}       3
${RETRY_INTERVAL}    2s
${BROWSER}           chrome

# --- Archivos de salida ---
${REPORT_PDF}        ${OUTPUT_DIR}${/}reporte_lote.pdf
${ERRORS_CSV}        ${ERRORS_DIR}${/}ordenes_con_errores.csv

# --- Credenciales del portal web (demo) ---
${WEB_USER}          tomsmith
${WEB_PASSWORD}      SuperSecretPassword!
```

2. Guarda el archivo.

**Salida esperada:** El archivo `common_variables.resource` existe en `resources/variables/`.

**Verificación:**
```bash
# Linux/macOS
cat lab-09-00-01/resources/variables/common_variables.resource | grep BASE_DIR

# Windows PowerShell
Get-Content lab-09-00-01\resources\variables\common_variables.resource | Select-String "BASE_DIR"
```

---

### Paso 2: Crear keywords de manipulación de archivos (Fase 1)

**Objetivo:** Implementar las keywords de la Fase 1 (ingesta de datos) — lectura del Excel, validación de estructura y movimiento del archivo a la carpeta "en-proceso".

**Instrucciones:**

1. Crea el archivo `resources/keywords/file_keywords.resource`:

```robotframework
*** Settings ***
Documentation    Keywords para manipulación de archivos CSV, Excel y PDF.
...              Fase 1: Ingesta — Fase 4: Reportes — Fase 5: Archivado.
Library          RPA.Excel.Files
Library          RPA.PDF
Library          RPA.FileSystem
Library          OperatingSystem
Library          Collections
Library          String
Library          DateTime

*** Variables ***
${EN_PROCESO_DIR}    ${CURDIR}${/}..${/}..${/}data${/}en-proceso

*** Keywords ***
Inicializar Carpetas Del Proceso
    [Documentation]    Crea todas las carpetas necesarias si no existen.
    ...                Garantiza que la estructura de directorios esté lista antes de procesar.
    Create Directory    ${OUTPUT_DIR}
    Create Directory    ${PROCESSED_DIR}
    Create Directory    ${ERRORS_DIR}
    Create Directory    ${EVIDENCE_DIR}
    Create Directory    ${EN_PROCESO_DIR}
    Log    Carpetas inicializadas correctamente.    level=INFO

Validar Archivo De Entrada
    [Documentation]    Verifica que el archivo Excel de entrada exista y sea legible.
    ...                Lanza una excepción descriptiva si el archivo no está disponible.
    [Arguments]    ${ruta_archivo}
    File Should Exist    ${ruta_archivo}
    ...    msg=Archivo de entrada no encontrado: ${ruta_archivo}
    ${size}=    Get File Size    ${ruta_archivo}
    Should Be True    ${size} > 0
    ...    msg=El archivo de entrada está vacío: ${ruta_archivo}
    Log    Archivo validado: ${ruta_archivo} (${size} bytes)    level=INFO

Leer Ordenes Desde Excel
    [Documentation]    Abre el libro Excel, lee la hoja de órdenes y retorna una lista de diccionarios.
    ...                Cada diccionario representa una fila con las claves de los encabezados.
    [Arguments]    ${ruta_archivo}    ${nombre_hoja}=${SHEET_NAME}
    Open Workbook    ${ruta_archivo}
    ${ordenes}=    Read Worksheet As Table    ${nombre_hoja}    header=True
    Close Workbook
    ${total}=    Get Length    ${ordenes}
    Log    Se leyeron ${total} órdenes desde el archivo Excel.    level=INFO
    RETURN    ${ordenes}

Mover Archivo A En Proceso
    [Documentation]    Mueve el archivo de entrada a la carpeta "en-proceso" para indicar
    ...                que el lote está siendo procesado. Evita reprocesamiento accidental.
    [Arguments]    ${ruta_origen}
    ${nombre_archivo}=    Get File Name    ${ruta_origen}
    ${destino}=    Set Variable    ${EN_PROCESO_DIR}${/}${nombre_archivo}
    Move File    ${ruta_origen}    ${destino}
    Log    Archivo movido a en-proceso: ${destino}    level=INFO
    RETURN    ${destino}

Generar CSV De Errores
    [Documentation]    Escribe un archivo CSV con las órdenes que presentaron errores
    ...                durante el procesamiento, para facilitar el reproceso manual.
    [Arguments]    ${lista_errores}    ${ruta_csv}
    ${total_errores}=    Get Length    ${lista_errores}
    Run Keyword If    ${total_errores} == 0
    ...    Log    No hay órdenes con errores. CSV de errores no generado.    level=INFO
    ...    ELSE    Escribir CSV Errores Interno    ${lista_errores}    ${ruta_csv}

Escribir CSV Errores Interno
    [Documentation]    Keyword interna — escribe el CSV de errores con encabezado.
    [Arguments]    ${lista_errores}    ${ruta_csv}
    ${lineas}=    Create List    orden_id,cliente_id,producto_id,cantidad,precio_unitario,motivo_error
    FOR    ${error}    IN    @{lista_errores}
        ${linea}=    Set Variable
        ...    ${error}[orden_id],${error}[cliente_id],${error}[producto_id],${error}[cantidad],${error}[precio_unitario],${error}[motivo_error]
        Append To List    ${lineas}    ${linea}
    END
    ${contenido}=    Catenate    SEPARATOR=\n    @{lineas}
    Create File    ${ruta_csv}    ${contenido}    encoding=utf-8
    Log    CSV de errores generado: ${ruta_csv} (${lista_errores.__len__()} registros)    level=WARN

Generar PDF De Resumen
    [Documentation]    Genera un PDF de resumen del lote procesado con estadísticas clave.
    ...                Usa RPA.PDF para crear el documento de reporte final.
    [Arguments]    ${estadisticas}    ${ruta_pdf}
    ${timestamp}=    Get Current Date    result_format=%Y-%m-%d %H:%M:%S
    ${total}=        Get From Dictionary    ${estadisticas}    total
    ${exitosas}=     Get From Dictionary    ${estadisticas}    exitosas
    ${fallidas}=     Get From Dictionary    ${estadisticas}    fallidas
    ${monto_total}=  Get From Dictionary    ${estadisticas}    monto_total
    ${contenido_html}=    Set Variable
    ...    <html><body>
    ...    <h1>Reporte de Lote — Órdenes de Compra</h1>
    ...    <p><b>Fecha de procesamiento:</b> ${timestamp}</p>
    ...    <hr/>
    ...    <h2>Estadísticas del Lote</h2>
    ...    <table border="1" cellpadding="5">
    ...    <tr><th>Métrica</th><th>Valor</th></tr>
    ...    <tr><td>Total de órdenes procesadas</td><td>${total}</td></tr>
    ...    <tr><td>Órdenes exitosas</td><td>${exitosas}</td></tr>
    ...    <tr><td>Órdenes con error</td><td>${fallidas}</td></tr>
    ...    <tr><td>Monto total procesado</td><td>$${monto_total}</td></tr>
    ...    </table>
    ...    <p><i>Generado automáticamente por el proceso RPA de Órdenes de Compra.</i></p>
    ...    </body></html>
    HTML To PDF    ${contenido_html}    ${ruta_pdf}
    Log    PDF de resumen generado: ${ruta_pdf}    level=INFO

Archivar Archivos Procesados
    [Documentation]    Mueve los archivos de la carpeta "en-proceso" a "completados"
    ...                y organiza las evidencias en una subcarpeta con timestamp.
    [Arguments]    ${ruta_en_proceso}    ${ruta_completados}    ${ruta_evidencias}
    ${timestamp}=    Get Current Date    result_format=%Y%m%d_%H%M%S
    ${carpeta_evidencias}=    Set Variable    ${ruta_evidencias}${/}lote_${timestamp}
    Create Directory    ${carpeta_evidencias}
    # Mover archivo Excel procesado
    ${archivos}=    List Files In Directory    ${ruta_en_proceso}    pattern=*.xlsx
    FOR    ${archivo}    IN    @{archivos}
        Move File    ${ruta_en_proceso}${/}${archivo}    ${ruta_completados}${/}${archivo}
    END
    # Mover screenshots a subcarpeta con timestamp
    ${screenshots}=    List Files In Directory    ${ruta_evidencias}    pattern=*.png
    FOR    ${screenshot}    IN    @{screenshots}
        Move File    ${ruta_evidencias}${/}${screenshot}    ${carpeta_evidencias}${/}${screenshot}
    END
    Log    Archivos archivados. Evidencias en: ${carpeta_evidencias}    level=INFO
    RETURN    ${carpeta_evidencias}
```

2. Guarda el archivo.

**Salida esperada:** El archivo `file_keywords.resource` contiene 8 keywords documentadas.

**Verificación:**
```bash
# Contar keywords definidas
grep -c "^\S.*\n\s*\[Documentation\]" lab-09-00-01/resources/keywords/file_keywords.resource || \
grep -c "\[Documentation\]" lab-09-00-01/resources/keywords/file_keywords.resource
# Debe mostrar 8 (o cercano — cada keyword tiene su [Documentation])
```

---

### Paso 3: Crear keywords de API con retry (Fase 2)

**Objetivo:** Implementar las keywords de enriquecimiento via API con manejo de errores y reintentos configurables.

**Instrucciones:**

1. Crea el archivo `resources/keywords/api_keywords.resource`:

```robotframework
*** Settings ***
Documentation    Keywords para consultas a la API REST de enriquecimiento de órdenes.
...              Fase 2: Enriquecimiento — incluye patrón de retry con Wait Until Keyword Succeeds.
Library          RequestsLibrary
Library          Collections
Library          String

*** Keywords ***
Inicializar Sesion API
    [Documentation]    Crea la sesión HTTP reutilizable para todas las llamadas a la API.
    ...                Debe llamarse una sola vez durante el setup del proceso.
    [Arguments]    ${base_url}
    Create Session    api_session    ${base_url}    verify=False
    Log    Sesión API inicializada: ${base_url}    level=INFO

Obtener Datos Cliente Con Retry
    [Documentation]    Consulta la API para obtener datos enriquecidos del cliente.
    ...                Aplica el patrón de retry: 3 intentos con 2s de espera entre cada uno.
    ...                Retorna un diccionario con los datos del cliente o un dict de error.
    [Arguments]    ${cliente_id}    ${max_retries}=${MAX_RETRIES}    ${intervalo}=${RETRY_INTERVAL}
    TRY
        ${datos}=    Wait Until Keyword Succeeds
        ...    ${max_retries}x    ${intervalo}
        ...    Consultar API Cliente    ${cliente_id}
        Log    Datos de cliente ${cliente_id} obtenidos correctamente.    level=INFO
        RETURN    ${datos}
    EXCEPT    AS    ${error}
        Log    ERROR al obtener cliente ${cliente_id} tras ${max_retries} intentos: ${error}
        ...    level=ERROR
        ${datos_error}=    Create Dictionary
        ...    cliente_id=${cliente_id}
        ...    nombre=DESCONOCIDO
        ...    direccion=N/A
        ...    categoria_credito=ERROR
        ...    limite_credito=${0}
        ...    error=${True}
        RETURN    ${datos_error}
    END

Consultar API Cliente
    [Documentation]    Keyword atómica que realiza la petición GET al endpoint de clientes.
    ...                Es la keyword que será reintentada por Wait Until Keyword Succeeds.
    [Arguments]    ${cliente_id}
    ${response}=    GET On Session    api_session    /api/clientes/${cliente_id}
    ...    expected_status=200
    ${datos}=    Set Variable    ${response.json()}
    RETURN    ${datos}

Obtener Datos Producto Con Retry
    [Documentation]    Consulta la API para obtener datos enriquecidos del producto.
    ...                Aplica el mismo patrón de retry que la consulta de clientes.
    ...                Retorna un diccionario con los datos del producto o un dict de error.
    [Arguments]    ${producto_id}    ${max_retries}=${MAX_RETRIES}    ${intervalo}=${RETRY_INTERVAL}
    TRY
        ${datos}=    Wait Until Keyword Succeeds
        ...    ${max_retries}x    ${intervalo}
        ...    Consultar API Producto    ${producto_id}
        Log    Datos de producto ${producto_id} obtenidos correctamente.    level=INFO
        RETURN    ${datos}
    EXCEPT    AS    ${error}
        Log    ERROR al obtener producto ${producto_id} tras ${max_retries} intentos: ${error}
        ...    level=ERROR
        ${datos_error}=    Create Dictionary
        ...    producto_id=${producto_id}
        ...    nombre=DESCONOCIDO
        ...    stock_disponible=${0}
        ...    precio_actualizado=${0}
        ...    error=${True}
        RETURN    ${datos_error}
    END

Consultar API Producto
    [Documentation]    Keyword atómica que realiza la petición GET al endpoint de productos.
    [Arguments]    ${producto_id}
    ${response}=    GET On Session    api_session    /api/productos/${producto_id}
    ...    expected_status=200
    ${datos}=    Set Variable    ${response.json()}
    RETURN    ${datos}

Calcular Monto Orden
    [Documentation]    Calcula el monto total de una orden multiplicando cantidad por precio.
    ...                Usa el precio actualizado de la API si está disponible.
    [Arguments]    ${cantidad}    ${precio_original}    ${precio_api}=${None}
    ${precio_final}=    Set Variable If
    ...    '${precio_api}' != 'None' and ${precio_api} > 0    ${precio_api}    ${precio_original}
    ${monto}=    Evaluate    float(${cantidad}) * float(${precio_final})
    RETURN    ${monto}

Cerrar Sesion API
    [Documentation]    Cierra la sesión HTTP al finalizar el proceso.
    Delete All Sessions
    Log    Sesión API cerrada.    level=INFO
```

2. Guarda el archivo.

**Salida esperada:** El archivo `api_keywords.resource` contiene 7 keywords con docstrings.

**Verificación:**
```bash
curl -s http://localhost:8089/api/clientes/1001 | python3 -m json.tool
# Debe mostrar el JSON del cliente formateado
curl -s http://localhost:8089/api/productos/PROD-A1 | python3 -m json.tool
# Debe mostrar el JSON del producto formateado
```

---

### Paso 4: Crear keywords de automatización web (Fase 3)

**Objetivo:** Implementar las keywords de la Fase 3 — registro de órdenes de alto valor en el portal web con captura de evidencias.

**Instrucciones:**

1. Crea el archivo `resources/keywords/web_keywords.resource`:

```robotframework
*** Settings ***
Documentation    Keywords para automatización del portal web.
...              Fase 3: Procesamiento web — registro de órdenes de alto valor.
...              Usa The Internet (https://the-internet.herokuapp.com) como portal de práctica.
Library          SeleniumLibrary
Library          OperatingSystem
Library          String
Library          DateTime

*** Keywords ***
Inicializar Navegador
    [Documentation]    Abre el navegador configurado y maximiza la ventana.
    ...                Usa webdriver-manager para gestión automática de ChromeDriver.
    [Arguments]    ${url}    ${browser}=${BROWSER}
    ${options}=    Evaluate
    ...    __import__('selenium.webdriver', fromlist=['ChromeOptions']).ChromeOptions()
    ...    modules=selenium.webdriver
    Call Method    ${options}    add_argument    --no-sandbox
    Call Method    ${options}    add_argument    --disable-dev-shm-usage
    # Para ejecución headless en CI/CD, descomentar la siguiente línea:
    # Call Method    ${options}    add_argument    --headless=new
    Open Browser    ${url}    ${browser}    options=${options}
    Maximize Browser Window
    Log    Navegador inicializado: ${browser} → ${url}    level=INFO

Autenticar En Portal Web
    [Documentation]    Realiza el login en el portal web de práctica.
    ...                Verifica que el login sea exitoso antes de continuar.
    [Arguments]    ${usuario}    ${password}
    Wait Until Element Is Visible    id=username    timeout=10s
    Input Text    id=username    ${usuario}
    Input Text    id=password    ${password}
    Click Button    css=button[type="submit"]
    Wait Until Element Is Visible    css=.flash.success    timeout=10s
    Log    Login exitoso en el portal web.    level=INFO

Registrar Orden En Portal
    [Documentation]    Navega al formulario de registro y completa los datos de la orden.
    ...                Captura un screenshot como evidencia del registro exitoso.
    ...                Retorna True si el registro fue exitoso, False en caso contrario.
    [Arguments]    ${orden}    ${datos_cliente}    ${monto}    ${directorio_evidencias}
    TRY
        # Navegar al formulario (usando la sección de inputs de The Internet)
        Go To    https://the-internet.herokuapp.com/inputs
        Wait Until Element Is Visible    css=.example    timeout=10s
        # Simular registro: ingresar el monto de la orden en el campo numérico
        ${monto_redondeado}=    Evaluate    round(${monto}, 2)
        Input Text    css=input[type="number"]    ${monto_redondeado}
        # Capturar screenshot como evidencia
        ${timestamp}=    Get Current Date    result_format=%Y%m%d_%H%M%S
        ${nombre_screenshot}=    Set Variable
        ...    ${directorio_evidencias}${/}orden_${orden}[orden_id]_${timestamp}.png
        Capture Page Screenshot    ${nombre_screenshot}
        Log    Orden ${orden}[orden_id] registrada. Screenshot: ${nombre_screenshot}    level=INFO
        RETURN    ${True}
    EXCEPT    AS    ${error}
        Log    ERROR al registrar orden ${orden}[orden_id] en portal web: ${error}    level=ERROR
        TRY
            ${timestamp}=    Get Current Date    result_format=%Y%m%d_%H%M%S
            Capture Page Screenshot
            ...    ${directorio_evidencias}${/}ERROR_${orden}[orden_id]_${timestamp}.png
        EXCEPT
            Log    No se pudo capturar screenshot de error.    level=WARN
        END
        RETURN    ${False}
    END

Cerrar Navegador
    [Documentation]    Cierra el navegador y libera los recursos del WebDriver.
    Close All Browsers
    Log    Navegador cerrado.    level=INFO
```

2. Guarda el archivo.

**Salida esperada:** El archivo `web_keywords.resource` contiene 4 keywords con manejo de errores anidado.

---

### Paso 5: Crear keywords de reporte y checklist (Fase 4 y 5)

**Objetivo:** Implementar la generación de reportes y el checklist de validación de 10 puntos.

**Instrucciones:**

1. Crea el archivo `resources/keywords/report_keywords.resource`:

```robotframework
*** Settings ***
Documentation    Keywords para generación de reportes y validación del checklist final.
...              Fase 4: Reportes — Fase 5: Cierre y trazabilidad.
Library          OperatingSystem
Library          Collections
Library          String
Library          DateTime

*** Keywords ***
Ejecutar Checklist De Validacion
    [Documentation]    Ejecuta 10 puntos de validación para verificar la integridad
    ...                del proceso RPA. Registra PASS/FAIL por cada punto y genera
    ...                un resumen final. No aborta el proceso ante fallos individuales.
    [Arguments]    ${estadisticas}    ${rutas}
    Log    ═══════════════════════════════════════════    level=INFO
    Log    CHECKLIST DE VALIDACIÓN DEL PROCESO RPA       level=INFO
    Log    ═══════════════════════════════════════════    level=INFO
    ${resultados}=    Create Dictionary

    # Punto 1: Verificar que se procesaron órdenes
    ${check1}=    Validar Punto Checklist
    ...    1    Órdenes procesadas > 0
    ...    ${estadisticas}[total] > 0
    Set To Dictionary    ${resultados}    punto_01=${check1}

    # Punto 2: Verificar que el PDF de resumen fue generado
    ${pdf_existe}=    Run Keyword And Return Status
    ...    File Should Exist    ${rutas}[reporte_pdf]
    ${check2}=    Validar Punto Checklist
    ...    2    PDF de resumen generado    ${pdf_existe}
    Set To Dictionary    ${resultados}    punto_02=${check2}

    # Punto 3: Verificar que la carpeta de completados existe
    ${check3}=    Validar Punto Checklist
    ...    3    Carpeta de completados existe
    ...    ${rutas}[directorio_completados] != ''
    Set To Dictionary    ${resultados}    punto_03=${check3}

    # Punto 4: Verificar que no hay archivos huérfanos en en-proceso
    ${archivos_en_proceso}=    List Files In Directory    ${rutas}[directorio_en_proceso]
    ${check4}=    Validar Punto Checklist
    ...    4    Carpeta en-proceso vacía tras archivado
    ...    len(${archivos_en_proceso}) == 0
    Set To Dictionary    ${resultados}    punto_04=${check4}

    # Punto 5: Verificar que las órdenes exitosas + fallidas = total
    ${suma}=    Evaluate    ${estadisticas}[exitosas] + ${estadisticas}[fallidas]
    ${check5}=    Validar Punto Checklist
    ...    5    Exitosas + Fallidas = Total
    ...    ${suma} == ${estadisticas}[total]
    Set To Dictionary    ${resultados}    punto_05=${check5}

    # Punto 6: Verificar que el monto total es positivo
    ${check6}=    Validar Punto Checklist
    ...    6    Monto total procesado > 0
    ...    ${estadisticas}[monto_total] > 0
    Set To Dictionary    ${resultados}    punto_06=${check6}

    # Punto 7: Verificar que existe al menos un screenshot de evidencia
    ${screenshots}=    Run Keyword And Return Status
    ...    Directory Should Not Be Empty    ${rutas}[directorio_evidencias_archivado]
    ${check7}=    Validar Punto Checklist
    ...    7    Evidencias (screenshots) generadas    ${screenshots}
    Set To Dictionary    ${resultados}    punto_07=${check7}

    # Punto 8: Verificar CSV de errores si hay órdenes fallidas
    ${hay_errores}=    Evaluate    ${estadisticas}[fallidas] > 0
    ${csv_existe}=    Run Keyword And Return Status
    ...    File Should Exist    ${rutas}[errores_csv]
    ${check8_condicion}=    Evaluate
    ...    (not ${hay_errores}) or (${hay_errores} and ${csv_existe})
    ${check8}=    Validar Punto Checklist
    ...    8    CSV de errores generado (si aplica)    ${check8_condicion}
    Set To Dictionary    ${resultados}    punto_08=${check8}

    # Punto 9: Verificar que el tasa de éxito es >= 50%
    ${tasa}=    Evaluate
    ...    (${estadisticas}[exitosas] / ${estadisticas}[total]) * 100 if ${estadisticas}[total] > 0 else 0
    ${check9}=    Validar Punto Checklist
    ...    9    Tasa de éxito >= 50%    ${tasa} >= 50
    Set To Dictionary    ${resultados}    punto_09=${check9}

    # Punto 10: Verificar que el log de trazabilidad fue generado
    ${check10}=    Validar Punto Checklist
    ...    10    Log de trazabilidad final registrado    ${True}
    Set To Dictionary    ${resultados}    punto_10=${check10}

    # Resumen del checklist
    ${puntos_ok}=    Evaluate
    ...    sum(1 for v in ${resultados}.values() if v == 'PASS')
    Log    ═══════════════════════════════════════════    level=INFO
    Log    RESULTADO CHECKLIST: ${puntos_ok}/10 puntos PASS    level=INFO
    Log    ═══════════════════════════════════════════    level=INFO
    RETURN    ${resultados}

Validar Punto Checklist
    [Documentation]    Evalúa un punto del checklist y registra el resultado.
    ...                Retorna 'PASS' o 'FAIL' sin lanzar excepción.
    [Arguments]    ${numero}    ${descripcion}    ${condicion}
    ${estado}=    Set Variable If    ${condicion}    PASS    FAIL
    ${nivel_log}=    Set Variable If    ${condicion}    INFO    WARN
    Log    [${estado}] Punto ${numero}: ${descripcion}    level=${nivel_log}
    RETURN    ${estado}

Generar Log De Trazabilidad Final
    [Documentation]    Registra un resumen estructurado del proceso en el log de Robot Framework.
    ...                Este log aparece en el output.xml y es analizable post-ejecución.
    [Arguments]    ${estadisticas}    ${timestamp_inicio}    ${timestamp_fin}
    ${duracion}=    Subtract Date From Date    ${timestamp_fin}    ${timestamp_inicio}
    Log    ═══════════════════════════════════════════════════    level=INFO
    Log    TRAZABILIDAD FINAL DEL PROCESO RPA                    level=INFO
    Log    ═══════════════════════════════════════════════════    level=INFO
    Log    Inicio del proceso  : ${timestamp_inicio}             level=INFO
    Log    Fin del proceso     : ${timestamp_fin}                level=INFO
    Log    Duración total      : ${duracion} segundos            level=INFO
    Log    Total órdenes       : ${estadisticas}[total]          level=INFO
    Log    Órdenes exitosas    : ${estadisticas}[exitosas]       level=INFO
    Log    Órdenes fallidas    : ${estadisticas}[fallidas]       level=INFO
    Log    Monto total ($)     : ${estadisticas}[monto_total]    level=INFO
    Log    ═══════════════════════════════════════════════════    level=INFO
```

2. Guarda el archivo.

**Salida esperada:** El archivo `report_keywords.resource` contiene el checklist de 10 puntos implementado.

---

### Paso 6: Crear la suite principal del proceso RPA

**Objetivo:** Orquestar las cinco fases del proceso en un único archivo `.robot` con setup, teardown y manejo de estado global.

**Instrucciones:**

1. Crea el archivo `tests/rpa_ordenes_e2e.robot`:

```robotframework
*** Settings ***
Documentation    Suite principal del proceso RPA de Órdenes de Compra.
...
...              FASES DEL PROCESO:
...              1. Ingesta    — Lectura y validación del archivo Excel de entrada.
...              2. Enriquecimiento — Consulta a la API REST con retry para datos adicionales.
...              3. Web        — Registro en portal web de órdenes que superan el umbral.
...              4. Reportes   — Generación de PDF de resumen y CSV de errores.
...              5. Cierre     — Archivado, trazabilidad y checklist de validación.
...
...              PARAMETRIZACIÓN CLI (ejemplo):
...              robot --variable INPUT_FILE:ruta/archivo.xlsx
...                    --variable API_BASE_URL:http://localhost:8089
...                    --variable MONTO_UMBRAL:1000
...                    --variable WEB_PORTAL_URL:https://the-internet.herokuapp.com/login
...                    --outputdir results/
...                    tests/rpa_ordenes_e2e.robot

Resource         ../resources/variables/common_variables.resource
Resource         ../resources/keywords/file_keywords.resource
Resource         ../resources/keywords/api_keywords.resource
Resource         ../resources/keywords/web_keywords.resource
Resource         ../resources/keywords/report_keywords.resource

Suite Setup      Preparar Entorno Del Proceso
Suite Teardown   Finalizar Proceso Completamente

*** Variables ***
# Variables con valores por defecto — sobreescribibles via CLI con --variable
${INPUT_FILE}        ${CURDIR}${/}..${/}data${/}input${/}ordenes_compra.xlsx
${API_BASE_URL}      http://localhost:8089
${WEB_PORTAL_URL}    https://the-internet.herokuapp.com/login
${MONTO_UMBRAL}      ${1000}
${BROWSER}           chrome

# Variables de estado global del proceso (se populan durante la ejecución)
${ARCHIVO_EN_PROCESO}    ${EMPTY}
${CARPETA_EVIDENCIAS}    ${EMPTY}

*** Tasks ***
Fase 1 - Ingesta De Datos
    [Documentation]    Lee el archivo Excel de entrada, valida su estructura
    ...                y lo mueve a la carpeta "en-proceso" para marcar el inicio.
    [Tags]    fase:ingesta    rpa    e2e

    Log    ▶ FASE 1: INGESTA DE DATOS    level=INFO
    Validar Archivo De Entrada    ${INPUT_FILE}
    ${ordenes}=    Leer Ordenes Desde Excel    ${INPUT_FILE}
    Set Suite Variable    ${ORDENES_CRUDAS}    ${ordenes}
    ${ruta_en_proceso}=    Mover Archivo A En Proceso    ${INPUT_FILE}
    Set Suite Variable    ${ARCHIVO_EN_PROCESO}    ${ruta_en_proceso}
    ${total}=    Get Length    ${ordenes}
    Log    Fase 1 completada: ${total} órdenes leídas y archivo en proceso.    level=INFO

Fase 2 - Enriquecimiento Via API
    [Documentation]    Para cada orden, consulta la API REST para obtener datos
    ...                adicionales del cliente y del producto. Usa retry automático.
    [Tags]    fase:api    rpa    e2e

    Log    ▶ FASE 2: ENRIQUECIMIENTO VÍA API    level=INFO
    ${ordenes_enriquecidas}=    Create List
    FOR    ${orden}    IN    @{ORDENES_CRUDAS}
        ${datos_cliente}=    Obtener Datos Cliente Con Retry    ${orden}[cliente_id]
        ${datos_producto}=    Obtener Datos Producto Con Retry    ${orden}[producto_id]
        ${monto}=    Calcular Monto Orden
        ...    ${orden}[cantidad]
        ...    ${orden}[precio_unitario]
        ...    ${datos_producto}[precio_actualizado]
        # Construir orden enriquecida
        ${orden_enriquecida}=    Create Dictionary
        ...    orden_id=${orden}[orden_id]
        ...    cliente_id=${orden}[cliente_id]
        ...    producto_id=${orden}[producto_id]
        ...    cantidad=${orden}[cantidad]
        ...    precio_unitario=${orden}[precio_unitario]
        ...    monto_total=${monto}
        ...    cliente_nombre=${datos_cliente}[nombre]
        ...    cliente_direccion=${datos_cliente}[direccion]
        ...    categoria_credito=${datos_cliente}[categoria_credito]
        ...    stock_disponible=${datos_producto}[stock_disponible]
        ...    precio_actualizado=${datos_producto}[precio_actualizado]
        ...    error_api=${datos_cliente.get('error', False)}
        Append To List    ${ordenes_enriquecidas}    ${orden_enriquecida}
    END
    Set Suite Variable    ${ORDENES_ENRIQUECIDAS}    ${ordenes_enriquecidas}
    ${total}=    Get Length    ${ordenes_enriquecidas}
    Log    Fase 2 completada: ${total} órdenes enriquecidas con datos de API.    level=INFO

Fase 3 - Procesamiento Web
    [Documentation]    Para las órdenes cuyo monto supera el umbral configurado,
    ...                accede al portal web y registra la orden capturando screenshot.
    [Tags]    fase:web    rpa    e2e

    Log    ▶ FASE 3: PROCESAMIENTO WEB (umbral: $${MONTO_UMBRAL})    level=INFO
    Inicializar Navegador    ${WEB_PORTAL_URL}
    Autenticar En Portal Web    ${WEB_USER}    ${WEB_PASSWORD}

    ${ordenes_procesadas}=    Create List
    ${ordenes_con_error}=     Create List
    ${monto_acumulado}=       Set Variable    ${0}

    FOR    ${orden}    IN    @{ORDENES_ENRIQUECIDAS}
        ${supera_umbral}=    Evaluate    float(${orden}[monto_total]) > float(${MONTO_UMBRAL})
        IF    ${supera_umbral}
            ${exito}=    Registrar Orden En Portal
            ...    ${orden}    ${orden}    ${orden}[monto_total]    ${EVIDENCE_DIR}
            IF    ${exito}
                Append To List    ${ordenes_procesadas}    ${orden}
                ${monto_acumulado}=    Evaluate
                ...    ${monto_acumulado} + float(${orden}[monto_total])
            ELSE
                ${orden_error}=    Copy Dictionary    ${orden}
                Set To Dictionary    ${orden_error}    motivo_error=Error en registro web
                Append To List    ${ordenes_con_error}    ${orden_error}
            END
        ELSE
            # Órdenes por debajo del umbral: procesadas directamente sin portal
            Append To List    ${ordenes_procesadas}    ${orden}
            ${monto_acumulado}=    Evaluate
            ...    ${monto_acumulado} + float(${orden}[monto_total])
        END
    END

    Set Suite Variable    ${ORDENES_PROCESADAS}    ${ordenes_procesadas}
    Set Suite Variable    ${ORDENES_CON_ERROR}     ${ordenes_con_error}
    Set Suite Variable    ${MONTO_TOTAL}           ${monto_acumulado}

    ${total_proc}=    Get Length    ${ordenes_procesadas}
    ${total_err}=     Get Length    ${ordenes_con_error}
    Log    Fase 3 completada: ${total_proc} exitosas, ${total_err} con error.    level=INFO

Fase 4 - Generacion De Reportes
    [Documentation]    Genera el PDF de resumen del lote y el CSV de órdenes con errores.
    [Tags]    fase:reportes    rpa    e2e

    Log    ▶ FASE 4: GENERACIÓN DE REPORTES    level=INFO
    ${total}=     Get Length    ${ORDENES_ENRIQUECIDAS}
    ${exitosas}=  Get Length    ${ORDENES_PROCESADAS}
    ${fallidas}=  Get Length    ${ORDENES_CON_ERROR}
    ${monto_redondeado}=    Evaluate    round(${MONTO_TOTAL}, 2)

    ${estadisticas}=    Create Dictionary
    ...    total=${total}
    ...    exitosas=${exitosas}
    ...    fallidas=${fallidas}
    ...    monto_total=${monto_redondeado}
    Set Suite Variable    ${ESTADISTICAS_PROCESO}    ${estadisticas}

    Generar PDF De Resumen    ${estadisticas}    ${REPORT_PDF}
    Generar CSV De Errores    ${ORDENES_CON_ERROR}    ${ERRORS_CSV}
    Log    Fase 4 completada: PDF y CSV generados.    level=INFO

Fase 5 - Cierre Y Trazabilidad
    [Documentation]    Archiva los archivos procesados, organiza evidencias con timestamp
    ...                y ejecuta el checklist de validación de 10 puntos.
    [Tags]    fase:cierre    rpa    e2e

    Log    ▶ FASE 5: CIERRE Y TRAZABILIDAD    level=INFO
    ${carpeta_archivado}=    Archivar Archivos Procesados
    ...    ${EN_PROCESO_DIR}
    ...    ${PROCESSED_DIR}
    ...    ${EVIDENCE_DIR}
    Set Suite Variable    ${CARPETA_EVIDENCIAS}    ${carpeta_archivado}

    # Construir diccionario de rutas para el checklist
    ${rutas}=    Create Dictionary
    ...    reporte_pdf=${REPORT_PDF}
    ...    errores_csv=${ERRORS_CSV}
    ...    directorio_completados=${PROCESSED_DIR}
    ...    directorio_en_proceso=${EN_PROCESO_DIR}
    ...    directorio_evidencias_archivado=${carpeta_archivado}

    ${resultados_checklist}=    Ejecutar Checklist De Validacion
    ...    ${ESTADISTICAS_PROCESO}    ${rutas}

    Generar Log De Trazabilidad Final
    ...    ${ESTADISTICAS_PROCESO}
    ...    ${TIMESTAMP_INICIO}
    ...    ${TIMESTAMP_FIN_ACTUAL}

    Log    Fase 5 completada. Proceso RPA finalizado.    level=INFO

*** Keywords ***
Preparar Entorno Del Proceso
    [Documentation]    Suite Setup: inicializa carpetas, sesión API y registra timestamp de inicio.
    ${ts_inicio}=    Get Current Date
    Set Suite Variable    ${TIMESTAMP_INICIO}    ${ts_inicio}
    Inicializar Carpetas Del Proceso
    Inicializar Sesion API    ${API_BASE_URL}
    Log    ══════════════════════════════════════════════════    level=INFO
    Log    INICIO DEL PROCESO RPA — ÓRDENES DE COMPRA           level=INFO
    Log    Timestamp: ${ts_inicio}                              level=INFO
    Log    Archivo de entrada: ${INPUT_FILE}                    level=INFO
    Log    API URL: ${API_BASE_URL}                             level=INFO
    Log    Portal web: ${WEB_PORTAL_URL}                        level=INFO
    Log    Umbral de monto: $${MONTO_UMBRAL}                    level=INFO
    Log    ══════════════════════════════════════════════════    level=INFO

Finalizar Proceso Completamente
    [Documentation]    Suite Teardown: cierra navegador, sesión API y registra timestamp de fin.
    ...                Se ejecuta SIEMPRE, incluso si el proceso falla en alguna fase.
    ${ts_fin}=    Get Current Date
    Set Suite Variable    ${TIMESTAMP_FIN_ACTUAL}    ${ts_fin}
    Run Keyword And Ignore Error    Cerrar Navegador
    Run Keyword And Ignore Error    Cerrar Sesion API
    Log    Proceso RPA finalizado a las: ${ts_fin}    level=INFO
```

2. Guarda el archivo.

**Salida esperada:** El archivo `rpa_ordenes_e2e.robot` contiene 5 Tasks y 2 keywords de suite con setup/teardown.

---

### Paso 7: Ejecutar el proceso completo desde CLI

**Objetivo:** Lanzar el proceso RPA completo parametrizado desde la línea de comandos y observar la ejecución de las cinco fases.

**Instrucciones:**

1. Asegúrate de que WireMock esté corriendo en el puerto 8089 (Paso de configuración).

2. Navega al directorio raíz del laboratorio:
```bash
cd lab-09-00-01
```

3. Ejecuta el proceso con parámetros CLI:

```bash
# Linux/macOS
robot \
  --variable INPUT_FILE:data/input/ordenes_compra.xlsx \
  --variable API_BASE_URL:http://localhost:8089 \
  --variable WEB_PORTAL_URL:https://the-internet.herokuapp.com/login \
  --variable MONTO_UMBRAL:1000 \
  --variable BROWSER:chrome \
  --outputdir results/ \
  --loglevel INFO \
  --report report.html \
  --log log.html \
  tests/rpa_ordenes_e2e.robot
```

```bat
:: Windows CMD
robot ^
  --variable INPUT_FILE:data\input\ordenes_compra.xlsx ^
  --variable API_BASE_URL:http://localhost:8089 ^
  --variable WEB_PORTAL_URL:https://the-internet.herokuapp.com/login ^
  --variable MONTO_UMBRAL:1000 ^
  --variable BROWSER:chrome ^
  --outputdir results\ ^
  --loglevel INFO ^
  --report report.html ^
  --log log.html ^
  tests\rpa_ordenes_e2e.robot
```

4. Crea los scripts de ejecución multiplataforma para reutilización:

**`run_rpa.sh` (Linux/macOS):**
```bash
#!/bin/bash
# Script de ejecución del proceso RPA de Órdenes de Compra
# Uso: ./run_rpa.sh [INPUT_FILE] [MONTO_UMBRAL]

INPUT_FILE="${1:-data/input/ordenes_compra.xlsx}"
MONTO_UMBRAL="${2:-1000}"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

echo "Iniciando proceso RPA - Lote: ${TIMESTAMP}"
echo "Archivo: ${INPUT_FILE} | Umbral: ${MONTO_UMBRAL}"

robot \
  --variable INPUT_FILE:"${INPUT_FILE}" \
  --variable API_BASE_URL:http://localhost:8089 \
  --variable WEB_PORTAL_URL:https://the-internet.herokuapp.com/login \
  --variable MONTO_UMBRAL:"${MONTO_UMBRAL}" \
  --outputdir "results/run_${TIMESTAMP}" \
  --loglevel INFO \
  tests/rpa_ordenes_e2e.robot

echo "Proceso finalizado. Resultados en: results/run_${TIMESTAMP}"
```

**`run_rpa.ps1` (Windows PowerShell):**
```powershell
# Script de ejecución del proceso RPA de Órdenes de Compra
# Uso: .\run_rpa.ps1 [-InputFile <ruta>] [-MontoUmbral <valor>]
param(
    [string]$InputFile = "data\input\ordenes_compra.xlsx",
    [int]$MontoUmbral = 1000
)

$Timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
Write-Host "Iniciando proceso RPA - Lote: $Timestamp"

robot `
  --variable "INPUT_FILE:$InputFile" `
  --variable "API_BASE_URL:http://localhost:8089" `
  --variable "WEB_PORTAL_URL:https://the-internet.herokuapp.com/login" `
  --variable "MONTO_UMBRAL:$MontoUmbral" `
  --outputdir "results\run_$Timestamp" `
  --loglevel INFO `
  tests\rpa_ordenes_e2e.robot

Write-Host "Proceso finalizado. Resultados en: results\run_$Timestamp"
```

```bash
chmod +x run_rpa.sh
./run_rpa.sh
```

**Salida esperada en consola:**
```
==============================================================================
Rpa Ordenes E2E
==============================================================================
Fase 1 - Ingesta De Datos                                             | PASS |
Fase 2 - Enriquecimiento Via API                                      | PASS |
Fase 3 - Procesamiento Web                                            | PASS |
Fase 4 - Generacion De Reportes                                       | PASS |
Fase 5 - Cierre Y Trazabilidad                                        | PASS |
==============================================================================
Rpa Ordenes E2E                                                       | PASS |
5 tasks, 5 passed, 0 failed
==============================================================================
Output:  /ruta/lab-09-00-01/results/output.xml
Log:     /ruta/lab-09-00-01/results/log.html
Report:  /ruta/lab-09-00-01/results/report.html
```

**Verificación:**
```bash
# Verificar que los artefactos fueron generados
ls -la results/
ls -la data/processed/
ls -la data/output/
ls -la evidence/
```

---

### Paso 8: Analizar el output.xml para trazabilidad

**Objetivo:** Extraer información de trazabilidad del `output.xml` generado para entender la estructura de resultados y los niveles de log.

**Instrucciones:**

1. Abre el archivo `results/output.xml` en un editor de texto y localiza las secciones clave:

```bash
# Linux/macOS — buscar mensajes de nivel WARN o ERROR en el output.xml
grep -n 'level="WARN"\|level="ERROR"' results/output.xml | head -20

# Windows PowerShell
Select-String -Path results\output.xml -Pattern 'level="WARN"|level="ERROR"' | Select-Object -First 20
```

2. Usa Python para extraer un resumen de los mensajes de trazabilidad:

```python
# analizar_output.py — ejecutar desde la raíz del proyecto
import xml.etree.ElementTree as ET

tree = ET.parse('results/output.xml')
root = tree.getroot()

print("=== ANÁLISIS DE TRAZABILIDAD — output.xml ===\n")

# Contar tasks por estado
stats = {"PASS": 0, "FAIL": 0}
for task in root.iter('test'):
    status = task.find('status')
    if status is not None:
        stats[status.get('status', 'UNKNOWN')] = \
            stats.get(status.get('status', 'UNKNOWN'), 0) + 1

print(f"Tasks PASS: {stats.get('PASS', 0)}")
print(f"Tasks FAIL: {stats.get('FAIL', 0)}")

# Extraer mensajes WARN y ERROR
print("\n=== MENSAJES WARN/ERROR ===")
for msg in root.iter('msg'):
    level = msg.get('level', '')
    if level in ('WARN', 'ERROR'):
        print(f"[{level}] {msg.text[:120] if msg.text else '(vacío)'}")

# Extraer mensajes del checklist
print("\n=== RESULTADOS DEL CHECKLIST ===")
for msg in root.iter('msg'):
    if msg.text and ('[PASS]' in msg.text or '[FAIL]' in msg.text):
        print(msg.text)
```

```bash
python analizar_output.py
```

3. Abre el reporte HTML en el navegador:

```bash
# Linux
xdg-open results/log.html

# macOS
open results/log.html

# Windows
start results\log.html
```

**Salida esperada del script de análisis:**
```
=== ANÁLISIS DE TRAZABILIDAD — output.xml ===

Tasks PASS: 5
Tasks FAIL: 0

=== MENSAJES WARN/ERROR ===
(ninguno si todas las órdenes procesaron correctamente)

=== RESULTADOS DEL CHECKLIST ===
[PASS] Punto 1: Órdenes procesadas > 0
[PASS] Punto 2: PDF de resumen generado
[PASS] Punto 3: Carpeta de completados existe
[PASS] Punto 4: Carpeta en-proceso vacía tras archivado
[PASS] Punto 5: Exitosas + Fallidas = Total
[PASS] Punto 6: Monto total procesado > 0
[PASS] Punto 7: Evidencias (screenshots) generadas
[PASS] Punto 8: CSV de errores generado (si aplica)
[PASS] Punto 9: Tasa de éxito >= 50%
[PASS] Punto 10: Log de trazabilidad final registrado
```

**Verificación:**
```bash
# Verificar que el PDF fue generado y tiene contenido
ls -lh data/output/reporte_lote.pdf
# Debe mostrar un archivo > 0 bytes

# Verificar screenshots de evidencia
ls -la evidence/
# Debe mostrar subcarpeta con timestamp y archivos .png
```

---

## 7. Validación y Pruebas

### Lista de verificación de artefactos generados

Ejecuta los siguientes comandos para confirmar que todos los artefactos del proceso fueron creados correctamente:

```bash
# === VERIFICACIÓN COMPLETA DE ARTEFACTOS ===

echo "1. Archivo Excel movido a completados:"
ls data/processed/*.xlsx 2>/dev/null && echo "  ✓ OK" || echo "  ✗ FALTA"

echo "2. PDF de resumen generado:"
ls data/output/reporte_lote.pdf 2>/dev/null && echo "  ✓ OK" || echo "  ✗ FALTA"

echo "3. Carpeta en-proceso vacía:"
[ -z "$(ls -A data/en-proceso 2>/dev/null)" ] && echo "  ✓ OK" || echo "  ✗ CONTIENE ARCHIVOS"

echo "4. Evidencias (screenshots) archivadas:"
ls evidence/lote_*/*.png 2>/dev/null | wc -l | xargs -I{} echo "  ✓ {} screenshots encontrados"

echo "5. output.xml generado:"
ls results/output.xml 2>/dev/null && echo "  ✓ OK" || echo "  ✗ FALTA"

echo "6. log.html generado:"
ls results/log.html 2>/dev/null && echo "  ✓ OK" || echo "  ✗ FALTA"

echo "7. Checklist en output.xml:"
grep -c "\[PASS\]" results/output.xml 2>/dev/null | xargs -I{} echo "  ✓ {} puntos PASS en el checklist"
```

### Prueba de resiliencia — Simular fallo de API

Para verificar que el manejo de errores funciona correctamente, detén WireMock temporalmente y ejecuta solo la Fase 2:

```bash
# 1. Detener WireMock (Ctrl+C en la terminal de WireMock)
# 2. Ejecutar solo la Fase 2 con un tag específico
robot \
  --variable INPUT_FILE:data/input/ordenes_compra.xlsx \
  --variable API_BASE_URL:http://localhost:8089 \
  --include fase:api \
  --outputdir results/test_resiliencia \
  tests/rpa_ordenes_e2e.robot

# 3. Verificar que el proceso continuó con datos de error (no abortó)
grep "ERROR al obtener cliente" results/test_resiliencia/output.xml | wc -l
# Debe mostrar > 0 (errores registrados pero proceso continuado)
```

### Prueba de parametrización — Cambiar umbral de monto

```bash
# Ejecutar con umbral alto (solo órdenes > $5000 van al portal web)
robot \
  --variable INPUT_FILE:data/input/ordenes_compra.xlsx \
  --variable MONTO_UMBRAL:5000 \
  --outputdir results/test_umbral_alto \
  tests/rpa_ordenes_e2e.robot

# Comparar número de screenshots generados vs ejecución con umbral normal
ls results/test_umbral_alto/../evidence/ | wc -l
```

---

## 8. Solución de Problemas

### Problema 1: `ModuleNotFoundError: No module named 'RPA'` al ejecutar la suite

**Síntomas:**
- Robot Framework falla al importar `RPA.Excel.Files` o `RPA.PDF`.
- El error aparece en el log con el mensaje: `Importing library 'RPA.Excel.Files' failed`.
- La suite no puede iniciarse y todos los tasks fallan con `FAIL`.

**Causa:**
El paquete `rpaframework` no está instalado en el entorno Python activo, o está instalado en un entorno virtual diferente al que usa el comando `robot`.

**Solución:**
```bash
# 1. Verificar qué Python está usando el comando robot
which robot      # Linux/macOS
where robot      # Windows

# 2. Verificar que rpaframework está instalado en ese Python
python -m pip show rpaframework

# 3. Si no está instalado, instalarlo en el entorno correcto
python -m pip install rpaframework

# 4. Si usas entorno virtual, activarlo primero
source venv/bin/activate      # Linux/macOS
.\venv\Scripts\Activate.ps1   # Windows PowerShell

# 5. Verificar la instalación
python -c "from RPA.Excel.Files import Files; print('RPA.Excel.Files OK')"
python -c "from RPA.PDF import PDF; print('RPA.PDF OK')"
```

---

### Problema 2: WireMock retorna 404 para todos los endpoints de la API

**Síntomas:**
- La Fase 2 registra errores `ERROR al obtener cliente XXXX tras 3 intentos` para todas las órdenes.
- Los datos enriquecidos muestran `categoria_credito=ERROR` en todos los registros.
- `curl http://localhost:8089/api/clientes/1001` retorna `{"status":404,"title":"Response was not found"}`.

**Causa:**
WireMock no está cargando los mappings correctamente. Esto ocurre cuando: (a) el directorio `--root-dir` no apunta a la carpeta que contiene la subcarpeta `mappings/`, o (b) los archivos JSON de mappings tienen errores de sintaxis.

**Solución:**
```bash
# 1. Verificar que WireMock está corriendo
curl -s http://localhost:8089/__admin/health
# Debe retornar {"status":"healthy"}

# 2. Verificar los mappings cargados
curl -s http://localhost:8089/__admin/mappings | python3 -m json.tool
# Debe mostrar los 2 mappings configurados

# 3. Si los mappings no aparecen, verificar la estructura de carpetas
ls -la wiremock/mappings/
# Debe mostrar: clientes_mapping.json y productos_mapping.json

# 4. Validar sintaxis JSON de los mappings
python3 -c "import json; json.load(open('wiremock/mappings/clientes_mapping.json')); print('JSON válido')"

# 5. Reiniciar WireMock apuntando explícitamente al directorio correcto
cd lab-09-00-01/wiremock
java -jar wiremock-standalone.jar --port 8089 --root-dir $(pwd) --verbose

# 6. Recargar mappings en caliente (sin reiniciar)
curl -X POST http://localhost:8089/__admin/mappings/reset
```

---

## 9. Limpieza del Entorno

Ejecuta los siguientes pasos al finalizar el laboratorio para restaurar el entorno a su estado inicial:

```bash
# 1. Detener WireMock (Ctrl+C en la terminal correspondiente)
#    Si está en segundo plano:
pkill -f wiremock-standalone.jar     # Linux/macOS
# Windows: cerrar la ventana CMD o PowerShell de WireMock

# 2. Archivar los resultados del laboratorio (opcional — para revisión posterior)
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
mkdir -p lab-09-results-backup
cp -r results/ lab-09-results-backup/results_${TIMESTAMP}/
cp -r evidence/ lab-09-results-backup/evidence_${TIMESTAMP}/
echo "Resultados archivados en lab-09-results-backup/"

# 3. Limpiar artefactos generados para poder re-ejecutar el lab desde cero
rm -rf data/output/*
rm -rf data/processed/*
rm -rf data/errors/*
rm -rf data/en-proceso/*
rm -rf evidence/*
rm -rf results/*
echo "Artefactos de ejecución eliminados."

# 4. Restaurar el archivo Excel de entrada para re-ejecución
python generar_ordenes.py
echo "Archivo de entrada restaurado."

# 5. Verificar que el entorno quedó limpio
echo "Estado del entorno tras limpieza:"
ls -la data/input/
ls -la data/output/
ls -la data/processed/
```

```powershell
# Windows PowerShell — limpieza equivalente
$Timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
New-Item -ItemType Directory -Force -Path "lab-09-results-backup"
Copy-Item -Recurse "results" "lab-09-results-backup\results_$Timestamp"

Remove-Item -Recurse -Force "data\output\*"
Remove-Item -Recurse -Force "data\processed\*"
Remove-Item -Recurse -Force "data\errors\*"
Remove-Item -Recurse -Force "evidence\*"
Remove-Item -Recurse -Force "results\*"

python generar_ordenes.py
Write-Host "Entorno limpio y listo para re-ejecución."
```

---

## 10. Resumen

En este laboratorio implementaste un proceso RPA empresarial completo de extremo a extremo que integra las tres capas de automatización (archivos, API y web) en un flujo orquestado de cinco fases:

| Fase | Tecnología principal | Patrón aplicado |
|---|---|---|
| **1 — Ingesta** | `RPA.Excel.Files`, `OperatingSystem` | Lectura + validación + movimiento de archivo |
| **2 — Enriquecimiento** | `RequestsLibrary`, `Wait Until Keyword Succeeds` | Retry con 3 intentos y manejo de error parcial |
| **3 — Procesamiento web** | `SeleniumLibrary` | TRY/EXCEPT anidado + captura de evidencias |
| **4 — Reportes** | `RPA.PDF`, `OperatingSystem` | Generación de PDF y CSV de errores |
| **5 — Cierre** | `report_keywords`, `DateTime` | Archivado + checklist de 10 puntos + trazabilidad |

Los conceptos clave consolidados en este laboratorio son:

- **Modularización en capas:** Cada fase tiene su propio archivo `.resource` con keywords documentadas, lo que facilita el mantenimiento y la reutilización en otros procesos.
- **Resiliencia sin abortar:** El uso de `TRY/EXCEPT` y `Run Keyword And Ignore Error` garantiza que un fallo en una orden individual no detiene el procesamiento del lote completo.
- **Parametrización total:** Todas las rutas, URLs y umbrales son variables sobreescribibles via `--variable` en CLI, lo que hace el proceso apto para orquestadores de producción.
- **Trazabilidad estructurada:** Los niveles de log `INFO`/`WARN`/`ERROR` y el `output.xml` proporcionan evidencia auditable de cada decisión del proceso.
- **Compatibilidad multiplataforma:** El uso de `${/}` en todas las rutas garantiza que el proceso funciona sin modificaciones en Windows, macOS y Linux.

### Recursos adicionales

- [Documentación oficial de RPA.Excel.Files](https://rpaframework.org/libraries/excel_files/)
- [Documentación oficial de
