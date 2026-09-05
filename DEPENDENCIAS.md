# Dependencias y entorno de ejecución — Control Cinemático Robot PhantomX v1.0

Repositorio del software: https://github.com/JuanHoKKeR/PincherArmInterface
(espacio de trabajo catkin completo; el código de autoría propia está en `src/test_movements/` y `iniciar_proyecto.sh`).

## Sistema base

| Componente | Versión | Fuente |
|---|---|---|
| Sistema operativo | Ubuntu 20.04 LTS (nativo o WSL 2 sobre Windows 10/11) | https://ubuntu.com |
| ROS | Noetic Ninjemys (`ros-noetic-desktop-full`) | https://wiki.ros.org/noetic |
| Python | 3.8 (incluido en Ubuntu 20.04) | — |
| USBIPD-WIN (solo Windows) | última versión estable | https://github.com/dorssel/usbipd-win |

## Paquetes ROS externos

Las versiones exactas quedan fijadas como submódulos git del repositorio (`.gitmodules`):

| Paquete | Repositorio | Commit fijado | Licencia | Uso en el software |
|---|---|---|---|---|
| `pincher_arm` (`pincher_arm_bringup`, `pincher_arm_moveit_config`, `pincher_arm_description`, `pincher_arm_ikfast_plugin`) | https://github.com/fictionlab/pincher_arm | `7b2b744a47e2893298a361301e4f1a12717ffef0` | BSD | `arm.launch` (driver ArbotiX + controlador de trayectoria) y `pincher_arm_moveit.launch` (RVIZ / MoveIt) |
| `arbotix_ros` (`arbotix_python`, `arbotix_msgs`, `arbotix_controllers`) | https://github.com/vanadiumlabs/arbotix_ros (rama `noetic-devel`) o paquete binario `ros-noetic-arbotix` | dependencia de `pincher_arm_bringup`, resuelta con `rosdep` | BSD | Driver de la tarjeta ArbotiX-M: tópicos `<joint>/command` y `/joint_states`, servicio `/servos/relax_all`, mensaje `arbotix_msgs/Relax` |

Paquetes ROS del sistema (apt): `ros-noetic-moveit`, `ros-noetic-control-msgs`, `ros-noetic-trajectory-msgs`,
`ros-noetic-actionlib`, `ros-noetic-sensor-msgs`, `ros-noetic-std-msgs`. Se resuelven con
`rosdep install --from-paths src --ignore-src -r -y`.

## Librerías Python

Versiones fijadas como submódulos en `external/` del repositorio:

| Librería | Repositorio | Commit fijado | Licencia |
|---|---|---|---|
| roboticstoolbox-python | https://github.com/petercorke/robotics-toolbox-python | `687d345d9994718d972a4b071dc061a3920d1733` | MIT |
| spatialmath-python | https://github.com/petercorke/spatialmath-python | `470962c3a77fda1bfd0022fea92a9b4474b1de97` | MIT |
| spatialgeometry (dependencia de roboticstoolbox) | https://github.com/jhavl/spatialgeometry | `132c0adf67e1618d9e361757807e908deb4ada92` | MIT |
| swift (dependencia de roboticstoolbox) | https://github.com/jhavl/swift | `efa25fb570aceed6070658b24bfe7526b8644fc8` | MIT |

Instaladas vía pip (ver `requirements.txt`): PyQt5 5.15.10 (GPL v3), numpy (BSD).

## Firmware de la tarjeta ArbotiX-M

La tarjeta ArbotiX-M ejecuta el firmware estándar de ArbotiX compatible con `arbotix_python`
(sketch `ros` de la librería ArbotiX para Arduino, https://github.com/vanadiumlabs/arbotix). Este firmware
y la librería `ax12` (LGPL 2.1, Michael E. Ferguson) son de terceros y no forman parte de la obra registrada.

## Reproducción del entorno

```bash
sudo apt install ros-noetic-desktop-full ros-noetic-moveit
git clone --recurse-submodules https://github.com/JuanHoKKeR/PincherArmInterface.git ~/PincherArm_catkin_ws
cd ~/PincherArm_catkin_ws
pip3 install -r requirements.txt
pip3 install -e external/spatialmath-python -e external/spatialgeometry -e external/swift -e external/robotics-toolbox-python
rosdep install --from-paths src --ignore-src -r -y
catkin_make && source devel/setup.bash
chmod +x src/test_movements/scripts/*.py iniciar_proyecto.sh
./iniciar_proyecto.sh
```
