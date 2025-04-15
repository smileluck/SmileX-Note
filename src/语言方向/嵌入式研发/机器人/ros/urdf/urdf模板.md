- continuous（轮子类）

  ```xml
  <?xml version="1.0"?>
  <robot name="tea">
      <link name="base_link">
          <visual>
              <geometry>
                  <mesh filename="base.obj" />
              </geometry>
          </visual>
      </link>
      <link name="mobile_link">
          <visual>
              <geometry>
                  <mesh filename="wheel.obj" />
              </geometry>
          </visual>
      </link>
      <joint name="joint" type="continuous">
          <parent link="base_link" />
          <child link="mobile_link" />
          <origin xyz="-0.14 1.15 0" />
          <axis xyz="-0.35 1 0" />
      </joint>
  </robot>
  
  ```

  需要填写：

  - link visual geometry mesh filename

  - joint orgin xyz rpy
  - joint axis xyz

  可选：

  - link orgin xyz rpy

- revolute（茶壶，首饰盒）

  ```xml
  <?xml version="1.0"?>
  <robot name="tea">
      <link name="base_link">
          <visual>
              <origin xyz="0 0 0.4" rpy="0 0.6 0"/>
              <geometry>
                  <mesh filename="box1.obj" />
              </geometry>
          </visual>
      </link>
      <link name="mobile_link">
          <visual>
              <origin xyz="0.7 -0.1 0.38" rpy="0 0.6 0"/>>
              <geometry>
                  <mesh filename="box2.obj" />
              </geometry>
          </visual>
      </link>
      <joint name="joint" type="revolute">
          <parent link="base_link" />
          <child link="mobile_link" />
          <origin xyz="0.1 0.1 0" rpy="0 0 0"/>
          <axis xyz="0 0 1" />
          <limit lower="0" upper="1.5" effort="0.0" velocity="0.0" />
      </joint>
  </robot>
  ```

  需要填写：

  - link visual geometry mesh filename

  - joint orgin xyz rpy
  - joint axis xyz
  - joint limit lower upper（绕轴的旋转角度 弧度制）

  可选：

  - link orgin xyz rpy

- prismatic（抽屉，注射器，抽拉类物体）

  ```xml
  <?xml version="1.0"?>
  <robot name="drawer">
      <link name="base_link">
          <visual>
              <geometry>
                  <mesh filename="drawer.obj" />
              </geometry>
          </visual>
      </link>
      <link name="mobile_link">
          <visual>
              <geometry>
                  <mesh filename="drawer2.obj" />
              </geometry>
          </visual>
      </link>
      <joint name="joint" type="prismatic">
          <parent link="base_link" />
          <child link="mobile_link" />
          <origin xyz="-0.14 1.15 0" />
          <axis xyz="-0.35 1 0" />
          <limit lower="-0.8175720572471619" upper="0.81" effort="0.0" velocity="0.0" />
      </joint>
  </robot>
  
  ```

  需要填写：

  - link visual geometry mesh filename

  - joint orgin xyz rpy
  - joint axis xyz
  - joint limit lower upper（沿轴的平移范围）

  可选：

  - link orgin xyz rpy

- 旋转+平移 螺旋（拧瓶盖）

  ```xml
  <?xml version="1.0"?>
  <robot name="screw_bottle_with_dummy_link">
  
    <!--================ 主要连杆定义 ================-->
    <!-- 瓶身 -->
    <link name="bottle">
      <visual>
        <geometry>
          <cylinder length="0.2" radius="0.05"/>
        </geometry>
        <material name="glass">
          <color rgba="0.7 0.7 1.0 0.5"/> <!-- 半透明玻璃色 -->
        </material>
      </visual>
      <collision>
        <geometry>
          <cylinder length="0.2" radius="0.05"/>
        </geometry>
      </collision>
      <inertial>
        <mass value="0.5"/>
        <inertia ixx="0.001" ixy="0" ixz="0" iyy="0.001" iyz="0" izz="0.001"/>
      </inertial>
    </link>
  
    <!-- 虚拟中间连杆（无实际几何，仅用于运动学） -->
    <link name="dummy_translation">
    </link>
  
    <link name="cap">
      <visual>
        <geometry>
          <box size="0.1 0.1 0.1"/>
        </geometry>
        <material name="metal">
          <color rgba="0.8 0.8 0.8 1"/> 
        </material>
      </visual>
      <collision>
        <geometry>
          <cylinder length="0.05" radius="0.06"/>
        </geometry>
      </collision>
    </link>
  
    <!--================ 关节定义 ================-->
    <!-- 平移关节（虚拟连杆到瓶盖） -->
    <joint name="translation_joint" type="prismatic">
      <parent link="dummy_translation"/>
      <child link="cap"/>
      <axis xyz="0 0 1"/> <!-- 沿 Z 轴平移 -->
      <limit lower="0" upper="0.02" effort="10" velocity="0.1"/> <!-- 螺距 2cm -->
    </joint>
  
    <!-- 旋转关节（瓶身到虚拟连杆） -->
    <joint name="rotation_joint" type="continuous">
      <parent link="bottle"/>
      <child link="dummy_translation"/>
      <origin xyz="0 0 0.1"/> <!-- 初始位置在瓶口 -->
      <axis xyz="0 0 1"/>     <!-- 绕 Z 轴旋转 -->
    </joint>
  
    
   
  
  </robot>
  ```

  由于urdf不支持将不同关节连接的两个节点设为一样，所以对于平移+旋转（螺旋）关节需要设置一个虚拟连杆，属性的物理意义见注释

- floating 和 planar关节在铰接物体中不常见，省略
- fixed关节为固定关节，省略