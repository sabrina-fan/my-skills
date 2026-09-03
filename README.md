# My Skills 整理与分享

我开发过程中使用的全部 skill，区分社区版（别人造的）和我沉淀的（自己整理的），方便复用和分享。

---

# 一、Matt Pocock Skills

仓库：https://github.com/mattpocock/skills

## 安装

克隆仓库，把 engineering、productivity、misc 三类的 skill 复制到**当前项目**的 skill 目录下。不要安装 in-progress 和 deprecated。

## 删除

删除以下 skill：wayfinder、wizard、to-questionnaire、wait-what、git-guardrails-claude-code、migrate-to-shoehorn、scaffold-exercises、setup-pre-commit、claude-handoff、implement-spec、loop-me、retro、setup-ts-deep-modules、writing-beats、writing-fragments、writing-shape。同步删掉 ask-matt 里指向这些 skill 的路由描述。

## 主线用法

ask-matt → grill-with-docs → to-spec → to-tickets → implement(+tdd) → code-review，由 domain-modeling / codebase-design 提供共享术语。

辅助：diagnosing-bugs、prototype、research、resolving-merge-conflicts、improve-codebase-architecture、triage。元层：handoff、teach、writing-for-agents、grilling。

---

# 二、uni-app 小程序的 wot-design-uni 组件接入

`wot-design-uni` 是 Vue3 + uni-app 的多端组件库。三步接好，模板里直接写 `<wd-xxx>` 即可，无需 import。

1. 装依赖：`wot-design-uni`、`@uni-helper/uni-app-types`、`@dcloudio/types`（uni-app 的官方类型）。
2. `src/pages.json` 配 easycom，实现组件自动按需引入：

   ```json
   "easycom": {
     "autoscan": true,
     "custom": { "^wd-(.*)": "wot-design-uni/components/wd-$1/wd-$1.vue" }
   }
   ```

3. `tsconfig.json` 的 `types` 加 `@uni-helper/uni-app-types`，`vueCompilerOptions.plugins` 加 `@uni-helper/uni-app-types/volar-plugin`，补齐模板类型提示。

主题换肤：在 `App.vue` 的 `page` 选择器上用 CSS 变量定义设计 token，业务样式和组件主题都引用它。用 `--wot-color-theme: var(--你的主色)` 把组件主色收敛到项目色板，改一处全站生效。样式单位统一用 `rpx`。

配套 skill：装了 `wot-ui-v2` 这个 skill 后，agent 处理 `wd-*` 组件代码时会自动查询组件库的离线知识库（属性、事件、插槽、样式变量），覆盖组件选型、API 查询、主题定制、常见坑排查。

---

# 三、社区 skill

## beautiful-ui（AI 原生界面复刻）

[beautifului.dev](https://www.beautifului.dev/) 是一套给 AI 应用的 UI 原语：流式文本（Streaming Text）、思考轨迹（Thinking）、人审确认卡（Approval Card）、提示输入栏（Prompt Bar）、洞察卡组（Insight Cards）等，自带 shimmer、闪烁光标、滑入悬停等动画，MIT 协议，copy-paste 即用。它面向的是 React/Web 技术栈。

**怎么复刻到 uni-app / wot-design-uni / Vue 等其它技术栈**——核心是拆交互、不抄样式。每个组件先抽出它的核心状态变量（比如流式文本的本质是 `text` + `active` 两个 prop——输出中显示光标、结束消失），状态对了换什么框架都一样；动画尽量用原生 CSS（`@keyframes`、小程序不支持的就降级）；宿主框架已有的原子控件（图标、加载、弹层）直接复用，组件逻辑自己写；颜色/圆角收敛到 CSS 变量统一换肤。具体每种 AI 组件的复刻方式要看它自己的交互特性，没有固定模板。

## RTK（Rust Token Killer）

装好后，在所有 agent 的全局配置（`AGENTS.md` 或 `CLAUDE.md`）里加一条规则：所有 shell 命令前加 `rtk` 前缀（如 `rtk git status`、`rtk npm run build`），它会压缩命令输出、节省 token。用 `rtk gain` 看节省统计。

## ponytail

AI 编码 agent 的"懒惰资深开发"规则。强制走一把梯子选最简方案：先问这事该不该做（YAGNI）→ 代码库里有没有现成的 → 标准库能不能干 → 平台原生功能够不够 → 已装的依赖能不能解决 → 能不能一行搞定 → 最后才写最少能跑的代码。禁止未要求的抽象、无用脚手架、"以后再说"的预留代码。支持 `/ponytail lite|full|ultra` 三档强度，默认 full，任何编码任务都自动生效。修 bug 时强制找根因而非补症状。

## pristine（纯净原则）

四层因果链对抗熵增——思想→规划→执行→输出，上层不纯会逐层放大成下游冗余。核心准则：一切都像第一次写那样写。思想层追溯前提、不藏假设；规划层目标先行、拒伪需求、先校验后执行；执行层只做计划内的事、工具最小化、每轮重述状态；输出层产物和对话都要纯净、清晰、节约。跨平台适配 Claude Code / Codex / OpenCode 等，在实现、重构、修 bug、写文档、做计划时自动触发。

## anti-slop

一组 Oxlint 规则，拦截 AI 生成代码的坏味道：链式类型断言、`unknown` 滥用、`Record<string, unknown>` 泛滥等。vendor 方式安装（源码拷进项目，不是 npm 依赖）。先判断项目是否适用（要有 TS/JS 源码），按框架关掉冲突规则（NestJS 关 `no-object-parameters`、测试 mock 关 `no-module-mocking` 等），再配置、试跑、处理报错。

## ego-browser

给 AI agent 用的浏览器自动化能力。基于 Chromium（ego-lite），agent 在隔离空间里复用用户的登录状态，不和人抢浏览器。通过 `ego-browser nodejs <<'EOF'` heredoc 调用，内置 snapshotText/click/js/cdp 等 helper 驱动真实浏览器：打开页面、填表单、点按钮、截图、抓数据、测 web 应用。优先于其它 web fetch/浏览器工具使用。

## vibehub

Vibe Coding 时的"术语翻译器"。用户用口语描述效果（"鼠标放上去有个小提示""点完变一下颜色"），它照常完成代码，同时在结果里点出对应的专业术语（tooltip、active 态等）并给通俗解释。边干活边补术语，不打断开发流程。

## finesse-ui

做"不廉价"的高品质 Web 界面。按用途分流：**brand 类**（落地页/品牌站/hero 页，靠 WebGL/Three.js/GSAP 做视觉奇观）vs **product 类**（后台/工作台/数据表/发布向导，重清晰、密度、可用性）。自带配色库（带色温的中性色阶 + 16 强调色）、反 AI-slop 黑名单、审计清单。支持动词命令（audit/bolder/soul/animate 等）做定向迭代。

## neon / neon-postgres

Neon 平台向导。Neon 现在不只是 Serverless Postgres，还打包了 Auth（托管 Better Auth）、对象存储（S3 兼容）、计算函数、AI Gateway，全部可分支。`neon` 是总览/路由，`neon-postgres` 是 Postgres 专项（连接池、分支、自动缩放、scale-to-zero、即时恢复）。核心工作流是"分支优先"。

## writing-great-skills

写 skill 的方法论参考。核心讲"可预测性"——agent 每次走同样流程而非产出同样输出才是美德；区分 model-invoked（靠描述自动触发、占 context）vs user-invoked（手敲、占记忆）两种调用方式该怎么选。写好 description 的原则（前置关键词、列触发分支、每个字都值 context load）。

---

# 四、我沉淀的 skill

以下都是实际项目开发过程中踩坑、提炼、泛化出来的，不是社区来源。均已开源（GitHub：sabrina-fan），克隆后把对应目录复制到 skill 目录即可使用。

## wechat-miniprogram-ui-restore

仓库：https://github.com/sabrina-fan/wechat-miniprogram-ui-restore

把 UI 设计稿（Figma 等）高保真还原成微信原生小程序页面（WXML/WXSS/JS）的 skill。核心流程：先看懂设计稿（页面类型、滚动/固定区、原生组件、设计宽度基准、提取共享 token、识别所有图标）→ 先搭共享布局基元（自定义导航组件、官方 tabBar、安全区计算）→ 先做静态布局再加动态状态（加载/空/错误/禁用/骨架屏）→ 真机验证（刘海屏、底部指示条、小屏高密屏）。关键技术点：自定义导航用 `wx.getMenuButtonBoundingClientRect()` 取胶囊坐标动态算高度，不硬编码；右侧胶囊区域是系统保留区，业务按钮挪到左侧；截图或真机检查前必须清 DevTools 缓存重新编译。

## rebuild-miniprogram

仓库：https://github.com/sabrina-fan/rebuild-miniprogram

改完小程序代码后，走完整的"重建 + 刷新微信开发者工具"流程，不接受热重载、旧产物或命令成功就算通过。自动探测项目的构建命令和 mp-weixin 产物路径（uni-app 一般在 `dist/build/mp-weixin`），校验 `app.json`/`project.config.json` 存在才往下走；清缓存只清 file/compile（不碰 storage/auth/all）；Cmd-Q 彻底退出开发者工具；只导入生成的 mp-weixin 目录（不导 repo 根/apps/dist）；编译后验证窗口标题、无报错、模拟器渲染、项目名四要素，失败保留证据不假装成功。

## wechat-devtools-automation-recovery

仓库：https://github.com/sabrina-fan/wechat-devtools-automation-recovery

微信开发者工具自动化连不上、白屏、真机空白时的诊断与恢复。先清 file/compile 缓存重新编译再排查（不拿旧产物诊断）；先分类故障——别因自动化连不上就重启后端，要分清 IDE HTTP 端口（9420，会返回 404）和自动化 WebSocket 端口（9422），localhost/[::1]/127.0.0.1 三种回环都试；用 `miniprogram-automator` 做协议级验证（connect + reLaunch + evaluate + screenshot，全程带超时）。白屏但几何非零时查原生渲染层（page-container/video/map/canvas 可能盖住 Vue 节点）；真机空白但模拟器正常时查模块级浏览器全局（如 TextDecoder 在手机运行时不存在，要 feature-detect + 降级）；旧 QR 是旧快照，改完代码要重建生成新 QR。文件存储超限时先清 file 缓存重试再查调用方。证据上报不含 token/cookie/.env。

## wechat-miniprogram-devtools-qa

仓库：https://github.com/sabrina-fan/wechat-miniprogram-devtools-qa

微信小程序在开发者工具里的协议优先验收与调试。用 CLI + `miniprogram-automator` 做 DOM 几何/路由/API 流程验证，Computer Use 仅作最后兜底。每次验证前必须走 clean-build gate（项目级清缓存 + 重新编译），不接受用构建成功或源码审查代替实际运行验收。截图、DOM 几何、API 响应、服务端日志都算证据，但单独一个不够——要交叉验证。和 rebuild-miniprogram（重建）、wechat-devtools-automation-recovery（恢复）组成小程序开发验证的完整链路。

## deploy-tencent-project

仓库：https://github.com/sabrina-fan/deploy-tencent-project

把当前项目部署/测试到腾讯云主机。用 Git 推代码、SSH、Docker Compose、健康检查、SSH 隧道、浏览器/API 测试。每项目按仓库隔离——独立部署目录、Compose 项目名、端口、容器、网络、卷，互不干扰。用 SSH alias 管理主机，不硬编码 IP。
