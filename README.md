# PincherArmInterface

Interfaz gráfica en PyQt5 para controlar el brazo robótico **PhantomX Pincher** (4 GDL + gripper) desde ROS Noetic.

Permite:

- Mover cada articulación con sliders (publica en `<joint>/command`).
- Ver la **cinemática directa** en tiempo real leyendo `/joint_states`.
- Calcular la **cinemática inversa** a partir de una posición X/Y/Z y ejecutar la trayectoria.
- Enviar el brazo a *Home* y relajar los servos.

## Requisitos

- Ubuntu 20.04 + ROS Noetic
- Python 3.8
- Brazo PhantomX Pincher con controladora ArbotiX (opcional: hay modo simulación)

## Instalación

```bash
# 1. Clonar el proyecto
git clone <URL-DE-TU-REPO> ~/PincherArmInterface
cd ~/PincherArmInterface

# 2. Dependencias de ROS (MoveIt, tf2, xacro, etc.)
sudo rosdep init      # solo la primera vez en la máquina
rosdep update
rosdep install --from-paths src --ignore-src -r -y

# 3. Dependencias de Python
sudo apt install python3-pyqt5
pip3 install "roboticstoolbox-python==1.1.0" spatialmath-python

# 4. Compilar
source /opt/ros/noetic/setup.bash
cd src && catkin_init_workspace && cd ..   # crea src/CMakeLists.txt
catkin_make
source devel/setup.bash
chmod +x src/test_movements/scripts/*.py
```

> La versión de `roboticstoolbox-python` está fijada a propósito: desde la 1.4 cambió la firma de `ikine_LM` y la cinemática inversa deja de funcionar.

> El paso 2 instala también el driver `arbotix` desde apt. Si `rosdep` no resolviera `arbotix_python` o `arbotix_controllers`, clónalo en el workspace y repite el paso:
> `git clone -b noetic-devel https://github.com/vanadiumlabs/arbotix_ros.git src/arbotix_ros`

## Uso

Con el brazo conectado por USB (requiere `sudo usermod -aG dialout $USER` la primera vez):

```bash
./iniciar_proyecto.sh
```

O paso a paso, una terminal por comando (con `source devel/setup.bash` en cada una):

```bash
roscore
roslaunch pincher_arm_bringup arm.launch
roslaunch pincher_arm_moveit_config pincher_arm_moveit.launch sim:=false
rosrun test_movements Interface.py
```

### Sin el brazo físico (simulación)

Solo la terminal del *bringup* lleva `sim:=true`; la de MoveIt se deja en `sim:=false`. Así el driver ArbotiX se simula (crea los tópicos de comando, publica `/joint_states` y el `robot_state_publisher`) y MoveIt solo lo visualiza, sin duplicar nodos:

```bash
roscore
roslaunch pincher_arm_bringup arm.launch sim:=true
roslaunch pincher_arm_moveit_config pincher_arm_moveit.launch sim:=false
rosrun test_movements Interface.py
```

Los sliders **sí** mueven el modelo en RViz. Si no ves el brazo en RViz, pon el *Fixed Frame* en `arm_base_link` y en el display *RobotModel* marca *Collision Enabled*. El botón de liberar servos no tiene efecto visible en simulación, porque no hay torque real que soltar.

## Estructura

| Ruta | Contenido |
|---|---|
| `src/test_movements/scripts/Interface.py` | Nodo principal: GUI + lógica de ROS |
| `src/test_movements/scripts/SimpleInterfaz.py` | Ventana generada con `pyuic5` desde `SimpleInterfaz.ui` |
| `src/pincher_arm/` | URDF, driver y configuración de MoveIt ([fictionlab](https://github.com/fictionlab/pincher_arm), BSD 3-Clause) |
| `iniciar_proyecto.sh` | Levanta roscore + bringup + MoveIt + interfaz |

Los scripts de prueba (lectura de joints, envío de trayectorias, cinemática inversa offline) están en `src/test_movements/scripts/experimental/` y no forman parte de la aplicación.
