# Configuración

## Variables de entorno

La configuración sensible o dependiente del despliegue debe ir por entorno.

Ejemplo orientativo:

```env
RTSP_URL=rtsp://user:password@mediamtx:8554/cam3
CAMERA_ID=habitacion
HTTP_PORT=8080
CONFIG_DIR=/config
DATA_DIR=/data
WEBHOOK_URL=
LOG_LEVEL=INFO
```

Nunca versionar `.env` con credenciales reales.

## Parámetros de captura

Valores iniciales razonables:

```yaml
capture:
  analysis_fps: 3
  width: 640
  height: 360
  reconnect_seconds: 3
```

Estos valores deben ser configurables. No se debe asumir que el stream original tiene esa resolución ni ese framerate.

## Zonas

Ejemplo de `zones.json`:

```json
{
  "zones": [
    {
      "id": "floor",
      "name": "Piso junto a la cama",
      "enabled": true,
      "points": [
        [0.50, 0.55],
        [0.90, 0.54],
        [0.92, 0.96],
        [0.48, 0.96]
      ],
      "threshold": 0.22,
      "hold_seconds": 3.0,
      "cooldown_seconds": 300
    }
  ]
}
```

## Coordenadas

Los puntos deben almacenarse normalizados entre `0.0` y `1.0`.

Ventajas:

- independencia de resolución;
- el frontend puede redimensionar el frame sin alterar la máscara;
- una máscara sigue siendo válida si se cambia la resolución interna de análisis.

## Threshold

`threshold` representa la fracción de píxeles de la ROI considerados cambiados.

Ejemplo:

```text
0.22 = 22 % de la zona
```

No fijar valores universales. Deben calibrarse con la escena real.

## hold_seconds

Tiempo durante el cual la actividad debe permanecer por encima del threshold antes de confirmar una alarma.

Esto filtra:

- ruido de compresión;
- flashes;
- sombras fugaces;
- cambios de un solo frame.

## cooldown_seconds

Tiempo mínimo entre eventos confirmados de una misma zona/episodio.

Debe combinarse con la lógica de retorno a normal para evitar alarmas repetidas mientras una situación continúa.

## Configuración del background

Posible esquema futuro:

```yaml
background:
  mode: running_average
  alpha: 0.005
  freeze_on_pending: true
  freeze_on_alarm: true
```

No exponer demasiados parámetros en la UI antes de comprobar que realmente son útiles.

## Persistencia

El contenedor debe poder destruirse y recrearse sin perder configuración:

```yaml
volumes:
  - ./config:/config
  - ./data:/data
```

`data` podrá considerarse opcional si se decide no conservar evidencia.
