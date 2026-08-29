# Video Vigía

Video Vigía es un sensor visual liviano que consume un stream RTSP existente y genera eventos cuando detecta cambios relevantes dentro de zonas configurables de la imagen.

El proyecto nace con una regla de arquitectura simple: **Video Vigía no conoce ni controla el sistema que produce el RTSP**. Para Video Vigía, la cámara es solamente una URL RTSP.

## Objetivo

Construir un servicio desacoplado, ejecutable en Docker, que permita:

- consumir uno o más streams RTSP;
- obtener frames a baja frecuencia, sin procesar video completo a 25/30 fps;
- dibujar y configurar máscaras/ROI desde una WebApp mínima;
- medir actividad/cambio dentro de cada zona;
- aplicar persistencia temporal, histéresis y cooldown para evitar falsas alarmas;
- generar eventos estructurados cuando una condición se cumple;
- guardar evidencia mínima del evento;
- exponer eventos hacia otros sistemas mediante una interfaz desacoplada, inicialmente webhook HTTP;
- permitir en una etapa posterior una segunda validación mediante un VLM, sin convertir al VLM en el detector primario.

## Principios

1. **Desacoplado del NVR**: recibe RTSP. Nada más.
2. **CPU primero**: el detector básico debe ser barato y poder funcionar continuamente sin GPU.
3. **La visión clásica es el sensor**: OpenCV detecta cambios; una IA puede ser un juez opcional posterior.
4. **No grabar video permanentemente**: conservar sólo evidencia asociada a eventos, salvo que más adelante se decida lo contrario.
5. **Configuración visual**: las zonas se dibujan sobre un frame real de la cámara.
6. **Detector y acciones separados**: detectar una condición no debe implicar saber cómo se llama por teléfono, se envía WhatsApp o se notifica a otro sistema.
7. **MVP pequeño y observable**: antes de agregar IA o automatizaciones externas, medir el comportamiento real del detector sobre la cámara real.

## Arquitectura conceptual

```text
      RTSP existente
           │
           ▼
    ┌───────────────┐
    │  Video Vigía  │
    │               │
    │ RTSP reader   │
    │      ↓        │
    │ preprocessing │
    │      ↓        │
    │ ROI detector  │
    │      ↓        │
    │ state machine │
    │      ↓        │
    │ event manager │
    └───────┬───────┘
            │
      ┌─────┴─────────────┐
      │                   │
      ▼                   ▼
 evidencia local       webhook/evento
                            │
                            ▼
                    sistemas externos
```

La integración actual esperada es:

```text
Star4Live / NVR → MediaMTX → RTSP → Video Vigía
```

Pero Video Vigía no debe depender de Star4Live ni de MediaMTX. Cualquier RTSP compatible debe servir como entrada.

## MVP

El primer prototipo debe demostrar solamente esto:

```text
RTSP
 ↓
frame reducido
 ↓
una ROI configurada
 ↓
medición de cambio
 ↓
threshold + tiempo mínimo
 ↓
evento + JPEG
```

Sin VLM. Sin telefonía. Sin WhatsApp. Sin lógica específica del NVR.

Cuando eso sea estable sobre imágenes reales, se agregan capas.

## Documentación

- [Arquitectura](docs/ARCHITECTURE.md)
- [MVP y criterios de aceptación](docs/MVP.md)
- [Configuración](docs/CONFIGURATION.md)
- [Modelo de eventos](docs/EVENTS.md)
- [Roadmap](docs/ROADMAP.md)
- [Guía para implementación con Codex](AGENTS.md)

## Estado

Proyecto en fase de diseño e implementación inicial.
