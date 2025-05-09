# Práctica 3 - Modelado de Robot

## Silvia Calvo Cabello

## 0. Introducción

Esta práctica está orientada al modelado de un robot en Blender y su integración posterior en ROS2. La práctica se divide en dos partes:

- **Parte A**: Crear un paquete en ROS2 utilizando Xacro para describir el robot, siguiendo las normativas REP-103 y REP-105.
- **Parte B**: Utilizar MoveIt y `ros2_control` para controlar el robot y simular la acción de recoger una caja en un entorno simulado.

Dentro del proyecto, se incluyen varios paquetes de ROS2 basados en la distribución `ros2-jazzy`.

### Carpeta `data`

Esta carpeta contiene recursos relevantes para el proyecto:

- [`Tree links`](data/frames_2025-05-05_11.22.41.pdf): Representación jerárquica (tree) de los links del brazo robótico.
- [`Rviz`](data/Rvizz_captura.png): Captura de pantalla de RViz mostrando el movimiento de las articulaciones del brazo.
- [`Video ParteA`](data/ParteA.mp4): Video de RViz moviendo las distintas articulaciones de las ruedas y del brazo.
- [`Video ParteB sin controladores`](data/ParteB.mp4): Video de RViz y Gazebo mostrando el mundo y las imágenes de las cámaras.
- [`Video ParteB con controladores`](data/Sacara3.mp4): Video de RViz y Gazebo mostrando el mundo moviendo el brazo en un intento de coger la caja ( no va las ruedas).
- [`Link_rosbag`](data/link_rosbag.txt): Link al rosbag con joint_state.
- [`Imagen_caja`](data/caja.png): Foto cogiendo la caja.




### Paquetes del proyecto

- **rover_description**: Este paquete contiene la descripción del robot, incluyendo sus partes físicas definidas en Xacro, así como los lanzadores necesarios para cargar el robot en Gazebo.
  
- **urjc-excavation-world**: Este paquete define el entorno en el que se probará el robot, con un mundo simulado denominado `urjc_excavation_msr`. Descargar del git del profesor

- **rover_model_moveit_config**: Este paquete contiene la configuracion de moveit mediante moveit_setup_assistanat.

### Lanzar el proyecto

Una vez hayas realizado el `colcon build` de ambos paquetes, puedes lanzar el proyecto con el siguiente comando en la terminal: (Esto es hasta gazebo, debido a que la parte B no está entera, se explica más adelante)

```bash
ros2 launch robot_description rover_gazebo.launch.py world_name:=urjc_excavation_msr

ros2 launch rover_model_moveit_config move_group.launch.py

ros2 launch rover_model_moveit_config move_group.launch.py
```
---

## Parte A - Generación del URDF con Xacro

### Enunciado

Al final de esta primera fase se debe tener un paquete ROS2 en el que el robot esté totalmente configurado y sea compatible con REP-103 y REP-105. El robot debe incluir:

- Un sensor IMU en el centro del robot (o en la base si la geometría no lo permite).
- Una cámara frontal en la parte delantera.
- Una cámara unida al eslabón final del manipulador.

---

### ¿Qué es Xacro?

Xacro es una herramienta de ROS que permite generar archivos URDF de manera modular, usando macros. Esto facilita la reutilización de componentes como ruedas, sensores, enlaces, etc., al dividirlos en distintos archivos reutilizables.

---

### Ejemplo de macro en un archivo `.xacro`

```xml
<?xml version="1.0"?>
<!-- created with Phobos 1.0.1 "Capricious Choutengan" -->
<robot name="robot" xmlns:xacro="http://wiki.ros.org/xacro">

  <xacro:macro name="sensor_camera_front" params="parent" >
    <joint name="Camara_frontal_link_joint" type="fixed">
      <origin rpy="-0.00000 1.57080 0.00000" xyz="1.55000 0.00000 0.40000"/>
      <parent link="${parent}"/> 
      <child link="Camara_frontal_link"/>
    </joint>
  </xacro:macro>

</robot>
```

Ejemplo de inclusión y uso del macro en el archivo principal

```xml
?xml version="1.0"?>
<!-- created with Phobos 1.0.1 "Capricious Choutengan" -->
<robot name="robot" xmlns:xacro="http://wiki.ros.org/xacro">
    <!-- includes -->
    <xacro:include filename="$(find robot_description)/urdf/sensors/camera.urdf.xacro"/>
    <xacro:sensor_camera_front parent="Base_link"/>
```
---

### Posibles problemas:

Si al compilar o cargar el robot recibes un error indicando que no se encuentran los `meshes` generados en Blender, las causas más comunes pueden ser las siguientes:

1. No has añadido la carpeta `meshes` en el archivo `CMakeLists.txt`.
2. La ruta del archivo del mesh no está correctamente especificada en el archivo URDF.

```xml
    <link name="Rueda_link">
      <collision name="Rueda_collision.000">
        <origin rpy="-0.00000 0.00000 0.00000" xyz="0.00000 -0.00000 0.25000"/>
        <geometry>
          <mesh filename="file://$(find robot_description)/meshes/dae/Torus.001.dae" scale="0.30000 0.30000 0.50000"/>
        </geometry>
      </collision>
      <inertial>
        <inertia ixx="0.36066" ixy="0.00000" ixz="0.00000" iyy="0.65979" iyz="0.00000" izz="0.36066"/>
        <origin rpy="0.00000 0.00000 0.00000" xyz="0.00000 -0.00000 0.25000"/>
        <mass value="10.00000"/>
      </inertial>
      <visual name="Rueda">
        <origin rpy="0.00000 0.00000 0.00000" xyz="0.00000 -0.00000 0.25000"/>
        <material name="Ruedas"/>
        <geometry>
          <mesh filename="file://$(find robot_description)/meshes/dae/Torus.001.dae" scale="0.30000 0.30000 0.50000"/>
        </geometry>
      </visual>
    </link>
```


## Parte B - Integración y estudio de dinámicas en Gazebo y ROS2

### Enunciado

En esta fase se realiza el análisis de coste del mecanismo *pick and place* (equivalente a la Fase 3 de la práctica 2) utilizando el simulador Gazebo y el framework MoveIt2. Para ello, se empleará el robot desarrollado en la Parte A de esta práctica.

A partir de los archivos generados en la Parte A, podemos obtener las posiciones de los componentes del brazo robótico y la pinza, lo cual nos permitirá realizar el agarre de la caja. Para manipular estas posiciones, utilizamos los *sliders* del "joint state" en RViz, así como la simulación proporcionada.

El escenario de simulación está basado en el mundo `urjc_excavation_msr`. El robot debe ser posicionado en las coordenadas `(0, 0)` del mundo para poder acercarse a la caja verde ubicada en las coordenadas `(5, 0)`. Una vez en posición, el robot debe agarrar la caja y depositarla en su contenedor.

---

### Sesores en gazebo
#### Camaras
Para hacer que las cámaras sean funcionales y se puedan visualizar correctamente en Gazebo, es necesario añadir el siguiente bloque de código al archivo Xacro de las cámaras. Asegúrate de que los nombres coincidan exactamente con las definiciones creadas en la Parte A.

```xml
  <gazebo reference="Camara_frontal_link">
    <sensor name="sensor_camera_front" type="camera">
      <visualize>true</visualize>
      <update_rate>30</update_rate>
      <topic>/front_camera/image</topic>
      <camera>
        <horizontal_fov>1.5708</horizontal_fov>  
        <image>
          <width>640</width>
          <height>480</height>
          <format>R8G8B8</format>
        </image>
        <clip>
          <near>0.10</near>
          <far>15.0</far>
        </clip>
        <noise>
          <type>gaussian</type>
          <mean>0.0</mean>
          <stddev>0.007</stddev>
        </noise>
        <optical_frame_id>Camara_frontal_link</optical_frame_id>
      </camera>
    </sensor>
  </gazebo>
```

#### IMU
Para añadir el sensor IMU al modelo del robot, se debe incluir el siguiente bloque de código en el archivo Xacro. De nuevo, asegúrate de que el nombre coincida con la definición creada previamente en la Parte A.

```xml
  <gazebo reference="IMU_link">
    <sensor name="sensor_IMU" type="imu">
      <always_on>1</always_on>
      <update_rate>30</update_rate>
      <topic>"imu_data"</topic>
    </sensor>
  </gazebo>

```

Además, debes añadir el archivo robot_bridge.yaml, descargado del aula virtual, a la carpeta config. Asegúrate de que el nombre del tema (ros_topic_name) coincida con el especificado en el archivo Xacro. El archivo robot_bridge.yaml debe tener el siguiente formato:

```xml

- ros_topic_name: "imu_data"
  gz_topic_name: "/imu_data"
  ros_type_name: "sensor_msgs/msg/Imu"
  gz_type_name: "gz.msgs.Pose_V"
  direction: GZ_TO_ROS
  lazy: false
```

#### Posibles problemas:

Si al lanzar las camaras no estan apuntando en la direccion correcta se debe a que faltaria un link de conexion, se puede crear uno vacio, de esta manera


```xml
  <link name="optical_frame">
  </link>
```

---

### Moveit y gazebo

Para esta práctica usaremos el mundo en gazebo encontrado en el paquete de este repositorio. 

Para moveit usamos el moveit_setup_assistant el cual solo funcionan en algunos ordenadores debido a una dependencia de librerias y paquetes fallidos.

Tras seguir todos los pasos del video para crear el paquete de moveit y los controladores ya se puede lanzar todo. 

#### Problemas 

1. El nombre del paquete con el robot no se puede llamar `robot_description`.
2. El nombre del robot en los xacros no se puede llamar `robot`.
3. Si no funciona el teleoperado se puede hacer publicando directamente en los topics con:
```bash

ros2 topic pub /rover_base_control/cmd_vel geometry_msgs/msg/TwistStamped "{header: {stamp: {sec: 0, nanosec: 0}, frame_id: 'base_link'}, twist: {linear: {x: 10.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}}"

```


