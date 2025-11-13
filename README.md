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

La ecuación general del control es:
La velocidad angular (`w`) se calcula como: `w = (Kp * error) + (Kd * derivada_del_error)`.
La velocidad lineal (`v`) se mantiene constante para simplificar el control.

#### Variantes y Ajuste
Durante el desarrollo se probaron distintas configuraciones:
* **Controlador P:** * La forma más básica. Corrige el error directamente en proporción a su magnitud. Puede ser inestable o lento si Kp no se ajusta correctamente.
* **Controlador PD:** * Añade la derivada del error para amortiguar las oscilaciones y anticipar curvas. Es el enfoque principal utilizado en esta solución.
* **Controlador PID completo:** * Incluye el término integral para eliminar errores persistentes. Resulta útil en circuitos con trayectorias largas o curvas suaves.
El ajuste de las ganancias (`Kp`, `Ki`, `Kd`) se realizó experimentalmente hasta obtener una respuesta estable y fluida.

| Parámetro | Valor |
| :-- | --: |
| **Kp** | 0.01 |
| **Ki** | 0.00005 |
| **Kd** | 0.0005 |


## 3. Instrucciones de Ejecución

### Prerrequisitos

* [cite_start]Tener Robotics Academy instalado y configurado (ver [guía de usuario](https://jderobot.github.io/RoboticsAcademy/user_guide/#installation))[cite: 8, 9].
* Python 3.x.
* [cite_start]Bibliotecas de Python: `opencv-python`, `numpy` (necesarias para el procesamiento de imágenes [cite: 163, 164]).

### Instalación

1.  Clona este repositorio en tu máquina local:
    ```bash
    git clone [https://github.com/TU_USUARIO/TU_REPOSITORIO.git](https://github.com/TU_USUARIO/TU_REPOSITORIO.git)
    cd TU_REPOSITORIO
    ```

2.  Instala las dependencias de Python (si aún no las tienes):
    ```bash
    pip install opencv-python numpy
    ```

### Lanzamiento

1.  En una terminal, lanza el ejercicio `follow_line` desde la aplicación de Robotics Academy.
2.  Abre una segunda terminal y navega al directorio donde clonaste este repositorio.
3.  Ejecuta el script de la solución:
    ```bash
    python follow_line_solution.py
    ```

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
