# Autonomous-Racing-Challenge

Este repositorio contiene la solución al ejercicio "Visual Follow Line" de Robotics Academy. El objetivo es programar un coche de Fórmula 1 simulado para que siga de forma autónoma la línea roja pintada en un circuito de carreras, utilizando un controlador PID reactivo.

## 1. Descripción del Funcionamiento

La solución se basa en un bucle de control que procesa imágenes de la cámara del vehículo para calcular y aplicar las velocidades lineal y angular necesarias.

### Detección de la Línea

El primer paso es identificar la línea a seguir. El proceso es el siguiente:

1.  **Obtención de Imagen:** En cada iteración, se captura la imagen de la cámara frontal del vehículo (en formato BGR8).
2.  **Filtrado de Color:** Se aplica un filtro de color específico (usando OpenCV) para aislar únicamente los píxeles que corresponden a la línea roja. Esto genera una imagen binaria (blanco y negro) llamada "máscara".
3.  **Cálculo del Error:** Se procesa la máscara para encontrar el centroide (el centro) de la línea detectada. El "error" se define como la diferencia horizontal entre el centroide de la línea y el "Set Point" (el punto objetivo, que es el centro horizontal de la imagen).
    * Si `error = 0`, el coche está perfectamente centrado.
    * Si `error > 0`, la línea está a la derecha del coche.
    * Si `error < 0`, la línea está a la izquierda del coche.

### Controlador PD
El seguimiento de la línea se realiza mediante un controlador PID (Proporcional–Integral–Derivativo), un mecanismo de control basado en la retroalimentación del sistema. Este controlador calcula continuamente el error entre la posición deseada y la actual, y ajusta el movimiento del vehículo en consecuencia.

#### Componentes del Controlador
* **Proporcional (P):** Corrige el error actual. La velocidad angular del coche es proporcional a la magnitud del error: cuanto más se desvía, más corrige la dirección.
* **Integral (I):** Compensa errores acumulados en el tiempo. Si el coche tiende a desviarse sistemáticamente, este término elimina el offset o sesgo residual.
* **Derivativo (D):** Actúa sobre la tasa de cambio del error, anticipando movimientos bruscos y suavizando la respuesta del sistema.

#### Variantes y Ajuste
Durante el desarrollo se probaron distintas configuraciones:
* **Controlador P (ControlProporcional):** La forma más básica. Corrige el error directamente en proporción a su magnitud. Puede ser inestable o lento si Kp no se ajusta correctamente.
     La velocidad angular (`control`) se calcula como: `control = (Kp * erro)`.
* **Controlador PD (ControlDerivativo):**  Añade la derivada del error para amortiguar las oscilaciones y anticipar curvas. Es el enfoque principal utilizado en esta solución.
    La velocidad angular (`control`) se calcula como: `control = (Kp * erro) + (kd * derivative_error)`, donde el `derivative_error` es la diferencia entre el error actual y el anterior, calculado como: `derivative_error = error - prev_error`.
* **Controlador PID completo (ContorlDerivativoIntegral):** Incluye el término integral para eliminar errores persistentes. Resulta útil en circuitos con trayectorias largas o curvas suaves.
  La velocidad angular (`control`) se calcula como: `control = (Kp * erro) + (kd * derivative_error) + (ki * integral_error)`, donde el `integral_error` es el error acumulado, calculado como: `integral_error += error`.
  
El ajuste de las ganancias (`Kp`, `Ki`, `Kd`) se realizó experimentalmente hasta obtener una respuesta estable y fluida.

| Parámetro | Valor |
| :-- | --: |
| **Kp** | 0.01 |
| **Ki** | 0.0000005 |
| **Kd** | 0.0005 |

La velocidad lineal (`v`) se ajusta dinámicamente según la magnitud del error (`err`):
```python
if err > 80 or err < -80:
  velocity = 3
elif err > 20 or err < -20:
  velocity = 7
elif err > 7 or err < -7:
  velocity = 10
else:
  velocity = 6
```

## 3. Instrucciones de Ejecución

### Prerrequisitos
* Tener Docker instalado y funcionando correctamente.
* Tener acceso a Unibotics o Robotics Academy.
* Python 3.x (solo si vas a ejecutar el controlador localmente).
* Bibliotecas de Python necesarias: opencv-python, numpy.

### Lanzamiento 
#### Inicie el contenedor Docker
Inicie el contenedor Docker en su máquina. Abre una terminal y ejecuta una de las siguientes opciones según tu sistema:
1. Sin aceleración gráfica (recomendado para la mayoría de usuarios):
   ```bash
      docker run --rm -it -p 6080-6090:6080-6090 -p 7163:7163 \
      jderobot/robotics-backend:latest
   ```
2. En Linux con tarjeta gráfica:
   ```bash
      docker run --rm -it --device /dev/dri \
     -p 6080-6090:6080-6090 -p 7163:7163 \
     jderobot/robotics-backend:latest
   ```
3. En sistemas con GPU Nvidia:
(asegúrate de tener instalados los drivers propietarios y el [Nvidia Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
    ```bash
       docker run --rm -it --device /dev/dri --gpus all \
       -p 6080-6090:6080-6090 -p 7163:7163 \
       jderobot/robotics-backend:latest
    ```
Mientras el contenedor esté en ejecución, podrá navegar por Unibotics sin necesidad de reiniciarlo.

##### Reiniciar el Docker
*   Identificar contenedores Docker activos: `docker ps -a`
*   Detener el contenedor: `docker stop ID_DEL_CONTENEDOR`
*   Eliminar el contenedor: `docker rm ID_DEL_CONTENEDOR`
   
#### Flujo de Trabajo en Unibotics
Una vez que el contenedor del Robotics Backend se esté ejecutando en tu terminal, sigue estos pasos para conectarte y ejecutar un ejercicio:

1. Acceder y Autenticarse:
* Ve a https://unibotics.org en tu navegador.
* Si eres un usuario nuevo, haz clic en REGISTER. Deberás completar los campos: Nombre (Name), Apellido (Surname), Nombre de usuario (Username), Email y Contraseña (Password).
* Si ya tienes una cuenta, haz clic en LOG IN e introduce tu Nombre de usuario (Username) y Contraseña (Password).

2. Navegar a la Academia:
* Después de iniciar sesión, serás dirigido al panel de aplicaciones (unibotics.org/apps).
* Haz clic en Robotics Academy (Learn robotics facing practical challenges).

3. Seleccionar un Ejercicio:
* Dentro de la Academia, selecciona un curso disponible, como el Free Course.
* Elige uno de los ejercicios prácticos disponibles. El PDF muestra la selección del ejercicio Follow Line.

4. Conectar y Ejecutar:
* Se abrirá la interfaz del ejercicio, mostrando el editor de código Python a la izquierda y los monitores de simulación a la derecha.
* Verás un mensaje que dice "Click Play to connect to the Robotics Backend".
* Haz clic en el botón Connect (o el botón de Play).
* La interfaz web se conectará a tu contenedor Docker local. Tu terminal de Docker mostrará un mensaje confirmando la conexión, como "client connected".

5. Probar el Código:
* Una vez conectado, la simulación se iniciará.
* Puedes ver la simulación 3D del robot y monitores adicionales como el Im Monitor que muestra la visión de la cámara del robot.
* El código Python se estará ejecutando, y la consola en la interfaz web mostrará cualquier salida.


## 4. Ejemplos y Rendimiento

### Capturas de Pantalla

A continuación, se muestra el funcionamiento del algoritmo. La imagen superior es la vista de la cámara, y la inferior es la imagen procesada (máscara) que usa el controlador para calcular el error.

| Vista de la Cámara | Imagen Procesada (Máscara) |
| :---: | :---: |
| ![Vista de la cámara en recta](./images/camera_straight.png) | ![Máscara procesada en recta](./images/mask_straight.png) |
| *Coche en una recta* | *Detección de la línea en la recta* |
| ![Vista de la cámara en curva](./images/camera_curve.png) | ![Máscara procesada en curva](./images/mask_curve.png) |
| *Coche tomando una curva* | *Detección de la línea en la curva* |


### Rendimiento
* **Término Proporcional (P):** 109,96
* **Término Integral (I):** 	96,67	
* **Término Derivativo (D):** 126,41

# 5. Conclusión
Este proyecto demuestra cómo el uso de visión por computadora combinada con control PID permite desarrollar un sistema de navegación autónomo eficaz y estable.
El ajuste fino de las ganancias del controlador es clave para lograr un equilibrio entre precisión, suavidad y velocidad de respuesta.
