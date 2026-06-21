# Guisómetro

## Qué es

Web 100% frontend y estática (sin backend, sin build, sin dependencias) que detecta el clima de tu zona y te dice si esta noche es **Noche de Guiso**.

La app principal vive en `index.html`. Las recetas viven como HTML estático en `recetas/` y comparten `styles.css`.

La regla: si la **mínima pronosticada de la noche** (ventana 20:00–08:00) es **≤ 10 °C**, es noche de guiso. Cuanto más frío, más ideal.

## Cómo funciona

### Flujo del test

1. El usuario apreta **"Hacer el Test"**
2. Se detecta la ubicación aproximada por IP (sin pedir permisos al navegador)
3. Se consulta el clima actual y pronóstico horario
4. Se calcula la **mínima nocturna** (ventana 20:00 a 08:00 del día siguiente)
5. Se muestra el resultado: gauge con temperatura, veredicto con meme argento, datos y ranking clickeable
6. Cada nivel del ranking abre una receta estática en una pestaña nueva

### APIs usadas (todas gratuitas, HTTPS, sin API key)

| Servicio | Uso | Fallback |
|---|---|---|
| [[ipwho-is]] [ipwho.is](https://ipwho.is) | Geolocalización por IP (primario) | — |
| [[geojs]] [get.geojs.io](https://get.geojs.io) | Geolocalización por IP (fallback 1) | Si ipwho falla |
| [[ipapi]] [ipapi.co](https://ipapi.co) | Geolocalización por IP (fallback 2) | Si geojs falla |
| [[open-meteo]] [Open-Meteo](https://open-meteo.com) | Clima actual + pronóstico horario | — |
| [[bigdatacloud]] [BigDataCloud Reverse Geocode](https://www.bigdatacloud.com/free-api/free-reverse-geocode-to-city-api) | Convierte coordenadas GPS en ciudad/zona legible | Si falla, se muestran coordenadas |
| [[cloudflare-web-analytics]] [Cloudflare Web Analytics](https://developers.cloudflare.com/web-analytics/) | Métricas de visitas mediante snippet JS | No requiere proxy DNS |

> Se usan 3 proveedores de IP en cascada porque los bloqueadores de anuncios (uBlock, EasyPrivacy) bloquean frecuentemente estos servicios. Cada petición tiene un **timeout de 8 segundos** via `AbortController`.

### Escala guisera — 7 franjas

Las frases están tomadas de memes argentos reales. Cada una tiene su fuente.

| Temperatura | Nivel | Frase | Fuente |
|---|---|---|---|
| +30°C | 🥵 Tengo la boca seca | "Tengo la boca seca... ¿no tenés algo para tomar?" | El Secreto de sus Ojos |
| 20–30°C | ☀️ Pero dale... | "¡Un escopetazo en el pecho! Comete un bife de chorizo, pelotudo." | Mozos (Rodriguez Galati) |
| 15–20°C | 🐑🍲 Andás con frío | "¿Qué te has puesto una bufanda? ¿Andás con frío? Pero qué lindo te quedó." | Oscar, andás con frío |
| 10–15°C | 👦🍲 Y sí hijo | "¿Guiso de fideos moñito? ¡Y SÍ HIJO!" | Johnny Primero de Mazo |
| 5–10°C | ❄️🍲 Ha venido el fresco | "La mierda che... ¡cómo se nota que ha venido el fresco eh!" | Se congeló del frío |
| 0–5°C | 🍲🔥 Alto Guiso | "Con 15 pesos sale un paty. ¡Con 15 pesos me hago alto guiso!" | El pibe del guiso |
| <0°C | 🇦🇷🍲 Guiso Nacional | "Sean eternos los laureles que supimos conseguir." | Himno Nacional Argentino |

### Recetas por ranking

Cada nivel del [[ranking-guisero]] tiene una receta estática propia y se abre con `target="_blank"` desde la lista o desde el badge del nivel activo.

| Temperatura | Nivel | Receta |
|---|---|---|
| +30°C | [[tengo-la-boca-seca]] | `recetas/tengo-la-boca-seca.html` |
| 20–30°C | [[pero-dale]] | `recetas/pero-dale.html` |
| 15–20°C | [[andas-con-frio]] | `recetas/andas-con-frio.html` |
| 10–15°C | [[y-si-hijo]] | `recetas/y-si-hijo.html` |
| 5–10°C | [[ha-venido-el-fresco]] | `recetas/ha-venido-el-fresco.html` |
| 0–5°C | [[alto-guiso]] | `recetas/alto-guiso.html` |
| <0°C | [[guiso-nacional]] | `recetas/guiso-nacional.html` |

### ING (Índice Guisero Nacional)

Mapeo no lineal de temperatura a un índice 0–100 que se usa internamente para decidir efectos visuales (partículas, humito, glow). No se muestra al usuario, pero controla la intensidad de los efectos.

```
Temp → ING:  40°C=0  |  30°C=20  |  20°C=40  |  15°C=60  |  10°C=80  |  5°C=90  |  0°C=95  |  -10°C=100
```

## Decisiones de diseño

### ¿Por qué estático?

- Sin build, sin dependencias, sin node_modules
- Se puede deployar en cualquier hosting estático (GitHub Pages, Netlify, etc.)
- La app principal y las recetas se pueden abrir con doble click en el navegador para probar
- Más fácil de mantener para un proyecto chico y memero

### ¿Por qué IP primero y GPS opcional?

- El GPS del navegador **pide permiso** al usuario → fricción
- La IP es **automática y silenciosa** → mejor UX para un test rápido
- La precisión no importa mucho: el clima no cambia en 50km a la redonda
- Si la temperatura no cierra, el usuario puede tocar **"📍 ¿La temperatura no te cierra? Tocá acá"**
- Si acepta permisos del navegador, se recalcula con GPS preciso
- Al aceptar GPS se muestra un tercer chip de resultado: **📍 Ubicación**, ubicado entre **🌡️ Ahora** y **🌙 Mín. noche**
- La ubicación se intenta mostrar como ciudad/zona mediante [[bigdatacloud]]; si falla, se muestran coordenadas redondeadas

### ¿Por qué 3 proveedores de IP?

Los bloqueadores de anuncios tienen a `ipapi.co` e `ipwho.is` en sus listas de bloqueo (EasyPrivacy, etc.). Tener 3 proveedores en cascada + timeout de 8s asegura que funcione para la mayoría de los usuarios sin pedirles que desactiven nada.

### ¿Por qué Open-Meteo?

- Gratis, sin API key, sin límite razonable de requests
- HTTPS
- Devuelve pronóstico horario (necesario para calcular la mínima nocturna)
- No requiere registro

### Efectos visuales

- **Humito animado**: 6 puffs CSS sobre la olla, se activa cuando ING ≥ 40
- **Fondo dinámico**: glow radial que cambia de color según la franja de temperatura
- **Partículas flotantes**: cantidad proporcional al ING, color cálido (guiso) o frío
- **Ranking guisero**: lista de 7 niveles con el nivel actual resaltado
- **Recetas estáticas**: una página por nivel del ranking, con estética hermana del sitio
- **Footer fijo**: `🛸 Un Argento al servicio del guiso.`
- **Favicon**: emoji 🍲 embebido como SVG data URL

## Hosting

- **GitHub Pages** desde rama `main`, carpeta raíz
- Dominio custom via archivo `CNAME` → `guisometro.pelotudonia.com`
- DNS: registro CNAME en Cloudflare apuntando a `nicolasjatip.github.io` (proxy off / DNS only)
- HTTPS activado con certificado [[lets-encrypt]] emitido por [[github-pages]]
- Métricas: [[cloudflare-web-analytics]] via snippet en `index.html`, funciona aunque el registro DNS esté en modo DNS only

## Stack

```
HTML + CSS + JS vanilla → GitHub Pages
```

No hay más. Así de simple.

## Recetas — 2026-06-21

Se incorporó el prototipo externo ubicado en:

`C:\Users\nicol\Documents\Codex\2026-06-21\quiero-que-cada-ranking-tenga-un`

Decisiones de integración:

- No se reemplazó la app principal por el prototipo.
- Se trajeron las 7 páginas de receta a `recetas/`.
- Se agregó `styles.css` para las páginas de receta, ajustado a la identidad visual actual.
- Se mantuvo todo estático.
- El ranking de `index.html` ahora renderiza enlaces reales.
- El badge `Nivel X/7` también abre la receta del resultado actual.
- Las recetas incluyen favicon y [[cloudflare-web-analytics]].

## Cierre de sesión — 2026-06-21

Cambios aplicados y publicados en [[github-pages]]:

- Se hizo pull de `origin/main` y se trabajó sobre `index.html`.
- Se reordenó la escala guisera de abajo hacia arriba:
  - `<0°C` → 🇦🇷🍲 **Guiso Nacional**
  - `0–5°C` → 🍲🔥 **Alto Guiso**
  - `5–10°C` → ❄️🍲 **Ha venido el fresco**
  - `10–15°C` → 👦🍲 **Y sí hijo**
  - `15–20°C` → 🐑🍲 **Andás con frío**
  - `20–30°C` → ☀️ **Pero dale...**
  - `+30°C` → 🥵 **Tengo la boca seca**
- Se agregó una UI de [[ranking-guisero]] con nivel actual, descripción y lista de rangos.
- Se dejó fijo el footer: `🛸 Un Argento al servicio del guiso.`
- Se agregó favicon de guiso 🍲.
- Se confirmó DNS de `guisometro.pelotudonia.com` en [[cloudflare]] como `CNAME → nicolasjatip.github.io`, modo DNS only.
- Se validó que [[github-pages]] emitió certificado HTTPS vía [[lets-encrypt]] y se habilitó el sitio seguro.
- Se agregó [[cloudflare-web-analytics]] con token público en el HTML para medir visitas.
- Se alineó el footer con la columna visual principal.
- Se agregó chip de ubicación para GPS preciso cuando el usuario acepta permisos del navegador.

Commits relevantes:

- `d1e2b2d` — Ajusta ranking guisero y footer
- `a8a120f` — Agrega favicon guisero
- `10aee27` — Agrega Cloudflare Web Analytics
- `3cc631f` — Alinea footer con contenido
- `59909e0` — Muestra ubicacion GPS en resultados
