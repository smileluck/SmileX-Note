[TOC]

---

# vue3-threejs

## 安装

- 安装 `Threejs`

```shell
npm install three --save

pnpm install three
```

- 安装 typescript 依赖库

```shell
pnpm install @types/three --save-dev
```

## 其它

- 3D高斯渲染
  
  ```shell
  https://github.com/mkkellogg/GaussianSplats3D
  ```

# 使用示例

## 双击选中对象

1. 创建存储对象数组
   
   ```typescript
   const objView:any[]  = [];
   let objCtrlRef: ObjCtrl = reactive<ObjCtrl>({
       focusObj: null,
       modeType: 'translate'
   })
   ```

2. 添加mesh对象
   
   ```typescript
     /**
      * glb 和 gltf 渲染
      *
      * @param url 地址
      */
     function renderGltf(url: string, cb: Function) {
       const loader: GLTFLoader = new GLTFLoader();
       progressState.value = true;
       loader.load(
         url,
         async (gltf: GLTF) => {
           // 解决模型为黑色的问题
           gltf.scene.traverse(function (child: any) {
             if (child.isMesh) {
               child.material.emissive = child.material.color;
               child.material.emissiveMap = child.material.map;
               child.castShadow = true;
               child.receiveShadow = true;
               // 增加父类说明，后续通过这个进行判断
               child.ancestors = gltf.scene;
             }
           });
           gltf.scene.castShadow = true;
   
           if (typeof cb === 'function') {
             await cb(gltf);
           }
           scene.add(gltf.scene);
   
           objView.push(gltf.scene)
         },
         function (xhr) {
           // 控制台查看加载进度xhr
           console.log(Math.floor((xhr.loaded / xhr.total) * 100));
           progressPercent.value = Math.floor(xhr.loaded / xhr.total);
         }
       );
     }
   ```

3. 双击事件绑定
   
   ```typescript
   import { TransformControls } from 'three/examples/jsm/controls/TransformControls.js'
   ```

```javascript
   renderer.domElement.addEventListener('dblclick', function (event: any) {
     // .offsetY、.offsetX以canvas画布左上角为坐标原点,单位px
     const px = event.offsetX;
     const py = event.offsetY;
     //屏幕坐标px、py转WebGL标准设备坐标x、y
     //width、height表示canvas画布宽高度
     const x = (px / canvasWidth) * 2 - 1;
     const y = -(py / canvasHeight) * 2 + 1;
     //创建一个射线投射器`Raycaster`
     const raycaster = new THREE.Raycaster();
     //.setFromCamera()计算射线投射器`Raycaster`的射线属性.ray
     // 形象点说就是在点击位置创建一条射线，射线穿过的模型代表选中
     raycaster.setFromCamera(new THREE.Vector2(x, y), camera);
     //.intersectObjects([mesh1, mesh2, mesh3])对参数中的网格模型对象进行射线交叉计算
     // 未选中对象返回空数组[],选中一个对象，数组1个元素，选中两个对象，数组两个元素
     const intersects = raycaster.intersectObjects(objView);
     // intersects.length大于0说明，说明选中了模型
     if (intersects.length > 0) {
       transformControl.detach()
       // 选中模型的第一个模型，设置为红色
       // intersects[0].object.material.color.set(0xff0000);
       const obj: any = intersects[0].object;
       if (obj.ancestors != null) {
         // 如果双击的是同一个对象则返回
         if (objCtrlRef.focusObj !== null && objCtrlRef.focusObj.id === obj.ancestors.id) {
           objCtrlRef.focusObj = null
         } else {
           objCtrlRef.focusObj = (obj.ancestors)
           transformControl.attach(obj.ancestors)
         }
       } else if (obj.parent instanceof DropInViewer) {
         if (objCtrlRef.focusObj !== null && objCtrlRef.focusObj.id === obj.parent.id) {
           objCtrlRef.focusObj = null
         } else {
           objCtrlRef.focusObj = (obj.parent)
           transformControl.attach(obj.parent)
         }
       } else {
         if (objCtrlRef.focusObj !== null && objCtrlRef.focusObj.id === obj.id) {
           objCtrlRef.focusObj = null
         } else {
           objCtrlRef.focusObj = (obj)
           transformControl.attach(obj)
         }
       }
     } else {
       objCtrlRef.focusObj = null
       transformControl.detach()
     }
   })
```

```typescript
// 摄像机围绕注视点旋转
1. 数据存储

 ```typescript
 // 摄像头注视位置
   let _cameraLookAt = new THREE.Vector3(0, 0, 0)
   let cameraCtrlRef: CameraCtrl = reactive<CameraCtrl>({
     isMove: false,
     isRotate: false
   });
```

2. 创建摄像头

```typescript
camera = new THREE.PerspectiveCamera(75, canvasWidth / canvasHeight, 1, 100);
camera.position.set(3, 3, 3);
camera.lookAt(_cameraLookAt);
```

3. 旋转方法
   
   ```typescript
     let animationFrameR: number | null = null;
     let rotateAngleR = 0; // 用于圆周运动计算的角度值
     let rotateR = 2; // 相机圆周运动的半径
   // 开始旋转
     function startRotate() {
       if (cameraCtrlRef.isRotate) {
         return;
       }
       cameraCtrlRef.isRotate = true;
       // 计算x,z的绝对距离
       // _cameraLookAt可以替换成物体的x，z则变为围绕物体旋转
       const x = Math.abs(camera.position.x - _cameraLookAt.x)
       const z = Math.abs(camera.position.z - _cameraLookAt.z)
       // 获取旋转半径
       rotateR = Math.sqrt(x * x + z * z);
       rotate();
     }
   // 旋转方法，
     function rotate() {
       rotateAngleR += 0.01;
       // 相机y坐标不变，在XOZ平面上做圆周运动，
       //  _cameraLookAt.x/z，如果不加，则变成原点，所以需要增加一个偏移量，即所围绕点的x,z
       camera.position.x = rotateR * Math.cos(rotateAngleR) + _cameraLookAt.x;
       camera.position.z = rotateR * Math.sin(rotateAngleR) + _cameraLookAt.z;
       camera.lookAt(_cameraLookAt);
       renderer.render(scene, camera);
       animationFrameR = requestAnimationFrame(rotate);
     }
     function stopRotate() {
       if (animationFrameR !== null) {
         cancelAnimationFrame(animationFrameR);
       }
       animationFrameR = null;
       cameraCtrlRef.isRotate = false;
     }
   ```

## 键盘控制摄像头

1. 存储数据
   
   ```typescript
     const keyState: any = {
       KeyW: false,
       KeyS: false,
       KeyA: false,
       KeyD: false,
     };
   ```

2. 控制方法
   
   ```shell
     function cameraKeyUp(event: any) {
       keyState[event.code] = false;
       updateMoveDirection();
     }
   
     function cameraKeyDown(event: any) {
       keyState[event.code] = true;
       updateMoveDirection();
     }
   // 移动速度
     const moveSpeed = 0.1
     function updateMoveDirection() {
       const direction = new THREE.Vector3(); // 存储相机前方方向
       const moveDirection = new THREE.Vector3(); // 计算移动向量
       const upVector = new THREE.Vector3(0, 1, 0); // 作为旋转轴辅助计算
   
       // 获取相机面向的方向
       camera.getWorldDirection(direction);
   
       // 根据按键状态调整移动向量
       if (keyState['KeyW']) moveDirection.add(direction);
       if (keyState['KeyS']) moveDirection.sub(direction);
       if (keyState['KeyA']) moveDirection.add(upVector.clone().cross(direction)); // 左转
       if (keyState['KeyD']) moveDirection.sub(upVector.clone().cross(direction)); // 右转
   
       // 确保移动向量有明确的方向，避免无效移动
       moveDirection.normalize();
   
       // 应用移动，乘以速度常量控制速度
       camera.position.add(moveDirection.multiplyScalar(moveSpeed));
     }
   ```

3. 绑定事件
   
   ```typescript
   function cameraStartKeyCtrl() {
       document.addEventListener('keydown', cameraKeyDown, false);
       document.addEventListener('keyup', cameraKeyUp, false);
       cameraCtrlRef.isMove = true
   }
   
   function cameraStopKeyCtrl() {
       document.removeEventListener('keydown', cameraKeyDown, false);
       document.removeEventListener('keyup', cameraKeyUp, false);
       cameraCtrlRef.isMove = false
   }
   ```

## 渲染

### 渲染点云

```typescript
import { PLYLoader } from 'three/examples/jsm/loaders/PLYLoader';

/**
   * ply点云文件渲染
   *
   * @param url 地址
   */
function renderPly(url: string) {
    const loader = new PLYLoader();
    progressState.value = true;
    loader.load(
        url,
        geometry => {
            const material = new THREE.MeshPhongMaterial({
                color: 0xff0000,
                shininess: 20 // 高光部分的亮度，默认30
            });
            geometry.computeBoundingBox();
            geometry.center();
            geometry.computeVertexNormals();
            const mesh = new THREE.Mesh(geometry, material);

            scene.add(mesh);
            progressState.value = false;
            objView.push(mesh)
        },
        function (xhr) {
            // 控制台查看加载进度xhr
            progressPercent.value = Math.floor(xhr.loaded / xhr.total);
        }
    );
}
```

### 渲染HDR环境贴图

```typescript
import { RGBELoader } from 'three/examples/jsm/Addons.js';
  /**
   * 渲染环境贴图
   *
   * @param url 路径
   */
  function renderHdr(url: string) {
    // 环境贴图
    const rgbeLoader = new RGBELoader();
    rgbeLoader.load(
      url,
      texture => {
        const pmremGenerator = new THREE.PMREMGenerator(renderer);
        pmremGenerator.compileEquirectangularShader();
        const envMap = pmremGenerator.fromEquirectangular(texture).texture;
        // 设备为背景（也可以用其他的的场景背景）
        scene.background = envMap;
        // 设为场景中所有物理材质的环境贴图
        scene.environment = envMap;
        texture.dispose();
        pmremGenerator.dispose();
      },
      undefined,
      error => {
        console.error('Error loading HDR texture', error);
      }
    );
  }
```

### 渲染glb

```typescript
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';

/**
   * glb 和 gltf 渲染
   *
   * @param url 地址
   */
function renderGltf(url: string, cb: Function) {
    const loader: GLTFLoader = new GLTFLoader();
    progressState.value = true;
    loader.load(
        url,
        async (gltf: GLTF) => {
            // 解决模型为黑色的问题
            gltf.scene.traverse(function (child: any) {
                if (child.isMesh) {
                    child.material.emissive = child.material.color;
                    child.material.emissiveMap = child.material.map;
                    child.castShadow = true;
                    child.receiveShadow = true;
                    child.ancestors = gltf.scene;
                }
            });
            gltf.scene.castShadow = true;

            if (typeof cb === 'function') {
                await cb(gltf);
            }
            scene.add(gltf.scene);

            objView.push(gltf.scene)
            progressState.value = false;
        },
        function (xhr) {
            // 控制台查看加载进度xhr
            console.log(Math.floor((xhr.loaded / xhr.total) * 100));
            progressPercent.value = Math.floor(xhr.loaded / xhr.total);
        }
    );
}
```

### 渲染3D高斯

```typescript
import { DropInViewer } from '@mkkellogg/gaussian-splats-3d';

  /**
   * 3d高斯渲染
   *
   * @param netUrl 网路地址
   */
  function render3DGS(netUrl: string) {
    gsView = new DropInViewer({
      gpuAcceleratedSort: true,
      sharedMemoryForWorkers: false,
      // 'showLoadingUI': true,
    });

    gsView
      .addSplatScene(
        netUrl,
        // fileUrl,
        {
          splatAlphaRemovalThreshold: 5,
          showLoadingSpinner: true,
          sharedMemoryForWorkers: true,
          format: 2
        }
      )
      .then(() => {
        const geometry = gsView.children[0].geometry;

        geometry.computeBoundingBox();
        geometry.center();
        geometry.computeVertexNormals();

        gsView.castShadow = true;
        // viewer.receiveShadow = true

        scene.add(gsView);
      });
  }
```
