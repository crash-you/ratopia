# Ratopia单页休闲游戏网站 - 技术架构与模块设计

## 技术栈
- HTML5：提供语义化标记结构
- TailwindCSS：实现响应式设计和现代UI组件

## 目录结构
```
ratopia/
├── index.html        # 主页面
├── assets/           # 静态资源
│   ├── images/       # 图片资源
│   ├── games/        # 游戏资源
│   └── icons/        # 图标资源
├── css/              
│   └── main.css      # 编译后的Tailwind CSS文件
└── js/               # JavaScript资源
    ├── main.js       # 主逻辑
    ├── gameLoader.js # 游戏加载管理
    ├── theme.js      # 主题管理
    ├── storage.js    # 本地存储管理
    └── utils.js      # 工具函数
```

## 模块设计

### 1. 核心布局模块
```html
<!-- 垂直三段式布局 -->
<div class="flex flex-col min-h-screen">
  <!-- 顶部导航栏 -->
  <header class="...">...</header>
  
  <!-- 中央游戏区域 -->
  <main class="flex-grow ...">...</main>
  
  <!-- 底部信息栏 -->
  <footer class="...">...</footer>
</div>
```

### 2. 响应式设计模块
使用Tailwind CSS断点系统实现：
- 移动端优先设计：默认样式适用于小屏幕
- 媒体查询断点：sm(640px), md(768px), lg(1024px), xl(1280px)
```html
<div class="w-full md:w-4/5 lg:w-3/4 xl:w-2/3 mx-auto">
  <!-- 响应式内容容器 -->
</div>
```

### 3. 游戏容器模块
```html
<div id="game-container" class="relative w-full aspect-video bg-gray-100 rounded-lg shadow-lg">
  <!-- 加载指示器 -->
  <div id="game-loader" class="absolute inset-0 flex items-center justify-center bg-white bg-opacity-80 z-10">
    <!-- 加载动画 -->
  </div>
  
  <!-- 游戏iframe -->
  <iframe id="game-frame" 
          class="w-full h-full" 
          src="" 
          frameborder="0"
          loading="lazy"
          sandbox="allow-scripts allow-same-origin">
  </iframe>
  
  <!-- 全屏按钮 -->
  <button id="fullscreen-btn" class="absolute bottom-4 right-4 bg-white p-2 rounded-full shadow-md">
    <!-- 全屏图标 -->
  </button>
</div>
```

### 4. 游戏选择器模块
```html
<div class="games-selector">
  <!-- 分类标签栏 -->
  <div class="flex space-x-2 overflow-x-auto pb-2">
    <button class="category-tab active">全部</button>
    <button class="category-tab">动作</button>
    <button class="category-tab">益智</button>
    <!-- 更多分类 -->
  </div>
  
  <!-- 游戏网格 -->
  <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4 mt-4">
    <!-- 游戏卡片 -->
    <div class="game-card">...</div>
    <!-- 更多游戏卡片 -->
  </div>
</div>
```

### 5. 用户数据管理模块
使用`localStorage`实现客户端存储：
```javascript
// storage.js
const storage = {
  // 游戏收藏
  saveGame: (gameId) => { ... },
  getFavorites: () => { ... },
  
  // 游戏历史
  addToHistory: (gameId) => { ... },
  getHistory: () => { ... },
  
  // 游戏成就
  saveAchievement: (gameId, achievement) => { ... },
  getAchievements: (gameId) => { ... },
  
  // 主题偏好
  saveTheme: (theme) => { ... },
  getTheme: () => { ... }
};
```

### 6. 主题切换模块
```javascript
// theme.js
const themeToggle = {
  init: () => {
    const currentTheme = storage.getTheme() || 'light';
    document.documentElement.classList.toggle('dark', currentTheme === 'dark');
    
    // 监听主题切换按钮
    document.getElementById('theme-toggle').addEventListener('click', () => {
      document.documentElement.classList.toggle('dark');
      const isDark = document.documentElement.classList.contains('dark');
      storage.saveTheme(isDark ? 'dark' : 'light');
    });
  }
};
```

### 7. 游戏加载模块
```javascript
// gameLoader.js
const gameLoader = {
  loadGame: (gameId) => {
    const gameContainer = document.getElementById('game-container');
    const gameFrame = document.getElementById('game-frame');
    const gameLoader = document.getElementById('game-loader');
    
    // 显示加载动画
    gameLoader.classList.remove('hidden');
    
    // 设置游戏源
    gameFrame.src = `./assets/games/${gameId}/index.html`;
    
    // 记录到历史
    storage.addToHistory(gameId);
    
    // 监听游戏加载完成
    gameFrame.onload = () => {
      gameLoader.classList.add('hidden');
    };
  }
};
```

### 8. 快捷操作栏模块
```html
<div class="game-actions flex space-x-2 mt-4">
  <button id="btn-favorite" class="action-btn">
    <!-- 收藏图标 -->
    <span>收藏</span>
  </button>
  
  <button id="btn-share" class="action-btn">
    <!-- 分享图标 -->
    <span>分享</span>
  </button>
  
  <div class="rating flex">
    <!-- 评分星星 -->
  </div>
</div>
```

### 9. 离线体验模块
使用Service Worker实现基本离线功能：
```javascript
// 注册Service Worker
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/sw.js').then(registration => {
      console.log('SW registered');
    }).catch(error => {
      console.log('SW registration failed:', error);
    });
  });
}
```

## 性能优化策略

### 1. 资源加载优化
- 懒加载图片与iframe
```html
<img loading="lazy" src="..." alt="...">
<iframe loading="lazy" src="..." frameborder="0"></iframe>
```

### 2. CSS优化
- 使用Tailwind的生产构建移除未使用的CSS
- 按需加载非关键CSS

### 3. JavaScript优化
- 延迟加载非关键JavaScript
```html
<script src="js/non-critical.js" defer></script>
```

## 数据存储方案
使用浏览器本地存储作为数据持久化方案：

1. **localStorage**：存储用户偏好设置、收藏游戏列表、历史记录等
2. **sessionStorage**：存储会话级别数据，如当前游戏状态
3. **IndexedDB**（可选）：存储更复杂的结构化数据，如详细游戏进度

## 安全考虑
1. **iframe安全设置**：正确配置sandbox属性
2. **内容安全策略(CSP)**：实施严格的CSP防止XSS攻击
3. **HTTPS服务**：确保所有资源通过HTTPS加载

## 可访问性设计
1. 语义化HTML结构
2. 适当的ARIA角色和属性
3. 键盘导航支持
4. 足够的颜色对比度

## 性能目标
1. 首屏加载时间：小于3秒（理想<1.5秒）
2. 游戏切换时间：小于2秒
3. 交互响应时间：小于100ms
