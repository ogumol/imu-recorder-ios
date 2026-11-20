# IMU Recorder para Safari en iPhone

Versión independiente de la idea de **IMU Recorder (9 values)** de la página de la asignatura CPS:
https://davidcastells.github.io/CPS/record_imu.html

La página original solicita sensores mediante la Generic Sensor API. Esta versión usa `DeviceMotionEvent`, compatible con Safari de iOS tras conceder permiso desde un toque del usuario.

## Qué registra

- `accel_x_m_s2`, `accel_y_m_s2`, `accel_z_m_s2`: `event.accelerationIncludingGravity` (m/s²). Si no hay esa lectura, usa `event.acceleration` como alternativa; en ese caso no incluye gravedad. Comprueba el comportamiento en tu dispositivo.
- `gyro_x_deg_s`, `gyro_y_deg_s`, `gyro_z_deg_s`: `event.rotationRate.beta`, `gamma`, `alpha` respectivamente (°/s). Se usa el sistema de ejes de la API del navegador, que puede no coincidir con otra implementación de IMU.
- `mag_x_uT`, `mag_y_uT`, `mag_z_uT`: **sin lectura** en Safari, se exportan vacíos (no son ceros).
- Marca temporal ISO en UTC y tiempo transcurrido desde el inicio de la grabación en milisegundos.

Los datos se procesan **solo en el navegador**: no hay backend, analytics ni librerías externas.

## Publicar mediante GitHub Pages

1. En GitHub, crea un repositorio **público** llamado `imu-recorder-ios`.
2. En **Add file → Upload files**, sube `index.html` y `README.md` al nivel raíz, y haz commit a `main`.
3. En **Settings → Pages → Build and deployment**, elige **Deploy from a branch**, `main`, carpeta `/(root)`, y guarda.
4. Abre `https://TU_USUARIO.github.io/imu-recorder-ios/` directamente en **Safari** del iPhone (HTTPS).
5. Pulsa **Activar sensores**, concede permiso, pulsa **Iniciar grabación**, mueve el teléfono, pulsa **Detener grabación** y finalmente **Descargar CSV** o **Compartir / Guardar**.

**Importante:** abrir `index.html` desde Archivos (`file://`) no equivale a una web HTTPS y puede impedir el permiso de sensores. iOS puede suspender los eventos si bloqueas la pantalla o cambias de aplicación.

## CSV

Cabeceras:

```csv
timestamp_iso,elapsed_ms,accel_x_m_s2,accel_y_m_s2,accel_z_m_s2,gyro_x_deg_s,gyro_y_deg_s,gyro_z_deg_s,mag_x_uT,mag_y_uT,mag_z_uT
```

Cada fila corresponde a un evento recibido del navegador mientras la grabación está activa. No se fuerza una frecuencia constante; para el análisis utiliza `elapsed_ms`. Incluye BOM UTF-8 para que Excel interprete bien el archivo.

## Limitaciones

- Los **tres ejes del magnetómetro no están disponibles** con la API de Safari. No se pueden reconstruir a partir de un simple rumbo de brújula.
- No es una adquisición calibrada de laboratorio: el ritmo, precisión y ejes los define el navegador y el hardware.
- La captura máxima por sesión es de 120 000 muestras para evitar consumo excesivo de memoria en el iPhone. Descarga el CSV antes de empezar otra sesión (la nueva grabación reemplaza las muestras anteriores).
- La gráfica recorta su escala visual a ±20 m/s² y ±180 °/s sin alterar los números exportados.
