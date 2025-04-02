[toc]

--- 


# 关于rpy的理解
均是以base的坐标点作为旋转的，其中rpy指的是x,y,z单位是弧度。旋转次序是先x，在y，再z。

rpy都是弧度制，PI = 3.14 rad = 180°。

所有变换均基于parent坐标系：base_footprint，这个很重要，parent坐标系是不变的，变的是child坐标系

rpy：roll、pitch、yaw，翻滚、俯仰、偏航，对应x、y、z顺序。从坐标原点为基准，看向坐标轴正方向，绕坐标轴顺时针为正，逆时针为负。

<origin xyz="0 0 1.0" rpy="1.57 1.57 0" />

先绕x轴顺时针90°，再绕y轴顺时针90°：

注意这里有一个先后顺序！！！
一定是先roll，再pitch，最后yaw！

可以说上面所谓的xyz的顺序是错误的，实际上是zyx的顺序。这种方式的旋转顺序才是正确的。

```javascript

const roll = (item.rpy[0])
const pitch = (item.rpy[1])
const yaw = (item.rpy[2])

const euler = new THREE.Euler(roll, pitch, yaw, 'ZYX');

// 应用旋转
obj.rotation.copy(euler);

```