# 🎆 烟花项目优化模块

## 📦 模块概述

本文件夹包含重构后的 JavaScript 模块，用于提高代码可维护性、可读性和可测试性。

### 模块一览表

| 模块 | 功能 | 类/导出 | 行数 |
|------|------|--------|------|
| **config.js** | 常量和配置 | `DEVICE`, `QUALITY`, `COLORS`, 等 | ~150 |
| **stateManager.js** | 状态管理 | `StateManager` | ~120 |
| **selectors.js** | 状态选择器 | `createSelectors()` | ~100 |
| **colorManager.js** | 颜色管理 | `ColorManager` | ~150 |
| **particleFactory.js** | 粒子生成 | `ParticleFactory` | ~200 |
| **uiManager.js** | UI操作 | `UIManager` | ~250 |
| **actionHandlers.js** | 行为处理 | `ActionHandlers` | ~150 |

---

## 🎯 各模块详解

### 1️⃣ config.js - 配置管理

**用途**: 集中管理所有常量、配置和选项

**主要导出**:
```javascript
- DEVICE           // 设备类型检测
- QUALITY          // 画质等级
- SKY_LIGHT        // 天空光照
- COLORS           // 颜色配置
- CANVAS           // 画布限制
- MATH_CONSTANTS   // 数学常量
- getDefaultConfig()      // 获取默认配置
- getDefaultScaleFactor() // 获取缩放因子
- HELP_CONTENT     // 帮助内容
- APP_NODE_SELECTORS      // DOM选择器
```

**使用示例**:
```javascript
import { DEVICE, COLORS, getDefaultConfig } from './config.js';

if (DEVICE.IS_MOBILE) {
  console.log("Mobile device detected");
}

const config = getDefaultConfig();
const redColor = COLORS.Red;
```

---

### 2️⃣ stateManager.js - 状态管理

**用途**: 使用观察者模式管理应用全局状态

**类**: `StateManager`

**核心方法**:
- `setState(nextState)` - 更新状态
- `subscribe(listener)` - 订阅状态变化
- `load()` - 从本地存储加载
- `persist()` - 保存到本地存储
- `reset(initialState)` - 重置状态

**使用示例**:
```javascript
import { StateManager } from './stateManager.js';

const store = new StateManager(initialState);

// 订阅状态变化
const unsubscribe = store.subscribe((newState, prevState) => {
  console.log('State updated:', newState);
});

// 更新状态
store.setState({ paused: false });

// 取消订阅
unsubscribe();
```

---

### 3️⃣ selectors.js - 状态选择器

**用途**: 从状态中提取特定值的函数集合

**函数**: `createSelectors(store)`

**返回对象中的选择器**:
- `isRunning()` - 是否正在运行
- `soundEnabled()` - 声音是否启用
- `canPlaySound()` - 是否可以播放声音
- `quality()` - 画质等级
- `shellName()` - 烟花类型
- `shellSize()` - 烟花大小
- `finale()` - 是否启用finale模式
- ...等等

**好处**: 
- 集中式访问状态数据
- 状态结构变化时只需修改此文件
- 易于测试和维护

**使用示例**:
```javascript
import { createSelectors } from './selectors.js';

const selectors = createSelectors(store);

if (selectors.canPlaySound()) {
  soundManager.play("burst");
}

const quality = selectors.quality();
```

---

### 4️⃣ colorManager.js - 颜色管理

**用途**: 处理所有与颜色相关的逻辑

**类**: `ColorManager`

**核心方法**:
- `randomSimple()` - 获取简单随机颜色
- `random(options)` - 获取带条件的随机颜色
- `makePistilColor(shellColor)` - 生成雌蕊颜色
- `whiteOrGold()` - 白色或金色
- `getColorTuple(hexColor)` - 获取RGB元组
- `getAllColorTuples()` - 获取所有颜色的RGB

**选项对象**:
```javascript
{
  notSame: true,      // 不返回上一个颜色
  notColor: '#fff',   // 不返回指定颜色
  limitWhite: true    // 限制白色出现频率
}
```

**使用示例**:
```javascript
import { ColorManager } from './colorManager.js';
import { COLORS } from './config.js';

const colorManager = new ColorManager(COLORS);

// 获取随机颜色
const randomColor = colorManager.random({ limitWhite: true });

// 获取对比颜色
const pistilColor = colorManager.makePistilColor(randomColor);
```

---

### 5️⃣ particleFactory.js - 粒子生成工厂

**用途**: 生成各种粒子爆炸效果的数学计算

**类**: `ParticleFactory` (静态方法)

**核心方法**:
- `createParticleArc()` - 在圆弧上分布粒子
- `createBurst()` - 在球面上分布粒子
- `createWordBurst()` - 从文字点阵创建爆炸
- `createGridBurst()` - 网格分布
- `createSpiralBurst()` - 螺旋分布
- `distance()` - 计算距离
- `angle()` - 计算角度
- `splitVector()` - 分解速度向量

**使用示例**:
```javascript
import { ParticleFactory } from './particleFactory.js';

// 创建环形爆炸
ParticleFactory.createParticleArc(0, Math.PI * 2, 50, 0.5, (angle) => {
  Star.add(x, y, color, angle, speed, life);
});

// 创建球形爆炸
ParticleFactory.createBurst(100, (angle, speedMult) => {
  Star.add(x, y, color, angle, speedMult * speed, life);
});
```

---

### 6️⃣ uiManager.js - UI管理器

**用途**: 处理所有DOM操作和UI更新

**类**: `UIManager`

**核心方法**:
- `renderApp(state, selectors)` - 更新整个UI
- `getConfigFromDOM()` - 从表单获取配置
- `setBackgroundColor()` - 设置背景色
- `showHelp()` - 显示帮助内容
- `hideElement()` / `showElement()` - 隐藏/显示元素
- `addFormListener()` - 添加表单事件监听

**使用示例**:
```javascript
import { UIManager } from './uiManager.js';
import { APP_NODE_SELECTORS } from './config.js';

const uiManager = new UIManager(APP_NODE_SELECTORS);

// 更新UI
uiManager.renderApp(state, selectors);

// 获取配置
const config = uiManager.getConfigFromDOM();

// 设置背景色
uiManager.setBackgroundColor(255, 100, 50);

// 显示帮助
uiManager.showHelp('quality', helpContent);
```

---

### 7️⃣ actionHandlers.js - 行为处理器

**用途**: 处理所有用户交互和应用操作

**类**: `ActionHandlers`

**核心方法**:
- `togglePause()` - 切换暂停
- `toggleSound()` - 切换声音
- `toggleMenu()` - 切换菜单
- `openHelp()` - 打开帮助
- `updateConfig()` - 更新配置
- `bindUIEvents()` - 绑定所有事件

**使用示例**:
```javascript
import { ActionHandlers } from './actionHandlers.js';

const handlers = new ActionHandlers(store, uiManager, soundManager);

// 用户交互
handlers.togglePause();
handlers.toggleSound();
handlers.openHelp('quality');

// 绑定所有事件
handlers.bindUIEvents(NODE_KEY_TO_HELP_KEY, uiManager);
```

---

## 🔄 模块依赖关系

```
config.js
    ↓
stateManager.js ← → selectors.js
    ↓
uiManager.js ← colorManager.js
    ↓            ↓
actionHandlers.js → particleFactory.js

script.js (主程序)
    ↓
使用所有上述模块
```

---

## 💡 使用场景

### 场景1: 添加新功能

假设要添加"自动颜色"功能：

1. 在 `config.js` 中添加配置选项
2. 在 `stateManager.js` 中添加到初始状态
3. 在 `selectors.js` 中添加选择器
4. 在 `uiManager.js` 中添加UI更新逻辑
5. 在 `actionHandlers.js` 中添加处理方法

### 场景2: 修改颜色系统

所有颜色相关代码集中在 `colorManager.js` 中，修改更容易。

### 场景3: 调试状态问题

在 `stateManager.js` 中添加日志，跟踪所有状态变化。

---

## 🧪 单元测试示例

```javascript
// stateManager.test.js
import { StateManager } from './stateManager.js';

describe('StateManager', () => {
  let store;

  beforeEach(() => {
    store = new StateManager({ count: 0 });
  });

  test('should update state', () => {
    store.setState({ count: 1 });
    expect(store.state.count).toBe(1);
  });

  test('should notify subscribers', () => {
    const listener = jest.fn();
    store.subscribe(listener);
    store.setState({ count: 1 });
    expect(listener).toHaveBeenCalled();
  });
});
```

---

## 🚀 性能优化建议

1. **使用 CachedSelector** - 缓存复杂的选择器计算
2. **防抖事件处理** - 避免频繁的UI更新
3. **虚拟列表** - 如果有大量项目列表
4. **Web Workers** - 将计算密集操作移到后台线程
5. **代码分割** - 使用动态导入加载模块

---

## 📚 进一步阅读

- [MIGRATION_GUIDE.md](./MIGRATION_GUIDE.md) - 详细迁移指南
- [主程序文件](../script.js) - 整体使用示例
- [原始代码](../../backup/original-code) - 参考原始实现

---

**最后更新**: 2026-03-17 10:05:11  
**版本**: 1.0.0