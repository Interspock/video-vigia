# AGENTS.md — Guía de implementación

Este archivo está dirigido a agentes de programación (Codex u otros) que trabajen sobre este repositorio.

## 1. Leer antes de modificar

Antes de escribir código, leer:

1. `README.md`
2. `docs/ARCHITECTURE.md`
3. `docs/MVP.md`
4. `docs/CONFIGURATION.md`
5. `docs/EVENTS.md`
6. `docs/ROADMAP.md`

Si una decisión de implementación contradice esos documentos, detenerse y explicitar la contradicción antes de cambiar la arquitectura.

## 2. Frontera estricta

Este repositorio es **Video Vigía**, no Star4Live.

No modificar, automatizar ni depender del código del gateway NVR/Star4Live.

La única entrada necesaria desde infraestructura externa es una URL RTSP configurable.

No asumir rutas, contenedores, nombres de servicio ni credenciales particulares del sistema que genera el RTSP.

## 3. Filosofía de implementación

Priorizar:

- código pequeño y legible;
- pocas dependencias;
- Python + OpenCV para backend/detección;
- WebApp simple con HTML/CSS/JavaScript vanilla;
- archivos JSON/YAML para configuración inicial;
- Docker reproducible;
- CPU baja;
- observabilidad suficiente para calibrar;
- separación entre detección y acciones.

Evitar salvo necesidad demostrada:

- React/Vue/Angular;
- bases SQL;
- Redis;
- colas externas;
- microservicios innecesarios;
- frameworks de visión pesados;
- inferencia IA continua;
- grabación permanente de video.

## 4. Trabajo por etapas

No implementar el roadmap completo de una sola vez.

La primera secuencia debe ser:

### Paso 1

Crear base ejecutable:

- Dockerfile;
- docker-compose.yml;
- `.env.example`;
- backend mínimo;
- `/api/health`.

Probar que construye y arranca.

### Paso 2

Agregar RTSP:

- leer `RTSP_URL`;
- mantener frame reciente;
- reconectar ante caída;
- `/api/frame.jpg`;
- mostrar estado del stream.

Probar con RTSP real antes de seguir.

### Paso 3

Agregar una única ROI y exponer `activity`.

No implementar todavía VLM ni alarmas externas.

### Paso 4

Agregar UI para dibujar/calibrar la ROI.

### Paso 5

Agregar máquina de estados, eventos y webhook.

Cada paso debe quedar funcionando antes del siguiente.

## 5. Dependencias externas

El servicio debe aceptar por entorno como mínimo:

```env
RTSP_URL=
CAMERA_ID=
HTTP_PORT=
CONFIG_DIR=
DATA_DIR=
WEBHOOK_URL=
```

No insertar secretos reales en commits.

## 6. Video y latencia

No procesar todos los frames del RTSP por defecto.

Objetivo inicial: 2–5 frames de análisis por segundo.

La aplicación debe favorecer el **frame más reciente** y descartar atraso acumulado. Para este caso de uso, un detector varios segundos detrás del stream es peor que saltarse frames.

## 7. Robustez RTSP

La desconexión del stream es un estado esperado, no un error fatal.

El proceso debe:

- seguir vivo;
- registrar el cambio de estado;
- reintentar conexión;
- recuperarse solo cuando reaparece el stream.

No generar alarmas de ROI usando frames viejos durante una desconexión.

## 8. WebApp

La UI debe ser funcional antes que estética.

Debe permitir como mínimo:

- ver imagen actual;
- superponer zonas;
- dibujar polígono;
- guardar;
- ver `activity` y estado por zona.

Canvas HTML es una opción adecuada.

Las coordenadas persistidas deben ser normalizadas `[0,1]`.

## 9. Detector

Mantener separadas estas capas:

```text
capture
  ↓
preprocess
  ↓
measure ROI
  ↓
state machine
  ↓
event manager
```

No mezclar envío de webhook dentro del código que calcula diferencias de píxeles.

Diseñar interfaces simples para poder cambiar el algoritmo de background sin reescribir el resto.

## 10. Pruebas

Agregar tests unitarios donde aporten valor, especialmente para:

- conversión de coordenadas normalizadas;
- cálculo de ROI;
- transiciones de máquina de estados;
- cooldown;
- serialización/configuración.

La lectura RTSP real debe probarse además manualmente/integración porque los mocks no validan comportamiento de codecs, buffering ni reconexión.

## 11. Cambios de alcance

No agregar por iniciativa propia:

- telefonía;
- WhatsApp;
- reconocimiento facial;
- VLM;
- detección de pose;
- autenticación compleja;
- soporte multicámara.

Si durante la implementación aparece una razón técnica fuerte para hacerlo, documentarla y proponerla antes.

## 12. Criterio general

Video Vigía debe sentirse como **un sensor visual pequeño**.

Si una solución vuelve al proyecto pesado, difícil de desplegar o dependiente de componentes que no necesita, buscar primero una alternativa más simple.
