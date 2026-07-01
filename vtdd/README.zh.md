# VTDD

VTDD 是一个用于前端 UI 像素级还原的 Codex skill。它把 Figma 设计、用户截图和浏览器运行时渲染结果整理成可测试的视觉契约，并要求先写失败测试，再实现，再用 DOM 度量和截图验证。

## 适用场景

- 用户要求“视觉 TDD”或“像素级还原”。
- 需要把 Figma frame/node 实现成前端页面或组件。
- 需要用截图、Playwright 或 DOM rect 验证 UI 视觉回归。
- 用户指出“看起来不像设计稿”，需要系统性补齐遗漏的节点、间距、字体、颜色和状态。

## 核心原则

Figma 是节点尺寸和设计 token 的事实来源，用户截图是当前视口下的视觉反馈来源，浏览器运行时测量是应用真实渲染结果的事实来源。三者都需要核对，不能只因为大容器测试通过就声称视觉一致。

## 工作流

1. 收集 Figma URL、node id、截图、目标路由、视口和现有实现文件。
2. 优先读取 Figma context、metadata 和 screenshot；不可用时必须明确说明。
3. 建立完整视觉清单，覆盖所有可见节点、复合行、隐藏 padding、状态和底部区域。
4. 在改生产代码前写失败测试，覆盖 CSS contract、DOM 结构、运行时尺寸和截图证据。
5. 确认红灯后做最小实现。
6. 用真实浏览器 route 测量 card-relative 坐标、尺寸、字体、颜色、gap、padding 和状态。
7. 最终报告使用的 Figma node、红绿测试、运行时测量值、命令结果和未验证风险。

## 安装

复制 `vtdd/` 到 Codex 全局技能目录：

```bash
cp -R vtdd ~/.codex/skills/vtdd
```

安装后重启 Codex，让新 skill 被加载。

## 触发示例

```text
使用 vtdd 按这个 Figma frame 做视觉 TDD，实现这个注册表单模块。
```

```text
这个页面和截图差异很大，按视觉 TDD 全量对比并修复。
```
