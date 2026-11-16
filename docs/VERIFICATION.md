# BIKE LAB 最终验收记录

日期：**2026-11-14**。本轮从阶段 5 续做，完成 UI / 导学验收、移动端修复、渲染收尾和阶段 6 交付。

环境：Windows、本地 Python HTTP 服务、Playwright CLI、Chromium 157 与本机 Microsoft Edge 149.0.4022.80。浏览器实际创建 WebGL 场景；通过鼠标、键盘、DOM 操作和只读场景快照交叉检查。移动触摸为 Chromium CDP 事件仿真。

## 验收结果

| 检查 | 结果 | 证据 |
| --- | --- | --- |
| 零件与界面 | 34 个组件；搜索、分类、关联、隔离、隐藏恢复与模型同步 | `output/playwright/phase5-check.js`、`phase5-edge.json` |
| 桌面操作 | 43 项断言通过：三模式、标签、5 段动力路径、9 镜头、快捷键、透视、显示设置、帮助、全屏 | Chromium 首轮执行与 Edge 最终结果 `phase5-edge.json` |
| 导学与巡航 | 34 项断言通过：12 个画面、目标选择和运动、前后翻页、重启、自动推进、暂停继续、参数恢复、巡航与鼠标接管 | `output/playwright/phase5-guide-check.js`，本轮执行结果见 `progress.md` |
| 单件 / 系统拆装 | 独立拆装、系统拆装、隔离保持、左右曲柄分离、动画暂停继续及反向通过 | `output/playwright/phase3-final.json` |
| 重复复装 | 连续 20 轮 `[100,37,82,14,0]` 进度变化后，位置 / 四元数 / 缩放 / 分体位置最大误差 **0** | `output/playwright/phase3-final.json` |
| 传动 | 50 / 34T × 12 档共 **24 组**全部通过；每组轮换一个 30 / 80 / 120 RPM 代表踏频，非 72 组全排列 | `output/playwright/phase4-final.json` |
| 机械关系 | 最大齿比误差约 **5.29e-14**；链条固定 108 节，最大链距与闭合误差 **< 1e-9 m**；后轮联动、前轮静止、脚踏水平、双导轮与路径同位 | `output/playwright/phase4-final.json` |
| 互斥与慢放 | 拆解暂停传动并禁止换挡；完整组装后恢复；0.25× 慢放实测比值约 **0.2569** | `output/playwright/phase4-final.json` |
| 响应式 | 1600×1000、1280×800、820×1180、390×844、360×780 无横向溢出，抽屉和教学可操作，标签避让控件 | `phase5-responsive-check.js` 与 `final-*-尺寸.png` |
| 触摸 | 单指拖动改变镜头；双指扩大距离使镜头靠近模型 | `output/playwright/phase5-responsive-check.js`，本轮实跑通过 |
| 手机修复 | 教学打开后仍可旋转；滚动保留标题与关闭按钮；新章节回到顶部；关闭抽屉不可见且 inert | `output/playwright/phase6-recovery.json`，4 项断言 |
| 错误恢复 | 主动阻断 CDN 与主动丢失 WebGL 上下文均出现重载入口，重载后恢复真实 3D 场景 | `output/playwright/phase6-recovery.json`，4 项断言 |
| 阶段快照 | 原稿 33 组件；phase-1 至 phase-6 均为 34 组件；7 个路径独立启动，运行错误为空 | `output/playwright/phase6-versions.json` |
| 代码与结构 | ES Module 语法检查、Git 空白检查通过；删除 10 条无引用旧样式；运行文件仍为单个 HTML | `node --check`、`git diff --check` |

正常场景的上述最终结果中捕获的页面错误均为 **0**。故障注入时预期出现的模块错误与上下文丢失信息不计为正常运行通过。

## 性能采样

机械传动近景，1600×1000，DPR **1**。每个状态采样 2.5 秒，基于 RAF 时间戳计算平均 FPS 和 P95 帧间隔；数值包含当时操作系统、GPU 和浏览器负载。

| 画质 | 静止 FPS | 传动 FPS | 静止 / 传动 P95 帧间隔 ms | 绘制调用 | 三角形计数 |
| --- | ---: | ---: | ---: | ---: | ---: |
| LOW | 60.0 | 60.0 | 16.8 / 16.8 | 177 | 196348 |
| MEDIUM | 58.8 | 60.0 | 16.9 / 16.8 | 459 | 369818 |
| HIGH | 56.4 | 60.0 | 33.2 / 16.9 | 459 | 369818 |

绘制调用和三角形数包含阴影及后处理 pass，不能解释为模型独立网格数量。DPR 1 下 MEDIUM / HIGH 分辨率相同，但阴影分辨率不同；没有以此声称高 DPI 屏幕达到相同帧率。

证据：`output/playwright/performance-before.json` 与 `performance-final.json`。关闭光晕改为直接场景渲染后，LOW 调用数 178→177，HIGH 关闭光晕 446→445，移除一次全屏处理。没有将采样波动描述为帧率提升。

LOW / HIGH 截图已目视检查，材质色彩正常，链条和机械细节保留。

## 本轮修复与交付

1. 保留并回归上次的模式按钮选择器修复，导学不再因 body 冒泡退出。
2. 工程标签按实际操作控件矩形避让，隐藏出界参考尺寸；调整手机主镜头，为顶部控制留空间。
3. 详情标题与关闭按钮改为 sticky，切换章节回到顶部；教学详情打开时取消场景遮罩；隐藏抽屉不再露出在长页面底部。
4. 关闭光晕时跳过后处理，保留 ACES 与 sRGB；清理旧 UI 的 10 条无引用样式。
5. 保存阶段 5 / 6 独立快照；更新 README、技术说明、计划和本验收记录，纠正旧版组件数、轴距、前盘及链条说明。

主要截图：`output/playwright/final-product.png`、`final-product-390.png`、`final-engineering-360.png`、`final-guide-390.png`、`final-mechanical-820.png`、`final-quality-low.png`、`final-quality-high.png`；爆炸图与传动回归截图位于 `artifacts/phase3-exploded.png`、`artifacts/phase4-running.png`。

## 重现检查

先按 README 启动 HTTP 服务。可选使用 Playwright CLI 运行验收脚本，工具不属于应用依赖：

```powershell
npx --yes --package @playwright/cli playwright-cli -s=bike-lab-check open http://127.0.0.1:8765/
npx --yes --package @playwright/cli playwright-cli -s=bike-lab-check run-code --filename output/playwright/phase5-check.js --raw
npx --yes --package @playwright/cli playwright-cli -s=bike-lab-check close
```

其余脚本同样通过 `run-code --filename` 执行。脚本证据路径按当前 Windows 工作区固定；迁移目录后需相应调整截图路径。

## 已知限制

- 性能为本机短时采样，未做长时间压力测试或实体低端设备测试。
- Edge 最终完成桌面界面回归；完整机械数值、故障恢复和触摸检查使用 Chromium。未验收 Safari、Firefox 和实体手机。
- 几何是参数化展示模型，不代表可制造 CAD、具体商品测量值或真实维修顺序；不模拟载荷、摩擦、换挡瞬态、链条弹性、制动动力学和自由轮滑行。
- 单文件运行仍需联网加载 Three.js；未制作离线引擎包或生产部署。
- WebGL 上下文丢失通过明确提示与手动重载恢复，不自动重建场景。

## 花鼓音效补充验收（2026-11-14）

根目录 `index.html` 新增花鼓模拟声和 0–100% 音量控制，使用 Ego Lite 实际加载 WebGL 与 Web Audio 验证。测试在增益节点之后连接 AnalyserNode 采集输出，除界面状态外，同时检查实际音频信号。

- 初始页面未创建音频上下文，默认音量 40%；用户启动传动后才发声。40% / 100% 的输出 RMS 约为 0.01354 / 0.08441，100% 峰值小于 1，没有削波。
- 音量为 0%、暂停和拆解后，测得 RMS 与峰值均为 0。音量静音期间后轮继续转动；重新启动复用一个上下文和一个循环音源。
- 前后齿数、踏频和演示倍率控制声音速率。0.25× 慢放同步降为四分之一；30 RPM / 34:34 / 0.25× 与 120 RPM / 50:11 / 2× 两个边界通过。
- 鼠标和键盘启停通过；切到其他标签页后暂停并静音，返回不会自行恢复。导学首次进入可解锁音频，运动章节播放、退出导学静音。
- 1600×1000、1280×800、1170×778、1024×768、820×1180、390×844 和 360×780 下音量控件无横向溢出、无重叠，手机控制区可滚动查看；桌面和手机截图已目视检查。
- 主动丢失 WebGL 上下文后，音频上下文关闭，无未处理的 Promise 拒绝；重新加载恢复。

结果文件：`output/hub-sound-check/audio-results.json`、`layout-results.json`、`guide-results.json`。截图：同目录 `desktop.png`、`mobile-control.png`。正常运行错误为 0。仅验证该浏览器的实际音频信号与界面，未验收实体扬声器听感、Safari、Firefox 或实体手机。音色为合成效果，未模拟真实自由轮滑行。

## 视觉与交互升级验收（2026-11-15）

本轮按用户确认实施模式布局、材质灯光和可读性三项。继续使用单个 `index.html`、Three.js r180、同一模型与状态源。环境为 Windows、Ego Lite Chromium、真实 WebGL；手机通过视口和 CDP 触摸仿真检查。

| 检查 | 本轮结果 | 证据 |
| --- | --- | --- |
| 桌面主要流程 | **20 项通过**：产品档案 / 目录、选件和聚焦、三模式分工、全拆复装、运行中换挡、导学与巡航；切回产品会暂停传动 | `output/visual-upgrade/desktop-results.json`、`round-02.mjs`、最终重跑入口 `round-10.mjs` |
| 额外交互 | **14 项通过**：快捷键进入对应工作区、未组装巡航禁用、真实射线命中后派发鼠标点击、单件拆解路由、透视说明避让、三档画质、触摸旋转 / 捏合、导学面板上方仍可旋转 | `output/visual-upgrade/interaction-results.json`、`round-12.mjs` |
| 平板 / 笔记本布局 | 1280×800、1024×768、820×1180，各 3 种模式共 **9 组通过**；无横向溢出，工程标签避让控件；平板选件与导学可操作 | `output/visual-upgrade/responsive-desktop-results.json`、`round-03.mjs` |
| 手机布局与触控尺寸 | 390×844、360×780，各 3 种模式共 **6 组通过**；无横向溢出，主要按钮和档位控件至少 44px；关闭的抽屉 inert，选件和导学可操作 | `output/visual-upgrade/responsive-mobile-results.json`、`round-04.mjs` |
| 视觉检查 | 已查看产品、工程、传动、拆解、导学、窄屏详情与手机完整传动控制区截图；手机近景调整后可同时观察前后传动部件 | `output/visual-upgrade/` 中相应模式 / 尺寸 PNG |
| 静态与自查 | 单 ES Module 语法通过、89 个 HTML ID 无重复、Git 空白检查通过；人工检查相对本轮前快照的行为 diff 与新样式 | `output/visual-upgrade/module.mjs`、`behavior-review.diff` |

上述最终正常流程捕获的页面错误为 **0**。本轮前运行文件保存在 `output/visual-upgrade/before.html`，历史 phase-1–6 快照保持原样。

本轮自查和实测修复了两个由布局分工引出的实际问题：隐藏控制区时，快捷键和单件拆解应进入对应可见工作区；窗口跨过抽屉断点时，布局同步不能依赖非零画布尺寸，也不能在 ResizeObserver 中改变被观察布局。最终通过窗口 resize / 模式 / 导学事件派生布局，ResizeObserver 仅负责渲染尺寸，未添加重试状态。

主要截图：`product-1440.png`、`engineering-1440.png`、`mechanical-1440.png`、`product-390.png`、`engineering-390.png`、`mechanical-390.png`、`mechanical-controls-390.png`、`guide-390.png`，均位于 `output/visual-upgrade/`。

本轮为界面与渲染呈现升级，复验主要机械操作与代表换挡关系，未重跑上一轮 20 次拆装、24 齿数组合和完整性能矩阵。实体手机、Safari / Firefox、长期性能与实体扬声器听感仍未验收；未提交、推送或部署。

## 工程标注与互动导学补充验收（2026-11-16）

完成先前剩余的两项：整车核心标注 / 按系统深入，以及传动因果说明 / 操作式导学。沿用单个 HTML、现有分类与控制器，未新增运行依赖。环境为 Windows、Ego Lite Chromium、真实 WebGL；手机为视口与 CDP 触摸仿真。

| 检查 | 结果 | 证据 |
| --- | --- | --- |
| 桌面功能 | **39 项通过**：默认 6 个核心标注，六系统目录与场景同步、系统拆装、单件拆装、换挡解释、12 个导学画面的操作与结论、退出后参数及场景恢复 | `output/playwright/learning-20261116/desktop-results.json`，`round-15.mjs` |
| 自动导学与布局 | **25 项通过**：等待操作后推进、拆解中退出复装，以及 1600×1000、1280×800、1184×792、1024×768、820×1180、390×844、360×780 × 三模式共 21 组布局 | 同目录 `layout-auto-results.json`，`round-08.mjs` |
| 手机操作 | **20 项通过**：隔离后切换系统、44px 标签和操作按钮、390px 下全部 12 个导学任务、重学重置、360px 下拆解任务自动推进并复装、退出系统同步、换挡解释换行 | 同目录 `mobile-results.json`，`round-14.mjs` |
| 导学入口视觉补检 | **3 组通过**：从产品、工程、机械模式进入导学均收起重复的模式说明，避免与目录入口重叠 | 同目录 `guide-entry-results.json`，`round-18.mjs` |

布局检查覆盖横向溢出、实际可见控件的中心点击命中及标签间重叠。水平滚动视角条中被裁切的按钮不计为当前可见按钮。正常流程捕获的页面错误为 **0**。39 项桌面功能在导学卡片更新逻辑调整后重跑；导学入口说明隐藏调整后另验 3 种模式。布局覆盖与手机交互检查各有独立结果文件。

换挡实测：50T 前盘、80 RPM 踏频，17T → 34T 后飞轮，显示“齿数翻倍，传动比减半”，后轮名义转速约 235 → 118 RPM，运行仪表为 118 RPM；暂停后仪表为 0，并明确保留设定关系。退出导学时，97 RPM、34T 前盘、第 10 档（13T）、2× 倍率及透视开启状态均恢复，教学隔离与拆解清除。

本轮发现并修复：新增解释区使矮窗口的工具栏与视角条重叠，调整为两列工具布局；旧后台暂停逻辑重建导学卡片导致节点与滚动位置变化，改为仅同步播放控件；选择系统时退出旧隔离，避免新系统零件不可见。

卡片原位更新补检：操作完成后按钮节点保持，详情 `scrollTop` 保持 220（`card-regression-results.json`，`round-17.mjs`）。该次补检未捕获后台切换事件，不作为真实后台切换通过的证据。

已目视检查系统标注、移动件 / 装配原位、桌面隔离与拆碟片、桌面和手机换挡结论、手机说明区截图。文件包括 `drivetrain-system-1600.png`、`disassembly-1600.png`、`guide-2-1600.png`、`guide-4-1600.png`、`guide-6-1600.png`、`mechanical-1184.png`、`engineering-390.png`、`guide-shift-390.png`、`guide-360.png`、`drive-explanation-360.png`。

ES Module 语法与 Git 空白检查通过。未重跑历史完整机械性能矩阵，未验实体手机、Safari / Firefox 或实体扬声器；本轮未提交、推送或部署。
