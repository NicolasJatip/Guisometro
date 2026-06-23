# 🍲 Guisómetro

Web 100% frontend (sin backend) que detecta tu ubicación por IP, mira el clima de tu zona
y te dice si esta noche es **Noche de Guiso**.

👉 La regla: si la **mínima pronosticada de la noche** (20:00–08:00) es **≤ 12 °C**, es noche de guiso.
Cuanto más frío, más ideal para tirar todo a la olla.

## Cómo funciona

- **Ubicación:** geolocalización por IP con [ipwho.is](https://ipwho.is) (HTTPS, sin API key),
  con fallback a [ipapi.co](https://ipapi.co).
- **Clima:** [Open-Meteo](https://open-meteo.com) (HTTPS, gratis, sin API key).
- **UI:** un gauge/termómetro con aguja animada y el veredicto.

La app principal vive en `index.html`. Las recetas son páginas estáticas en `recetas/`
y usan `styles.css` compartido. Las referencias visuales de memes viven en
`assets/memes/`. No hay build ni dependencias.

## Recetas por ranking

Después del test, cada nivel del ranking abre su receta en una pestaña nueva:

- `<3°C` → `recetas/guiso-nacional.html`
- `3–6°C` → `recetas/y-si-hijo.html`
- `6–9°C` → `recetas/alto-guiso.html`
- `9–12°C` → `recetas/andas-con-frio.html`
- `12–18°C` → `recetas/ha-venido-el-fresco.html`
- `+18°C` → `recetas/tengo-la-boca-seca.html`

Cada receta incluye una imagen/referencia directa al meme del nivel. Las fuentes están documentadas en `assets/memes/SOURCES.md`.

## Más detalles del proyecto

La memoria larga del proyecto está en `memory/guisometro-project.md`. Ahí quedan las decisiones de diseño, hosting, analytics, ranking, recetas y referencias visuales con formato compatible con Obsidian.

## Probar localmente

Abrí `index.html` en el navegador. (Algunas extensiones/bloqueadores pueden frenar las
llamadas a las APIs de IP; si pasa, deshabilitalas para el sitio.)

## Deploy en GitHub Pages

1. Subí el repo a GitHub.
2. **Settings → Pages → Source: Deploy from a branch** → rama `main`, carpeta `/ (root)`.
3. El archivo `CNAME` ya apunta a `guisometro.pelotudonia.com`.
4. En tu DNS, creá un registro **CNAME** `guisometro` → `<tu-usuario>.github.io`.
5. Esperá la propagación y activá **Enforce HTTPS**.
