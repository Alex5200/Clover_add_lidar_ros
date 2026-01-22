

###  1. Убедитесь, что у вас установлен стек Clover и Gazebo
Официальная среда симуляции Clover основана на **Gazebo** и предоставляет `.launch`-файлы для запуска дрона в симуляторе .

Если вы ещё не установили:
```bash
# Для ROS Noetic (Ubuntu 20.04)
sudo apt install ros-noetic-clover-simulation

# Или клонируйте репозиторий
git clone https://github.com/CopterExpress/clover_simulation.git ~/catkin_ws/src/clover_simulation
```

Модель дрона Clover 4 описана в URDF-файлах и совместима с Gazebo .

---

###  2. Добавьте LiDAR в URDF-модель дрона

Откройте файл описания дрона, например:  
`~/catkin_ws/src/clover/clover_description/urdf/clover4.xacro`

Добавьте в него **LiDAR-сенсор** (пример для RPLiDAR A2 или аналога):

```xml
<!-- LiDAR -->
<link name="laser_link">
  <collision>
    <origin xyz="0 0 0" rpy="0 0 0"/>
    <geometry>
      <cylinder radius="0.03" length="0.05"/>
    </geometry>
  </collision>
  <visual>
    <origin xyz="0 0 0" rpy="0 0 0"/>
    <geometry>
      <cylinder radius="0.03" length="0.05"/>
    </geometry>
    <material name="black"/>
  </visual>
  <inertial>
    <mass value="0.1"/>
    <inertia ixx="1e-4" ixy="0" ixz="0" iyy="1e-4" iyz="0" izz="1e-4"/>
  </inertial>
</link>

<joint name="laser_joint" type="fixed">
  <parent link="base_link"/>
  <child link="laser_link"/>
  <origin xyz="0 0 -0.1" rpy="0 0 0"/> <!-- под дроном -->
</joint>

<!-- Gazebo plugin for LiDAR -->
<gazebo reference="laser_link">
  <sensor type="ray" name="laser">
    <pose>0 0 0 0 0 0</pose>
    <visualize>true</visualize>
    <update_rate>10</update_rate>
    <ray>
      <scan>
        <horizontal>
          <samples>360</samples>
          <resolution>1</resolution>
          <min_angle>-3.14159</min_angle>
          <max_angle>3.14159</max_angle>
        </horizontal>
      </scan>
      <range>
        <min>0.1</min>
        <max>12.0</max>
        <resolution>0.01</resolution>
      </range>
    </ray>
    <plugin name="gazebo_ros_laser" filename="libgazebo_ros_laser.so">
      <topicName>/scan</topicName>
      <frameName>laser_link</frameName>
    </plugin>
  </sensor>
</gazebo>
```

> Это стандартный способ добавления 2D LiDAR в Gazebo через `ray`-сенсор и ROS-плагин .

---

###  3. Запустите симуляцию с LiDAR

Создайте или измените launch-файл, например `clover_lidar.launch`:

```xml
<launch>
  <include file="$(find clover_simulation)/launch/gazebo.launch">
    <arg name="model" value="$(find clover_description)/urdf/clover4_lidar.xacro"/>
  </include>

  <!-- Запуск RViz для визуализации -->
  <node name="rviz" pkg="rviz" type="rviz" args="-d $(find clover_simulation)/config/lidar.rviz"/>
</launch>
```

Запустите:
```bash
roslaunch your_pkg clover_lidar.launch
```

В Gazebo вы увидите дрон с LiDAR (визуализация лучей включена), а в RViz — данные `/scan`.

---

###  4. Проверка работы
В терминале выполните:
```bash
rostopic echo /scan
```
Если приходят данные — всё работает.

Теперь вы можете:
- Запускать **SLAM** (`gmapping`, `cartographer`)
- Строить карты
- Тестировать алгоритмы избегания препятствий

---

