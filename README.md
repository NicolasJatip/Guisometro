# 🍲 Guisómetro

Web 100% frontend (sin backend) que detecta tu ubicación por IP, mira el clima de tu zona
y te dice si esta noche es **Noche de Guiso**.

👉 La regla: si la **mínima pronosticada de la noche** (20:00–08:00) es **≤ 10 °C**, es noche de guiso.
Cuanto más frío, más ideal para tirar todo a la olla.

## Cómo funciona

- **Ubicación:** geolocalización por IP con [ipwho.is](https://ipwho.is) (HTTPS, sin API key),
  con fallback a [ipapi.co](https://ipapi.co).
- **Clima:** [Open-Meteo](https://open-meteo.com) (HTTPS, gratis, sin API key).
- **UI:** un gauge/termómetro con aguja animada y el veredicto.

Todo vive en un único `index.html`. No hay build ni dependencias.

## Probar localmente

Abrí `index.html` en el navegador. (Algunas extensiones/bloqueadores pueden frenar las
llamadas a las APIs de IP; si pasa, deshabilitalas para el sitio.)

## Deploy en GitHub Pages

1. Subí el repo a GitHub.
2. **Settings → Pages → Source: Deploy from a branch** → rama `main`, carpeta `/ (root)`.
3. El archivo `CNAME` ya apunta a `guisometro.pelotudonia.com`.
4. En tu DNS, creá un registro **CNAME** `guisometro` → `<tu-usuario>.github.io`.
5. Esperá la propagación y activá **Enforce HTTPS**.
