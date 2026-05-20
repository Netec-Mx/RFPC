# Práctica 4 — Integrador y Plan de Adopción Regional

## Metadatos

| Campo         | Valor                                      |
|---------------|--------------------------------------------|
| **Duración**  | 36 minutos                                 |
| **Complejidad** | Media                                    |
| **Nivel Bloom** | Evaluar (Evaluate)                       |
| **Módulo**    | 10 — CLI avanzada, reportes y cierre RFCP  |

---

## Descripción General

Este laboratorio integrador cierra el curso consolidando las habilidades adquiridas en los módulos 7, 8 y 9. A través de cuatro bloques secuenciales, el participante dominará la CLI avanzada de Robot Framework, analizará reportes de ejecución reales con fallos intencionales, diseñará conceptualmente un pipeline CI/CD y resolverá un simulacro de 20 preguntas estilo RFCP con revisión guiada. Al completarlo, el participante dispondrá de un artefacto ejecutable (`run_all.sh` / `run_all.bat`) y un template de pipeline YAML listo para adaptar a proyectos reales.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio, el participante será capaz de:

- [ ] Construir y ejecutar comandos `robot` avanzados usando `--include`, `--exclude`, `--variable`, `--outputdir` y `--rerunfailed` sobre suites de múltiples módulos.
- [ ] Analizar un `output.xml` con fallos intencionales para identificar los tests más lentos, patrones de fallo recurrentes y calcular KPIs de calidad por suite y por tag.
- [ ] Completar un template de pipeline YAML (GitHub Actions / GitLab CI) con stages, quality gates y estrategia de artefactos.
- [ ] Resolver un simulacro de 20 preguntas tipo RFCP y autoevaluar el resultado identificando áreas de mejora.

---

## Prerrequisitos

### Conocimiento previo
- Laboratorios 07-00-01 (web con SeleniumLibrary), 08-00-01 (API con RequestsLibrary) y 09-00-01 (RPA con rpaframework) completados.
- Comprensión de la estructura de proyectos Robot Framework (suites, recursos, variables).
- Conocimiento conceptual de pipelines CI/CD (stages, artefactos, notificaciones).
- Familiaridad con tags en Robot Framework (`smoke`, `regression`, `e2e`).

### Acceso y archivos requeridos
- Resultados de ejecución previos: `output.xml`, `log.html` y `report.html` de los laboratorios 07, 08 y 09 (o los archivos pre-generados del repositorio del curso).
- Repositorio del curso clonado localmente con la estructura de módulos intacta.
- Git instalado y configurado.
- Conexión a internet (opcional para Bloque 3; los bloques 1, 2 y 4 son completamente offline).

---

## Entorno de Laboratorio

### Hardware mínimo recomendado

| Recurso     | Mínimo          | Recomendado     |
|-------------|-----------------|-----------------|
| CPU         | Core i5 8ª gen  | Core i7 / Ryzen 7 |
| RAM         | 8 GB            | 16 GB           |
| Almacenamiento | 10 GB libres | 20 GB libres    |
| Pantalla    | 1280×768        | 1920×1080       |

### Software requerido

| Herramienta               | Versión mínima | Verificación                         |
|---------------------------|----------------|--------------------------------------|
| Python                    | 3.10           | `python --version`                   |
| Robot Framework           | 7.0            | `robot --version`                    |
| SeleniumLibrary           | 6.2            | `pip show robotframework-seleniumlibrary` |
| RequestsLibrary           | 0.9.4          | `pip show robotframework-requests`   |
| rpaframework              | 28.x           | `pip show rpaframework`              |
| rebot (incluido con RF)   | 7.0            | `rebot --version`                    |
| Git                       | 2.40           | `git --version`                      |
| VS Code + RF Language Server | 1.85 / 1.12 | Verificación visual                  |

### Preparación del entorno

Ejecuta los siguientes comandos para verificar que el entorno está listo antes de comenzar:

```bash
# 1. Verificar instalaciones clave
python --version
robot --version
rebot --version
git --version

# 2. Ubicarse en la raíz del repositorio del curso
cd ~/curso-robot-framework   # Ajustar según tu ruta

# 3. Verificar que existen los resultados de laboratorios anteriores
ls results/lab07/
ls results/lab08/
ls results/lab09/

# 4. Si no existen resultados previos, usar los pre-generados del repositorio
cp -r course-assets/pregenerated-results/ results/
```

**Windows (PowerShell):**
```powershell
# Verificar instalaciones
python --version
robot --version
rebot --version
git --version

# Ubicarse en la raíz del repositorio
Set-Location "$env:USERPROFILE\curso-robot-framework"

# Verificar resultados previos
Get-ChildItem results\lab07\
Get-ChildItem results\lab08\
Get-ChildItem results\lab09\

# Si no existen, copiar los pre-generados
Copy-Item -Recurse course-assets\pregenerated-results\ results\
```

> **Nota sobre proxy corporativo:** Si trabajas en una red con proxy, configura las variables de entorno antes de ejecutar comandos pip:
> ```bash
> export HTTP_PROXY=http://proxy.empresa.com:8080
> export HTTPS_PROXY=http://proxy.empresa.com:8080
> ```
> En Windows: `$env:HTTP_PROXY = "http://proxy.empresa.com:8080"`

---

## Desarrollo del Laboratorio

### BLOQUE 1 — Maestría CLI (10 minutos)

**Objetivo:** Construir comandos `robot` avanzados que ejecuten selectivamente las suites de los módulos 7, 8 y 9, y empaquetar todo en un script único `run_all`.

---

#### Paso 1.1 — Verificar la estructura de tags en las suites existentes

**Objetivo:** Confirmar qué tags están disponibles en cada suite antes de filtrar.

**Instrucciones:**

1. Abre una terminal en la raíz del repositorio del curso.
2. Ejecuta el siguiente comando para listar todos los tags disponibles sin ejecutar pruebas:

```bash
# Modo dry-run: lista tags sin ejecutar
robot --dryrun --outputdir results/dryrun/ tests/
```

3. Revisa la salida en consola. Deberías ver los tags asignados en los laboratorios anteriores. Si no tienes tags asignados, agrega los siguientes a los archivos `.robot` de cada módulo:

```robotframework
# En tests/lab07/web_suite.robot — agregar a cada test:
# [Tags]    smoke    web    e2e

# En tests/lab08/api_suite.robot — agregar a cada test:
# [Tags]    smoke    api    regression

# En tests/lab09/rpa_suite.robot — agregar a cada test:
# [Tags]    regression    rpa    e2e
```

4. Vuelve a ejecutar el dry-run para confirmar los tags.

**Salida esperada:**
```
==============================================================================
Dry Run
==============================================================================
tests/lab07/web_suite.robot                                           | PASS |
tests/lab08/api_suite.robot                                           | PASS |
tests/lab09/rpa_suite.robot                                           | PASS |
==============================================================================
```

**Verificación:** La consola muestra todos los tests sin ejecutarlos realmente (modo dry-run). No debe haber errores de sintaxis.

---

#### Paso 1.2 — Ejecución filtrada por tags

**Objetivo:** Ejecutar únicamente los tests marcados como `smoke` de todas las suites.

**Instrucciones:**

1. Ejecuta solo los smoke tests de todas las suites:

```bash
# Ejecutar solo smoke tests en todas las suites
robot \
  --include smoke \
  --outputdir results/run_smoke/ \
  --log log_smoke.html \
  --report report_smoke.html \
  tests/
```

2. Ejecuta los tests de regresión excluyendo los marcados como `wip`:

```bash
# Regression sin tests en progreso
robot \
  --include regression \
  --exclude wip \
  --outputdir results/run_regression/ \
  --log log_regression.html \
  --report report_regression.html \
  tests/
```

3. Ejecuta tests que tengan el tag `api` **Y** `smoke` simultáneamente:

```bash
# Lógica AND: tests que sean api Y smoke
robot \
  --include apiANDsmoke \
  --outputdir results/run_api_smoke/ \
  tests/
```

**Salida esperada:**
```
==============================================================================
Tests
==============================================================================
tests/lab08/api_suite.robot :: Suite de pruebas API                   | PASS |
------------------------------------------------------------------------------
Tests                                                                  | PASS |
3 tests, 3 passed, 0 failed
==============================================================================
```

**Verificación:** Solo los tests con ambos tags (`api` y `smoke`) aparecen en la ejecución.

---

#### Paso 1.3 — Inyección de variables de entorno desde CLI

**Objetivo:** Parametrizar la ejecución para diferentes entornos sin modificar los archivos fuente.

**Instrucciones:**

1. Ejecuta la suite de API apuntando a un entorno específico:

```bash
# Sobreescribir BASE_URL y TIMEOUT para entorno de staging
robot \
  --variable BASE_URL:https://reqres.in \
  --variable TIMEOUT:15 \
  --variable ENV:staging \
  --include smoke \
  --outputdir results/run_staging/ \
  tests/lab08/
```

2. Crea un archivo de variables para el entorno de producción:

```bash
# Crear archivo de variables (config/vars_prod.py)
cat > config/vars_prod.py << 'EOF'
BASE_URL = "https://api.produccion.com"
TIMEOUT = 30
ENV = "production"
RETRY_COUNT = 3
EOF
```

**Windows (PowerShell):**
```powershell
# Crear el archivo de variables en Windows
New-Item -ItemType Directory -Force -Path config
@"
BASE_URL = "https://api.produccion.com"
TIMEOUT = 30
ENV = "production"
RETRY_COUNT = 3
"@ | Out-File -FilePath config\vars_prod.py -Encoding UTF8
```

3. Ejecuta usando el archivo de variables:

```bash
robot \
  --variablefile config/vars_prod.py \
  --include smoke \
  --outputdir results/run_prod/ \
  tests/lab08/
```

**Salida esperada:** Los tests se ejecutan usando los valores del archivo de variables. En el `log.html` generado, las variables deben mostrar los valores inyectados.

**Verificación:** Abre `results/run_staging/log.html` en el navegador. En la sección de variables globales, confirma que `${BASE_URL}` muestra `https://reqres.in`.

---

#### Paso 1.4 — Mecanismo de rerun para tests fallidos

**Objetivo:** Practicar el ciclo completo de ejecución → fallo → rerun → merge.

**Instrucciones:**

1. Ejecuta todas las suites para generar un `output.xml` de referencia (es normal que algunos tests fallen si los servicios no están disponibles):

```bash
# Ejecución inicial completa
robot \
  --outputdir results/initial_run/ \
  --log log_initial.html \
  --report report_initial.html \
  tests/
```

2. Relanza únicamente los tests que fallaron:

```bash
# Rerun de fallos usando el output.xml anterior
robot \
  --rerunfailed results/initial_run/output.xml \
  --outputdir results/rerun/ \
  --log log_rerun.html \
  --report report_rerun.html \
  tests/
```

3. Fusiona ambos resultados en un reporte consolidado:

```bash
# Merge de resultados: el rerun sobreescribe el estado de los tests que repitió
rebot \
  --merge \
  --outputdir results/final/ \
  --log log_final.html \
  --report report_final.html \
  results/initial_run/output.xml \
  results/rerun/output.xml
```

**Salida esperada:**
```
Rebot command line:  rebot --merge --outputdir results/final/ ...
Merging outputs.
==============================================================================
Output:  /ruta/results/final/output.xml
Log:     /ruta/results/final/log.html
Report:  /ruta/results/final/report.html
==============================================================================
```

**Verificación:** En `results/final/report.html`, los tests que pasaron en el rerun aparecen en verde. Los que volvieron a fallar permanecen en rojo.

---

#### Paso 1.5 — Construir el script `run_all`

**Objetivo:** Empaquetar toda la lógica de ejecución en un único script ejecutable multiplataforma.

**Instrucciones:**

1. Crea el script para Linux/macOS (`run_all.sh`):

```bash
cat > run_all.sh << 'SCRIPT'
#!/usr/bin/env bash
# =============================================================================
# run_all.sh — Script integrador de ejecución de suites
# Curso: Robot Framework Professional
# Uso: ./run_all.sh [ENV] [TAGS]
# Ejemplo: ./run_all.sh staging smoke
# =============================================================================

set -euo pipefail

# Parámetros con valores por defecto
ENV="${1:-qa}"
TAGS="${2:-smoke}"
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
RESULTS_DIR="results/${TIMESTAMP}_${ENV}_${TAGS}"
BASE_URL="${BASE_URL:-https://reqres.in}"

echo "=============================================="
echo "  Robot Framework — Ejecución Integrada"
echo "  Entorno: ${ENV} | Tags: ${TAGS}"
echo "  Resultados: ${RESULTS_DIR}"
echo "=============================================="

# Crear directorio de resultados
mkdir -p "${RESULTS_DIR}"

# PASO 1: Ejecución inicial de smoke tests
echo "[1/3] Ejecutando smoke tests..."
robot \
  --variable BASE_URL:"${BASE_URL}" \
  --variable ENV:"${ENV}" \
  --include "${TAGS}" \
  --exclude wip \
  --outputdir "${RESULTS_DIR}/initial" \
  --log log_initial.html \
  --report report_initial.html \
  tests/ || true  # Continuar aunque haya fallos

# PASO 2: Rerun de fallos (si existen)
if [ -f "${RESULTS_DIR}/initial/output.xml" ]; then
  FAILED=$(python3 -c "
import xml.etree.ElementTree as ET
tree = ET.parse('${RESULTS_DIR}/initial/output.xml')
root = tree.getroot()
failed = [s.get('name') for s in root.iter('test') if s.find('.//status[@status=\"FAIL\"]') is not None]
print(len(failed))
" 2>/dev/null || echo "0")

  if [ "${FAILED}" -gt "0" ]; then
    echo "[2/3] Reejecutando ${FAILED} test(s) fallido(s)..."
    robot \
      --rerunfailed "${RESULTS_DIR}/initial/output.xml" \
      --variable BASE_URL:"${BASE_URL}" \
      --variable ENV:"${ENV}" \
      --outputdir "${RESULTS_DIR}/rerun" \
      tests/ || true

    echo "[3/3] Fusionando resultados..."
    rebot \
      --merge \
      --outputdir "${RESULTS_DIR}/final" \
      --log log_final.html \
      --report report_final.html \
      "${RESULTS_DIR}/initial/output.xml" \
      "${RESULTS_DIR}/rerun/output.xml"
  else
    echo "[2/3] Sin fallos detectados. Omitiendo rerun."
    cp -r "${RESULTS_DIR}/initial" "${RESULTS_DIR}/final"
  fi
fi

echo ""
echo "=============================================="
echo "  Ejecución completada."
echo "  Reporte final: ${RESULTS_DIR}/final/report_final.html"
echo "=============================================="
SCRIPT

chmod +x run_all.sh
```

2. Crea el equivalente para Windows (`run_all.bat`):

```batch
@echo off
REM =============================================================================
REM run_all.bat — Script integrador de ejecucion de suites (Windows)
REM Uso: run_all.bat [ENV] [TAGS]
REM Ejemplo: run_all.bat staging smoke
REM =============================================================================

SET ENV=%1
IF "%ENV%"=="" SET ENV=qa

SET TAGS=%2
IF "%TAGS%"=="" SET TAGS=smoke

FOR /F "tokens=1-3 delims=/ " %%a IN ('date /t') DO SET DATE_STR=%%c%%a%%b
FOR /F "tokens=1-2 delims=: " %%a IN ('time /t') DO SET TIME_STR=%%a%%b
SET TIMESTAMP=%DATE_STR%_%TIME_STR%
SET RESULTS_DIR=results\%TIMESTAMP%_%ENV%_%TAGS%

IF "%BASE_URL%"=="" SET BASE_URL=https://reqres.in

echo ==============================================
echo   Robot Framework -- Ejecucion Integrada
echo   Entorno: %ENV% ^| Tags: %TAGS%
echo   Resultados: %RESULTS_DIR%
echo ==============================================

mkdir "%RESULTS_DIR%\initial" 2>nul

echo [1/3] Ejecutando smoke tests...
robot ^
  --variable BASE_URL:%BASE_URL% ^
  --variable ENV:%ENV% ^
  --include %TAGS% ^
  --exclude wip ^
  --outputdir "%RESULTS_DIR%\initial" ^
  --log log_initial.html ^
  --report report_initial.html ^
  tests\

echo [2/3] Reejecutando tests fallidos (si aplica)...
robot ^
  --rerunfailed "%RESULTS_DIR%\initial\output.xml" ^
  --outputdir "%RESULTS_DIR%\rerun" ^
  tests\ || echo Sin fallos para rerun.

echo [3/3] Fusionando resultados...
rebot ^
  --merge ^
  --outputdir "%RESULTS_DIR%\final" ^
  --log log_final.html ^
  --report report_final.html ^
  "%RESULTS_DIR%\initial\output.xml" ^
  "%RESULTS_DIR%\rerun\output.xml" 2>nul || (
    echo No se pudo hacer merge. Copiando resultado inicial.
    xcopy /E /I "%RESULTS_DIR%\initial" "%RESULTS_DIR%\final"
  )

echo.
echo ==============================================
echo   Ejecucion completada.
echo   Reporte: %RESULTS_DIR%\final\report_final.html
echo ==============================================
```

3. Verifica el script ejecutándolo:

```bash
# Linux/macOS
./run_all.sh qa smoke

# Windows
run_all.bat qa smoke
```

**Salida esperada:** El script ejecuta las tres suites, realiza el rerun automático si hay fallos, y genera el reporte final consolidado.

**Verificación:** Confirma que existe el directorio `results/<timestamp>_qa_smoke/final/` con `report_final.html` y `log_final.html`.

---

### BLOQUE 2 — Análisis de Reportes (10 minutos)

**Objetivo:** Analizar un `output.xml` con fallos intencionales para extraer métricas de calidad y proponer acciones correctivas.

---

#### Paso 2.1 — Preparar el output.xml de análisis

**Objetivo:** Disponer de un archivo `output.xml` con datos representativos para analizar.

**Instrucciones:**

1. Usa el archivo pre-generado del repositorio del curso o genera uno propio:

```bash
# Opción A: usar el archivo pre-generado (recomendado para ahorrar tiempo)
cp course-assets/analysis/output_with_failures.xml results/analysis/output.xml

# Opción B: generar uno propio ejecutando con fallos intencionales
robot \
  --variable FORCE_FAILURES:true \
  --outputdir results/analysis/ \
  tests/
```

2. Verifica que el archivo existe y tiene contenido:

```bash
# Linux/macOS
wc -l results/analysis/output.xml
head -50 results/analysis/output.xml

# Windows PowerShell
(Get-Content results\analysis\output.xml).Count
Get-Content results\analysis\output.xml | Select-Object -First 50
```

**Salida esperada:** El archivo debe tener al menos 200 líneas y mostrar la estructura XML con elementos `<suite>`, `<test>` y `<status>`.

---

#### Paso 2.2 — Identificar los 3 tests más lentos

**Objetivo:** Usar Python para extraer métricas de tiempo del `output.xml`.

**Instrucciones:**

1. Crea el script de análisis `analyze_output.py`:

```python
#!/usr/bin/env python3
"""
analyze_output.py — Extrae métricas de calidad de output.xml
Uso: python analyze_output.py results/analysis/output.xml
"""

import sys
import xml.etree.ElementTree as ET
from datetime import datetime

def parse_elapsed_time(start_time: str, end_time: str) -> float:
    """Calcula el tiempo transcurrido en segundos entre dos timestamps RF."""
    fmt = "%Y%m%d %H:%M:%S.%f"
    try:
        start = datetime.strptime(start_time, fmt)
        end = datetime.strptime(end_time, fmt)
        return (end - start).total_seconds()
    except (ValueError, TypeError):
        return 0.0

def analyze(output_xml_path: str):
    tree = ET.parse(output_xml_path)
    root = tree.getroot()

    tests = []
    passed = 0
    failed = 0
    tag_stats = {}

    for test in root.iter("test"):
        name = test.get("name", "Unknown")
        status_elem = test.find("status")
        if status_elem is None:
            continue

        status = status_elem.get("status", "UNKNOWN")
        start = status_elem.get("starttime", "")
        end = status_elem.get("endtime", "")
        elapsed = parse_elapsed_time(start, end)

        # Tags del test
        tags = [t.text for t in test.findall("tags/tag") if t.text]

        tests.append({
            "name": name,
            "status": status,
            "elapsed": elapsed,
            "tags": tags,
        })

        if status == "PASS":
            passed += 1
        else:
            failed += 1

        for tag in tags:
            if tag not in tag_stats:
                tag_stats[tag] = {"pass": 0, "fail": 0}
            if status == "PASS":
                tag_stats[tag]["pass"] += 1
            else:
                tag_stats[tag]["fail"] += 1

    total = passed + failed
    pass_rate = (passed / total * 100) if total > 0 else 0

    # ── REPORTE ──────────────────────────────────────────────────────────────
    print("=" * 60)
    print("  ANÁLISIS DE output.xml — Robot Framework")
    print("=" * 60)

    # KPI global
    print(f"\n📊 RESUMEN GLOBAL")
    print(f"   Total tests : {total}")
    print(f"   Pasaron     : {passed} ({pass_rate:.1f}%)")
    print(f"   Fallaron    : {failed} ({100 - pass_rate:.1f}%)")

    # Top 3 tests más lentos
    sorted_by_time = sorted(tests, key=lambda t: t["elapsed"], reverse=True)
    print(f"\n⏱️  TOP 3 TESTS MÁS LENTOS")
    for i, t in enumerate(sorted_by_time[:3], 1):
        print(f"   {i}. [{t['status']}] {t['name'][:50]} — {t['elapsed']:.2f}s")

    # Tests fallidos y patrón
    failed_tests = [t for t in tests if t["status"] == "FAIL"]
    print(f"\n❌ TESTS FALLIDOS ({len(failed_tests)})")
    for t in failed_tests:
        print(f"   - {t['name'][:55]} | Tags: {', '.join(t['tags'])}")

    # KPI por tag
    print(f"\n🏷️  KPI POR TAG")
    print(f"   {'Tag':<20} {'Pass':>6} {'Fail':>6} {'%Pass':>8}")
    print(f"   {'-'*20} {'-'*6} {'-'*6} {'-'*8}")
    for tag, counts in sorted(tag_stats.items()):
        total_tag = counts["pass"] + counts["fail"]
        pct = (counts["pass"] / total_tag * 100) if total_tag > 0 else 0
        print(f"   {tag:<20} {counts['pass']:>6} {counts['fail']:>6} {pct:>7.1f}%")

    print("\n" + "=" * 60)

    # Acciones correctivas sugeridas
    print("\n💡 ACCIONES CORRECTIVAS SUGERIDAS")
    if pass_rate < 95:
        print(f"   ⚠ Tasa de éxito {pass_rate:.1f}% < 95% (quality gate). Revisar fallos.")
    if sorted_by_time and sorted_by_time[0]["elapsed"] > 30:
        print(f"   ⚠ Test más lento: {sorted_by_time[0]['elapsed']:.1f}s. Considerar timeout o refactor.")
    if len(failed_tests) > 0:
        failing_tags = [tag for t in failed_tests for tag in t["tags"]]
        if failing_tags:
            most_common = max(set(failing_tags), key=failing_tags.count)
            print(f"   ⚠ Tag con más fallos: '{most_common}'. Investigar causa raíz.")
    print()

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Uso: python analyze_output.py <ruta/output.xml>")
        sys.exit(1)
    analyze(sys.argv[1])
```

2. Ejecuta el script de análisis:

```bash
python analyze_output.py results/analysis/output.xml
```

**Salida esperada:**
```
============================================================
  ANÁLISIS DE output.xml — Robot Framework
============================================================

📊 RESUMEN GLOBAL
   Total tests : 18
   Pasaron     : 14 (77.8%)
   Fallaron    : 4 (22.2%)

⏱️  TOP 3 TESTS MÁS LENTOS
   1. [PASS] Verificar carga completa de tabla dinámica   — 12.45s
   2. [FAIL] Crear usuario con datos inválidos            — 8.32s
   3. [PASS] Procesar archivo Excel con 500 filas         — 7.18s

❌ TESTS FALLIDOS (4)
   - Crear usuario con datos inválidos      | Tags: api, regression
   - Login con credenciales expiradas       | Tags: web, smoke
   - Validar esquema JSON respuesta 404     | Tags: api, smoke
   - Subir archivo mayor a límite permitido | Tags: rpa, e2e

🏷️  KPI POR TAG
   Tag                   Pass   Fail    %Pass
   -------------------- ------ ------ --------
   api                      5      2    71.4%
   e2e                      3      1    75.0%
   regression               4      1    80.0%
   rpa                      3      1    75.0%
   smoke                    4      2    66.7%
   web                      5      1    83.3%

💡 ACCIONES CORRECTIVAS SUGERIDAS
   ⚠ Tasa de éxito 77.8% < 95% (quality gate). Revisar fallos.
   ⚠ Tag con más fallos: 'api'. Investigar causa raíz.
```

**Verificación:** El script produce métricas legibles. Anota en papel o en un comentario de código las tres acciones correctivas que propondrías para el equipo.

---

#### Paso 2.3 — Debugging con log.html

**Objetivo:** Usar `log.html` para identificar la causa raíz de un fallo específico.

**Instrucciones:**

1. Abre `results/analysis/log.html` en el navegador (o el log del laboratorio 08 si no tienes el pre-generado).
2. Localiza el test fallido "Crear usuario con datos inválidos" (o cualquier test fallido disponible).
3. Expande el árbol de keywords hasta encontrar el mensaje de error exacto.
4. Responde mentalmente (o en un archivo de notas) las siguientes preguntas:

```
CHECKLIST DE DEBUGGING EN log.html:
□ ¿En qué keyword exacta ocurrió el fallo?
□ ¿Cuál es el mensaje de error completo?
□ ¿Qué valor tenía la variable en el momento del fallo?
□ ¿Cuánto tiempo tardó el test antes de fallar?
□ ¿Hay capturas de pantalla adjuntas (para tests web)?
□ ¿El fallo parece determinístico o intermitente (flaky)?
```

5. Usa `rebot` para generar un reporte filtrado solo con los fallos:

```bash
# Generar reporte solo con tests fallidos para revisión focalizada
rebot \
  --include FAILED \
  --outputdir results/failures_only/ \
  --log log_failures.html \
  --report report_failures.html \
  results/analysis/output.xml
```

> **Nota:** `--include FAILED` no es una opción nativa de `rebot`. Usa la siguiente alternativa con Python para filtrar:

```bash
# Alternativa: usar rebot con --test para casos específicos conocidos
rebot \
  --outputdir results/failures_only/ \
  results/analysis/output.xml
```

**Verificación:** El reporte `report_failures.html` muestra el porcentaje de éxito y la lista de fallos con sus mensajes de error.

---

### BLOQUE 3 — Diseño CI/CD Conceptual (6 minutos)

**Objetivo:** Completar un template de pipeline YAML con stages, quality gates y estrategia de artefactos.

---

#### Paso 3.1 — Completar el template de GitHub Actions

**Instrucciones:**

1. Crea el archivo `.github/workflows/robot_pipeline.yml` en el repositorio:

```bash
mkdir -p .github/workflows
```

2. Crea el pipeline completo con los stages requeridos:

```yaml
# .github/workflows/robot_pipeline.yml
# =============================================================================
# Pipeline CI/CD — Robot Framework Professional
# Stages: install-deps → run-smoke → run-regression → publish-report → notify
# =============================================================================

name: Robot Framework CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  schedule:
    # Ejecución nocturna de regresión completa (lunes-viernes a las 2 AM UTC)
    - cron: '0 2 * * 1-5'

env:
  PYTHON_VERSION: "3.11"
  RF_VERSION: ">=7.0"
  BASE_URL: ${{ vars.BASE_URL || 'https://reqres.in' }}
  SMOKE_PASS_THRESHOLD: 95   # Quality gate: mínimo 95% smoke
  REGRESSION_PASS_THRESHOLD: 85

jobs:
  # ─────────────────────────────────────────────────────────────────────────
  # STAGE 1: Instalación de dependencias
  # ─────────────────────────────────────────────────────────────────────────
  install-deps:
    name: "📦 Instalar Dependencias"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout del repositorio
        uses: actions/checkout@v4

      - name: Configurar Python ${{ env.PYTHON_VERSION }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'

      - name: Instalar dependencias Python
        run: |
          python -m pip install --upgrade pip
          pip install \
            "robotframework>=${{ env.RF_VERSION }}" \
            "robotframework-seleniumlibrary>=6.2" \
            "robotframework-requests>=0.9.4" \
            "rpaframework>=28.0" \
            "robotframework-datadriver>=1.11" \
            "pabot>=2.18" \
            "webdriver-manager"

      - name: Verificar instalaciones
        run: |
          robot --version
          rebot --version
          python -c "import SeleniumLibrary; print('SeleniumLibrary OK')"
          python -c "import RequestsLibrary; print('RequestsLibrary OK')"

      - name: Cache de ChromeDriver
        uses: actions/cache@v4
        with:
          path: ~/.wdm
          key: ${{ runner.os }}-chromedriver-${{ hashFiles('**/requirements.txt') }}

  # ─────────────────────────────────────────────────────────────────────────
  # STAGE 2: Smoke Tests (Quality Gate primario)
  # ─────────────────────────────────────────────────────────────────────────
  run-smoke:
    name: "🔥 Smoke Tests"
    needs: install-deps
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'

      - name: Instalar dependencias (desde cache)
        run: pip install -r requirements.txt

      - name: Instalar Chrome y ChromeDriver
        uses: browser-actions/setup-chrome@v1

      - name: Ejecutar Smoke Tests
        run: |
          robot \
            --variable BASE_URL:${{ env.BASE_URL }} \
            --variable ENV:ci \
            --variable BROWSER:headlesschrome \
            --include smoke \
            --exclude wip \
            --outputdir results/smoke/ \
            --log log_smoke.html \
            --report report_smoke.html \
            tests/
        continue-on-error: true   # Permitir continuar para publicar reportes

      - name: Verificar Quality Gate de Smoke
        run: |
          python scripts/check_quality_gate.py \
            results/smoke/output.xml \
            ${{ env.SMOKE_PASS_THRESHOLD }}

      - name: Subir artefactos de smoke
        uses: actions/upload-artifact@v4
        if: always()   # Siempre subir, incluso si hay fallos
        with:
          name: smoke-results-${{ github.run_number }}
          path: results/smoke/
          retention-days: 30

  # ─────────────────────────────────────────────────────────────────────────
  # STAGE 3: Regression Tests (solo en main y schedule)
  # ─────────────────────────────────────────────────────────────────────────
  run-regression:
    name: "🔄 Regression Tests"
    needs: run-smoke
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' || github.event_name == 'schedule'
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'

      - name: Instalar dependencias
        run: pip install -r requirements.txt

      - name: Instalar Chrome
        uses: browser-actions/setup-chrome@v1

      - name: Ejecutar Regression Tests
        run: |
          robot \
            --variable BASE_URL:${{ env.BASE_URL }} \
            --variable ENV:ci \
            --variable BROWSER:headlesschrome \
            --include regression \
            --exclude wip \
            --outputdir results/regression/ \
            tests/
        continue-on-error: true

      - name: Rerun de fallos (hasta 1 reintento)
        run: |
          if [ -f results/regression/output.xml ]; then
            robot \
              --rerunfailed results/regression/output.xml \
              --variable BASE_URL:${{ env.BASE_URL }} \
              --outputdir results/regression/rerun/ \
              tests/ || true

            rebot \
              --merge \
              --outputdir results/regression/final/ \
              results/regression/output.xml \
              results/regression/rerun/output.xml 2>/dev/null || \
              cp -r results/regression/ results/regression/final/
          fi

      - name: Subir artefactos de regresión
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: regression-results-${{ github.run_number }}
          path: results/regression/
          retention-days: 60

  # ─────────────────────────────────────────────────────────────────────────
  # STAGE 4: Publicar Reporte Consolidado
  # ─────────────────────────────────────────────────────────────────────────
  publish-report:
    name: "📊 Publicar Reportes"
    needs: [run-smoke, run-regression]
    runs-on: ubuntu-latest
    if: always()   # Publicar siempre, incluso con fallos
    steps:
      - uses: actions/checkout@v4

      - name: Descargar artefactos de smoke
        uses: actions/download-artifact@v4
        with:
          name: smoke-results-${{ github.run_number }}
          path: results/smoke/

      - name: Descargar artefactos de regresión
        uses: actions/download-artifact@v4
        with:
          name: regression-results-${{ github.run_number }}
          path: results/regression/
        continue-on-error: true   # Puede no existir en PRs

      - name: Publicar reportes en GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        if: github.ref == 'refs/heads/main'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: results/
          destination_dir: reports/${{ github.run_number }}

  # ─────────────────────────────────────────────────────────────────────────
  # STAGE 5: Notificación en caso de fallo
  # ─────────────────────────────────────────────────────────────────────────
  notify-on-failure:
    name: "🔔 Notificar Fallos"
    needs: [run-smoke, run-regression]
    runs-on: ubuntu-latest
    if: failure()   # Solo ejecutar si algún stage anterior falló
    steps:
      - name: Notificar por Slack (webhook)
        uses: slackapi/slack-github-action@v1.26.0
        with:
          payload: |
            {
              "text": "❌ *Pipeline Robot Framework falló*",
              "attachments": [{
                "color": "danger",
                "fields": [
                  {"title": "Repositorio", "value": "${{ github.repository }}", "short": true},
                  {"title": "Branch", "value": "${{ github.ref_name }}", "short": true},
                  {"title": "Run #", "value": "${{ github.run_number }}", "short": true},
                  {"title": "Commit", "value": "${{ github.sha }}", "short": true}
                ]
              }]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
```

3. Crea el script auxiliar de quality gate:

```python
# scripts/check_quality_gate.py
"""
Verifica que la tasa de éxito de output.xml supere el umbral mínimo.
Uso: python check_quality_gate.py <output.xml> <threshold_percent>
Retorna exit code 0 si pasa, 1 si falla el quality gate.
"""

import sys
import xml.etree.ElementTree as ET

def check_gate(output_xml: str, threshold: float) -> bool:
    tree = ET.parse(output_xml)
    root = tree.getroot()

    # Buscar estadísticas en el elemento <statistics>
    total = passed = 0
    for stat in root.iter("stat"):
        if stat.get("type") == "total" or stat.text == "All Tests":
            pass_count = int(stat.get("pass", 0))
            fail_count = int(stat.get("fail", 0))
            if pass_count + fail_count > total:
                total = pass_count + fail_count
                passed = pass_count

    if total == 0:
        print("⚠ No se encontraron estadísticas en output.xml")
        return False

    pass_rate = (passed / total) * 100
    print(f"Quality Gate: {pass_rate:.1f}% >= {threshold}% requerido")
    print(f"Tests: {passed}/{total} pasaron")

    if pass_rate >= threshold:
        print("✅ Quality Gate APROBADO")
        return True
    else:
        print(f"❌ Quality Gate FALLIDO: {pass_rate:.1f}% < {threshold}%")
        return False

if __name__ == "__main__":
    if len(sys.argv) < 3:
        print("Uso: python check_quality_gate.py <output.xml> <threshold>")
        sys.exit(1)

    xml_path = sys.argv[1]
    threshold = float(sys.argv[2])

    if not check_gate(xml_path, threshold):
        sys.exit(1)
```

**Salida esperada:** Los archivos `.github/workflows/robot_pipeline.yml` y `scripts/check_quality_gate.py` creados sin errores de sintaxis YAML.

**Verificación:** Valida la sintaxis YAML:

```bash
# Validar YAML (requiere pip install pyyaml)
python -c "import yaml; yaml.safe_load(open('.github/workflows/robot_pipeline.yml'))"
echo "YAML válido ✓"
```

---

#### Paso 3.2 — Reflexión sobre el diseño del pipeline (actividad guiada)

**Instrucciones:**

Revisa el pipeline creado y responde las siguientes preguntas de diseño en el archivo `docs/pipeline_decisions.md`:

```markdown
# Decisiones de Diseño del Pipeline CI/CD

## Quality Gates
- **Smoke threshold:** 95% — ¿Por qué 95% y no 100%?
  > _Respuesta:_ Los smoke tests validan funcionalidad crítica. Un 5% de tolerancia
  > permite tests intermitentes (flaky) sin bloquear el pipeline, pero exige
  > investigación inmediata.

- **Regression threshold:** 85% — ¿Es apropiado para tu proyecto?
  > _Respuesta:_ [Completa aquí con tu análisis]

## Estrategia de Artefactos
- ¿Por qué `if: always()` en los pasos de upload?
  > _Respuesta:_ Para garantizar que los reportes estén disponibles incluso cuando
  > los tests fallan, permitiendo el diagnóstico post-mortem.

## Política de Rerun
- ¿Cuántos reintentos son apropiados? ¿Por qué no más de 2?
  > _Respuesta:_ [Completa aquí]

## Notificaciones
- ¿Qué información mínima debe incluir una notificación de fallo?
  > _Respuesta:_ Repositorio, branch, número de run, enlace al reporte y
  > resumen de tests fallidos.
```

**Verificación:** El archivo `docs/pipeline_decisions.md` existe con al menos 3 de las 5 preguntas respondidas.

---

### BLOQUE 4 — Simulacro RFCP (10 minutos)

**Objetivo:** Autoevaluar el conocimiento del curso mediante 20 preguntas tipo RFCP con revisión guiada.

---

#### Paso 4.1 — Simulacro de 20 preguntas

**Instrucciones:**

Lee cada pregunta, selecciona tu respuesta y anótala. Luego revisa las respuestas correctas en el Paso 4.2.

---

**Pregunta 1** *(CLI — Módulo 10)*
¿Cuál es el comando correcto para ejecutar solo los tests con el tag `smoke` pero excluir los que también tengan el tag `wip`?

- A) `robot --include smoke --exclude wip tests/`
- B) `robot --tag smoke --no-tag wip tests/`
- C) `robot --filter smoke:NOT:wip tests/`
- D) `robot --include smokeNOTwip tests/`

---

**Pregunta 2** *(CLI — Módulo 10)*
¿Qué herramienta de Robot Framework se usa para combinar múltiples archivos `output.xml` en un único reporte?

- A) `robot --merge`
- B) `rebot --merge`
- C) `rfmerge`
- D) `robot --combine`

---

**Pregunta 3** *(Sintaxis RF — Módulo 7)*
¿Cuál es la sintaxis correcta para definir una variable de suite en un archivo `.robot`?

- A) `*** Variables ***` con `${VAR}    valor`
- B) `*** Settings ***` con `Variable    ${VAR}    valor`
- C) `*** Variables ***` con `VAR = valor`
- D) `@variable    ${VAR}    valor`

---

**Pregunta 4** *(SeleniumLibrary — Módulo 7)*
¿Qué keyword de SeleniumLibrary se usa para esperar que un elemento sea visible antes de interactuar con él?

- A) `Wait Until Element Exists`
- B) `Wait Until Element Is Visible`
- C) `Element Should Be Visible`
- D) `Wait For Element`

---

**Pregunta 5** *(Page Object Model — Módulo 7)*
En el patrón Page Object Model con Robot Framework, ¿dónde se almacenan los locators de los elementos web?

- A) Directamente en los archivos `.robot` de tests
- B) En archivos de recursos `.resource` o `.robot` específicos de cada página
- C) En el archivo `__init__.robot` de la suite
- D) En variables de entorno del sistema operativo

---

**Pregunta 6** *(RequestsLibrary — Módulo 8)*
¿Cuál es el keyword correcto para crear una sesión HTTP en RequestsLibrary?

- A) `Open HTTP Session`
- B) `Create HTTP Connection`
- C) `Create Session`
- D) `Initialize Session`

---

**Pregunta 7** *(API Testing — Módulo 8)*
¿Qué keyword se usa para validar que una respuesta HTTP tiene el código de estado 200?

- A) `Response Status Should Be    200`
- B) `Status Code Should Be    ${response}    200`
- C) `Should Be Equal As Integers    ${response.status_code}    200`
- D) `Verify Status    200`

---

**Pregunta 8** *(JSON Schema — Módulo 8)*
¿Qué librería de Robot Framework permite validar un JSON contra un esquema JSON Schema?

- A) `RequestsLibrary`
- B) `JSONLibrary` con `Validate Json By Schema File`
- C) `SeleniumLibrary`
- D) `Collections`

---

**Pregunta 9** *(RPA Framework — Módulo 9)*
¿Cuál es la keyword de RPA Framework para leer el contenido de un archivo Excel?

- A) `Read Excel File`
- B) `Open Workbook` + `Read Sheet Data`
- C) Depende de la librería específica (ej: `RPA.Excel.Files`)
- D) `Get Excel Content`

---

**Pregunta 10** *(Manejo de errores — Módulo 9)*
¿Qué keyword permite ejecutar una keyword y continuar aunque falle, capturando el mensaje de error?

- A) `Try Keyword`
- B) `Run Keyword And Ignore Error`
- C) `Execute And Continue`
- D) `Safe Run Keyword`

---

**Pregunta 11** *(Tags — Módulo 10)*
¿Cuál es la sintaxis CLI para ejecutar tests que tengan el tag `api` O el tag `web`?

- A) `--include api --include web`
- B) `--include apiORweb`
- C) `--include api OR web`
- D) Tanto A como B son correctas

---

**Pregunta 12** *(Estructura de proyecto — Módulo 7)*
¿Cuál es la práctica recomendada para separar las capas de abstracción en un proyecto Robot Framework?

- A) Un único archivo `.robot` con todas las keywords y tests
- B) Separar en: `tests/` (casos), `resources/` (keywords), `variables/` (datos)
- C) Usar únicamente archivos Python para todas las keywords
- D) Mezclar keywords de negocio con keywords técnicas en el mismo archivo

---

**Pregunta 13** *(CLI — Módulo 10)*
¿Qué opción CLI de `robot` permite relanzar solo los tests que fallaron en una ejecución anterior?

- A) `--rerun-failures`
- B) `--retry-failed`
- C) `--rerunfailed`
- D) `--run-failed-only`

---

**Pregunta 14** *(DataDriver — Módulo 8)*
¿Qué librería permite ejecutar el mismo test con múltiples conjuntos de datos desde un archivo CSV o Excel?

- A) `robotframework-datadriver`
- B) `robotframework-datatable`
- C) `RequestsLibrary`
- D) `Collections`

---

**Pregunta 15** *(output.xml — Módulo 10)*
¿Qué información NO se puede extraer directamente del `output.xml`?

- A) Tiempo de ejecución de cada keyword
- B) Capturas de pantalla embebidas
- C) El código fuente de las keywords Python
- D) Mensajes de log de cada paso

---

**Pregunta 16** *(Buenas prácticas — Módulo 7)*
¿Cuál es la ventaja principal de usar `${/}` en lugar de `/` o `\` en rutas de archivos en Robot Framework?

- A) Es más rápido en tiempo de ejecución
- B) Garantiza compatibilidad entre Windows, Linux y macOS
- C) Permite usar rutas relativas
- D) Es requerido por SeleniumLibrary

---

**Pregunta 17** *(CI/CD — Módulo 10)*
¿Por qué se recomienda usar `continue-on-error: true` (o equivalente) en el paso de ejecución de Robot Framework en un pipeline CI/CD?

- A) Para que el pipeline nunca falle
- B) Para permitir que los pasos de publicación de reportes se ejecuten incluso cuando hay tests fallidos
- C) Porque Robot Framework siempre retorna código de salida 0
- D) Para evitar que el pipeline se detenga por errores de sintaxis

---

**Pregunta 18** *(Logging — Módulo 9)*
¿Qué keyword de Robot Framework permite escribir mensajes en el log con nivel configurable (INFO, WARN, DEBUG)?

- A) `Print Message`
- B) `Log`
- C) `Write Log`
- D) `Console Log`

---

**Pregunta 19** *(Pabot — Módulo 10)*
¿Qué herramienta permite ejecutar suites de Robot Framework en paralelo?

- A) `robot --parallel`
- B) `pabot`
- C) `robot --threads`
- D) `rfparallel`

---

**Pregunta 20** *(Arquitectura — Módulo 7-9)*
¿Cuál de las siguientes afirmaciones describe correctamente el patrón de tres capas en Robot Framework?

- A) Capa de datos → Capa de keywords → Capa de tests (de más específico a más abstracto)
- B) Capa de tests (alto nivel) → Capa de keywords de negocio → Capa de keywords técnicas (de más abstracto a más específico)
- C) Capa web → Capa API → Capa RPA
- D) Capa de configuración → Capa de ejecución → Capa de reporte

---

#### Paso 4.2 — Revisión guiada de respuestas

**Instrucciones:** Compara tus respuestas con las correctas. Para cada error, lee la explicación y anota el módulo correspondiente para repaso.

| # | Respuesta Correcta | Explicación | Módulo |
|---|-------------------|-------------|--------|
| 1 | **A** | `--include smoke --exclude wip` es la sintaxis estándar. La opción D (`smokeNOTwip`) es válida como expresión de tag único pero menos legible. | 10 |
| 2 | **B** | `rebot` es la herramienta de post-procesamiento. `robot` no tiene opción `--merge`. | 10 |
| 3 | **A** | La sección `*** Variables ***` con `${VAR}    valor` es la sintaxis correcta. | 7 |
| 4 | **B** | `Wait Until Element Is Visible` espera activamente. `Element Should Be Visible` falla inmediatamente si no está visible. | 7 |
| 5 | **B** | En POM, los locators y keywords de página se encapsulan en archivos `.resource` específicos por página. | 7 |
| 6 | **C** | `Create Session` es el keyword estándar de RequestsLibrary para inicializar una sesión HTTP. | 8 |
| 7 | **C** | `Should Be Equal As Integers    ${response.status_code}    200` es la forma correcta. RequestsLibrary retorna un objeto `response` con atributo `status_code`. | 8 |
| 8 | **B** | `JSONLibrary` incluye `Validate Json By Schema File` para validación de esquemas. | 8 |
| 9 | **C** | `RPA.Excel.Files` es la librería específica. El keyword exacto varía; lo importante es usar la librería correcta del framework RPA. | 9 |
| 10 | **B** | `Run Keyword And Ignore Error` retorna el status y el mensaje de error sin fallar el test. | 9 |
| 11 | **D** | Ambas formas son válidas: múltiples `--include` actúan como OR, y `apiORweb` es la expresión booleana equivalente. | 10 |
| 12 | **B** | La separación en `tests/`, `resources/` y `variables/` es la arquitectura de tres capas recomendada. | 7 |
| 13 | **C** | `--rerunfailed` es la opción correcta. Las otras no existen en Robot Framework. | 10 |
| 14 | **A** | `robotframework-datadriver` es la librería oficial para pruebas data-driven con CSV/Excel. | 8 |
| 15 | **C** | El `output.xml` no contiene el código fuente Python de las keywords, solo los resultados de ejecución. | 10 |
| 16 | **B** | `${/}` es la variable built-in de Robot Framework que resuelve al separador de directorios del SO en tiempo de ejecución. | 7 |
| 17 | **B** | Con `continue-on-error: true`, el pipeline puede publicar reportes y enviar notificaciones aunque los tests fallen, lo que es esencial para visibilidad. | 10 |
| 18 | **B** | `Log` es el keyword built-in de Robot Framework. Acepta niveles: `INFO`, `WARN`, `DEBUG`, `ERROR`, `TRACE`. | 9 |
| 19 | **B** | `pabot` (Parallel Robot Framework Executor) es la herramienta estándar para ejecución paralela. | 10 |
| 20 | **B** | El patrón correcto va de lo abstracto (tests de negocio) a lo específico (keywords técnicas de librería). | 7-9 |

**Calcula tu puntuación:**

```
Puntuación: ___ / 20 correctas

Interpretación:
  18-20 ✅  Excelente — Listo para el examen RFCP
  15-17 🟡  Bueno — Repasa los módulos con errores
  12-14 🟠  Regular — Refuerza conceptos de CLI, API y arquitectura
   < 12 🔴  Necesita repaso integral — Revisa todos los módulos
```

---

## Validación y Pruebas

Ejecuta los siguientes comandos para validar que todos los artefactos del laboratorio están correctamente creados:

```bash
#!/usr/bin/env bash
# validate_lab10.sh — Validación completa del laboratorio 10-00-01

echo "=== VALIDACIÓN LAB 10-00-01 ==="
PASS=0
FAIL=0

check() {
  local desc="$1"
  local cmd="$2"
  if eval "$cmd" > /dev/null 2>&1; then
    echo "  ✅ $desc"
    PASS=$((PASS + 1))
  else
    echo "  ❌ $desc"
    FAIL=$((FAIL + 1))
  fi
}

# Bloque 1: Scripts de ejecución
check "run_all.sh existe y es ejecutable" "[ -x run_all.sh ]"
check "run_all.bat existe" "[ -f run_all.bat ]"
check "Directorio results/ existe" "[ -d results/ ]"
check "output.xml generado en algún run" "find results/ -name 'output.xml' | grep -q ."

# Bloque 2: Scripts de análisis
check "analyze_output.py existe" "[ -f analyze_output.py ]"
check "check_quality_gate.py existe" "[ -f scripts/check_quality_gate.py ]"
check "analyze_output.py ejecuta sin errores" \
  "python analyze_output.py results/analysis/output.xml 2>/dev/null || \
   python analyze_output.py \$(find results/ -name 'output.xml' | head -1)"

# Bloque 3: Pipeline CI/CD
check "Pipeline YAML existe" "[ -f .github/workflows/robot_pipeline.yml ]"
check "Pipeline YAML es válido" \
  "python -c \"import yaml; yaml.safe_load(open('.github/workflows/robot_pipeline.yml'))\""
check "pipeline_decisions.md existe" "[ -f docs/pipeline_decisions.md ]"

# Bloque 4: Simulacro
check "Simulacro completado (archivo de notas)" \
  "[ -f docs/simulacro_rfcp_respuestas.md ] || \
   [ -f simulacro_respuestas.txt ]"

echo ""
echo "=== RESULTADO: $PASS/10 verificaciones pasaron ==="
[ $FAIL -eq 0 ] && echo "🎉 ¡Laboratorio completado exitosamente!" || \
  echo "⚠ Revisa los items marcados con ❌"
```

```bash
chmod +x validate_lab10.sh
./validate_lab10.sh
```

**Criterio de aprobación:** Al menos 8 de 10 verificaciones deben pasar (✅).

---

## Solución de Problemas

### Problema 1: `rebot --merge` falla con "No output files given"

**Síntoma:**
```
[ ERROR ] No output files given.
```
O bien:
```
[ ERROR ] Reading XML source 'results/rerun/output.xml' failed:
          [Errno 2] No such file or directory
```

**Causa:** El directorio de rerun no se creó porque no hubo tests fallidos en la ejecución inicial (todos pasaron), por lo que el archivo `output.xml` del rerun no existe.

**Solución:**
```bash
# Verificar si existen ambos archivos antes de hacer merge
if [ -f "results/initial/output.xml" ] && [ -f "results/rerun/output.xml" ]; then
  rebot --merge \
    --outputdir results/final/ \
    results/initial/output.xml \
    results/rerun/output.xml
else
  echo "Sin fallos para merge. Copiando resultado inicial como final."
  cp -r results/initial/ results/final/
fi
```

En el script `run_all.sh`, este caso ya está manejado con la verificación del contador de fallos. Si el problema persiste, asegúrate de que el comando `robot --rerunfailed` se ejecutó con `|| true` para no abortar el script ante código de salida no cero.

---

### Problema 2: El script `analyze_output.py` retorna ceros en todas las métricas

**Síntoma:**
```
📊 RESUMEN GLOBAL
   Total tests : 0
   Pasaron     : 0 (0.0%)
   Fallaron    : 0 (0.0%)
```

**Causa:** La estructura del `output.xml` varía ligeramente entre versiones de Robot Framework. En RF 7.x, los atributos de tiempo (`starttime`, `endtime`) pueden estar en el elemento `<status>` o como atributos del elemento `<test>`. Además, el elemento `<statistics>` puede tener una estructura diferente a la esperada por el parser.

**Solución:**
```python
# Versión robusta del parser — reemplazar la función parse_elapsed_time y el bucle de tests

def get_elapsed_ms(status_elem, test_elem):
    """Intenta obtener elapsed de múltiples ubicaciones según versión de RF."""
    # RF 7.x: atributo 'elapsed' directo en status
    elapsed = status_elem.get("elapsed")
    if elapsed:
        return float(elapsed) / 1000  # milisegundos a segundos

    # RF 6.x: starttime/endtime en status
    start = status_elem.get("starttime") or test_elem.get("starttime", "")
    end = status_elem.get("endtime") or test_elem.get("endtime", "")
    return parse_elapsed_time(start, end)

# Diagnóstico rápido para identificar la estructura real:
# python -c "
# import xml.etree.ElementTree as ET
# root = ET.parse('results/analysis/output.xml').getroot()
# for test in list(root.iter('test'))[:2]:
#     print('TEST:', test.attrib)
#     for child in test:
#         print('  CHILD:', child.tag, child.attrib)
# "
```

Ejecuta el diagnóstico primero para ver la estructura real del XML y ajusta el parser según corresponda a tu versión de Robot Framework.

---

## Limpieza

Una vez completado y validado el laboratorio, ejecuta los siguientes comandos para organizar los artefactos y liberar espacio:

```bash
# 1. Conservar solo los artefactos finales relevantes
mkdir -p lab10_final_artifacts/

# Copiar artefactos clave
cp run_all.sh run_all.bat lab10_final_artifacts/ 2>/dev/null || true
cp analyze_output.py lab10_final_artifacts/
cp scripts/check_quality_gate.py lab10_final_artifacts/
cp .github/workflows/robot_pipeline.yml lab10_final_artifacts/
cp docs/pipeline_decisions.md lab10_final_artifacts/ 2>/dev/null || true

# 2. Limpiar resultados intermedios (mantener solo 'final')
find results/ -name "*.xml" -not -path "*/final/*" -delete 2>/dev/null || true
find results/ -name "*.html" -not -path "*/final/*" -delete 2>/dev/null || true

# 3. Comprimir artefactos para entrega (opcional)
zip -r lab10_entrega_$(date +%Y%m%d).zip lab10_final_artifacts/ 2>/dev/null || \
  tar -czf lab10_entrega_$(date +%Y%m%d).tar.gz lab10_final_artifacts/

echo "✅ Limpieza completada. Artefactos en: lab10_final_artifacts/"
```

**Windows (PowerShell):**
```powershell
# Crear directorio de artefactos finales
New-Item -ItemType Directory -Force -Path lab10_final_artifacts

# Copiar artefactos clave
Copy-Item run_all.bat, analyze_output.py -Destination lab10_final_artifacts\ -ErrorAction SilentlyContinue
Copy-Item scripts\check_quality_gate.py -Destination lab10_final_artifacts\
Copy-Item .github\workflows\robot_pipeline.yml -Destination lab10_final_artifacts\

# Comprimir para entrega
Compress-Archive -Path lab10_final_artifacts\* `
  -DestinationPath "lab10_entrega_$(Get-Date -Format 'yyyyMMdd').zip" `
  -Force

Write-Host "✅ Limpieza completada. Artefactos en: lab10_final_artifacts\"
```

> **Importante:** No elimines los archivos `output.xml` de `results/final/` si planeas usarlos como referencia en el proyecto integrador del curso.

---

## Resumen

En este laboratorio integrador completaste los cuatro pilares del cierre del curso:

| Bloque | Habilidad adquirida | Artefacto generado |
|--------|--------------------|--------------------|
| **1 — CLI Avanzada** | Ejecución granular con `--include`, `--exclude`, `--variable`, `--rerunfailed` y `rebot --merge` | `run_all.sh` / `run_all.bat` |
| **2 — Análisis de Reportes** | Extracción de KPIs de `output.xml`: tests lentos, patrones de fallo, tasa de éxito por tag | `analyze_output.py` |
| **3 — Pipeline CI/CD** | Diseño de pipeline GitHub Actions con 5 stages, quality gates y notificaciones | `.github/workflows/robot_pipeline.yml` |
| **4 — Simulacro RFCP** | Autoevaluación de 20 preguntas cubriendo todos los módulos del curso | Puntuación y plan de repaso |

Los conceptos clave que debes dominar para el examen RFCP son:
- La diferencia entre `--include A --include B` (OR) y `--include AANDB` (AND)
- El ciclo completo: `robot` → `output.xml` → `robot --rerunfailed` → `rebot --merge`
- La arquitectura de tres capas y el patrón Page Object Model
- El uso de `${/}` para portabilidad multiplataforma

### Recursos adicionales

- [Documentación oficial Robot Framework — opciones CLI](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#using-command-line-options)
- [Guía de usuario — rebot y post-procesamiento](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#post-processing-outputs)
- [Repositorio oficial Robot Framework](https://github.com/robotframework/robotframework)
- [Guía de estudio RFCP (Robot Framework Foundation)](https://robotframework.org/foundation/)
- [GitHub Actions — documentación oficial](https://docs.github.com/en/actions)
- [GitLab CI/CD — documentación oficial](https://docs.gitlab.com/ee/ci/)

---
*Lab 10-00-01 — Práctica 4: Integrador y Plan de Adopción Regional | Curso Robot Framework Professional*
