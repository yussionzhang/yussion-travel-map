# 走遍世界旅行地图：跨电脑维护与自动发布指南

本项目是一个静态 HTML 旅行档案网站。以后在工作电脑、家用电脑或其他智能体上维护时，统一遵循本文件。

## 一、换电脑后的首次准备

安装 Git，并确认 GitHub 账号 `yussionzhang` 有仓库写入权限。

```bash
git clone https://github.com/yussionzhang/yussion-travel-map.git
cd yussion-travel-map
```

项目入口是根目录的 `index.html`。本地预览可以直接双击打开，也可以启动简单服务器：

```bash
python3 -m http.server 8000
```

然后打开 `http://localhost:8000/`。

首次配置身份（只需做一次）：

```bash
git config --global user.name "yussion"
git config --global user.email "你的 GitHub 邮箱"
```

如果 HTTPS 推送要求密码，不要输入 GitHub 登录密码。应使用 GitHub Personal Access Token，或改用 SSH：

```bash
git remote set-url origin git@github.com:yussionzhang/yussion-travel-map.git
```

## 二、每次开始修改前

先同步远程版本，避免家里和工作电脑互相覆盖：

```bash
cd "/你的路径/yussion-travel-map"
git pull --rebase origin main
git status --short
```

只处理用户明确要求的文件。不要把 `.DS_Store`、未要求删除的照片或其他并行修改一起提交。

## 三、照片更新标准

### 1. 路径与引用

- 图片放在对应目录，例如 `assets/photos/namibia/`、`assets/photos/maldives/`。
- HTML 中使用相对路径，保证 GitHub Pages 和本地打开都能工作。
- 文件名尽量使用英文、数字、短横线，避免空格和特殊字符。
- 更新文件夹后必须检查 HTML 是否仍引用已删除的旧文件。

### 2. 去重

更新照片后必须做两层去重：

- 文件层：同一张照片不能因为不同文件名重复存在。
- 页面层：同一文件不能被同一个分页重复引用。

可用以下命令检查引用：

```bash
rg -n "assets/photos/目标文件夹" index.html pages
```

不要仅凭文件名判断是否重复；应结合图片尺寸、缩略图或哈希检查。重复照片必须删除引用或保留更高清的一张。

### 3. 人物和主体完整显示

人物、建筑、动物等主体不能被画框裁掉。默认原则：

```css
object-fit: contain;
object-position: center;
```

如果用户要求“不要留空白”，不要简单改成 `object-fit: cover`，因为它可能切掉人物。应根据照片比例调整画框比例，必要时为横图和竖图使用不同布局。

验收重点：

- 人物头部、身体和关键动作完整；
- 建筑顶部、桥梁、船只等主体不被裁掉；
- 画框内没有明显不自然的空白；
- 手机端和桌面端都检查一次。

## 四、视频更新标准

所有网页视频必须明确给用户播放控制，避免自动播放、卡顿和无法暂停：

```html
<video controls preload="metadata" playsinline poster="封面图路径">
  <source src="视频路径" type="video/mp4">
</video>
```

默认不要使用 `autoplay`、`muted`、`loop`，除非用户明确要求背景视频自动播放。视频更新后必须检查：

- 页面上能看到播放/暂停、进度、音量控件；
- 点击播放后能正常播放；
- 可以暂停和拖动进度；
- 视频比例与画框匹配，不出现黑边或大面积空白；
- 使用高清原文件，不误用低清预览文件；
- 手机端也能点击播放。

如果视频很大，要保留原画质但合理使用 `preload="metadata"`，避免页面一打开就全部加载导致卡顿。

## 五、地图城市与省份联动规则

在国内地图新增城市时，必须同时完成两件事：

1. 在城市位置增加城市点和文字标签；
2. 将该城市所属省级行政区改为“已到访”颜色。

例如新增长春时，长春所在的吉林省也必须点亮；新增汕头时，广东省必须保持已到访状态。

标签不能与附近城市重叠。若空间紧张：

- 保留城市点在真实位置；
- 将文字移到空白处；
- 加短引线连接城市点和文字。

地图修改后必须查看实际渲染图，不能只检查代码或坐标。

## 六、响应式与移动端验收

每次修改图片、视频、地图卡片后，都要检查：

- 桌面宽屏；
- 普通笔记本宽度；
- 手机窄屏；
- 横向和纵向照片；
- 页面顶部轮播、分页内容和地图卡片。

特别注意：`object-fit: cover`、固定高度、固定宽高比和 `overflow:hidden` 都可能导致内容被裁掉。地图需要完整显示时，优先使用经过裁剪的专用素材加 `object-fit: contain`，不要直接拿一张超宽原图硬塞进窄卡片。

## 七、标准工作流程

每次用户提出更新时，智能体按以下顺序执行：

1. 读取当前 `git status`，保护用户已有未提交修改；
2. 扫描新增、删除和改名的照片/视频；
3. 检查 HTML 引用是否失效、重复；
4. 修改页面或素材；
5. 本地预览并目视检查主体完整、标签不重叠、视频控件生效；
6. 只暂存本次任务相关文件；
7. 提交清晰的 commit；
8. 推送到 `origin main`；
9. 用 `git ls-remote origin refs/heads/main` 确认远程已更新；
10. 告知用户提交号、修改内容和强制刷新方法。

推荐命令：

```bash
git add index.html pages/相关页面.html assets/maps/相关素材.png
git commit -m "Update travel map content"
git push origin main
git ls-remote origin refs/heads/main
```

如果 GitHub Pages 尚未显示最新内容，等待构建完成后用 `Command + Shift + R` 强制刷新。

## 八、给另一个 Codex / 智能体的标准提示词

复制下面这段作为开场指令：

> 这是我的静态旅行地图网站项目。请先读取项目根目录的 `CODEX_WORKFLOW.md`，再检查当前 `git status`，保护所有未提交的用户修改。按照我的本次要求修改网站，并完成本地预览验收后，只提交本次相关文件并自动推送到 GitHub 的 `main` 分支。请特别检查：图片主体和人物不能被裁切；手机端不能显示不全；更新照片后不能有重复；新增城市时对应省份必须同步点亮；视频必须有可见且可用的 controls，默认不要自动播放；图片和视频更新后要检查引用路径、清晰度、比例和空白；地图标签不能重叠。完成后报告修改文件、commit 编号、推送结果和需要强制刷新的方法。

## 九、完成报告模板

每次完成后，智能体应明确报告：

- 修改了什么；
- 检查了哪些图片/视频和页面；
- 是否确认人物、主体、城市标签、省份颜色和视频控件正常；
- Git commit 编号；
- 是否已成功推送到 GitHub；
- 用户如何刷新线上页面。

