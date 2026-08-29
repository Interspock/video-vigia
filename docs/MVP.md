# MVP

## Meta

Validar con la cámara real que una máscara/ROI puede detectar cambios relevantes con pocos falsos positivos y muy bajo costo de CPU.

El MVP no intenta todavía resolver toda la interpretación semántica de la escena.

## Etapa 0 — Arranque

Entregables:

- `Dockerfile`;
- `docker-compose.yml` de ejemplo;
- `.env.example`;
- aplicación que arranca y expone `/api/health`;
- lectura de `RTSP_URL` desde entorno.

Criterio de aceptación:

```text
docker compose up -d
curl http://localhost:<port>/api/health
```

debe indicar servicio saludable.

## Etapa 1 — Lectura RTSP

Entregables:

- conexión al RTSP;
- reconexión automática;
- captura del frame más reciente;
- endpoint `/api/frame.jpg`;
- métricas/log básicos de conexión.

Criterios de aceptación:

- la aplicación sobrevive a una caída temporal del RTSP;
- al volver el stream, se reconecta sin reiniciar el contenedor;
- no acumula segundos de video atrasado.

## Etapa 2 — Una ROI y telemetría

Entregables:

- una zona poligonal configurable;
- cálculo de porcentaje de cambio;
- `/api/status` con el valor actual;
- WebApp que superpone la máscara y muestra el porcentaje.

Ejemplo:

```json
{
  "zone": "floor",
  "activity": 0.084,
  "state": "NORMAL"
}
```

Criterio de aceptación:

Al entrar una persona/objeto grande en la zona, la actividad debe aumentar de forma claramente distinguible del ruido normal de la cámara.

## Etapa 3 — Configurador visual

Entregables:

- dibujar un polígono sobre el frame;
- editar/borrar zona;
- guardar la zona;
- controles para threshold, `hold_seconds` y `cooldown_seconds`;
- persistencia tras reiniciar el contenedor.

Criterio de aceptación:

No debe ser necesario editar coordenadas manualmente.

## Etapa 4 — Máquina de estados

Entregables:

Estados iniciales:

```text
NORMAL → PENDING → ALARM → COOLDOWN → NORMAL
```

Criterios de aceptación:

- un pico breve no genera alarma;
- una condición persistente sí;
- un mismo episodio produce un solo evento;
- durante cooldown no se producen llamadas repetidas al webhook.

## Etapa 5 — Evidencia y eventos

Entregables:

- guardar JPEG de la alarma;
- guardar JSON asociado;
- historial de eventos;
- webhook HTTP configurable.

Criterio de aceptación:

Al provocar una alarma artificialmente debe aparecer:

```text
/data/events/<fecha>/<timestamp>-<zone>.jpg
/data/events/<fecha>/<timestamp>-<zone>.json
```

y debe enviarse exactamente un webhook.

## Etapa 6 — Prueba real

Antes de agregar VLM o telefonía, ejecutar el detector durante horas/días y registrar:

- actividad normal de cada zona;
- máximos sin evento real;
- alarmas reales provocadas a propósito;
- falsas alarmas;
- uso de CPU/memoria;
- reconexiones RTSP.

El objetivo es encontrar thresholds y tiempos razonables basados en datos reales, no por intuición.

## Fuera del MVP

Queda expresamente fuera de esta primera implementación:

- reconocimiento facial;
- detección de pose;
- clasificación de personas;
- inferencia continua con VLM;
- llamadas VoIP directas;
- WhatsApp/Telegram directos;
- almacenamiento permanente de video;
- multiusuario complejo;
- base de datos SQL;
- dependencia del software Star4Live/NVR.
