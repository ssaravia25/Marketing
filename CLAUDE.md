# Marketing - Analisis GEO

Proyecto Vercel donde se publican los reportes comerciales de **Analisis GEO** (visibilidad en IA + SEO) para prospectos de SFinance.
Deploy automatico en: https://marketing-mu-fawn.vercel.app

## Stack

- HTML estatico (sin framework)
- Vercel, con deploy automatico desde GitHub main
- Repo: github.com/ssaravia25/Marketing

## Proceso unico: Analisis GEO

Cuando el usuario pida analizar, auditar o revisar un sitio web, sigue **este** flujo. Reemplaza a la antigua "Auditoria SEO". Los reportes `auditoria-seo-{dominio}.html` quedan como historicos: no uses su flujo ni su estructura.

- **Objetivo:** un reporte comercial que muestre al prospecto que dice la IA de su marca, por que no lo elige primero y que hacer, y que lo lleve a escribir a SFinance.
- **Entrada:** el dominio del prospecto.
- **Salida:** `auditoria-seo-geo-{marca}.html`, publicado y agregado a `index.html`.
- **Plantilla:** `auditoria-seo-geo-pinpilinpausha.html`. Se copia y se reemplaza el contenido; no se redisena.

### Antes de empezar

- Chrome con la extension Claude in Chrome conectada y una sesion de Google iniciada, para Gemini y Google Modo IA. ChatGPT funciona sin sesion. Perplexity pide login y **no se usa**: nunca iniciar sesion por el usuario.
- Carpeta de trabajo en el scratchpad: `{scratchpad}/{marca}/`, para HTML crudo, robots, PDFs y `evidencia.md`.
- Regla de oro: **cada cifra del reporte sale de la evidencia**. Lo que no se pudo medir se dice, no se estima.

### Paso 1. Recoleccion tecnica (todo en paralelo)

| Que | Como | Que registrar |
|---|---|---|
| Contenido de la home | `firecrawl_scrape` en markdown, con `onlyMainContent: false` | Rubro, propuesta, platos o productos, ocasiones, direccion, horario, contacto, canal de reserva o compra, redes |
| Inventario de URLs | `firecrawl_map` con `limit: 500` | Cantidad de URLs y PDFs publicados |
| HTML crudo | `curl -sL` de la home, parseado con `python3 -I` | Title, meta description, canonical, `lang`, H1-H3 (cantidad y duplicados), OG, bloques JSON-LD, `img` totales y sin alt, alts con nombre de archivo, links `#`, links externos y redes, plugins/tema/version del CMS, GTM/GA |
| Rastreo | `curl` de `/robots.txt`, `/llms.txt`, `/llms-full.txt` y `/sitemap.xml` | Reglas para GPTBot, OAI-SearchBot, ClaudeBot, PerplexityBot y Google-Extended; `crawl-delay`; si llms.txt da 404; URLs del sitemap |
| Redirecciones y servidor | `curl -sI` de http, sin www y con www | Redirecciones a https://www; header `x-powered-by` (version de PHP); TTFB en 3 mediciones |
| Peso de imagenes | HEAD en paralelo de cada imagen de la home | Cantidad, MB totales (2 decimales), cantidad en WebP, la mas pesada |
| Carta, catalogo o PDFs | `pdftotext`, `pdffonts` y `pdfinfo` | Si tiene texto o es solo imagen; paginas y peso; versiones antiguas aun publicadas |

PageSpeed API suele estar sin cuota. Si falla, no se estima: se reporta peso de imagenes y TTFB.

### Paso 2. Contexto de negocio y competidores

Usa `firecrawl_search` con `sources: ["web"]`:

1. `"{marca} {direccion o rubro} opiniones"`: nota y cantidad de resenas en Google y Tripadvisor, prensa y directorios.
2. `"mejores {rubro} en {ciudad}"`: competidores directos, guias y rankings.

Anota los 5-8 competidores probables y las fuentes que se repiten (Tripadvisor, prensa, guias tematicas).

### Paso 3. Disenar las 8 preguntas

Son preguntas que haria un cliente real. Todas incluyen la ciudad.

| # | Tipo | Ejemplo (restaurante) |
|---|---|---|
| Q1 | Generica del rubro | Cuales son los mejores restaurantes de comida espanola en Santiago de Chile? |
| Q2 | Producto o especialidad estrella | Donde comer una buena paella en Santiago de Chile? |
| Q3 | Identidad o diferenciador | Recomiendame un restaurante de comida vasca en Santiago de Chile |
| Q4 | Ocasion 1, con barrio | Que restaurante me recomiendas en el barrio El Golf para un almuerzo de negocios? |
| Q5 | Producto secundario o de temporada | Donde comer churros con chocolate en Santiago de Chile? |
| Q6 | Ocasion 2, con atributo (terraza, familia, etc.) | Recomiendame un restaurante en Las Condes para un almuerzo familiar el domingo, con terraza |
| Q7 | Producto o categoria cercana | Donde puedo comer jamon iberico y tortilla espanola en Santiago de Chile? |
| Q8 | **Marca** | Que me puedes contar de {marca}? Horario, precios, como reservar/comprar y {atributo clave} |

Q1 a Q7 se usan para medir visibilidad (7 x 3 = 21 respuestas). Q8 sirve para comprobar si la IA da datos correctos.

### Paso 4. Medicion en IA (3 motores x 8 preguntas = 24 respuestas)

Usa `claude-in-chrome`: primero `tabs_context_mcp` con `createIfEmpty: true`, despues **una sola pestana** para todo.

| Motor | Como | Detalle |
|---|---|---|
| ChatGPT | `navigate` a `https://chatgpt.com/?q={pregunta url-encoded}`, esperar 20 s y hacer `get_page_text` | Se envia sola y funciona sin sesion |
| Google Modo IA | `navigate` a `https://www.google.com/search?udm=50&hl=es&gl=cl&q={pregunta}`, esperar 16-20 s y hacer `get_page_text` | Si el texto sale vacio, repetir con mas espera. El AI Overview normal no siempre aparece; por eso se usa Modo IA |
| Gemini | `navigate` a `https://gemini.google.com/app`, esperar **7 s**, clic en el centro del input, `type`, `Return`, esperar 20 s y hacer `get_page_text` | Si la pagina queda en "Por donde empezamos?", la consulta no se envio: tomar screenshot, ubicar el input y repetir |

- Agrupa 2-3 preguntas por `browser_batch`.
- Ante captcha, limite de uso o login: **detenerse y avisar al usuario**. No reintentar en loop.
- Las conversaciones quedan en el historial de ChatGPT, Gemini y Google del usuario. Mencionarlo al entregar.
- **Registra en cada respuesta:**
  - posicion de la marca: `#1` (primera recomendacion o la que la IA elige), `#2`, `#3`..., `mencion menor` (solo nombrada en texto, al final) o `no aparece`;
  - quien es la primera opcion cuando no es la marca;
  - competidores nombrados y fuentes citadas;
  - una frase textual util de menos de 15 palabras;
  - en Q8, horario, precio por persona, canal de reserva o compra, nota y cualquier dato erroneo.
- **No registres** nombres de autores de resenas (Gemini los muestra) ni correos personales.

### Paso 5. Evidencia y calculos

Guarda todo en `{scratchpad}/{marca}/evidencia.md`:

- Una matriz Q1-Q7 x motor con la posicion y la primera opcion.
- Totales: **presencia** X/21 (y por motor), **primera opcion** Y/21 y menciones por competidor (share of voice).
- Para Q8, una tabla de dato x fuente (sitio, ChatGPT, Google Modo IA, Gemini) con las inconsistencias.
- Los datos tecnicos del paso 1 y los de reputacion del paso 2.

**Score GEO /100** (la formula va visible en el reporte):

| Pilar | Pts | Rubrica |
|---|---|---|
| Visibilidad en IA | 40 | 40 x (0,5 x presencia + 0,5 x primera opcion) |
| Acceso para buscadores e IA | 15 | Bots IA no bloqueados (5) + sin crawl-delay (3) + llms.txt (3) + sitemap con las URLs relevantes (4) |
| Datos estructurados | 15 | Tipo de negocio correcto (6) + direccion, horario y contacto (4) + menu, productos u oferta (3) + sameAs o FAQ (2) |
| Contenido citable | 15 | Oferta en texto HTML (5) + secciones por intencion u ocasion (5) + FAQ o respuestas directas (3) + title, description y H1 claros (2) |
| Autoridad y consistencia | 15 | Resenas, por volumen y nota (5) + prensa y guias (3) + consistencia de horario, NAP y canales entre plataformas (4) + perfiles conectados (3) |

**Score SEO /100** = promedio de Tecnico, On-Page, Contenido y Local/Reputacion (cada uno /100). Si es e-commerce, se suma E-commerce al promedio.

### Paso 6. Analisis

- Lanza en paralelo `geo-specialist` y `seo-specialist` (`.claude/agents/`) pasandoles `evidencia.md` completa. Si la evidencia ya esta en el contexto, el analisis se puede hacer directamente.
- Define el **gancho**: la brecha mas concreta entre lo que el negocio es y lo que la IA recomienda. Ejemplo: "La IA te conoce (17/21) pero no te elige (5/21); para comida vasca, las 3 IA eligen a El Txoko".
- Prepara: 3 hallazgos con su consecuencia comercial, 6 quick wins, plan de 90 dias, schema JSON-LD, llms.txt propuesto y KPIs (hoy vs meta).

### Paso 7. Reporte HTML

Copia la plantilla a `auditoria-seo-geo-{marca}.html` y reemplaza el contenido manteniendo el CSS y los componentes. Estructura:

| # | Seccion | Contenido |
|---|---|---|
| 0 | Portada | Titular con el gancho, dominio, fecha DD-MM-AAAA, rubro, ubicacion, CMS, 2 scores (GEO y SEO) y boton CTA |
| - | Resumen ejecutivo | Conclusion en negrita, 3 hallazgos con su consecuencia comercial en rojo, "la buena noticia" y 5 tarjetas de score (GEO, Tecnico, On-Page, Contenido, Local) |
| - | Banner CTA 1 | |
| 1 | Que dice la IA de tu marca | 4 metricas, matriz de posiciones, alerta de lectura, 2 citas `.ai-quote`, share of voice (menciones y primera opcion), tabla de la consulta de marca y tabla de fuentes |
| - | Banner CTA 2 | |
| 2 | Por que la IA no te elige primero | Barras de los 5 pilares, la formula y tarjetas `.issue` con HOY vs RECOMENDADO, de critico a medio |
| 3 | Base SEO | 4 metricas y tablas de Tecnico, On-page, Contenido y Local/Reputacion con su score |
| 4 | Quick wins a 30 dias | 6 tarjetas `.opp-card` con impacto y esfuerzo |
| 5 | Plan de 90 dias | 4 prioridades (critica semana 1-2, alta semana 3-4, media mes 2, baja mes 3+), schema, llms.txt, 6 KPIs hoy vs meta y timeline |
| 6 | Como lo hacemos en SFinance | 3 pasos (Diagnostico, Implementacion, Medicion mensual) y bloque `.cta-final` |
| - | Footer | SFINANCE, correo, fecha y metodologia (motores, fecha de las consultas y "las respuestas de IA varian") |

### Paso 8. Fact-check (obligatorio antes de publicar)

1. Lanza un agente `general-purpose` en segundo plano que compare **cada cifra y afirmacion** del reporte contra `evidencia.md` y el HTML crudo. Debe revisar la aritmetica (X/21, Y/21, %, scores) y marcar lo que contradiga o no respalde la evidencia.
2. Corrige los errores. Las afirmaciones "sin respaldo" que si estan en los textos capturados se mantienen. Las que no estan, se eliminan o se suavizan.
3. Controles con `grep` sobre el reporte:
   - `grep -c 'href="mailto:contact@sfinance.world' {archivo}` debe dar 5 (4 CTA + footer).
   - `grep -cP '[\x{00A1}\x{00BF}\x{00C0}-\x{00FF}]' {archivo}` debe dar 0 (sin tildes crudas, ni n con tilde, ni signos de apertura).
   - `grep -nE '[0-9]{1,2}\.[0-9]{3}\.[0-9]{3}-[0-9kK]' {archivo}` no debe encontrar nada (sin RUT).
   - `grep -oE '[A-Za-z0-9._-]+@[A-Za-z0-9-]+\.[A-Za-z.]+' {archivo} | sort | uniq -c` solo puede mostrar el correo comercial publico del negocio y `contact@sfinance.world`.

### Paso 9. QA visual

1. Servidor local en segundo plano: `python3 -m http.server 8765 --bind 127.0.0.1`.
2. **Escritorio:** abre la URL local en la pestana de Chrome y revisa portada, matriz, barras y CTA final con screenshots.
3. **Movil:** `resize_window` no funciona si la ventana esta maximizada. Inyecta un `<iframe>` de 390 px con `javascript_tool` (`action: "javascript_exec"`) y mide `scrollWidth` contra `innerWidth`, ademas de los elementos que se salen fuera de `.table-wrap` y `.code-block`. La captura headless bajo 500 px sale cortada y no sirve.
4. **PDF:** `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --no-pdf-header-footer --print-to-pdf=...`. Revisa con `pdftoppm -r 28` y una hoja de contacto: sin paginas casi vacias y con el correo del CTA impreso. Referencia: 15 paginas.
5. El CSS ya trae las reglas que evitan los errores conocidos: responsive solo en `@media screen and (max-width: 760px)`, porque el ancho A4 activaria el layout movil, y sin `page-break-inside: avoid` en `.section`. No cambiarlas.

### Paso 10. Publicar

1. Agrega una entrada al **inicio** de la lista en `index.html`:
   ```html
   <li><a href="/auditoria-seo-geo-{marca}"><div class="icon">GEO</div><div class="report-info">
     <h3>Auditoria GEO + SEO - {dominio}</h3>
     <span>Score GEO X/100 · SEO Y/100 · {rubro} · 24 respuestas de ChatGPT, Gemini y Google Modo IA · Aparece en X/21, primera opcion en Y/21</span>
   </div></a></li>
   ```
2. `git add` **solo** de los archivos del reporte, `index.html` y docs. Nunca `.claude/settings.local.json`.
3. Commit (`feat: add GEO + SEO audit for {dominio} (GEO X, SEO Y)`) y push a main.
4. Verifica en vivo: `curl` de `https://marketing-mu-fawn.vercel.app/auditoria-seo-geo-{marca}` hasta obtener 200 y el `mailto` presente. El CDN puede servir la version anterior unos segundos: prueba con `?v=n` y despues sin parametros.

### Paso 11. Cierre y entrega

- Detiene el servidor local y cierra la pestana de Chrome.
- Entrega al usuario:
  - la URL;
  - el gancho y los scores;
  - que no se pudo medir;
  - que las consultas quedaron en su historial de IA;
  - que revise la seccion 6 (descripcion del servicio).

## Reglas comerciales y de contenido

- **CTA solo por correo:** `mailto:contact@sfinance.world?subject=Auditoria%20GEO%20%2B%20SEO%20-%20{dominio}`. No usar sfinance.world/contacto, porque no funciona. Va en 4 bloques (portada, despues del resumen, despues de la seccion 1 y CTA final) mas el footer, con `contact@sfinance.world` visible en texto para que salga en el PDF.
- Sin planes ni precios del servicio, y sin inventar condiciones (duracion de reuniones, plazos garantizados, etc.).
- No inventar cifras: nada de volumenes de busqueda, trafico, ventas ni ROI. Los KPIs son "hoy medido vs meta" y van acompanados de la nota de que las metas no son resultados garantizados.
- Sin testimonios ni casos de otros clientes, porque los reportes son confidenciales.
- Sin RUT, correos personales ni nombres de autores de resenas. Las citas de IA, de menos de 15 palabras.
- Tono: espanol neutro, trato de "tu", sin emojis y escrito para el dueno del negocio. Primero la consecuencia comercial, despues el detalle tecnico.
- Ser honesto con lo medido: si la marca aparece mucho, el gancho es la brecha entre aparecer y ser la primera opcion, no "no apareces".

## Convenciones

- Todos los reportes son HTML standalone, con el CSS inline en el archivo.
- Sin tildes en el contenido del HTML (evita problemas de encoding al imprimir). La "n" con tilde se escribe como entidad `&ntilde;`, que es ASCII-safe. No usar signos de apertura de pregunta ni de exclamacion.
- Score global /100 y scores por seccion /100. Colores: menos de 30 critical, 30-49 low, 50-69 mid y 70 o mas good.
- Badges de severidad: critical (rojo), high (naranja), medium (amarillo), low (verde), ok (verde) y none (gris).
- Alertas: alert-critical, alert-warning, alert-success y alert-info.
- Prioridades: critica (semana 1-2), alta (semana 3-4), media (mes 2) y baja (mes 3+).

## Archivos

- `index.html`: landing con la lista de todos los reportes (el mas reciente arriba).
- `vercel.json`: config de Vercel (cleanUrls habilitado).
- `auditoria-seo-geo-*.html`: reportes de Analisis GEO. Este es el formato vigente.
- `auditoria-seo-geo-pinpilinpausha.html`: **plantilla** de referencia.
- `auditoria-seo-*.html`, `prospeccion-*.html` y otros: reportes historicos.
- `.claude/agents/`: agentes `geo-specialist` y `seo-specialist`.
