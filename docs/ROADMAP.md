# Roadmap

## Fase A — Base ejecutable

- estructura mínima del proyecto;
- Dockerfile;
- compose de ejemplo;
- `.env.example`;
- health endpoint;
- logs claros.

Resultado: contenedor reproducible que arranca y se puede inspeccionar.

## Fase B — Ingesta RTSP

- lectura estable;
- último frame disponible;
- reconexión automática;
- estado `STREAM_UP/DOWN`;
- endpoint de snapshot.

Resultado: Video Vigía puede observar un RTSP real sin depender del origen.

## Fase C — Detección mínima

- una ROI;
- diferencia de imagen;
- porcentaje de actividad;
- threshold configurable;
- telemetría visible.

Resultado: saber si la escena real ofrece una señal útil.

## Fase D — Calibración Web

- dibujar polígonos;
- múltiples zonas;
- guardar configuración;
- mostrar overlay;
- barras/valores de actividad;
- controles simples de threshold/hold/cooldown.

Resultado: calibración sin editar archivos a mano.

## Fase E — Estados y eventos

- `NORMAL/PENDING/ALARM/COOLDOWN`;
- evidencia JPEG/JSON;
- historial;
- webhook;
- eventos de caída/recuperación del stream.

Resultado: detector utilizable como sensor externo.

## Fase F — Prueba prolongada

- varias horas/días en escena real;
- medir falsos positivos;
- ajustar background y thresholds;
- medir consumo CPU/RAM;
- analizar iluminación diurna/nocturna;
- documentar presets útiles.

Resultado: comportamiento conocido antes de sumar complejidad.

## Fase G — Integración de alarmas

Crear o conectar un componente externo que consuma el webhook y ejecute acciones.

Primera candidata:

```text
ZONE_ALARM → webhook → FreeSWITCH → llamada VoIP
```

Video Vigía continúa sin conocer FreeSWITCH.

## Fase H — Validación VLM opcional

Sólo si la tasa de falsas alarmas lo justifica:

- snapshot asociado al evento;
- consulta a VLM local;
- salida JSON restringida;
- política de confirmación/rechazo;
- timeout y fallback.

Resultado: IA bajo demanda, no inferencia continua.

## Fase I — Mejoras futuras posibles

No implementar preventivamente:

- buffer circular y clip corto alrededor del evento;
- varias cámaras;
- presets por cámara;
- zonas con reglas combinadas;
- detección de dirección/entrada-salida;
- horario día/noche;
- autenticación WebApp;
- métricas Prometheus;
- integración Home Assistant/MQTT;
- detector especializado de personas/pose.

Cada mejora debe aparecer por una necesidad observada, no por anticipación.
