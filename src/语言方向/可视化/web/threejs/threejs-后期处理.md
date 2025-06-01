[toc]

---

# three.js-后期处理
## 基础使用(以outline为示例)
1. 引入并创建一个后期处理容器
```javascript

import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer';

// 创建一个webGL对象
let renderer = new THREE.WebGLRenderer({
    //增加下面两个属性，可以抗锯齿
    antialias: true,
    alpha: true,
    logarithmicDepthBuffer: true // 解决模型闪烁问题
});
// 创建一个EffectComposer对象
this.composer = new EffectComposer(renderer); 

this.composer = new EffectComposer(this.renderer, new THREE.WebGLRenderTarget(this.canvasWidth, this.canvasHeight, {
    samples: 4 // 增加采样次数来提高抗锯齿效果
}));

```

2. 添加渲染器通道
**需要添加到第一个**
```javascript

import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass';

const renderPass = new RenderPass(this.scene, this.camera);
this.composer.addPass(renderPass);
```

3. 添加outline高亮通道
```javascript
import { OutlinePass } from 'three/examples/jsm/postprocessing/OutlinePass';
 
this.outlinePass = new OutlinePass(
    new THREE.Vector2(this.canvasWidth, this.canvasHeight),
    this.scene,
    this.camera
);
// 将此通道结果渲染到屏幕
this.outlinePass.renderToScreen = true// 设置这个参数的目的是马上将当前的内容输出
this.outlinePass.edgeGlow = 1 // 发光强度
this.outlinePass.usePatternTexture = false // 是否使用纹理图案
this.outlinePass.edgeThickness = 2 // 边缘浓度
this.outlinePass.edgeStrength = 5 // 边缘的强度，值越高边框范围越大
this.outlinePass.pulsePeriod = 2// 闪烁频率，值越大频率越低
this.outlinePass.visibleEdgeColor.set('#00FF00') // 呼吸显示的颜色
this.outlinePass.hiddenEdgeColor.set('#00FF00') // 不可见边缘的颜色

this.outlinePass.selectedObjects = []
this.composer.addPass(this.outlinePass);

```

4. 选中需要高亮的物体

```javascript
this.outlinePass.selectedObjects = [this.mesh] // 选中需要高亮的物体

this.outlinePass.selectedObjects = [] // 清空选中物体
```

5. 渲染
**注意避免循环渲染，要采用第二种写法**
```javascript

this.renderer.render(this.scene, this.camera); // 当有composer的时候，可以不用这个渲染。不然重复渲染容易导致性能降低。
if (this.composer) {
    this.composer.render();
}

// 即采用下面的代码
if (this.composer) {
    this.composer.render();
} else {
    this.renderer.render(this.scene, this.camera);
}

```

## gltf 后处理颜色异常

### 方式一：使用GammaCorrectionShader

加载gltf模型如果出现颜色偏差，需要设置renderer.outputEncoding解决。如果你使用threejs后处理功能EffectComposer，renderer.outputEncoding会无效，自然会出现颜色偏差。
GammaCorrectionShader.js的功能就是进行伽马校正，具体点说就是可以用来解决gltf模型后处理时候，颜色偏差的问题。
threejs并没有直接提供伽马校正的后处理通道，提供了一个伽马校正的Shader对象GammaCorrectionShader，这时候可以把Shader对象作为ShaderPass的参数创建一个通道。

```javascript
import { GammaCorrectionShader } from 'three/examples/jsm/shaders/GammaCorrectionShader.js';

// 颜色修正
const gammaCorrectionShader = new ShaderPass(GammaCorrectionShader);
gammaCorrectionShader.renderToScreen = true;
this.composer.addPass(gammaCorrectionShader);
```

### 方式二：使用OutputPass
创建一个OutputPass对象，并设置renderToScreen为true，将此通道添加到composer中。
```javascript
import { OutputPass } from 'three/examples/jsm/postprocessing/OutputPass.js';
const outputPass = new OutputPass();
outputPass.renderToScreen = true;
this.composer.addPass(outputPass);
```

**需要在添加到outline后面**

## FXAAShader 快速近似抗锯齿

**FXAAShader**运用了快速近似抗锯齿（FXAA）算法。该算法不会增加渲染的几何复杂度，而是直接对渲染后的图像进行后期处理。它会检测图像中的边缘区域，然后有针对性地对这些区域进行模糊处理，从而达到消除锯齿的视觉效果。由于这种处理方式是在图像层面进行的，所以性能开销相对较低，特别适合在移动设备或者性能有限的硬件上使用。

### **一、FXAAShader 的核心作用**
#### 1. **消除锯齿边缘**
- **锯齿**：当低分辨率屏幕（或渲染缓冲区）无法精确表示复杂几何形状时，边缘会出现阶梯状的锯齿效果。
- **FXAA**：通过分析相邻像素的颜色差异，智能识别边缘并应用模糊算法，使边缘过渡更平滑。

#### 2. **性能优化**
- 相比传统的 MSAA（多重采样抗锯齿），FXAA 是一种**后处理技术**，计算开销更低，适合性能有限的设备（如移动设备或低配电脑）。

#### 3. **兼容性强**
- 不依赖硬件支持，只要浏览器支持 WebGL 就能使用，通用性好。


### **二、如何在 Three.js 中使用 FXAAShader**
#### 1. **引入必要模块**
```javascript
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js';
import { FXAAShader } from 'three/examples/jsm/shaders/FXAAShader.js';
```

#### 2. **配置后期处理链**
```javascript
// 创建渲染器时启用抗锯齿（可选）
const renderer = new THREE.WebGLRenderer({ antialias: false }); // 即使不启用硬件抗锯齿，FXAA 也能工作

// 创建后期处理合成器
const composer = new EffectComposer(renderer);

// 添加渲染通道
const renderPass = new RenderPass(scene, camera);
composer.addPass(renderPass);

// 添加 FXAA 通道
const fxaaPass = new ShaderPass(FXAAShader);
composer.addPass(fxaaPass);

// 根据屏幕分辨率调整 FXAA 参数
function onWindowResize() {
  const pixelRatio = renderer.getPixelRatio();
  const width = window.innerWidth * pixelRatio;
  const height = window.innerHeight * pixelRatio;
  
  fxaaPass.material.uniforms['resolution'].value.set(
    1 / width,
    1 / height
  );
}
window.addEventListener('resize', onWindowResize);
onWindowResize(); // 初始化时调用一次
```

#### 3. **使用合成器渲染场景**
```javascript
function animate() {
  requestAnimationFrame(animate);
  composer.render(); // 使用 composer 替代 renderer.render()
}
animate();
```


### **三、FXAAShader 的优缺点**
#### **优点**
1. **性能高效**：比 MSAA 等硬件抗锯齿更节省资源。
2. **质量稳定**：在各种场景下都能提供一致的抗锯齿效果。
3. **易于集成**：通过后期处理链即可添加，无需修改几何体或材质。

#### **缺点**
1. **可能模糊细节**：在平滑边缘的同时，可能会轻微模糊图像中的精细纹理或文字。
2. **依赖分辨率**：需要根据屏幕分辨率和像素密度调整参数，否则效果可能不佳。
3. **不处理内部锯齿**：仅处理图像边缘，对模型内部的锯齿（如纹理重复）无效。


### **四、FXAAShader 与其他抗锯齿技术的对比**
| 技术     | 工作原理                   | 性能开销 | 质量 | 兼容性             |
| -------- | -------------------------- | -------- | ---- | ------------------ |
| **FXAA** | 后期处理，分析颜色差异     | 低       | 良好 | 软件实现，100%兼容 |
| **MSAA** | 多重采样，硬件级抗锯齿     | 中高     | 优秀 | 依赖硬件支持       |
| **TAA**  | 时间性抗锯齿（结合多帧）   | 高       | 极佳 | 需要运动矢量       |
| **SSAA** | 超采样（渲染到更高分辨率） | 极高     | 最高 | 性能消耗大         |


### **五、调试和优化建议**
1. **调整分辨率参数**：
   ```javascript
   // 手动调整 FXAA 效果强度（值越小，效果越明显，但可能过度模糊）
   fxaaPass.material.uniforms['resolution'].value.set(1 / (window.innerWidth * 0.5), 1 / (window.innerHeight * 0.5));
   ```

2. **与其他效果结合使用**：
   - FXAAShader 通常作为最后一个后期处理通道添加，以确保平滑所有前面的效果（如 Bloom、DOF 等）。

3. **移动设备优化**：
   - 在移动设备上，FXAA 是比 MSAA 更推荐的抗锯齿方案，可显著提升性能。


### **六、何时应使用 FXAAShader？**
- **性能敏感场景**：如游戏、交互式应用。
- **硬件抗锯齿不足**：当 MSAA 无法提供足够质量或导致性能下降时。
- **兼容性要求高**：需要在各种设备上一致运行的应用。

通过合理使用 FXAAShader，你可以在不显著影响性能的情况下，大幅提升 Three.js 渲染的视觉质量。



以下是Three.js中常见后期处理效果的**简单示例代码**与**功能说明**，按类别整理：

## 其它

### **1. 抗锯齿 (Antialiasing)**
#### **FXAA (快速近似抗锯齿)**
```javascript
// 需引入的模块
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js';
import { FXAAShader } from 'three/examples/jsm/shaders/FXAAShader.js';

// 创建后期处理链
const composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));

// 添加FXAA通道
const fxaaPass = new ShaderPass(FXAAShader);
fxaaPass.material.uniforms['resolution'].value.set(
  1 / window.innerWidth, 
  1 / window.innerHeight
);
composer.addPass(fxaaPass);
```
**说明**：  
通过屏幕空间边缘检测和颜色插值，减少多边形边缘的锯齿感，提升视觉平滑度。性能开销低，适合移动设备。


### **2. 视觉增强效果**
#### **Bloom (泛光)**
```javascript
import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass.js';

const bloomPass = new UnrealBloomPass(
  new THREE.Vector2(window.innerWidth, window.innerHeight),
  1.5,  // 强度
  0.4,  // 半径
  0.85  // 阈值（控制哪些区域发光）
);
composer.addPass(bloomPass);
```
**说明**：  
使明亮区域产生光晕扩散效果，增强视觉层次感，常用于模拟光源发光、霓虹灯等效果。

#### **Vignette (暗角)**
```javascript
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js';
import { VignetteShader } from 'three/examples/jsm/shaders/VignetteShader.js';

const vignettePass = new ShaderPass(VignetteShader);
vignettePass.uniforms['offset'].value = 1.0;   // 边缘过渡强度
vignettePass.uniforms['darkness'].value = 1.2; // 暗角深度
composer.addPass(vignettePass);
```
**说明**：  
使画面边缘逐渐变暗，突出中心内容，常用于电影感渲染或增强沉浸感。


### **3. 深度与空间效果**
#### **DOF (景深)**
```javascript
import { DepthOfFieldPass } from 'three/examples/jsm/postprocessing/DepthOfFieldPass.js';

// 需启用深度纹理
renderer.setDepthTexture(true);

const dofPass = new DepthOfFieldPass(scene, camera);
dofPass.focusDistance = 0.0;  // 聚焦距离
dofPass.aperture = 0.025;     // 光圈大小（值越大，背景虚化越明显）
composer.addPass(dofPass);
```
**说明**：  
模拟相机聚焦效果，使特定距离的物体清晰，其他区域模糊，增强场景真实感和视觉重点。

#### **SSAO (屏幕空间环境光遮蔽)**
```javascript
import { SSAOPass } from 'three/examples/jsm/postprocessing/SSAOPass.js';

const ssaoPass = new SSAOPass(scene, camera);
ssaoPass.radius = 0.02;       // 遮蔽半径
ssaoPass.intensity = 1.5;     // 强度
ssaoPass.aoClamp = 0.25;      // 钳位值
composer.addPass(ssaoPass);
```
**说明**：  
通过屏幕空间算法计算物体间的间接光照，增强阴影和角落的深度感，无需复杂光照计算。


### **4. 色彩校正**
#### **Color Grading (色彩分级)**
```javascript
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js';
import { ColorCorrectionShader } from 'three/examples/jsm/shaders/ColorCorrectionShader.js';

const colorCorrectionPass = new ShaderPass(ColorCorrectionShader);
colorCorrectionPass.uniforms['powRGB'].value.set(1.1, 1.1, 1.1); // 色彩幂次（提亮）
colorCorrectionPass.uniforms['mulRGB'].value.set(1.05, 1.05, 1.05); // 色彩乘法（饱和度）
composer.addPass(colorCorrectionPass);
```
**说明**：  
调整画面整体色调、亮度和饱和度，常用于创建特定风格（如复古、冷色调等）。


### **5. 特效与滤镜**
#### **Glitch (故障艺术)**
```javascript
import { GlitchPass } from 'three/examples/jsm/postprocessing/GlitchPass.js';

const glitchPass = new GlitchPass();
glitchPass.renderToScreen = true;
glitchPass.goWild = false;  // 是否持续故障效果（false为随机触发）
composer.addPass(glitchPass);
```
**说明**：  
模拟数字信号干扰效果，常用于赛博朋克风格、UI警告状态或过渡动画。

#### **Film Grain (胶片颗粒)**
```javascript
import { FilmPass } from 'three/examples/jsm/postprocessing/FilmPass.js';

const filmPass = new FilmPass(
  0.5,    // 噪点强度
  0.025,  // 扫描线强度
  648,    // 扫描线数量
  false   // 是否禁用色彩（黑白效果）
);
filmPass.renderToScreen = true;
composer.addPass(filmPass);
```
**说明**：  
添加类似胶片电影的颗粒感和扫描线效果，增强复古或电影质感。


### **6. 性能优化**
#### **SMAA (增强型多帧采样抗锯齿)**
```javascript
import { SMAAPass } from 'three/examples/jsm/postprocessing/SMAAPass.js';

const smaaPass = new SMAAPass(window.innerWidth, window.innerHeight);
composer.addPass(smaaPass);
```
**说明**：  
比FXAA更精确的抗锯齿算法，尤其适合处理对角线边缘和文字边缘，性能略高于TAA。


### **使用步骤总结**
1. **初始化后期处理链**：
   ```javascript
   const composer = new EffectComposer(renderer);
   composer.addPass(new RenderPass(scene, camera));
   ```

2. **添加效果通道**（按顺序添加，影响最终效果）：
   ```javascript
   composer.addPass(bloomPass);
   composer.addPass(dofPass);
   // ...其他通道
   ```

3. **在动画循环中渲染**：
   ```javascript
   function animate() {
     requestAnimationFrame(animate);
     composer.render(); // 替代 renderer.render(scene, camera)
   }
   animate();
   ```


### **性能注意事项**
- **复杂效果慎用**：SSAO、DOF等算法开销大，移动端需谨慎使用
- **通道顺序**：先应用抗锯齿，再叠加特效，最后使用全屏效果（如Vignette）
- **分辨率调整**：可降低EffectComposer的渲染分辨率以提升性能
  ```javascript
  composer.setSize(window.innerWidth / 2, window.innerHeight / 2); // 半分辨率
  ```