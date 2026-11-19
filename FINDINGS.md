# 审计记录

审计范围：`index.html` 的资源引用、内联 JavaScript、HTML 渲染入口及答案显隐状态。未发现 high 或 medium 级别问题。

| 严重程度 | 位置 | 风险 | 处理 |
| --- | --- | --- | --- |
| low | `index.html:172`、`esc()`、`renderQuestions()`、`exCard()`、`textCard()`、`transCard()`、`unitSection()` | 字符串直接拼入 `innerHTML`，原 `esc()` 不做转义；当前数据均为本地常量，没有发现可由外部输入触发的 XSS，但未来导入内容可能被当作 HTML 执行。 | 实现文本转义并用于普通文本字段；富文本仅保留现有材料实际使用的固定标记（粗体、换行、英文段落、填空和不换行空格），其余一律转义。答案继续通过 `textContent` 写入。 |
| low | `index.html:63`（原 `.show-ans .ans,.ans.on` 规则）、`setItem()` | 先显示整节答案再收起单道选择题时，祖先的 `.show-ans` 仍强制显示答案，按钮与可见状态不一致。 | CSS 改为仅按 `.ans.on` 显示，服从已有逐题状态；未改答案或 JavaScript 状态逻辑。 |
| low | `docs/preview-celeste.png`（截图生成步骤） | 环境限制导致无法生成截图，也无法完成真实浏览器视觉和布局验证。 | reported only：按要求执行截图命令后，Chromium 以退出码 133 失败，报错 `setsockopt: Operation not permitted (1)`。未生成或伪造预览；README 保留指定截图引用并注明失败。 |

未发现 `eval`、`Function` 构造器、字符串定时器、`document.write`、外部脚本/样式、混合内容资源请求或硬编码密钥、令牌、凭据。图标中的 `http://www.w3.org/2000/svg` 是 SVG 命名空间，不是网络资源请求。页面没有作答输入、自动判分或持久化状态；分值是考试说明，各项合计 100 分。

验证：原始学习数据脚本、思政词表及 base64 图片逐字节比对一致；除样式、配色元信息和渲染辅助代码外，其余源文件内容一致。使用 Node.js VM 和临时 DOM 桩验证了 101 道题、67 个空、2 道双空题、9 组整节控制和 20 个词条；逐题反复显隐、整节显隐及两者交叉操作的状态检查通过。原版与改版生成的 164 个 HTML 片段经 HTML 解析后，文本、标签和属性一致；渲染入口的注入探针也通过。上述检查不代替真实浏览器验证；临时检查文件已清理。

在可运行 Chromium 的环境中，于仓库根目录重试：

```sh
mkdir -p docs
/home/hatch/.cache/ms-playwright/chromium-1254/chrome-linux64/chrome --headless=new --disable-gpu --no-sandbox --hide-scrollbars --window-size=1280,900 --virtual-time-budget=8000 --screenshot=docs/preview-celeste.png file://$PWD/index.html
```
