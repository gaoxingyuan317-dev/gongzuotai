# WorkBuddy 全权交接文档

> 本文档是占星工作台项目的完整交接文档。WorkBuddy (TRAE) 在任何新对话中读取本文档即可全权接手所有工作台的开发、维护和优化工作。

---

## 一、用户信息

- **职业**: 占星师
- **专业领域**: 占星、星座、月运/周运、因果业力
- **主平台**: 抖音，跨平台同步
- **目标受众**: 25-30岁女性白领用户，主要用苹果手机
- **内容特点**: 利他、前5秒抓人、借鉴达人热点
- **变现方式**: 1对1占星看盘 + 占星课程
- **技术偏好**: GUI操作，双击即可，避免命令行
- **所有技术维护和功能开发由 TRAE (WorkBuddy) 负责，用户不碰代码**

---

## 二、项目核心信息

### GitHub 配置
- **用户名**: `gaoxingyuan317-dev`
- **仓库名**: `gongzuotai`
- **分支**: `main`
- **Token**: 需用户提供有效的 Personal Access Token (repo权限)
- **GitHub Pages 地址**: `https://gaoxingyuan317-dev.github.io/gongzuotai/`
- **Pages 配置**: 从 main 分支根目录启用

### 本地文件路径
- **项目根目录**: `C:\Users\Gaoyang\AppData\Roaming\TRAE SOLO CN\ModularData\ai-agent\work-mode-projects\6a6761d8a091770b04e95bf6\占星内容个人永久工作台\`
- **主入口文件**: `index.html` (约 300KB，单文件应用)
- **控制台文件**: `工作台控制台.html` (约 300KB，功能与 index.html 基本一致)
- **数据文件**: `workbench_data.json` (静态备份)
- **PWA配置**: `manifest.json`

### 上传脚本
- 路径: `c:\Users\Gaoyang\.trae-cn\work\6a6761d8a091770b04e95bf9\upload_both.js`
- 用途: 将 index.html 和 工作台控制台.html 上传到 GitHub
- 运行: `node upload_both.js`
- 注意: 中文文件名需要 `encodeURIComponent()` 编码

---

## 三、技术架构

### 前端架构
- 纯原生 HTML + CSS + JavaScript，无框架
- 单文件应用 (SPA)，所有代码在 index.html 内
- PWA 支持 (manifest.json + Service Worker 自动注销)

### 数据存储 (四层保护)
1. **localStorage** (主存储) - key: `astro_organ_workbench_v3`
2. **workbench_data.json** (静态备份) - 同目录文件
3. **IndexedDB** (全量动态备份) - 数据库: `astro_workbench_db`，store: `diaries`
4. **GitHub 云同步** - workbench_data.json 上传到仓库

### 数据结构 (state 对象)
```javascript
state = {
  modules: ["daily","content","editing","viral","writing","brain","skills","library"],
  activeModule: "library",  // 默认主页是图书馆
  // 八大模块数据 (数组，每项有 id/text/created 等字段)
  daily: [],      // 每日完成任务
  content: [],    // 内容灵感
  editing: [],    // 视频剪辑
  viral: [],      // 爆款视频
  writing: [],    // 灵感知识库
  brain: [],      // 第二大脑
  skills: [],     // Skills管理
  library: [],    // 图书馆
  // 日记系统
  diaries: [],           // 日记数组 {id, date, time, title, content}
  selectedDiaryId: null,
  diary_chat: [],        // 日记AI对话
  deletedIds: [],        // 已删除ID追踪 (防同步恢复)
  // 图书馆扩展
  bookNotes: [],         // 读书笔记
  crucible: [],          // 思想熔炉
  questions: [],         // 问题库
  // 其他
  fileSyncConnected: false,
  dataVersion: "2026-08-03-questions"
}
```

### 模块定义
| 模块key | 图标 | 名称 | 类型 |
|---------|------|------|------|
| daily | ✅ | 每日完成任务 | tasks |
| content | ⛏️ | 内容灵感 | cards |
| editing | ✂️ | 视频剪辑 | cards |
| viral | 🔥 | 爆款视频 | cards |
| writing | 💡 | 灵感知识库 | cards |
| brain | 🧠 | 第二大脑 | brain |
| skills | 🛠️ | Skills管理 | skills |
| library | 📚 | 图书馆 | cards |

### 图书馆分类
| key | 名称 | 图标 | 颜色 |
|-----|------|------|------|
| astrology | 占星 | 🔮 | #6b46c1 |
| karma | 因果 | ♾️ | #2d7d5b |
| growth | 心灵成长 | 🌱 | #d9744a |
| relationship | 亲密关系 | 💗 | #c54b6c |

---

## 四、硬性约束 (必须遵守)

### 界面
- 默认主页是**图书馆**页面，不是每日计划
- 右上角只保留两个按钮：**☁️云同步** 和 **🔍搜索**
- 左侧侧边栏保留，可切换所有模块
- 模块支持拖拽排序
- AI助手用浮动层 (🤖助手按钮触发)，不常驻右栏
- 界面风格参考苹果备忘录，简洁不杂乱
- 不添加大段说明文字、复杂信息块、底部对话框

### 日记系统
- 苹果备忘录风格两栏布局 (左列表+右编辑器)
- 选中项暖黄色高亮 (#ffe8b3)
- 标题"记录"可折叠/展开，带旋转动画和状态持久化
- 日期分组显示 (今天/昨天/具体日期)
- 字体用 SF Pro 系统字体
- **删除方式**: 长按0.5秒触发，卡片缩放+红色反馈+确认框
- 移除了左滑删除和悬停删除按钮
- 长按不触发系统复制菜单
- 保存后0.6秒自动返回列表，显示"已保存"提示
- 移动端初始只显示列表，点日记才打开编辑区
- **deletedIds 防恢复**: 删除日记时把ID加入 state.deletedIds，所有数据合并逻辑(7处)都要检查 !isDeletedId()

### 全局搜索
- 🔍按钮打开搜索弹窗
- 实时搜索所有内容类型：书籍、读书笔记、日记、灵感、内容灵感、爆款、剪辑、任务、技能、思想熔炉、问题库
- 搜索结果带类型标签和关键词高亮
- 点击结果跳转到对应模块

### GitHub 同步
- 同步前先拉取云端数据合并，再上传 (防止覆盖)
- saveDiaryNow() 保存后自动触发 githubAutoSync() (3秒延迟)
- 合并逻辑只补充本地没有的数据，不覆盖已有
- 所有合并处必须检查 deletedIds
- Service Worker 自动注销确保加载最新代码
- 版本检测机制: localStorage 存 `_page_version`，版本号变化时清除缓存强制刷新

### 飞书同步
- 飞书多维表格"占星工作台日记"用于手机端日记录入
- 字段: title, content, record time(自动), category(diary/inspiration/voice/material)
- 同步是增量-only，只添加飞书新条目，不覆盖本地
- 每3小时自动同步，也可手动触发
- 飞书日记ID以 `feishu_` 开头

### 内容灵感模块
- 每天早上9点自动更新平台爆款内容
- 知乎搜索: 整理高热问题、高赞回答、评论区痛点
- 小红书搜索用官方站内搜索: `https://www.xiaohongshu.com/search_result?keyword=关键词&source=web_search_result_notes`
- 小红书搜索按钮是单个按钮，无单独登录按钮，灰色标签不显示"需登录"

---

## 五、工程约定

### 文件夹结构
```
占星内容个人永久工作台/
├── 00_工作台入口/          # 8栏归类规则、README、模块映射、账号定位
├── 01_内容挖掘开发/        # 选题池、内容灵感、栏目库、关键词库
├── 02_视频剪辑/           # 剪辑SOP、封面标题库
├── 02_内容灵感/           # 每日爆款追踪
├── 03_爆款视频搜集/        # 对标账号库、爆款拆解表
├── 04_灵感知识库/         # 钩子库、口播模板、灵感记录卡、写作流程
├── 05_第二大脑/           # 知识卡片、产品经理规则、账号优化、粉丝问题库
├── 06_Skills管理/         # AI提示词库、GitHub下载的Skills (MediaCrawler等)
├── 07_数据复盘/           # (数据复盘相关)
├── 08_图书馆/            # (图书馆相关)
├── index.html            # 主工作台入口 (约300KB)
├── 工作台控制台.html       # 控制台 (约300KB，功能基本一致)
├── workbench_data.json    # 数据备份
├── manifest.json          # PWA配置
└── WORKBUDDY_HANDOFF.md   # 本交接文档
```

### 关键函数清单
| 函数名 | 用途 |
|--------|------|
| `loadState()` | 加载localStorage数据，补齐新模块 |
| `saveState()` | 保存到localStorage + IndexedDB + 触发GitHub同步 |
| `switchModule(key)` | 切换模块，默认 "library" |
| `renderModule()` | 渲染当前模块 |
| `deleteDiary(id)` | 删除日记 + 加入deletedIds |
| `isDeletedId(id)` | 检查ID是否在deletedIds中 |
| `saveDiaryNow()` | 保存日记，带错误处理和反馈 |
| `saveDiaryAndExit()` | 保存并0.6秒后返回列表 |
| `githubPushNow()` | 上传到GitHub (先拉取合并再上传) |
| `githubPullOnLoad()` | 页面加载时从GitHub拉取 |
| `githubAutoSync()` | 3秒延迟自动同步 |
| `restoreDiariesFromFile()` | 从workbench_data.json恢复 |
| `idbBackupDiaries()` | IndexedDB全量备份 |
| `idbRestoreDiaries()` | IndexedDB恢复 |
| `loadFeishuDiaries()` | 从飞书拉取日记 |
| `openGlobalSearch()` | 打开全局搜索弹窗 |
| `doGlobalSearch(query)` | 执行全局搜索 |
| `diaryLongPressStart/Move/End()` | 长按删除手势处理 |
| `diaryHandleClick(id)` | 长按后拦截click事件 |
| `smartClassify()` | 智能归类粘贴的内容 |

### 图书馆特有功能
- 思想熔炉: 不是独立页面，在每本书详情里显示关联话题
- AI自动识别书籍笔记与思想熔炉话题的关联 (关键词匹配评分)
- 全局思想熔炉管理只在图书馆网格视图出现
- 每本书详情卡有per-book熔炉关联区
- AI荐书引擎: 分析书架、笔记、熔炉、问题库，生成阅读画像，从43本书数据库推荐
- 作者名标准化: 去空格、中点(·)、句点(.)、连字符(-)，转小写合并重复

---

## 六、踩过的坑 (避免重复)

1. **服务器同步覆盖导致数据丢失** - 已移除服务器同步，改用GitHub
2. **restoreDiariesFromFile()未在初始化调用** - localStorage清空时日记丢失，已修复
3. **GitHub用户名包含仓库路径** - 必须分开用户名和仓库名字段
4. **变量名错误 LIB_CATS vs LIB_CATEGORIES** - 导致JS崩溃，已修复
5. **搜索框用oninput导致频繁重渲染** - 改为回车或按钮触发
6. **saveDiaryNow()静默失败** - 加了try-catch和用户反馈
7. **删除日记被同步恢复** - 用deletedIds追踪，7处合并逻辑都检查
8. **koa-connect wrapper导致ctx泄漏** - 需原生Koa重写
9. **移动端Service Worker缓存** - 用无痕模式或清除站点数据解决
10. **中文文件名上传GitHub** - 需要encodeURIComponent编码

---

## 七、操作指南

### 修改工作台代码
1. 读取 `index.html` 和/或 `工作台控制台.html` 的相关部分
2. 用 SearchReplace 修改代码
3. 两个文件保持同步 (功能一致)
4. 运行 `node upload_both.js` 上传到GitHub
5. 更新版本号 `CURRENT_VERSION` 触发缓存清除

### 版本号更新
在两个HTML文件末尾的版本检测脚本中:
```javascript
var CURRENT_VERSION = '20260804_library_search'; // 改这个值
```

### 上传脚本位置
```
c:\Users\Gaoyang\.trae-cn\work\6a6761d8a091770b04e95bf9\upload_both.js
```

### 临时工作目录
```
c:\Users\Gaoyang\.trae-cn\work\6a6761d8a091770b04e95bf9\
```

### 项目记忆位置
```
c:\Users\Gaoyang\.trae-cn\memory\projects\aoyang-AppData-Roaming-TRAE-SOLO-CN-ModularData-ai-agent-work-mode-projects-6a6761d8a091770b04e95bf6-uf8p8o--p2-3303740f6f4122f17093\
├── project_memory.md    # 项目级记忆
└── YYYYMMDD/
    ├── session_memory_*.jsonl  # 会话级记忆
    └── topics.md               # 话题级记忆
```

### 用户级记忆
```
c:\Users\Gaoyang\.trae-cn\memory\user_profile.md
```

---

## 八、后续开发方向

- 工作台所有功能优化和新功能开发由 TRAE 负责
- 用户只需用中文描述需求，TRAE 负责全部技术实现
- 修改后自动上传GitHub，更新版本号触发缓存清除
- 两个HTML文件 (index.html / 工作台控制台.html) 需保持同步
- 遇到移动端缓存问题: 版本号更新 + Service Worker注销 + caches.delete()

---

## 九、当前版本状态

- **数据版本**: `2026-08-03-questions`
- **页面版本**: `20260804_library_search`
- **最后更新**: 2026-08-04
- **当前主页**: 图书馆
- **右上角按钮**: ☁️云同步 + 🔍搜索
- **日记删除**: 长按0.5秒
- **全局搜索**: 已实现，搜全部内容类型

---

> 本文档随项目更新而更新。WorkBuddy在接手新对话时，应先读取本文档了解全部背景，再根据用户需求进行开发。
