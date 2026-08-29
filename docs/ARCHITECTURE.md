# Arquitectura

## 1. Frontera del sistema

Video Vigía empieza en una URL RTSP y termina en eventos estructurados.

No debe:

- iniciar sesión contra el NVR;
- abrir túneles P2P;
- administrar MediaMTX;
- modificar la configuración de Star4Live;
- conocer credenciales o protocolos internos del NVR;
- realizar directamente llamadas VoIP, envíos de WhatsApp u otras acciones externas específicas.

Esas responsabilidades pertenecen a otros componentes.

## 2. Despliegue esperado

El servicio se ejecutará inicialmente como un tercer contenedor independiente:

```text
┌──────────────────┐
│ Star4Live gateway│
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│     MediaMTX     │
└────────┬─────────┘
         │ RTSP
         ▼
┌──────────────────────────────┐
│          video-vigia         │
│                              │
│ capture → detector → events  │
│              │               │
│              └→ WebApp/API   │
└──────────────────────────────┘
```

Puede compartir una red Docker con MediaMTX si resulta conveniente, o consumir un RTSP publicado por host/IP. No debe requerir estar en el mismo `docker-compose.yml` que Star4Live.

## 3. Componentes internos

### 3.1 RTSP reader

Responsabilidad: mantener acceso al stream y entregar frames recientes al detector.

Requisitos iniciales:

- reconexión automática;
- timeout razonable;
- descarte de frames atrasados;
- procesamiento a baja frecuencia configurable, inicialmente 2–5 fps;
- resize antes del análisis, por ejemplo 640×360 o inferior.

El objetivo no es procesar cada frame del stream original.

### 3.2 Preprocesamiento

Pipeline inicial sugerido:

1. resize;
2. conversión a escala de grises;
3. Gaussian blur suave;
4. cálculo de diferencia contra referencia;
5. threshold;
6. operaciones morfológicas opcionales para quitar ruido.

Debe ser posible inspeccionar visualmente el resultado durante calibración.

### 3.3 Zonas / máscaras

Una zona es un polígono definido en coordenadas normalizadas respecto de la imagen.

Ejemplo conceptual:

```json
{
  "id": "floor",
  "name": "Piso junto a la cama",
  "points": [
    [0.52, 0.55],
    [0.88, 0.54],
    [0.91, 0.95],
    [0.48, 0.95]
  ]
}
```

Usar coordenadas normalizadas evita acoplar las máscaras a una resolución concreta.

Cada zona tendrá parámetros propios de detección.

### 3.4 Detector

Para cada zona debe calcular al menos:

```text
activity = changed_pixels_inside_roi / total_pixels_inside_roi
```

El detector no decide acciones externas. Sólo produce señales y eventos.

Debe diferenciar dos conceptos:

- **movimiento instantáneo**: comparación con frame previo;
- **cambio persistente de escena**: comparación con background/referencia.

En el MVP puede implementarse inicialmente sólo uno de ellos si simplifica la validación, pero la arquitectura debe permitir ambos.

### 3.5 Background model

Primera implementación posible: promedio exponencial lento.

Conceptualmente:

```text
background = (1 - alpha) * background + alpha * current_frame
```

con `alpha` pequeño.

Regla importante: cuando una zona está en estado sospechoso/alarma, la actualización del background debe poder congelarse para evitar que una situación anormal termine aprendida como normal.

Más adelante se podrá evaluar MOG2/KNN si aportan una mejora real.

### 3.6 Máquina de estados

Cada zona mantiene su propio estado.

Modelo inicial:

```text
NORMAL
  │ threshold superado
  ▼
PENDING
  │ persiste hold_seconds
  ▼
ALARM
  │ evento emitido
  ▼
COOLDOWN
  │ cooldown vencido y condición normalizada
  ▼
NORMAL
```

Objetivos:

- ignorar cambios fugaces;
- emitir un solo evento por episodio;
- impedir tormentas de eventos;
- saber cuándo una zona vuelve a estado normal.

### 3.7 Event manager

Recibe eventos del detector y se ocupa de efectos internos del proyecto:

- timestamp;
- guardar JPEG de evidencia;
- guardar metadata JSON;
- mantener historial reciente;
- publicar el evento a través de la API/webhook configurado.

Las integraciones externas deben quedar detrás de una interfaz simple.

### 3.8 WebApp / API

La misma aplicación debe exponer una interfaz web mínima para:

- ver el frame actual;
- dibujar, editar y borrar zonas poligonales;
- activar/desactivar zonas;
- modificar threshold, tiempo mínimo y cooldown;
- ver porcentaje de actividad en vivo;
- ver estado actual de cada zona;
- guardar configuración;
- consultar eventos recientes.

No se necesita un framework frontend complejo. HTML/CSS/JavaScript simple es suficiente.

## 4. Persistencia

Para la primera versión no hace falta una base de datos.

Sugerencia:

```text
/config
  config.yaml
  zones.json

/data
  events/
    YYYY-MM-DD/
      timestamp-zone.jpg
      timestamp-zone.json
```

El directorio `/config` y, opcionalmente, `/data` deben montarse como volúmenes.

## 5. API mínima esperada

No es contrato definitivo, pero orienta la implementación:

```text
GET  /api/health
GET  /api/frame.jpg
GET  /api/zones
PUT  /api/zones
GET  /api/status
GET  /api/events
POST /api/test-event
```

El endpoint de frame debe usarse sólo para calibración/WebApp; el detector debe leer el stream internamente.

## 6. Eventos externos

Video Vigía debe poder hacer un `POST` configurable a un webhook cuando aparezca un evento confirmado.

Ejemplo:

```json
{
  "type": "zone_alarm",
  "camera": "habitacion",
  "zone": "floor",
  "activity": 0.31,
  "duration": 4.2,
  "timestamp": "2026-08-29T18:30:00-03:00",
  "evidence": "events/2026-08-29/183000-floor.jpg"
}
```

Un servicio externo podrá traducir esto a llamada FreeSWITCH, WhatsApp, Telegram, etc.

## 7. VLM como segunda capa

No forma parte del MVP.

Arquitectura futura:

```text
ROI detector
     │ sospecha
     ▼
 snapshot
     │
     ▼
    VLM
     │ clasificación
     ▼
 event policy
```

El VLM nunca debe ser requisito para que el detector básico funcione.

## 8. Recursos

Prioridades:

- CPU baja;
- memoria moderada;
- no GPU requerida para detección clásica;
- no grabación continua a disco;
- tolerancia a caída/reconexión RTSP.

## 9. Seguridad

- RTSP URL y credenciales mediante variables de entorno o `.env`, nunca versionadas;
- WebApp inicialmente pensada para red confiable/local;
- si se publica fuera de LAN, agregar autenticación antes de exponerla;
- evitar logs que impriman URLs RTSP con credenciales.
