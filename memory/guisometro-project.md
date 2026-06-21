# Guisómetro

## Qué es

Web 100% frontend (un solo `index.html`, sin backend, sin build, sin dependencias) que detecta el clima de tu zona y te dice si esta noche es **Noche de Guiso**.

La regla: si la **mínima pronosticada de la noche** (ventana 20:00–08:00) es **≤ 10 °C**, es noche de guiso. Cuanto más frío, más ideal.

## Cómo funciona

### Flujo del test

1. El usuario apreta **"Hacer el Test"**
2. Se detecta la ubicación aproximada por IP (sin pedir permisos al navegador)
3. Se consulta el clima actual y pronóstico horario
4. Se calcula la **mínima nocturna** (ventana 20:00 a 08:00 del día siguiente)
5. Se muestra el resultado: gauge con temperatura, veredicto con meme argento, y datos

### APIs usadas (todas gratuitas, HTTPS, sin API key)

| Servicio | Uso | Fallback |
|---|---|---|
| [[ipwho-is]] [ipwho.is](https://ipwho.is) | Geolocalización por IP (primario) | — |
| [[geojs]] [get.geojs.io](https://get.geojs.io) | Geolocalización por IP (fallback 1) | Si ipwho falla |
| [[ipapi]] [ipapi.co](https://ipapi.co) | Geolocalización por IP (fallback 2) | Si geojs falla |
| [[open-meteo]] [Open-Meteo](https://open-meteo.com) | Clima actual + pronóstico horario | — |

> Se usan 3 proveedores de IP en cascada porque los bloqueadores de anuncios (uBlock, EasyPrivacy) bloquean frecuentemente estos servicios. Cada petición tiene un **timeout de 8 segundos** via `AbortController`.

### Escala guisera — 7 franjas

Las frases están tomadas de memes argentos reales. Cada una tiene su fuente.

| Temperatura | Nivel | Frase | Fuente |
|---|---|---|---|
| +30°C | 🥵 Tengo la boca seca | "Tengo la boca seca... ¿no tenés algo para tomar?" | El Secreto de sus Ojos |
| 20–30°C | ☀️ Pero dale... | "¡Un escopetazo en el pecho! Comete un bife de chorizo, pelotudo." | Mozos (Rodriguez Galati) |
| 15–20°C | 🍲 Y sí hijo | "¿Guiso de fideos moñito? ¡Y SÍ HIJO!" | Johnny Primero de Mazo |
| 10–15°C | 🍲🔥 Alto Guiso | "Con 15 pesos sale un paty. ¡Con 15 pesos me hago alto guiso!" | El pibe del guiso |
| 5–10°C | ❄️🍲 Ha venido el fresco | "La mierda che... ¡cómo se nota que ha venido el fresco eh!" | Se congeló del frío |
| 0–5°C | 🧣🍲 Andás con frío | "¿Qué te has puesto una bufanda? ¿Andás con frío?" | Oscar, andás con frío |
| <0°C | ☢️🍲 Guiso Nacional | "Oíd mortales, el grito sagrado..." | Himno Nacional Argentino |

### ING (Índice Guisero Nacional)

Mapeo no lineal de temperatura a un índice 0–100 que se usa internamente para decidir efectos visuales (partículas, humito, glow). No se muestra al usuario, pero controla la intensidad de los efectos.

```
Temp → ING:  40°C=0  |  30°C=20  |  20°C=40  |  15°C=60  |  10°C=80  |  5°C=90  |  0°C=95  |  -10°C=100
```

## Decisiones de diseño

### ¿Por qué un solo HTML?

- Sin build, sin dependencias, sin node_modules
- Se puede deployar en cualquier hosting estático (GitHub Pages, Netlify, etc.)
- Se puede abrir con doble click en el navegador para probar
- Más fácil de mantener para un proyecto chico y memero

### ¿Por qué IP y no GPS?

- El GPS del navegador **pide permiso** al usuario → fricción
- La IP es **automática y silenciosa** → mejor UX para un test rápido
- La precisión no importa mucho: el clima no cambia en 50km a la redonda
- No mostramos la ubicación al usuario, así que no se nota la imprecisión

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
- **Footer memero**: frase random de un pool de ~15, rota en cada test

## Hosting

- **GitHub Pages** desde rama `main`, carpeta raíz
- Dominio custom via archivo `CNAME` → `guisometro.pelotudonia.com`
- DNS: registro CNAME en Cloudflare apuntando a `nicolasjatip.github.io` (proxy off / DNS only)

## Stack

```
HTML + CSS + JS vanilla → GitHub Pages
```

No hay más. Así de simple.
