# PRACTICA 3 MODELADO 
## Silvia Calvo Cabello
## 0 Introduccion
Esta practita trata de tras haber modelado un robot en blender.  Se divide en dos partes, la primera: Realizar un paquete para ros2 mediante xacro. Y la parte B usar moveit y ros2 controll para mover al robot y hacer que recoja una caja en un escenadrio simualdo. Siguiendo las normas 

## Parte A
eNUCIADO: Al final estas dos primeras sesiones se debería tener un paquete de ROS, en el cual el robot se
encuentre totalmente configurado y compatible con REP-103 y REP-105.
El robot además de estar plenamente funcional debe contener:
un sensor IMU ubicado en el centro del robot, o dentro de la base del mismo si su geometría no
lo permite.
una cámara frontal posicionada en la parte delantera del robot
una cámara que vaya unida al eslabón final del manipulador robot

Xacro es una herramineta para generear como macros de archivos urdf. Esto se hace de la separarlos en distinats partes, ruedas, base...
HAy que tenr lo siugiente en la cabecera
'''bash
<?xml version="1.0"?>
<!-- created with Phobos 1.0.1 "Capricious Choutengan" -->
<robot name="robot" xmlns:xacro="http://wiki.ros.org/xacro">

  <xacro:macro name="sensor_camera_front" params="parent" >
   <joint name="Camara_frontal_link_joint" type="fixed">
    <origin rpy="-0.00000 1.57080 0.00000" xyz="1.55000 0.00000 0.40000"/>
    <parent link="${parent}"/> 
    <child link="Camara_frontal_link"/>

'''

Y a la hora de llamarlos en un  archivo comun es de la siguiente manera:

'''bash
?xml version="1.0"?>
<!-- created with Phobos 1.0.1 "Capricious Choutengan" -->
<robot name="robot" xmlns:xacro="http://wiki.ros.org/xacro">
    <!-- includes -->
    <xacro:include filename="$(find robot_description)/urdf/sensors/camera.urdf.xacro"/>
    <xacro:sensor_camera_front parent="Base_link"/>
'''


