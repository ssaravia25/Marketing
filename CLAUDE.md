# Marketing - Auditorias SEO

Proyecto Vercel para publicar reportes de auditoria SEO y marketing digital.
Deploy automatico en: https://marketing-mu-fawn.vercel.app

## Stack

- HTML estatico (sin framework)
- Vercel (deploy automatico desde GitHub main)
- Repo: github.com/ssaravia25/Marketing

## Proceso de Auditoria SEO

Cuando el usuario pida analizar un sitio web, seguir este flujo:

### 1. Scraping con Firecrawl

Ejecutar en paralelo:

- **Scrape homepage** con formato JSON: extraer meta title, meta description, H1-H3, canonical, og tags, schema markup, imagenes sin alt, total imagenes, links internos/externos, contacto, redes sociales.
- **Map del sitio** (`firecrawl_map`): descubrir todas las URLs indexadas.
- **Scrape paginas clave** (catalogo, contacto, productos, blog): repetir extraccion JSON en cada una.

### 2. Analisis con seo-specialist agent

Lanzar el agente `seo-specialist` pasandole TODA la data extraida del scraping. Pedirle:

- Auditoria tecnica (SSL, robots, canonical, schema, sitemap, velocidad)
- SEO on-page (titles, descriptions, headings, imagenes, og tags)
- Contenido y keywords (catalogo, blog, keywords objetivo)
- Off-page y local SEO (redes sociales, Google Business, backlinks, NAP)
- E-commerce SEO si aplica (WooCommerce, productos, categorias)
- Plan de accion priorizado en 4 fases con timeline

### 3. Generar reporte HTML

- Usar el mismo template CSS de los reportes existentes (ver `auditoria-seo-dicoinchile.html` como referencia de estilos).
- Estructura del reporte:
  - Portada con score global, dominio, fecha, CMS, industria
  - Resumen ejecutivo con scores por area
  - Seccion 1: SEO Tecnico (tabla de elementos, alertas, estructura URLs)
  - Seccion 2: SEO On-Page (meta tags, headings, imagenes, og tags)
  - Seccion 3: Contenido y Keywords (catalogo, blog, keywords recomendadas)
  - Seccion 4: Off-Page y Local SEO (redes, GBP, backlinks, competidores)
  - Seccion 5: E-Commerce SEO (si aplica)
  - Seccion 6: Plan de Accion (4 niveles de prioridad + schema recomendado + KPIs + timeline)
- Nombre de archivo: `auditoria-seo-{dominio}.html`

### 4. Publicar

- Agregar el nuevo reporte al `index.html` (el mas reciente arriba)
- Commit y push a main (Vercel deploya automatico)
- URL final: `https://marketing-mu-fawn.vercel.app/auditoria-seo-{dominio}`

## Proceso de Auditoria GEO + SEO (comercial)

Version comercial para vender el servicio: GEO al frente ("que dice la IA de tu marca") y SEO como base tecnica. Plantilla de referencia: `auditoria-seo-geo-pinpilinpausha.html`.

### 1. Recoleccion tecnica (en paralelo)

- `firecrawl_scrape` de la homepage (markdown) y `firecrawl_map` del sitio.
- `curl` + Python para HTML crudo: title, meta, H1-H3, canonical, OG, JSON-LD, imagenes/alt, links "#", plugins y version del CMS.
- `curl` de `robots.txt` (revisar reglas para GPTBot, ClaudeBot, PerplexityBot, Google-Extended y crawl-delay), `/llms.txt` y `sitemap.xml`.
- Peso de imagenes (HEAD de cada imagen) y TTFB. Si PageSpeed API no tiene cuota, no estimar.
- Cartas, catalogos o PDF: revisar si tienen texto (`pdftotext`, `pdffonts`) o son solo imagen.

### 2. Rubro, competidores y consultas

- `firecrawl_search` para identificar competidores y fuentes (Tripadvisor, prensa, guias).
- Definir 8 consultas de comprador: 7 de descubrimiento (generica del rubro, por plato o producto, por ocasion, por barrio) y 1 de marca (horario, precio, reserva, datos clave).

### 3. Medicion en IA con Chrome (`claude-in-chrome`)

- ChatGPT: `https://chatgpt.com/?q=...` (funciona sin sesion).
- Google Modo IA: `https://www.google.com/search?udm=50&hl=es&gl=cl&q=...`
- Gemini: `gemini.google.com/app`, escribir la consulta en el input; esperar 7 s antes de hacer clic.
- Perplexity pide login: no iniciar sesion por el usuario. Reemplazar por Gemini o preguntar.
- Usar `browser_batch` (navigate, wait 20 s, `get_page_text`). Registrar por respuesta: posicion de la marca, primera opcion, competidores, fuentes citadas, dato textual y fecha.
- Si aparece captcha, limite o login: detenerse y avisar.

### 4. Analisis

- Agentes `geo-specialist` y `seo-specialist` (`.claude/agents/`) con toda la evidencia.
- Score GEO /100 = Visibilidad IA 40 (40 x [0,5 x presencia + 0,5 x primera opcion]) + Acceso bots/llms.txt 15 + Datos estructurados 15 + Contenido citable 15 + Autoridad y consistencia 15.
- Score SEO /100 = promedio de Tecnico, On-Page, Contenido y Local/Reputacion.
- Antes de publicar: fact-check de cada cifra del reporte contra la evidencia.

### 5. Reporte HTML

- Nombre: `auditoria-seo-geo-{dominio}.html`
- Estructura:
  - Portada: titular con el hallazgo principal, 2 scores (GEO y SEO) y CTA
  - Resumen ejecutivo: conclusion, 3 hallazgos con su consecuencia comercial, scores por area, banner CTA
  - Seccion 1: Que dice la IA de tu marca (matriz consultas x motores, citas, share of voice, consulta de marca, fuentes) y banner CTA
  - Seccion 2: Por que la IA no te elige primero (5 pilares con barras y tarjetas HOY vs RECOMENDADO)
  - Seccion 3: Base SEO condensada (tecnico, on-page, contenido, local)
  - Seccion 4: Quick wins a 30 dias
  - Seccion 5: Plan de 90 dias (4 prioridades, schema, llms.txt, KPIs actual vs meta, timeline)
  - Seccion 6: Como lo hacemos en SFinance (3 pasos) y CTA final
  - Footer: marca SFinance y metodologia

### Reglas comerciales

- CTA siempre a `https://sfinance.world/contacto` (URL exacta, sin UTM), en 4 puntos: portada, despues del resumen, despues de la seccion IA y bloque final. La URL va visible en texto para que salga en PDF.
- Sin planes ni precios del servicio.
- No inventar cifras: nada de volumenes, trafico ni ROI. KPIs como "hoy medido vs meta".
- Sin testimonios ni casos de otros clientes (los reportes son confidenciales).
- Sin RUT, correos personales ni nombres de autores de resenas.

## Convenciones

- Todos los reportes son HTML standalone (CSS inline en el archivo)
- Sin tildes en el contenido del HTML (evitar problemas de encoding en impresion). La "n" con tilde se escribe como entidad `&ntilde;` (ASCII-safe)
- Score global /100, scores por seccion /100
- Badges de severidad: critical (rojo), high (naranja), medium (amarillo), low (verde), ok (verde), none (gris)
- Alertas: alert-critical, alert-warning, alert-success, alert-info
- Prioridades: critica (semana 1-2), alta (semana 3-4), media (mes 2), baja (mes 3+)

## Archivos

- `index.html` - Landing con lista de todos los reportes
- `vercel.json` - Config Vercel (cleanUrls habilitado)
- `auditoria-seo-*.html` - Reportes individuales
- `auditoria-seo-geo-*.html` - Reportes GEO + SEO comerciales
- `.claude/agents/` - Agentes `seo-specialist` y `geo-specialist`
