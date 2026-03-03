# Folio 2025

![image info](./static/social/share-image.png)

## 技术栈分析

### 核心技术

| 技术 | 版本 | 用途 |
|---|---|---|
| [Three.js](https://threejs.org/) (WebGPU) | ^0.182.0 | 3D 渲染引擎，使用最新 WebGPU 后端（兼容 WebGL 回退） |
| [Three.js Shading Language (TSL)](https://github.com/mrdoob/three.js/wiki/Three.js-Shading-Language) | (随 Three.js) | 节点式着色器材质系统，替代 GLSL |
| [Rapier3D](https://rapier.rs/) (WASM) | ^0.17.3 | Rust 编译的 WebAssembly 物理引擎 |
| [GSAP](https://greensock.com/gsap/) | ^3.12.5 | 高性能补间动画库 |
| [Howler.js](https://howlerjs.com/) | ^2.2.4 | Web 音频管理库 |
| [camera-controls](https://github.com/yomotsu/camera-controls) | ^3.1.2 | 摄像机控制器 |
| [Tweakpane](https://tweakpane.github.io/docs/) | ^4.0.4 | 调试面板 UI |
| [Vite](https://vitejs.dev/) | ^7.2.4 | 前端构建工具与开发服务器 |
| [Stylus](https://stylus-lang.com/) | ^0.64.0 | CSS 预处理器（`.styl` 文件） |
| [msgpack-lite](https://github.com/kawanet/msgpack-lite) | ^0.1.26 | WebSocket 二进制序列化 |

### 架构模式

- **游戏引擎式循环** — 带优先级的有序 tick 系统（见下方 Game Loop 章节）
- **单例模式** — `Game` 类作为全局单例，各子系统通过 `Game.getInstance()` 访问
- **事件驱动** — 自定义 `Events` 类解耦各模块通信
- **物理/视觉分离** — `PhysicsVehicle` 处理物理，`VisualVehicle` 处理渲染，相互解耦

### 3D 资源工具链

| 工具 | 文件类型 | 用途 |
|---|---|---|
| [Blender](https://www.blender.org/) | `.blend` | 3D 建模与场景制作 |
| [Adobe Photoshop](https://www.adobe.com/products/photoshop.html) | `.psd` | 贴图绘制 |
| [Substance Designer](https://www.adobe.com/products/substance3d-designer.html) | `.sbs` | PBR 材质创建 |
| [@gltf-transform](https://gltf-transform.dev/) | `.glb` / `.ktx` | GLB 压缩与 KTX2/ETC1S 纹理转换 |
| [sharp](https://sharp.pixelplumbing.com/) | `.png`/`.jpg`→`.webp` | UI 图片转 WebP |

### 修改此项目需要的技能

#### 必备技能

1. **JavaScript (ES Modules)** — 所有源码均为现代 ES Module 风格，无 TypeScript
2. **Three.js** — 场景、网格、材质、灯光、后处理
3. **Three.js Shading Language (TSL)** — 节点材质系统，用于自定义着色效果
4. **WebGPU / WebGL** — 了解渲染管线基础知识有助于调试性能问题
5. **Vite** — 配置开发环境与生产构建
6. **Stylus CSS** — 修改 UI 样式
7. **Node.js** — 运行构建脚本与资源压缩管线

#### 进阶技能

8. **Rapier3D / 物理引擎** — 修改车辆物理、碰撞体等
9. **GSAP** — 调整过渡动画与时间线
10. **Blender** — 修改 3D 场景与模型，并使用项目内的导出预设
11. **KTX2 / ETC1S 纹理压缩** — 使用 `npm run compress` 管线处理新贴图
12. **WebSocket / msgpack** — 如需修改多人或服务器同步逻辑

## Setup

Create `.env` file based on `.env.example`

Download and install [Node.js](https://nodejs.org/en/download/) then run this followed commands:

``` bash
# Install dependencies
npm install --force

# Serve at localhost:1234
npm run dev

# Build for production in the dist/ directory
npm run build
```

## Game loop

#### 0

- Time
- Inputs

#### 1

- Player:pre-physics (Inputs)

#### 2

- PhysicalVehicle:pre-physics (Player:pre-physics)

#### 3

- Physics

#### 4

- PhysicsWireframe (Physics)
- Objects (Physics)

#### 5

- PhysicalVehicle:post-physics (Player:pre-physics)

#### 6

- Player:post-physics (Physics, PhysicalVehicle:post-physics)

#### 7

- View (Inputs, Player:post-physics)

#### 8

- Intro
- DayCycles
- YearCycles
- Weather (DayCycles, YearCycles)
- Zones (Player:post-physics)
- VisualVehicle (PhysicalVehicle:post-physics, Inputs, Player:post-physics, View)

#### 9

- Wind (Weather)
- Lighting (DayCycles, View)
- Tornado (DayCycles, PhysicalVehicle)
- InteractivePoints (Player:post-physics)
- Tracks (VisualVehicle)

#### 10

- Area++ (View, PhysicalVehicle:post-physics, Player:post-physics, Wind)
- Foliage (VisualVehicle, View)
- Fog (View)
- Reveal (DayCycles)
- Terrain (Tracks)
- Trails (PhysicalVehicle)
- Floor (View)
- Grass (View, Wind)
- Leaves (View, PhysicalVehicle)
- Lightnings (View, Weather)
- RainLines (View, Weather, Reveal)
- Snow (View, Weather, Reveal, Tracks)
- VisualTornado (Tornado)
- WaterSurface (Weather, View)
- Benches (Objects)
- Bricks (Objects)
- ExplosiveCrates (Objects)
- Fences (Objects)
- Lanterns (Objects)
- Whispers (Player)

#### 13

- InstancedGroup (Objects, [SpecificObjects])

#### 14

- Audio (View, Objects)
- Notifications
- Title (PhysicalVehicle:post-physics)

#### 998

- Rendering

#### 999

- Monitoring

## Blender

### Export

- Mute the palette texture node (loaded and set in Three.js `Material` directly)
- Use corresponding export presets
- Don't use compression (will be done later)

### Compress

Run `npm run compress`

Will do the following

#### GLB

- Traverses the `static/` folder looking for glb files (ignoring already compressed files)
- Compresses embeded texture with `etc1s --quality 255` (lossy, GPU friendly)
- Generates new files to preserve originals

#### Texture files

- Traverses the `static/` folder looking for `png|jpg` files (ignoring non-model related folders)
- Compresses with default preset to `--encode etc1s --qlevel 255` (lossy, GPU friendly) or specific preset according to path
- Generates new files to preserve originals

#### UI files

- Traverses the `static/ui.` folder looking for `png|jpg` files
- Compresses to WebP

#### Resources

- https://gltf-transform.dev/cli
- https://github.com/KhronosGroup/KTX-Software?tab=readme-ov-file
- https://github.khronos.org/KTX-Software/ktxtools/toktx.html