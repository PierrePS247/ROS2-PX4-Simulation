
# Simulación ROS2-PX4

## Descripción
Este repositorio explica cómo usar ROS2 y PX4 para controlar la velocidad de un UAV simulado. Incluyendo comandos que nos sirvieron para solucionar errores tanto en la instalación como en la simulación.

### Requisitos
* ROS2 Humble
* PX4 Autopilot 1.15
* Micro XRCE-DDS Agent
* px4_msgs
* Ubuntu 22.04
* Python 3.10


## Pasos de configuración

### Instalar PX4 Autopilot
Ejecuta este código 
```
git clone https://github.com/PX4/PX4-Autopilot.git --recursive -b release/1.15
```

Ejecute este script en un shell bash para instalar todo

```
bash ./PX4-Autopilot/Tools/setup/ubuntu.sh
```

Ahora deberá reiniciar su computadora antes de continuar.


### Instalar ROS2 Humble
Para instalar ROS2 Humble siga los pasos de [aquí](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html)

### Instalar dependencias

Instale las dependencias de Python como se menciona en el [PX4 Docs](https://docs.px4.io/main/en/ros/ros2_comm.html#install-ros-2) con este código.

```
pip3 install --user -U empy pyros-genmsg setuptools
```

También descubrimos que sin estos paquetes instalados, Gazebo tiene problemas para cargar.

```
pip3 install kconfiglib
pip install --user jsonschema
pip install --user jinja2
```

#### (Posible error ! )
Si se presenta un error relacionado con pip3, ejecute los siguinetes comandos:

```
pip3 uninstall empy -y
pip3 install empy==3.3.4
```


### Construir Micro DDS
Como se menciona en el [PX4 Docs](https://docs.px4.io/main/en/ros/ros2_comm.html#setup-micro-xrce-dds-agent-client) Ejecute este código para crear MicroDDS en su máquina

```
git clone https://github.com/eProsima/Micro-XRCE-DDS-Agent.git
cd Micro-XRCE-DDS-Agent
mkdir build
cd build
cmake ..
make
sudo make install
sudo ldconfig /usr/local/lib/
```

### Configurar el espacio de trabajo
Este repositorio git está destinado a ser un paquete ROS2 que se clona en un espacio de trabajo ROS2.

Vamos a crear un espacio de trabajo en nuestro directorio de inicio y luego clonar en este repositorio y también en el repositorio px4_msgs.

Para obtener más información sobre la creación de espacios de trabajo, consulte [aquí](https://docs.ros.org/en/humble/Tutorials/Workspace/Creating-A-Workspace.html)

Ejecute este código para crear un espacio de trabajo en su directorio de inicio

```
mkdir -p ~/ros2_px4_offboard_example_ws/src
cd ~/ros2_px4_offboard_example_ws/src
```

*ros2_px4_offboard_example_ws* es solo el nombre que elegí para el espacio de trabajo. Puedes nombrarlo como quieras. Sin embargo, después de ejecutar *colcon build*, podrías tener problemas para cambiar el nombre de tu espacio de trabajo, así que elige con cuidado.

Ahora estamos en el directorio src de nuestro espacio de trabajo. Aquí se almacenan los paquetes de ROS2 y donde clonaremos nuestros dos repositorios.

### Clonar en paquetes
Primero necesitaremos el paquete px4_msgs. Nuestros nodos ROS2 se basarán en las definiciones de mensajes de este paquete para comunicarse con PX4. Lea [aquí](https://docs.px4.io/main/en/ros/ros2_comm.html#overview:~:text=ROS%202%20applications,different%20PX4%20releases) para más información.

Asegúrate de estar en el directorio src de tu espacio de trabajo y luego ejecuta este código para clonar en el repositorio px4_msgs

```
git clone https://github.com/PX4/px4_msgs.git -b release/1.15
```

Asegúrate de que sigues en el directorio src de tu espacio de trabajo. Ejecuta este código para clonar en nuestro paquete de ejemplo.

```
git clone https://github.com/ARK-Electronics/ROS2_PX4_Offboard_Example.git
```

Ejecute este código para clonar el repositorio



### Construyendo el espacio de trabajo
Los dos paquetes en este espacio de trabajo son px4_msgs y px4_offboard. px4_offboard es un paquete de ROS2 que contiene el código del nodo de control externo que implementaremos. Se encuentra dentro del directorio ROS2_PX4_Offboard_Example.

Antes de compilar estos dos paquetes, necesitamos ubicar la instalación de ROS2. Ejecute este código para ello.

```
source /opt/ros/humble/setup.bash
```

Esto deberá ejecutarse en cada terminal que desee ejecutar comandos de ROS2. Una forma sencilla de solucionarlo es añadir este comando a tu archivo .bashrc. Esto lo ejecutará cada vez que abras una nueva ventana de terminal.

Para crear estos dos paquetes, debe estar en el directorio del espacio de trabajo, no en src, ejecute este código para cambiar el directorio de src a un paso atrás, es decir, la raíz de su espacio de trabajo y crear los paquetes.

```
cd ..
colcon build
```
Recibirás algunas advertencias sobre setup.py pero mientras no haya errores, debería estar todo bien.


Después de ejecutar esto, no deberíamos tener que compilar px4_msgs nunca más. Sin embargo, sí tendremos que compilar px4_offboard cada vez que modifiquemos el código. Para ello y ahorrar tiempo, podemos ejecutar

```
colcon build --packages-select px4_offboard
```

#### (Posible error ! )
Si se presenta un error relacionado con colcon build, ejecute los siguinetes comandos:

```
rm -rf build install log
colcon build --symlink-install
```

#### (Posible error ! )
Si se presentan errores relacioandos a los paquetes: 'package not processed'
Ejecute:

```
pip3 install --upgrade packaging
pip3 uninstall setuptools packaging
pip3 install setuptools packaging
```


Si intentaras ejecutar nuestro código ahora, no funcionaría. Esto se debe a que necesitamos obtener el código de nuestro espacio de trabajo actual. Esto siempre se hace después de una compilación. Para ello, asegúrate de estar en el directorio src y luego ejecuta este código.

```
source install/setup.bash
```

Lo ejecutaremos cada vez que compilemos. También deberá ejecutarse en cada terminal donde queramos ejecutar comandos de ROS2.


### Ejecutando el código
Este ejemplo se ha diseñado para ejecutarse desde un archivo de inicio que iniciará todos los nodos necesarios. Este archivo ejecutará un script de Python que usa la terminal de Gnome para abrir una nueva ventana de terminal para MicroDDS y Gazebo.

Ejecute este código para iniciar el ejemplo

```
ros2 launch px4_offboard offboard_velocity_control.launch.py
```

#### (Posible error ! )
Si se presentan errores relacioandos al gazebo bridge: 'gz_bridge failed to start and spawn model'
Ejecute:

```
sudo apt install ros-humble-gz-ros2-bridge
```

This will run numerous things. In no particular order, it will run:

* process.py en una nueva ventana
   * MicroDDS en una nueva ventana de terminal
   * Gazebo se abrirá en una segunda pestaña de la misma ventana de terminal
      * La interfaz gráfica de Gazebo se abrirá en su propia ventana
* control.py en una nueva ventana
   * Envía comandos ROS2 Teleop al tema /offboard_velocity_cmd según la entrada del teclado
* RVIZ se abrirá en una nueva ventana
* velocity_control.py se ejecuta como un nodo propio y es el nodo principal de este ejemplo

Una vez que todo esté en funcionamiento, debería poder centrarse en la ventana de terminal control.py, armar y despegar. Los controles imitan los del transmisor RC Modo 2, con WASD como joystick izquierdo y las teclas de flecha como joystick derecho. Los controles son los siguientes:
* W: Arriba
* S: Abajo
* A: Guiñada a la izquierda
* D: Guiñada a la derecha
* Flecha arriba: Inclinación hacia adelante
* Flecha abajo: Inclinación hacia atrás
* Flecha izquierda: Giro a la izquierda
* Flecha derecha: Giro a la derecha
* Espacio: Activar/Desactivar

Al pulsar *Espacio* se armará el dron. Espere un momento y despegará y pasará al modo fuera de borda. Ahora puede controlarlo con las teclas anteriores. Si aterriza el dron, se desarmará y, para despegar de nuevo, deberá activar y desactivar el interruptor del brazo con la barra espaciadora.

Con los controles, pulsa *W* para enviar una señal de velocidad vertical y despegar. Una vez en el aire, puedes controlarlo como prefieras.

## Cerrar la simulación *IMPORTANTE*
Al cerrar la simulación, es muy tentador cerrar las ventanas de terminal. Sin embargo, esto dejará a Gazebo ejecutándose en segundo plano, lo que podría causar problemas al ejecutarlo en el futuro. Para finalizar correctamente la simulación de Gazebo, vaya a la ventana de terminal y presione *Ctrl+C*. Esto cerrará Gazebo y todos sus procesos secundarios. Después, puede cerrar las demás ventanas de terminal.
 

 ## Explicación de processes.py
 Este código ejecuta cada conjunto de comandos bash en una nueva pestaña de la terminal de Gnome. Se asume que la instalación de PX4 es accesible desde el directorio raíz y que utiliza la simulación gz_x500. No existe una implementación actual para cambiar estos comandos al ejecutar el archivo de inicio; sin embargo, puede modificar la cadena de comandos en el archivo process.py para ajustar estos valores a sus necesidades.

 Si la línea 17 de process.py no estuviera comentada
```
17     # "cd ~/QGroundControl && ./QGroundControl.AppImage"
```
Luego, QGroundControl se ejecutará en una nueva pestaña de la ventana del terminal y se abrirá la interfaz gráfica de usuario de QGroundControl. Esto está comentado por defecto, ya que no es necesario para la ejecución de la simulación, pero es útil para la depuración y es un ejemplo sencillo que muestra cómo agregar otro comando al archivo de inicio.

## Problemas conocidos
Si el vehículo no se arma al presionar Enter, verifique que el parámetro NAV_DLL_ACT esté configurado en 0. Es posible que necesite descargar QGroundControl y deshabilitar este parámetro si desea ejecutar esta demostración sin necesidad de tener QGC abierto.

