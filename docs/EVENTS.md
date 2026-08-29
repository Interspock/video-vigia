# Modelo de eventos

## Objetivo

El detector debe producir hechos estructurados. Las acciones externas se resuelven fuera del núcleo de detección.

Esto permite conectar Video Vigía con telefonía, mensajería, automatizaciones u otros sistemas sin acoplar esas tecnologías al procesamiento de imagen.

## Eventos internos

Tipos iniciales sugeridos:

```text
ZONE_PENDING
ZONE_ALARM
ZONE_CLEAR
STREAM_DOWN
STREAM_UP
```

No todos necesitan enviarse al webhook externo.

## ZONE_PENDING

Una zona superó su threshold, pero todavía no cumplió el tiempo mínimo.

Ejemplo:

```json
{
  "type": "ZONE_PENDING",
  "camera": "habitacion",
  "zone": "floor",
  "activity": 0.28,
  "timestamp": "2026-08-29T18:30:00-03:00"
}
```

## ZONE_ALARM

La condición persistió lo suficiente y se confirma el evento.

```json
{
  "type": "ZONE_ALARM",
  "event_id": "20260829T183003-floor",
  "camera": "habitacion",
  "zone": "floor",
  "activity": 0.31,
  "duration": 3.2,
  "timestamp": "2026-08-29T18:30:03-03:00",
  "evidence": "events/2026-08-29/20260829T183003-floor.jpg"
}
```

Este es el evento principal que inicialmente debe enviarse al webhook.

## ZONE_CLEAR

La zona volvió a condición normal después de un episodio.

```json
{
  "type": "ZONE_CLEAR",
  "camera": "habitacion",
  "zone": "floor",
  "timestamp": "2026-08-29T18:31:10-03:00"
}
```

Puede ser útil más adelante para cerrar alarmas o medir duración total.

## Eventos de stream

`STREAM_DOWN` y `STREAM_UP` son importantes porque una cámara inaccesible no debe confundirse con una escena sin movimiento.

La WebApp debe mostrar claramente el estado de conexión.

## Webhook

Configuración:

```env
WEBHOOK_URL=http://otro-servicio:8080/alarm
```

El envío debe:

- tener timeout corto;
- no bloquear el detector;
- registrar error si falla;
- evitar reintentos infinitos;
- permitir reintentos limitados en una etapa posterior.

## Integración futura con acciones

Arquitectura deseada:

```text
Video Vigía
    │
    │ POST evento
    ▼
Alarm Dispatcher
    ├── FreeSWITCH / VoIP
    ├── WhatsApp
    ├── Telegram
    └── otras acciones
```

Video Vigía no debe importar librerías de FreeSWITCH ni contener números telefónicos salvo que en el futuro exista una razón explícita para cambiar esta frontera.

## Segunda validación con VLM

Más adelante, un evento sospechoso podrá tener un estado de validación adicional:

```json
{
  "vision": {
    "validator": "vlm",
    "result": "person_or_body",
    "confidence": null
  }
}
```

No asumir que un VLM entrega una confianza probabilística fiable. La política debe basarse principalmente en categorías y reglas configurables.
