---
name: wechat-mp-article
description: 写一篇微信公众号文章，排好版，做好封面，再通过官方 API 推进草稿箱（只存草稿，不发表）。适用于“写公众号文章 / 推到草稿箱 / 换封面 / 重排版”。
---

# 公众号文章：写作 → 排版 → 封面 → 推草稿箱

方法来源（读过源码和文档后提炼）：
- 排版红线、组件、视觉层级：[isjiamu/gzh-design-skill](https://github.com/isjiamu/gzh-design-skill)（SKILL.md、references/theme-graphite-minimal.md、archive/themes-v1/theme-terminal.md、references/common-components.md）
- Markdown→内联样式的做法、字号/行高档位：[doocs/md](https://github.com/doocs/md)（packages/shared/src/configs/style.ts、theme-css/default.css、core/src/theme/cssProcessor.ts）
- 写作结构、加粗配额、摘要、封面构图与裁剪框、发布错误码：[aiworkskills/wechat-article-skills](https://github.com/aiworkskills/wechat-article-skills)（aws-wechat-article-writing、-formatting、-images/references/cover-method.md、-publish/references/branches.md）
- 组件质量基线、40164 分类：[zhouke0929/wechat-visual-director](https://github.com/zhouke0929/wechat-visual-director)（docs/组件视觉质量基线-V1.0.md、apps/api/.../wechat_publisher.py）
- 主题写法（蓝图、终端等，全部 inline style 字符串）：[laogou717/md-wechat](https://github.com/laogou717/md-wechat)（src/lib/themes.js、renderer.js）
- API 客户端（token 缓存、uploadimg、add_material、draft/add）：[lyhue1991/wxgzh](https://github.com/lyhue1991/wxgzh)（src/core/wechat.ts）
- 工具全景：[md2wechat/awesome-wechat-markdown](https://github.com/md2wechat/awesome-wechat-markdown)
- 文风与自检：[KKKKhazix/khazix-skills · khazix-writer](https://github.com/KKKKhazix/khazix-skills/tree/main/khazix-writer)（数字生命卡兹克，MIT）

环境变量：`WECHAT_MP_APPID`、`WECHAT_MP_APPSECRET`。永远不要打印 secret。

---

## 1. 写作方法

### 1.1 选题与结构（aiworkskills + Khazix）
- 选题过 HKR：H 有趣有悬念、K 有信息量、R 能共鸣。至少占两项。
- 结构：标题 → 摘要（≤128 字，和正文第一段不要是同一句）→ 开头 2–3 段 → 主体若干节 → 结尾 1–2 段 → 文末区块（一句互动/关注，唯一一处）。
- 开头：从一个具体、当下的事件切入，第一句要让人想问“然后呢”。可用：荒诞事实、热点破题、“事情是这样的”。禁止“在当今 AI 飞速发展的时代”。
- 结尾：回环呼应开头的意象（契诃夫之枪），或一句短促留白，或一个开放问题。不要“综上所述”。
- 多条目的盘点（N 件事）可以用编号小节；否则尽量少用小标题，用口语转场（“说到这个”“回到 xxx 这块”）。
- 逐一展示按“升番”排：弱的在前，最炸的放最后或放开头做钩子。

### 1.2 卡兹克（Khazix）文风要点
- 定位一句话：“有见识的普通人在认真聊一件打动他的事”。敢用“我觉得”。
- 节奏：长短句交替，关键处一句话单独成段（全文 ≥3 处），偏出去就用一句“扣主线”拉回来，用疑问句做刹车和转向。
- 知识“聊着聊着顺手掏出来”，不说“下面科普一下”；讲观点前先承认对方处境合理。
- 硬禁用：说白了、这意味着/意味着什么、本质上、换句话说、不可否认、综上所述、值得注意的是、不难发现、首先其次最后、让我们来看看。
- 标点：正文少用冒号和破折号，引号用「」（嵌套『』），可以用“。。。”“？？？”表情绪（克制使用）。
- 不编造亲历和细节：没有真实经历就不写“有一次我……”；工具、模型写具体名字。
- 写完按四层自检：L1 禁用词/标点扫描 → L2 开头、节奏、口语化 → L3 每个观点有具体支撑、有一处文化/历史升维 → L4 通读看有没有“AI 在输出信息”的味道。

### 1.3 强调配额（aiworkskills）
- 把全文加粗/标记按顺序抽出来连读，应该是一篇能看懂的缩写版。
- 约每 100–200 字一个落点，每节至少一处，一段最多两处；标记要能独立看懂（数字连着意思，比如“撤回 3 篇”）。
- 最强强调（深色块、主色大字）全文 ≤5 处（gzh-design 的三层：锚点层 / 标记层 / 容器层）。

### 1.4 事实纪律
- 每个数字、日期、引语都对应文末来源；改写只动语气和节奏，不动事实。改完用脚本把新旧稿的数字做多重集对比。

---

## 2. 微信 HTML 红线（gzh-design + doocs/md + md-wechat 共识）

- 只给正文片段：从一个 `<section>` 全局容器开始，没有 `<html>/<head>/<body>`。
- 只用内联 `style`。禁止：`<style>`、`<script>`、`<div>`（用 `<section>`）、`class`、`id`、外部 CSS/字体、CSS 变量 `var(--x)`、`@media`、`@keyframes`、`position:absolute/fixed/sticky`、`float`、`display:grid`。
- 可用：`<section> <p> <span> <strong> <em> <img> <br> <h1-h3>`、`display:flex`（有限）、`border-radius`、`box-shadow`、`border-*`。
- 渐变可渲染但可能触发 `darkmode-no-gradient` 检测；文字和卡片背景用纯色。
- `font-size` ≤24px；同一个 `<p>` 里不要混多个字号；不要把 `font-size/border-bottom` 打在 `<strong>` 上，挂在外层 `<span>` 上。
- 装饰性空元素会被剥掉：要么内部放 `<span leaf=""><br></span>`，要么改成给有文字的元素加 border。
- 手动粘贴进编辑器的流程：所有文字包 `<span leaf="">`（否则粘贴后样式丢失）。走 API 的 `draft/add` 时不是必须。
- 图片：`max-width:100%;height:auto;display:block;margin:0 auto;`，不要 `width:100%` 拉伸小图。正文图片必须先 `media/uploadimg` 换成 mmbiz 链接。
- 代码块：每行一个 `<p style="margin:0">`，不要 `white-space:pre`；缩进用全角空格。
- 外链：非认证号正文里的 `<a>` 外链会失效，来源写成纯文本 URL（`word-break:break-all`）。
- 手机宽度：内容最大 677px，预览按 375/390px 检查，无横向滚动。

## 3. 排版数值（参考值）

| 项 | 值 | 来源 |
|---|---|---|
| 正文字号 | 15px（doocs 档位 14–18，16 也常见） | gzh-design、doocs |
| 行高 | 1.75–1.85 | doocs 档位 1.5/1.65/1.75/1.9；gzh 1.8 |
| 字距 | 0.3–0.5px（约 0.03–0.1em） | gzh、md-wechat |
| 段距 | 段后 16px（约 1–1.5em） | doocs `margin:1.5em 8px` |
| 二级标题 | 18–20px 粗体，上距 40–56px | gzh 章节距 56px |
| 三级/小标签 | 13–16px，左竖条或药丸标签 | doocs h3 `border-left:3px` |
| 引用 | 左竖条 3–4px + 浅底或无底 | doocs blockquote |
| 正文色 / 标题色 | #3A4150 左右 / #0B1220–#27272A | gzh 石墨 |
| 辅助文字 | 12–13px，#8A94A6 | |

## 4. 主题模板：Ink & Cyan（科技、克制，已实测通过 draft/update）

设计变量：墨 `#0B1220`、正文 `#3A4150`、青 `#00A3C4`（白底可读）、亮青 `#22D3EE`（下划线/深底）、浅青底 `#E6F7FB`、细线 `#E3E8EF`、灰 `#8A94A6`、浅底 `#F5F8FB`。等宽 `'SF Mono',Menlo,Consolas,monospace` 只用于编号和标签，正文仍用系统黑体。全篇只有一个深色块（开头钩子卡）。

```html
<!-- 全局容器 -->
<section style="max-width:677px;margin:0 auto;font-family:-apple-system,BlinkMacSystemFont,'PingFang SC','Hiragino Sans GB','Microsoft YaHei',sans-serif;font-size:15px;line-height:1.8;letter-spacing:0.5px;color:#3A4150;">

<!-- 开头钩子卡（唯一深色块） -->
<section style="margin:4px 0 28px;padding:18px 18px 16px;background:#0B1220;border-radius:6px;">
  <p style="margin:0 0 10px;font-family:'SF Mono',Menlo,Consolas,monospace;font-size:12px;letter-spacing:2px;color:#22D3EE;">// 本周 AI 五件事</p>
  <p style="margin:0 0 6px;font-size:17px;font-weight:700;line-height:1.6;color:#FFFFFF;">{钩子第一句}</p>
  <p style="margin:0 0 12px;font-size:17px;font-weight:700;line-height:1.6;color:#22D3EE;">{反转短句}</p>
  <p style="margin:0;font-size:14px;line-height:1.8;color:#C9D3E0;">{导语}</p>
</section>

<!-- 章节标题：等宽编号 + 粗标题 + 细线 -->
<section style="margin:48px 0 18px;padding-bottom:10px;border-bottom:1px solid #E3E8EF;">
  <p style="margin:0 0 4px;font-family:'SF Mono',Menlo,Consolas,monospace;font-size:13px;letter-spacing:2px;color:#00A3C4;">01 / 05</p>
  <p style="margin:0;font-size:19px;font-weight:700;line-height:1.5;color:#0B1220;letter-spacing:0.5px;">{章节标题}</p>
</section>

<!-- 正文段 + 关键词标记（挂在 span 上） -->
<p style="margin:0 0 16px;font-size:15px;line-height:1.8;letter-spacing:0.5px;color:#3A4150;text-align:justify;">……<span style="color:#0B1220;font-weight:600;border-bottom:2px solid #22D3EE;">{关键短语}</span>……</p>

<!-- 小标签（需要时） -->
<p style="margin:22px 0 10px;"><span style="display:inline-block;padding:2px 8px;background:#E6F7FB;color:#007A94;font-size:13px;font-weight:600;letter-spacing:1px;border-radius:3px;">{标签}</span></p>

<!-- 点评/旁注 -->
<section style="margin:18px 0 8px;padding:12px 14px;background:#F5F8FB;border-left:3px solid #00A3C4;">
  <p style="margin:0 0 4px;font-family:'SF Mono',Menlo,Consolas,monospace;font-size:12px;letter-spacing:1px;color:#00A3C4;">&gt; 点评</p>
  <p style="margin:0;font-size:14px;line-height:1.8;color:#0B1220;">{一句判断}</p>
</section>

<!-- 引语 -->
<section style="margin:4px 0 18px;padding:2px 0 2px 14px;border-left:3px solid #E3E8EF;"><p style="margin:0;font-size:15px;line-height:1.8;color:#0B1220;font-weight:600;">「{引语}」（据 {来源}）</p></section>

<!-- 结尾标题用 EOF 代替编号；文末一句互动，居中灰字 -->
<p style="margin:28px 0 0;font-size:14px;line-height:1.8;color:#8A94A6;text-align:center;">觉得有用的话，随手点个赞、在看、转发。</p>

<!-- 参考来源 -->
<section style="margin:40px 0 0;padding:14px 14px 10px;background:#F5F8FB;border-radius:4px;">
  <p style="margin:0 0 8px;font-family:'SF Mono',Menlo,Consolas,monospace;font-size:12px;letter-spacing:2px;color:#8A94A6;">REFERENCES · 参考来源</p>
  <p style="margin:0 0 6px;font-size:12px;line-height:1.6;color:#8A94A6;word-break:break-all;"><span style="color:#3A4150;">{名称}</span>：{URL}</p>
</section>
</section>
```

生成后自检：`grep -E '<(div|style|script)|class=|id=|position:|float:|@media|var\(--|display:grid'` 必须零命中；用 headless Chrome 以 390px 宽截图看一遍。

## 5. 封面配方

- 尺寸 900×383（2.35:1）。信息流里只有约 345×147，缩到这么小还要一眼认出主元素。
- 1:1 分享卡从中心裁：所有关键内容放在中间 383×383 安全区（x 258–641）。
- 元素 ≤3 个（主数字/短标题算一个）；主元素字高占画面 25–35%；文字 4–7 字或一个数字。
- 科技但克制：近黑底 `#090C12` 或深蓝 `#07122A`；极淡点阵或细网格（alpha 10–60，中心亮、四周暗）；一条发光细线（青 `#00E5FF`，模糊半径 6px）；等宽体大数字（JetBrains Mono 800）+ 一行中文小标题（Noto Sans CJK SC Bold 20px）；可选四角细刻度。不要渐变满屏、扫描线太密、霓虹过曝。
- 做法：Pillow 以 2 倍尺寸（1800×766）绘制，GaussianBlur 做辉光，最后 LANCZOS 缩到 900×383，JPEG quality 92。至少出 2–3 个变体，自己看图后再选。
- 字体：`fonts-noto-cjk`（ttc index 2 = SC）、JetBrains Mono / Space Grotesk 可变字体（`set_variation_by_axes([wght])`）。

## 6. 官方 API 流程

基址 `https://api.weixin.qq.com/cgi-bin/`，所有 JSON 用 UTF-8 发送：`data=json.dumps(obj, ensure_ascii=False).encode('utf-8')`，否则中文会变成 `\uXXXX`。

1. **取 token**：`POST stable_token`，body `{"grant_type":"client_credential","appid":$WECHAT_MP_APPID,"secret":$WECHAT_MP_APPSECRET}`。返回 `access_token`（7200s）。stable_token 不会让别处的 token 失效，优于旧的 `GET token`；缓存到过期前 5 分钟。
2. **正文图片**：`POST media/uploadimg?access_token=`，multipart 字段 `media`（jpg/png，<1MB），返回 `url`（mmbiz.qpic.cn），替换正文 `<img src>`。不占素材库。
3. **封面**：`POST material/add_material?access_token=&type=image`，multipart `media`，返回 `media_id` 作为 `thumb_media_id`（永久素材）。
4. **新建草稿**：`POST draft/add`，`{"articles":[{"title","author","digest","content","content_source_url","thumb_media_id","need_open_comment":1,"only_fans_can_comment":0}]}`，返回草稿 `media_id`。标题 ≤64 字，作者 ≤8 字，摘要 ≤120/128 字。可选 `pic_crop_235_1`、`pic_crop_1_1`（归一化 `X1_Y1_X2_Y2`，比例不对会报 53402），不传就居中裁。
5. **更新草稿**：先 `POST draft/get {"media_id"}` 取出现有字段，再 `POST draft/update {"media_id","index":0,"articles":{...}}`。注意 update 的 `articles` 是对象不是数组；不想改的字段（标题、作者、摘要、评论设置）原样带回。
6. **验证**：再 `draft/get` 一次，核对标题、作者、摘要、评论、`thumb_media_id`、正文里的特征字符串；`draft/count` 看篇数。
7. **永不调用** `freepublish/submit`（发表）或群发接口，除非用户明确要求。

常见错误：
- **40164 invalid ip**：调用 IP 不在白名单。`errmsg` 里会写出 IP（`invalid ip x.x.x.x, not in whitelist`）。把它告诉用户加到「设置与开发 → 基本配置 → IP 白名单」。出口 IP 可能轮换，可以多试几次收集同一网段的几个 IP 一起加。可以短暂重试。
- 40001/42001：token 失效，强制刷新后重试一次。
- 45009：调用次数超限。
- 53402：封面裁剪框比例不对。
- 48001：接口无权限（没开草稿箱权限，可能需要认证）。

## 7. 交付清单
- [ ] 事实核对：新旧稿数字多重集一致，来源齐全
- [ ] 文风：L1 禁用词扫描零命中
- [ ] HTML 红线 grep 零命中，390px 截图看过
- [ ] 封面 2–3 个变体看过，主元素在中心安全区
- [ ] draft/update 返回 `{"errcode":0,"errmsg":"ok"}`，draft/get 回读核对
- [ ] 没有发表

## 两种文章形态：播报+点评 / 单篇长文

**1. 每日播报+点评（每天一篇）**
- 选 5–8 条当日科技/AI 要闻，按从弱到强的顺序排，最炸的一条放最后。
- 每条写成一段：先用 2–3 句说清发生了什么，再接一段「小g点评」。点评要有明确立场：你怎么看，为什么，后面会怎样。不要说两边都对的话。
- 开头用一句话钩子点出今天的主线。结尾选一条，抛一个让读者站队的问题。
- 标题围绕最炸的那条来起，再点明这是一篇合集，例如「…：今天最值得聊的 6 件事」。

**2. 单篇长文（遇到真正炸裂的事件才写）**
- 只写一件事，2500–4000 字。
- 结构：先从一个具体的荒诞细节切入，再讲清来龙去脉，然后说关键人物和各方立场，接着拆它为什么重要，最后讲争议点，以及小g的判断和预测。
- 每个论点都要有事实撑着，数字、引语都标明出处。不编造亲身经历。
- 和当天的播报文章不重复，播报里那条点评可以写短一点，注一句「详见今日长文」。

## 来源与敏感词（国内公众号）
- 默认不在文末附来源列表，用户要求时才附。事实照样要核实，核实记录留在本地。
- 正文和来源都不出现在国内属敏感或违禁的媒体名，比如美国之音、自由亚洲电台、大纪元、新唐人等。需要引用时改写成「据报道」「据外媒报道」，或者换用公司官方公告、中立的科技和财经媒体作为出处。
- 推送前检查一遍全文，不出现这些敏感媒体名。
