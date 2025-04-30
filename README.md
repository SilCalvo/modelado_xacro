# Práctica 3 - Modelado de Robot

## Silvia Calvo Cabello

## 0. Introducción

Esta práctica está orientada al modelado de un robot en Blender y su integración posterior en ROS2. La práctica se divide en dos partes:

- **Parte A**: Crear un paquete en ROS2 utilizando Xacro para describir el robot, siguiendo las normativas REP-103 y REP-105.
- **Parte B**: Utilizar MoveIt y `ros2_control` para controlar el robot y simular la acción de recoger una caja en un entorno simulado.

Dentro del proyecto, se incluyen varios paquetes de ROS2 basados en la distribución `ros2-foxy`.

### Paquetes del proyecto

- **robot_description**: Este paquete contiene la descripción del robot, incluyendo sus partes físicas definidas en Xacro, así como los lanzadores necesarios para cargar el robot en Gazebo.
  
- **urjc-excavation-world**: Este paquete define el entorno en el que se probará el robot, con un mundo simulado denominado `urjc_excavation_msr`.

### Lanzar el proyecto

Una vez hayas realizado el `colcon build` de ambos paquetes, puedes lanzar el proyecto con el siguiente comando en la terminal:

```bash
ros2 launch robot_description robot_gazebo.launch.py world_name:=urjc_excavation_msr
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


