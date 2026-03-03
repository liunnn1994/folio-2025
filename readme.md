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

## 项目结构分析

### 顶层目录

```
folio-2025/
├── sources/          # 应用源码（JS、HTML、Stylus CSS）
├── static/           # 编译后的游戏资源（模型、贴图、音效等）
├── resources/        # 原始设计文件（Blender、PSD、SBS 等）
├── scripts/          # 构建辅助脚本
├── vite.config.js    # Vite 构建配置
├── package.json      # 项目依赖与 npm 脚本
└── .env.example      # 环境变量模板
```

### sources/ — 源码目录

```
sources/
├── index.html            # HTML 入口，挂载 canvas 及 UI 根节点
├── index.js              # JS 入口，实例化 Game 并启动
├── threejs-override.js   # 对 Three.js 默认行为的局部覆盖
├── Game/                 # 游戏核心逻辑（见下方详解）
├── data/                 # 静态数据定义
└── style/                # Stylus CSS 样式文件
```

#### sources/Game/ — 游戏核心

| 文件 / 目录 | 说明 |
|---|---|
| `Game.js` | 全局单例，持有所有子系统引用，负责启动与帧循环 |
| `Ticker.js` | 带优先级的 tick 调度器，驱动 Game Loop |
| `Time.js` | 封装帧时间、delta、elapsed 等时间数据 |
| `Rendering.js` | WebGPU/WebGL 渲染器初始化与后处理管线 |
| `Viewport.js` | 视口尺寸监听与响应式适配 |
| `Events.js` | 全局自定义事件总线，解耦模块通信 |
| `Player.js` | 玩家状态管理（当前区域、交互、重生等） |
| `View.js` | 摄像机位置与朝向控制 |
| `Physics/` | Rapier3D 物理世界、车辆物理、调试线框 |
| `World/` | 所有 3D 场景对象与视觉效果（见下方详解） |
| `Inputs/` | 键盘、鼠标、触控、手柄、滚轮输入统一封装 |
| `Cycles/` | 昼夜循环（`DayCycles`）与四季循环（`YearCycles`） |
| `Materials/` | 自定义 TSL 节点材质（网格默认材质、网格材质等） |
| `Geometries/` | 自定义几何体（风线、传送门石板等） |
| `Passes/` | 自定义后处理通道（廉价景深等） |
| `BlackFriday/` | 黑色星期五特殊活动逻辑 |
| `utilities/` | 通用工具函数（数学、时间、响应式集合/映射） |
| `Audio.js` | Howler.js 音频管理，按区域与事件播放音效 |
| `Weather.js` | 天气状态（晴/雨/雪/雷暴）管理 |
| `Terrain.js` | 地形高度图采样与碰撞数据 |
| `ResourcesLoader.js` | 资源加载队列与进度管理 |
| `Server.js` | WebSocket 连接与 msgpack 二进制通信 |
| `Menu.js` / `Modals.js` | 主菜单与弹窗 UI 逻辑 |
| `Notifications.js` | 屏幕通知提示系统 |
| `Achievements.js` | 成就解锁检测与展示 |
| `Quality.js` | 画质档位（低/中/高/超）动态切换 |
| `Debug.js` | Tweakpane 调试面板注册与管理 |
| `Monitoring.js` | 性能监控（FPS、内存等） |

#### sources/Game/World/ — 3D 世界对象

| 文件 / 目录 | 说明 |
|---|---|
| `World.js` | 世界根节点，统一初始化所有场景对象 |
| `VisualVehicle.js` | 玩家车辆的视觉表现（模型、动画、车轮变形等） |
| `Areas/` | 各功能区域（登陆区、项目区、社交区、职业区等） |
| `Foliage.js` / `Trees.js` / `Bushes.js` / `Flowers.js` / `Grass.js` | 植被系统（GPU 实例化渲染） |
| `Leaves.js` / `Snow.js` / `RainLines.js` / `Lightnings.js` | 天气粒子效果 |
| `Tornado.js` / `VisualTornado.js` | 龙卷风物理与视觉效果 |
| `WaterSurface.js` | 水面波纹与反射 |
| `Terrain.js` / `Floor.js` / `Grid.js` | 地形网格与地面 |
| `Tracks.js` / `Trails.js` | 车辙痕迹系统 |
| `Wind.js` / `WindLines.js` | 风向模拟与风线视觉 |
| `Scenery.js` | 场景装饰物统一管理 |
| `Benches.js` / `Bricks.js` / `Fences.js` / `Lanterns.js` | 可交互/物理道具 |
| `ExplosiveCrates.js` | 可爆炸箱子物理对象 |
| `Whispers.js` | 隐藏彩蛋提示文字 |
| `Intro.js` | 开场动画与过渡效果 |
| `Bubble.js` / `Confetti.js` / `Fireballs.js` | 特殊粒子效果 |

#### sources/Game/World/Areas/ — 互动区域

| 文件 | 说明 |
|---|---|
| `LandingArea.js` | 登陆/起始区域 |
| `ProjectsArea.js` | 项目展示区域 |
| `SocialArea.js` | 社交媒体链接区域 |
| `CareerArea.js` | 职业经历区域 |
| `LabArea.js` | 实验室/实验项目区域 |
| `AchievementsArea.js` | 成就展示区域 |
| `CircuitArea.js` | 赛道竞速区域 |
| `BowlingArea.js` | 保龄球小游戏区域 |
| `AltarArea.js` | 祭坛/特殊互动区域 |
| `BehindTheSceneArea.js` | 幕后/开发者区域 |
| `TimeMachine.js` | 时间机器特殊区域 |
| `ToiletArea.js` | 彩蛋区域 |
| `CookieArea.js` | Cookie 彩蛋区域 |

#### sources/data/ — 数据定义

| 文件 | 说明 |
|---|---|
| `projects.js` | 展示的项目列表（名称、链接、描述等） |
| `social.js` | 社交媒体账号列表 |
| `achievements.js` | 成就定义（触发条件、图标、文案） |
| `lab.js` | 实验项目列表 |
| `countries.js` | 国家数据（用于访客地图等） |
| `consoleLog.js` | 控制台欢迎信息内容 |

#### sources/style/ — 样式文件

所有样式均使用 **Stylus** 编写（`.styl`），按功能模块拆分，通过 `index.styl` 统一导入：

| 文件 | 说明 |
|---|---|
| `general.styl` / `fonts.styl` | 全局基础样式与字体声明 |
| `menu.styl` / `modals.styl` | 主菜单与弹窗样式 |
| `controls.styl` / `options.styl` | 控制说明与设置面板样式 |
| `achievements.styl` | 成就弹出提示样式 |
| `notifications.styl` | 通知提示样式 |
| `circuit.styl` / `circuit-end.styl` | 赛道计时 UI 样式 |
| `map.styl` | 小地图样式 |
| `tabs.styl` | 标签页样式 |
| `tooltips.styl` / `interactiveButtons.styl` | 交互提示与按钮样式 |
| `tweakpane.styl` | 调试面板自定义样式 |
| `whispers.styl` | 彩蛋文字样式 |
| `discord.styl` / `behindTheScene.styl` / `blackFriday.styl` | 特殊活动/页面样式 |

### static/ — 游戏资源目录

存放经过压缩处理的**运行时资源**，由 `npm run compress` 生成：

| 子目录 | 内容 |
|---|---|
| `models/` | 压缩后的 `.glb` 3D 模型 |
| `terrain/` | 地形高度图与贴图（KTX2 格式） |
| `areas/` | 各区域专用资源 |
| `sounds/` / `jukebox/` | 音效与背景音乐（`.ogg`/`.mp3`） |
| `foliage/` / `particles/` | 植被与粒子贴图 |
| `ui/` | UI 图片（WebP 格式） |
| `fonts/` | 字体文件 |
| `favicons/` | 网站图标 |
| `social/` / `projects/` | 社交头像与项目截图 |
| `overlay/` | 屏幕叠加层资源 |
| `achievements/` | 成就图标 |

### resources/ — 原始设计文件

存放**不参与构建**的原始创作文件，仅供设计迭代使用：

| 文件 / 目录 | 内容 |
|---|---|
| `folio-2025.blend` | 主 Blender 场景文件 |
| `models/` | 各角色/物件的 `.blend` 源文件及导出的 `.glb` |
| `textures/` | 原始贴图（`.psd`、`.png` 等） |
| `sounds/` | 原始音频文件 |
| `renders/` | 渲染输出图 |
| `slabs.sbs` | Substance Designer 材质源文件 |
| `palette.png` / `stars.psd` | 调色板与星空贴图源文件 |

### scripts/ — 构建脚本

| 文件 | 说明 |
|---|---|
| `compress.js` | 遍历 `static/` 目录，对 GLB 模型执行 ETC1S 纹理压缩，对 PNG/JPG 执行 KTX2 转换，对 UI 图片执行 WebP 转换 |

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