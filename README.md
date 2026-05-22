# GEO Visibility Tracker

> El "rank tracker" de la era de respuestas generativas. En vez de medir posiciones en Google, mide cuántas veces un dominio aparece **citado como fuente** en las respuestas de Claude con web search.

Mientras el SEO clásico optimiza para los 10 enlaces azules, **GEO (Generative Engine Optimization)** optimiza para ser citado dentro de la respuesta que genera el LLM. Este script te permite medirlo de forma sistemática para tu nicho.

## ¿Qué hace?

Dado un conjunto de queries representativas de un nicho:

1. Las envía a Claude con la herramienta de web search activada.
2. Captura todas las URLs citadas en cada respuesta.
3. Agrega los datos por dominio y calcula el **Share of Voice (SoV)**: en qué % de queries aparece cada dominio.
4. Compara contra un dominio objetivo (el tuyo) y lista las queries donde ganas o pierdes.
5. Exporta el resultado a JSON para análisis posterior.

## Instalación

```bash
git clone https://github.com/<tu-usuario>/geo-visibility-tracker.git
cd geo-visibility-tracker
pip install anthropic
export ANTHROPIC_API_KEY=sk-ant-...
```

## Uso

Edita la lista `queries` y el `target_domain` al final de `geo_tracker.py`:

```python
queries = [
    "best way to integrate Odoo with Astro framework",
    "Odoo REST API authentication with API token tutorial",
    # ...
]

data = analyze(queries, target_domain="tudominio.com")
```

Luego corre:

```bash
python geo_tracker.py
```

## Salida

Consola:

```
================================================================
GEO VISIBILITY REPORT  ·  modelo: claude-sonnet-4-6
Queries analizadas: 7
================================================================

🎯 Target: patagrowth.com
   Apariciones: 2/7  (28.6%)
   Queries ganadas:
     ✓ Odoo REST API authentication with API token tutorial
     ✓ secure environment variables for Odoo API in Astro

Top 15 dominios por share of voice:

  #  Dominio                                Apar.    SoV  Citas
----------------------------------------------------------------
  1  odoo.com                                   6  85.7%     11
  2  github.com                                 4  57.1%      7
  3  stackoverflow.com                          3  42.9%      4
  4  patagrowth.com                             2  28.6%      3
  ...
```

Y un archivo `geo_report.json` con todos los datos crudos: respuesta completa, citas, títulos y URLs por query.

## Configuración

En la cabecera de `geo_tracker.py`:

| Variable | Default | Descripción |
|---|---|---|
| `MODEL` | `claude-sonnet-4-6` | Modelo a usar. Sonnet es buen balance precio/calidad. |
| `MAX_TOKENS` | `1500` | Tokens máximos por respuesta. |
| `MAX_SEARCHES` | `5` | Máximo de búsquedas web por query. |
| `SLEEP_SECONDS` | `1.0` | Pausa entre queries para evitar rate limits. |

## Costos

Cada query usa:
- ~1 a 5 búsquedas web ($10 por 1.000 búsquedas)
- Tokens de input + output del modelo

Para un análisis típico de 20 queries con Sonnet: aproximadamente **USD 0.30 – 0.80**.

## Cómo funciona

1. Llama al endpoint `/v1/messages` con `tools=[{"type": "web_search_20250305", "name": "web_search"}]`.
2. Claude decide autónomamente cuándo buscar y qué buscar para responder.
3. La respuesta incluye bloques `text` con un campo `citations` que contiene las URLs originales.
4. Parseamos esas citas, normalizamos los dominios (quitando `www.`) y agregamos.

## Ideas para extender

- **Tracking temporal**: corre el script semanalmente, guarda los JSON con timestamp y grafica la evolución del SoV.
- **Cross-engine**: añade Perplexity y OpenAI con web search para comparar qué dominios prefiere cada motor.
- **Análisis de contenido**: para cada URL citada, haz fetch y analiza patrones (longitud, schema, formato listas vs prosa) → factores GEO.
- **Clustering de queries**: usa embeddings para agrupar queries similares y ver qué clusters domina cada competidor.
- **Detección de citas a sí mismo**: detecta cuándo el LLM cita aggregadores vs fuentes primarias.
- **Modo batch**: usa la Batch API de Anthropic (50% descuento) para volúmenes grandes.

## Limitaciones

- La web search de Anthropic usa su propio índice; los resultados pueden diferir de Google.
- Los LLMs no son deterministas: dos corridas pueden devolver citas distintas. Para análisis serio, promedia varias corridas.
- El script no maneja queries en bulk con paralelización: una a una para evitar rate limits.

## Licencia

MIT

---

Creado por [Agencia SEO Chile · Ridgeseo](https://ridgeseo.com)
