# WebView Demo · H5 欢迎页

一个轻量、温暖、响应式的 H5 欢迎页，面向 Android WebView 与桌面浏览器。页面为单文件内联 CSS/JS，无 CDN、外部字体或外部图片，离线也能正常渲染。

## 目录

- `web/index.html`：欢迎页（含内联样式、交互与埋点占位）
- `web/favicon.svg`：本地图标
- `web/.nojekyll`：告诉 GitHub Pages 跳过 Jekyll 处理
- `.github/workflows/deploy-pages.yml`：从 `web/` 发布到 GitHub Pages 的工作流

## 本地预览

在项目根目录（包含 `web` 与 `.github` 的目录）打开 PowerShell：

```powershell
python -m http.server 8000 -d web
```

然后访问 <http://localhost:8000/>，按 `Ctrl+C` 停止。也可以直接双击 `web\index.html` 用浏览器打开（`file://` 下功能正常）。

## H5 埋点与 Android 原生桥

`web/index.html` 内定义了全局的 `track(eventName, params)`：

- 页面运行在 WebView 且发现 Android 注入的 `window.UmengBridge` 时，事件通过原生桥进入友盟 U-App。
- 在普通浏览器中，Android 桥不存在，函数自动回退到 `console.debug`，页面可独立预览。
- 页面加载触发 `page_view`，点击 CTA 触发 `cta_click`。
- 页面上的通道状态会显示当前使用「Android 原生桥」还是「浏览器控制台」。

桥梁调用契约：

```js
window.UmengBridge.track(
  'cta_click',
  JSON.stringify({
    page: 'welcome',
    area: 'hero-cta',
    label: '开始体验'
  })
);
// 返回 JSON 字符串，例如：
// {"ok":true,"transport":"android-bridge","event":"cta_click"}
```

Android 端只暴露这一个桥对象，并校验：

- 当前页面必须是配置的 HTTPS 主机；
- 事件名和参数键只允许字母、数字、`_`、`.`、`-`；
- 最多 20 个参数，单个值最多 256 字符；
- 桥内通过 `MobclickAgent.onEvent(...)` 发送到友盟 U-App。

浏览器里打开时不会有桥，只会看到控制台日志，这是预期行为。Android 源码中的实现位于 `android/app/src/main/java/com/lovelyqianyun/webviewdemo/UmengWebBridge.java`。

## 首次发布到 GitHub Pages

1. 在 GitHub 新建**公开**仓库 `webview-demo`（保持空仓库，不初始化 README）。
2. 在项目根目录初始化并推送（本任务只生成文件，未执行任何 git 操作）：

   ```powershell
   git init
   git add .
   git commit -m "feat: add H5 welcome page and Pages deploy workflow"
   git branch -M main
   git remote add origin https://github.com/Lovely-Qianyun/webview-demo.git
   git push -u origin main
   ```

3. 打开仓库 **Settings → Pages → Build and deployment**，将 **Source** 设为 **GitHub Actions**。
4. 打开 **Actions**，等待 `Deploy Pages` 完成；也可以在 Actions 中手动执行 `Run workflow`（即 `workflow_dispatch`）。如果首次推送时尚未开启 Pages、导致工作流失败，开启 Pages 后重新运行一次即可。
5. 预期访问地址：<https://lovely-qianyun.github.io/webview-demo/>

工作流会在推送到 `main` 且 `web/**` 或工作流文件发生变化时自动触发。

## 注意

- 如果仓库名改变，GitHub Pages URL 会随之改变；Android WebView 中加载的地址需要同步修改。
- URL 中的账号与仓库名通常使用小写；最终以仓库 Pages 设置页展示的地址为准。
- 若之后把项目改成 `Lovely-Qianyun.github.io` 这类用户主页仓库，页面根地址会变为 `https://lovely-qianyun.github.io/`，部署目录也需相应调整。
- 页面文案、CTA、`track()` 函数体和 `favicon.svg` 都可以直接替换；替换后重新推送即可发布。
