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

## Convenciones

- Todos los reportes son HTML standalone (CSS inline en el archivo)
- Sin tildes en el contenido del HTML (evitar problemas de encoding en impresion)
- Score global /100, scores por seccion /100
- Badges de severidad: critical (rojo), high (naranja), medium (amarillo), low (verde), ok (verde), none (gris)
- Alertas: alert-critical, alert-warning, alert-success, alert-info
- Prioridades: critica (semana 1-2), alta (semana 3-4), media (mes 2), baja (mes 3+)

## Archivos

- `index.html` - Landing con lista de todos los reportes
- `vercel.json` - Config Vercel (cleanUrls habilitado)
- `auditoria-seo-*.html` - Reportes individuales
