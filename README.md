# ◆ StaticSentinel

**Motor de análisis estático de malware.** Le das un binario sospechoso y, sin
ejecutarlo, te devuelve un dossier forense completo: identidad, anatomía del
ejecutable, comportamiento inferido, indicadores de compromiso, una puntuación
de riesgo explicada y una **regla YARA autogenerada** para cazar la amenaza.

> Proyecto de ciberseguridad defensiva. Analiza muestras de forma **estática**
> (nunca las ejecuta), pensado para laboratorios y práctica en entornos
> controlados.

---

## ¿Qué hace?

Cuando analizas un fichero, StaticSentinel lo disecciona por capas:

| Fase | Qué extrae |
|------|-----------|
| **Identidad** | Hashes (MD5/SHA-1/SHA-256/SHA-512), tamaño, y el **tipo real** por *magic bytes* (detecta extensiones que mienten). |
| **Entropía** | Entropía de Shannon global y por sección. Detecta empaquetado/cifrado (packers como UPX) cuando supera los 7.2 bits/byte. |
| **Anatomía PE** | Arquitectura, fecha de compilación, subsistema, imphash, secciones con permisos, tabla de *imports* y *exports*, y detección de *overlay*. |
| **Capacidades** | Mapea las APIs importadas a técnicas **MITRE ATT&CK**: inyección de código, persistencia, anti-análisis, C2, ransomware, keylogging... |
| **IOCs** | Extrae cadenas ASCII y UTF-16LE y saca URLs, IPs, dominios, emails, rutas, claves de registro y direcciones de criptomoneda. |
| **Scoring** | Combina todas las señales en una puntuación 0-100 **con el motivo de cada punto** (auditable). |
| **YARA** | Genera automáticamente una regla con las cadenas más distintivas + imphash, lista para desplegar. |

Salida en tres formatos: resumen en **terminal**, informe **HTML** (autocontenido,
ideal para demos) y **JSON** (para integrar con otras herramientas).

---

## Instalación

Requiere **Python 3.9+**. La única dependencia es opcional:

```bash
git clone https://github.com/papaya1477/staticsentinel.git
cd malware-analyzer
pip install -r requirements.txt     # instala pefile (recomendado)
```

> Sin `pefile`, la herramienta sigue funcionando con un **parser PE nativo de
> respaldo** (solo librería estándar). Con `pefile` obtienes imphash y exports.

---

## Uso

```bash
# Análisis básico (genera <fichero>_report.html junto a la muestra)
python -m analyzer.cli muestra.exe

# Control total de la salida
python -m analyzer.cli muestra.exe -o informe.html --json datos.json --yara regla.yar

# Solo resumen en terminal, sin HTML
python -m analyzer.cli muestra.exe --no-html
```

Opciones principales:

```
-o, --output     Ruta del informe HTML
    --json       Guardar resultados crudos en JSON
    --yara       Guardar la regla YARA en un .yar
    --no-html    No generar HTML
    --no-color   Terminal sin color
```

---

## Arquitectura

```
analyzer/
├── cli.py                  # Orquesta el pipeline y escribe los informes
└── core/
    ├── identity.py         # Hashes y detección de tipo por magic bytes
    ├── entropy.py          # Entropía de Shannon y heurística de empaquetado
    ├── pe.py               # Parseo PE (motor dual: pefile + nativo)
    ├── strings_ioc.py      # Extracción de strings e IOCs
    ├── capabilities.py     # Inferencia de comportamiento (MITRE ATT&CK)
    ├── scoring.py          # Motor de puntuación de riesgo
    ├── yara_gen.py         # Generación automática de reglas YARA
    └── report.py           # Informes HTML y JSON
tests/
└── test_core.py            # Tests unitarios
```

Diseño modular: cada módulo es independiente y testeable por separado. El `cli`
solo coordina.

---

## ¿Dónde conseguir muestras para practicar?

En un entorno aislado (VM sin red, snapshot previo):

- **MalwareBazaar** (abuse.ch) — repositorio público de muestras reales.
- **theZoo** — colección educativa en GitHub.
- **VirusShare** — requiere registro.

También sirve cualquier binario legítimo de Windows (`.exe`/`.dll` de
`C:\Windows\System32`) para ver cómo se comporta con software benigno.

---

## Roadmap (siguientes fases)

- [ ] Consulta opcional a la API de VirusTotal por hash (con tu API key).
- [ ] Detección de *packers* conocidos por firmas de sección.
- [ ] Soporte ELF y Mach-O a nivel de secciones e imports.
- [ ] Extracción de recursos embebidos (iconos, manifiestos, binarios anidados).
- [ ] Análisis de macros en documentos Office (OLE/OOXML).
- [ ] Modo *batch* para analizar carpetas enteras y generar un índice.

---

## Nota ética

StaticSentinel **no ejecuta** las muestras: todo el análisis es estático, por lo
que es seguro frente a la mayoría del malware (aunque conviene trabajar siempre
en una VM aislada). Es una herramienta **defensiva**, orientada a comprender,
clasificar y detectar amenazas. El análisis estático complementa —no sustituye—
al análisis dinámico en sandbox.

---

*Desarrollado como proyecto personal de ciberseguridad.*
