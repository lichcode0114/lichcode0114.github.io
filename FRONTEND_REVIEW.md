# LiCode 博客：界面与性能评估

评估日期：2026-10-16。对象：Hexo 8.1.2 + 自定义 `moment-grid` 主题。

## 判断与边界

项目已经具备静态生成、系统字体、构建期代码高亮、SVG 封面和代码复制功能，这些选择适合个人技术博客。最有价值的改进集中在阅读布局、移动端导航、图片体积和评论加载方式。

本次通读了全部 10 个主题模板、1,296 行主样式、站点及主题配置、两篇文章、页面源文件、脚手架、包与锁文件的相关元数据、现有部署说明，并核查生成产物。依赖库和部署目录的 Git 历史不属于本次逐文件审查范围。

验证：`npm run build -- --force` 成功，重新生成 26 个文件，其中 HTML 21 个。浏览器预览工具连续启动失败，未取得实际截图、网络瀑布或 Lighthouse 分数。以下明确区分源码已确认事实、基于布局规则的风险和待实测目标；构建耗时不代表页面加载速度。

## 已确认的规模

- 当前内容：2 篇文章，2 个分类，9 个标签。
- 生成目录：26 个文件，总原始大小 1,991,310 字节，约 1.99 MB；这不是单页下载量。
- 首页 HTML：7,371 字节；本地 gzip 计算为 2,640 字节。
- 主 CSS：23,994 字节；本地 gzip 计算为 5,268 字节。
- 首页 Logo：1,174,079 字节，PNG，1,254 × 1,254。
- 全站背景：348,405 字节，JPEG。
- 关于页头像：272,251 字节，JPEG，CSS 展示为 150 × 150。
- 通用 SVG 封面：1,575 字节。
- 两篇文章 HTML 分别为 43,818 和 26,756 字节，另加载 Gitalk 外部资源，其传输量未测。

gzip 数字由本地算法计算，不能据此断定 GitHub Pages 实际使用了对应压缩方式。首页 Logo、背景和 SVG 合计约 1.52 MB，优先优化图片的收益明显高于先拆分约 5.27 KB gzip 的 CSS。

## P0：先处理公开密钥

### 1. Gitalk 的 OAuth Client Secret 被输出到公开 HTML

证据：[post.ejs:122](D:/QLL/QLL/blog/themes/moment-grid/layout/post.ejs:122)。此前静态检查曾在两篇生成文章中确认 `clientSecret` 字段存在；本次上传前已从模板移除该值。`GITALK_CONFIG.md` 虽写了“不要泄露到公开仓库”，原集成仍把密钥放入浏览器代码。

建议立即在 GitHub 撤销并轮换现有 Secret。只删除模板中的值无法让已泄露的密钥失效；改成构建时环境变量后再注入前端，同样会暴露。后续可以评估 Giscus，或将需要 Secret 的 OAuth 交换放到可信服务端。Secret 的泄露不等于 GitHub 账号已被接管，但该凭据应视为公开。

实施成本：轮换与移除旧配置较低；更换评论方案约半天，取决于历史评论迁移。

## P1：当前就影响界面和阅读的问题

### 2. 移动端主导航缺失

证据：[style.css:1254](D:/QLL/QLL/blog/themes/moment-grid/source/css/style.css:1254)、[layout.ejs:20](D:/QLL/QLL/blog/themes/moment-grid/layout/layout.ejs:20)。700 px 以下 `.nav-links` 和 `.nav-button` 同时隐藏，模板没有移动菜单；页脚也没有这些入口。

建议保留“文章、归档、关于”三个主要入口，或增加菜单按钮和折叠导航。按钮应有可访问名称、`aria-expanded`、`aria-controls`，支持键盘打开、关闭和正确恢复焦点。无需为了五个链接引入组件库。

验收：320、390、700 px 下都能直接进入归档、分类、标签和关于；仅使用键盘也可完成导航。

### 3. 文章页的三列尺寸与断点不匹配

证据：[style.css:561](D:/QLL/QLL/blog/themes/moment-grid/source/css/style.css:561)、[style.css:916](D:/QLL/QLL/blog/themes/moment-grid/source/css/style.css:916)。宽屏列宽固定为 `250px 820px 356px`，加两个 48 px 间距，合计 1,522 px；容器却是视口宽度减 40 px，直到 1,200 px 以下才切换。因此在 1,201–1,561 px 的视口区间，三列所需宽度超过外层容器。1,366 和 1,440 px 都落在这个区间。这是按 CSS 计算得到的溢出风险，尚无浏览器截图验证。

内部 `.article-shell` 又使用另一组固定列宽并重复限制容器宽度；980 px 以下虽将内部改成一列，外层要到 700 px 才改成一列，侧栏仍被指定在第 2 列，平板布局也容易出现不协调的偏移。

建议先简化嵌套网格。桌面采用“约 200–240 px 目录 + `minmax(0, 1fr)` 正文”，正文容器约 720–800 px，互动区移到文末。若保留第三列，按真实最小空间设置断点，并使正文可收缩。对 Grid/Flex 子项设置必要的 `min-width: 0`；长代码和表格在自身容器内横向滚动。

验收：390、768、1,024、1,280、1,366、1,440 和 1,920 px 下，页面根节点不出现横向滚动；代码的横向滚动只发生在代码块内部。

### 4. 首页首屏偏重品牌展示，文章出现较晚

证据：[style.css:150](D:/QLL/QLL/blog/themes/moment-grid/source/css/style.css:150)、[index.ejs:18](D:/QLL/QLL/blog/themes/moment-grid/layout/index.ejs:18)。Hero 的最小高度为 `100vh - 72px`，接近整屏；标题同时拼接站名和副标题，最大字号 86 px；平板以下图片还会排到文案之后，进一步拉长首屏。

建议缩短 Hero：一句明确的研究/工程方向、一段简短介绍、一个“浏览文章”入口即可。桌面让首屏露出第一篇文章，移动端优先展示标题和文章列表。蓝青色可以保留为主色，但品牌标题、头像、文章卡片不必都使用渐变与重阴影。

当前“精选”其实固定取最新文章 `posts[0]`。可改名为“最新文章”，或通过 `featured: true` / `sticky` 指定真正精选。两篇文章无需在“精选”和“文章导航”中重复占用大量空间；统计栏可以缩成一行。

实施成本：约半天，包括桌面与移动端排版回归。

### 5. 文章互动区比正文更复杂

证据：[post.ejs:30](D:/QLL/QLL/blog/themes/moment-grid/layout/post.ejs:30)、[post.ejs:157](D:/QLL/QLL/blog/themes/moment-grid/layout/post.ejs:157)、[style.css:581](D:/QLL/QLL/blog/themes/moment-grid/source/css/style.css:581)。赞助、点赞收藏、评论默认全部展开；同时还保留右侧固定悬浮按钮。桌面侧栏有独立滚动但隐藏滚动条，读者不易发现其中仍有内容。窄屏固定按钮有遮挡正文的风险，需实测。

建议正文阅读时仅保留目录。点赞/本地收藏/分享放在文末一行；评论位于正文之后，默认折叠或点击加载；未配置收款码时不显示赞助入口。移动端目录使用 `<details>` 或等效折叠控件，避免长目录把正文推得很远。

目录还应支持当前章节高亮、标题 `scroll-margin-top`，防止点击锚点后标题被粘性页头遮住。当前源码没有相关处理。

## P1：优先做的性能优化

### 6. 优化图片并明确加载策略

证据：[index.ejs:29](D:/QLL/QLL/blog/themes/moment-grid/layout/index.ejs:29)、[post.ejs:18](D:/QLL/QLL/blog/themes/moment-grid/layout/post.ejs:18)、[about.ejs:11](D:/QLL/QLL/blog/themes/moment-grid/layout/about.ejs:11)。相关 `<img>` 没有显式 `width/height`，也没有响应式尺寸或懒加载策略。CSS 的 `max-width: 100%` 不能减少下载字节数。

建议：

- Logo 按实际显示尺寸和屏幕像素密度导出 WebP/AVIF 版本，例如 480/960 px，具体规格以最终布局为准；保留必要兼容格式。若原始素材本身为简单矢量图，可直接使用 SVG，不需要把复杂图片机械转换成 SVG。
- 用 `srcset` 和 `sizes` 让手机不下载桌面大图。头像输出约 150/300 px 的版本。
- 所有内容图片设置正确的固有尺寸或 `aspect-ratio`，减少图片加载后的布局跳动；当前缺失尺寸意味着存在 CLS 风险，不代表已测得 CLS 超标。
- 首屏关键图正常加载。确认实际 LCP 元素后，再对关键图片考虑 `fetchpriority="high"`；不要把首屏关键图设为懒加载。
- 首页下方的文章封面、正文后续插图、收款码等非首屏图片按需使用 `loading="lazy"` 和适合的解码策略。
- 背景图优先在文章页移除；若保留，则压缩并为手机提供更小版本。

建议预算：Logo 初步争取压到 100–150 KB，背景压到 80–120 KB，头像争取 20–40 KB；这些是优化目标，需以画质检查和实际编码结果为准。

### 7. 评论加载独立于文章交互

证据：[post.ejs:114](D:/QLL/QLL/blog/themes/moment-grid/layout/post.ejs:114)。每篇文章同步引用 `gitalk@1` 脚本并立即执行 `new Gitalk()`、`render()`，在用户阅读正文前就加载评论资源并触发相应请求。

第三方 CDN 若不可达，`new Gitalk()` 会抛错，中断同一个脚本后续的执行。后面的事件监听注册与 `const articleId` 初始化会受影响，点赞、收藏、面板初始展开也可能失败。这不是单纯的下载大小问题，还涉及功能之间的故障隔离。

建议用“打开评论时加载”或评论区域即将进入视口时加载，确保只初始化一次。评论加载失败应显示可重试状态，文章复制、分享、本地收藏照常运行。核心交互单独初始化；若使用普通外部脚本，可用 `defer` 并正确安排初始化时序，仅给脚本加 `defer` 而保留紧随其后的立即调用会出错。

若保留第三方资源，固定具体版本，避免 `@1` 漂移；评论方案更换时一并复核。是否需要自托管资源应根据目标读者的访问网络实测决定。

### 8. 减少全页模糊和滚动绘制负担

证据：[style.css:27](D:/QLL/QLL/blog/themes/moment-grid/source/css/style.css:27)、[style.css:47](D:/QLL/QLL/blog/themes/moment-grid/source/css/style.css:47)、[style.css:1020](D:/QLL/QLL/blog/themes/moment-grid/source/css/style.css:1020)。全站固定背景、粘性页头和多处 `backdrop-filter: blur(20px)` 叠加，卡片还普遍使用阴影及 `transition: all`。

建议文章页使用稳定的纯色正文底，减少大面积毛玻璃；只保留少量有意义的层次。过渡只声明实际需要的属性，通常是 `transform`、`opacity`、颜色；避免统一过渡所有属性。为 pulse、heartbeat 和平滑滚动提供 `prefers-reduced-motion` 降级。

这是基于效果组合的渲染成本风险，应通过真实手机 Performance 面板比较滚动前后的绘制与掉帧，不能据此宣称已存在某个帧率问题。

## P2：信息架构、可访问性与维护

### 9. 补齐分页，避免文章增多后入口失效

证据：[index.ejs:2](D:/QLL/QLL/blog/themes/moment-grid/layout/index.ejs:2)、[index.ejs:89](D:/QLL/QLL/blog/themes/moment-grid/layout/index.ejs:89)、[archive.ejs:2](D:/QLL/QLL/blog/themes/moment-grid/layout/archive.ejs:2)。首页从全站文章获取列表，只展示前四篇，不使用当前分页的 `page.posts`；归档会使用 `page.posts`，却没有输出分页控件。

目前 2 篇文章尚未触发缺页体验。超过首页配置的 10 篇后，后续索引页仍可能展示相同的最新四篇，而归档后续页缺少正常到达入口。

建议首页列表基于 `page.posts`，输出分页导航；归档、分类和标签详情页复用带分页的列表。给分类/标签详情页输出当前名称，例如“标签：MOMENT”，而不是都显示“文章归档”。短期无需引入全文搜索；内容增长后再增加构建期搜索索引并按需加载。

### 10. 修复缺失资源与互动语义

证据：[post.ejs:67](D:/QLL/QLL/blog/themes/moment-grid/layout/post.ejs:67)、[post.ejs:185](D:/QLL/QLL/blog/themes/moment-grid/layout/post.ejs:185)。两篇文章都请求不存在的 `/images/reward-qr.jpg`。错误回退虽然避免长期破图，仍会发送无效请求并展示“暂未设置”的占位内容。

点赞和收藏计数仅写入当前浏览器 `localStorage`，不是全站累计数据。建议把收藏明确标记为“本地收藏”，点赞如果只表达个人状态，就不展示易被理解为全站人数的计数。当前也没有收藏列表页，可在确实需要此能力时再补。

本地存储读写加异常兜底，避免存储不可用时阻断交互。两处 `DOMContentLoaded` 重复恢复互动状态，应合并为一次。复制降级函数应检查 `document.execCommand('copy')` 的布尔返回值，避免调用失败却提示成功。

### 11. 补齐键盘操作与正文链接辨识

证据：[style.css:37](D:/QLL/QLL/blog/themes/moment-grid/source/css/style.css:37)、[style.css:668](D:/QLL/QLL/blog/themes/moment-grid/source/css/style.css:668)、[post.ejs:57](D:/QLL/QLL/blog/themes/moment-grid/layout/post.ejs:57)。全局链接继承颜色并移除下划线，正文普通链接难与文字区分。关闭按钮只有“×”；关闭面板只改变透明度和高度，没有同步移出焦点导航。

建议给正文链接可识别的颜色/下划线，提供清晰的 `:focus-visible`，增加跳到正文入口。关闭按钮提供“关闭评论”等明确名称。折叠后通过 `hidden` / `inert` 或等效方式处理焦点和可访问树，并同步按钮展开状态；复制成功提示可用 `aria-live`。

白字渐变按钮和浅色小字需测对比度，按普通文字至少 4.5:1、大字至少 3:1 验收；不能只凭颜色代码断定整站合格或不合格。

### 12. 关于页使用真实、可维护的信息

证据：[about.ejs:3](D:/QLL/QLL/blog/themes/moment-grid/layout/about.ejs:3)、[about.ejs:35](D:/QLL/QLL/blog/themes/moment-grid/layout/about.ejs:35)、[about.ejs:91](D:/QLL/QLL/blog/themes/moment-grid/layout/about.ejs:91)。页面有两个 H1；“50+ 文章、10+ 开源项目、1000+ 提交”写死在模板里，邮箱仍为 `licode@example.com`。

建议一个页面保留一个主要 H1，姓名使用下一级标题。本博客文章数从 `site.posts.length` 生成；如果统计覆盖其他平台，应写明范围并提供来源。邮箱改为真实公开联系方式，或移除入口。个人资料和技术栈移入主题配置/数据文件，避免每次改资料都改 HTML 模板。

### 13. 补充分享与收录信息

证据：[layout.ejs:6](D:/QLL/QLL/blog/themes/moment-grid/layout/layout.ejs:6)。目前有标题、description 和 viewport，但未输出 canonical、Open Graph、Twitter Card、文章结构化数据，也没有站点地图产物。

建议用 Hexo helper 输出规范 URL 和 Open Graph，添加适度的文章结构化数据及 sitemap，设置 favicon。description 应统一去 HTML、规范空白并合理截断，避免未来有 HTML 的 excerpt 被编码成难读摘要。首页摘要函数当前顺序为 `excerpt || content || description`，正文几乎总是存在，会掩盖手写 description；建议优先使用明确编写的摘要字段。

这些改善的是链接分享呈现与抓取信息完整性，不应承诺搜索排名提升。

### 14. 让构建和主题演进更可控

证据：[package.json](D:/QLL/QLL/blog/package.json)、[about.ejs:115](D:/QLL/QLL/blog/themes/moment-grid/layout/about.ejs:115)、[tags.ejs:31](D:/QLL/QLL/blog/themes/moment-grid/layout/tags.ejs:31)。样式分散在主 CSS 与模板内联样式；主题菜单配置与全局菜单配置存在，但导航链接仍写死；当前仅有 Dependabot 配置，没有本项目的构建工作流。教程推荐 Node 14+，而安装的 Hexo 8.1.2 实际要求 Node >=20.19.0。

建议统一设计变量与响应式规则，按职责整理 CSS 源码，保持最终浏览器资源简洁。小型交互脚本独立成文件，按页面需要加载，有助于缓存与异常隔离。清理确认未使用的主题/渲染器依赖，可以减少安装维护成本，但这些 Node 依赖本身不是浏览器下载包，不能把依赖数量当作页面负载。

配置 `engines.node` 与版本文件，使用满足依赖要求的受支持 LTS；同步修正教程。CI 使用锁文件执行 `npm ci`、构建和内部资源引用检查。发布后如需要长缓存，先为资源提供内容指纹，再按托管平台支持情况配置缓存；不要给无版本的 CSS 或 HTML 机械套用 immutable。

## 推荐推进顺序

1. **立即**：撤销并轮换公开 OAuth Secret，决定后续评论集成方案。
2. **第一批，约 1 天**：移动导航、文章页网格、缺失二维码、图片压缩与尺寸。这一批最容易让读者直接感受到改善。
3. **第二批，约 1–2 天**：首页压缩 Hero、简化文章侧栏、评论延迟加载及失败隔离、正文链接与目录体验。
4. **第三批，约 1 天**：分页、关于页数据、分享元信息、构建流程。文章超过 10 篇前完成分页。

时间是单人实施的粗估，不含评论数据迁移及设计素材重做。

## 改造后的验收方式

- 页面范围：首页、两篇文章、归档、分类/标签详情、关于。
- 布局：覆盖上述手机、平板和桌面宽度；长标题、长代码、长链接不撑宽页面。
- 网络：冷缓存移动网络下记录请求瀑布；评论未打开前不请求评论脚本/接口；缺失资源请求为零。
- 可用性：键盘走通导航、目录、复制和评论折叠；模拟评论 CDN 不可达、本地存储异常，正文仍可正常阅读和分享。
- 性能目标：LCP <=2.5 s、CLS <=0.1、INP <=200 ms。目标应在代表性真实用户第 75 百分位评估；无真实用户数据时，先用固定条件的实验室测量做前后对比。Lighthouse 可辅助评估加载与 TBT，INP 需结合真实用户/实际交互采样，不能直接用 TBT 替代。
- 性能测试保持相同页面、设备、网络和缓存条件，多次运行看中位数；报告改造前后图片字节数、请求数与指标变化，不只展示一个综合分数。

建议保留 Hexo 的静态架构。当前页面体量小，先把内容入口、阅读体验、图片和第三方加载做好，再评估更复杂的交互功能。
