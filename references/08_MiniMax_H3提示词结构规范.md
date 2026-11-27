# MiniMax H3 提示词结构规范

当目标视频模型为 **MiniMax H3** 时读取本文件。MiniMax H3 支持 4–15 秒，覆盖 `T2VA / I2VA / FL2VA / L2VA / Ref2VA` 五种音画同步模式；其中官方 `h3-prompt-writing` 规范使用严格的英文结构化多模态格式。

> **双份对照输出规则（默认）**：
> 除非用户明确只要单语（如“只要英文版”或“只要中文版”），否则**默认同时输出两份**：
> 1. **官方英文结构版**（主用，直接粘贴最稳）：所有结构字段正文用**英文**撰写；仅对白/歌词 `<d>...</d>` 内容与画面中实际可见的文字 `"..."` 严格保留原始语言（如中文），不翻译、不改标点。
> 2. **中文对照版**（核对与中文直出用）：保留完全一致的字段顺序、时间轴、人物左右手、运镜三要素与音画拆解，用自然中文对应呈现，方便用户快速核对调度细节，也可直接用于中文输入。

## 1. 模式选择与首行对齐指令

首行对齐指令必须放在最终提示词的**第一行**，紧接**一个空行**后再写核心字段（`S.SS` 为视频总时长，精确到两位小数；`N` 为最终镜号）：

| 模式 | 适用场景 | 首行指令（必须严格保留格式） |
| :--- | :--- | :--- |
| **T2VA** | 纯文生视频 | *无首行指令*，直接从 `integrated_multimodal_description:` 开始 |
| **I2VA** | 首帧图生视频 | `For the target video, at 0.00 seconds into the target video, <Picture 1> (from [Shot 1]) is fully referenced.` |
| **FL2VA** | 首尾帧图生视频 | `How the reference pictures align with the target video — Picture 1 (from Shot 1) aligns with the 0.00-second mark of the target video; Picture 2 (from Shot N) aligns with the S.SS-second mark of the target video.` |
| **L2VA** | 尾帧图生视频 | `How the reference pictures align with the target video — <Picture 1> (from [Shot N]) aligns with the S.SS-second mark of the target video.` |
| **Ref2VA** | 多图主体/场景全参考 | *不使用上述首行指令*，改用第 3 节的六段式结构 |

## 2. 基础四模式（T2VA / I2VA / FL2VA / L2VA）三段结构

按以下严格顺序输出三个字段：

```text
integrated_multimodal_description: [Shot 1] Live-action, cinematic, ...

overall_soundscape: ...

non_diegetic_music: ...
```

### 字段写作细则
1. **`integrated_multimodal_description`**：
   - `[Shot 1]` 开头标明画风与初始构图（如 `Live-action, cinematic, a medium-wide shot frames...`），**首镜不写时间戳**。
   - **I2VA**：先锚定 `<Picture 1>` 的主体、服装、站位与场景，再顺着时间线推进动作与反应（`首帧锚定 → 动作启动 → 持续发展 → 落幅/反应`）。
   - **FL2VA**：不重复写两张静态图，重点描写从 `Picture 1` 到 `Picture 2` 的连续过渡路径，结尾稳稳落回 `Picture 2` 的姿态与构图。
   - **多镜切镜**：后续镜头按顺序编号并带递增时间戳，如 `[Shot 2] At 00:03.500, the camera cuts to...`（切镜须在总时长内）。
   - **运镜写法（Motion Type + Amplitude + Speed）**：融入自然英文句子，不堆砌标签。可用动作：`Push In / Pull Out / Zoom In / Zoom Out / Pan Left / Pan Right / Truck Left / Truck Right / Tilt Up / Tilt Down / Pedestal Up / Pedestal Down / Arc Shot / Tracking Shot / Static Shot / Shake Slightly / POV`；幅度：`with small amplitude / with large amplitude`；速度：`at slow speed / at fast speed`（中等幅度与正常速度可省略）。
     - 示例：`The camera pushes in with small amplitude at slow speed toward the kraft envelope in his left hand.`
   - **对白与画外音**：
     - 开口说话的角色分配稳定 ID `(S1)`、`(S2)`（不说话的角色不编号）；
     - 写为：`The woman in a light gray trench coat with a calm voice (S1) says: <d>[Chinese] 给你的。</d>`
     - 画外音固定写：`says in an off-screen voiceover: <d>[Chinese] ...</d> while her lips remain completely closed.`
     - 跨镜对白在断开与衔接处用 `<scenetrans>` 并注明 `continues seamlessly across the cut`。
   - **画面文字**：可见招牌/门牌用英文双引号保留原文，如 `a doorplate reading "302"`。
2. **`overall_soundscape`**（1–4 句英文单段）：
   - 概括全片环境声、物理动作声与非语言人声（脚步声、推门声、纸张摩擦、呼吸声等），不重复对白；仅在用户明确要求全片绝对静音时写 `N/A`。
   - **声画时序对齐（防音效抢跑/错拍）**：严格按与画面描述一致的时间先后顺序写声音，**禁止把跨阶段音效打包成无时序的一句**，也**禁止在前段提前出现后段音效词**（如动作刚开始就写 `latch click` 易触发提前落锁声）：
     - **瞬态接触/撞击音**（关门落锁、放杯、拍桌、开关灯、落地等）：仅在物理接触/闭合完成瞬间发声；带缓慢/阻尼（`slowly / gently / resistance`）的动作须给足现实耗时，拆写为「移动过程无撞击声 → 闭合瞬间单次轻响」；**若镜头结束时动作未完成，严禁写完成态音效**。
     - **持续/周期过程音**（脚步、物品摩擦、倒水、拉帘、拉链等）：绑定动作起止边界（随动而起、随停而止），不虚构易与画面步数冲突的死数字（如随意写 `three footsteps`）。
     - **环境底噪/画外音**：标明 `distant / off-screen` 防模型在画内凭空造声源；无动作留白期用低电平 room tone 占满。
3. **`non_diegetic_music`**（1–3 句英文）：
   - 仅描述角色听不到、观众能听到的背景配乐，写清乐器、速度、节奏与音量变化，不写抽象情绪词；无配乐时写 `N/A`。

## 3. 全参考模式（Ref2VA）六段结构

当使用多张人物/场景/道具参考图生成时，按以下严格顺序输出 6 个英文段落：

1. **`subject_definitions`**：逐行定义 `<Subject N>`（复用的人物/场景/服装/道具，并在定义中引用其来源 `<Picture N>`）与 `<Picture N>`（仅当某图直接作为某镜首帧/尾帧/分镜锚点时单列）。标签一旦分配，全文一致。
   - 示例：`<Subject 1> is the woman in <Picture 1>, wearing a light gray trench coat with shoulder-length black hair.`
2. **`summary`**：1 段简短英文，以方括号任务类型开头（如 `[reference generation]`、`[keyframe completion + reference generation]`），概括主体、镜头流与参考关系，不引入新标签。
3. **`retention_analysis`**：逐行说明每个标签在目标视频中如何保留（fully preserved / partially preserved / transferred / reused）。
4. **`detailed_description`**：按播放顺序详写每一镜的构图、站位、动作、左右手、运镜、声音与参考元素出现节点（共享第 2 节的 `[Shot N]`、运镜与 `<d>` 规范）。
5. **`overall_soundscape`**：同第 2 节。
6. **`non_diegetic_music`**：同第 2 节。

## 4. 交付前自查（H3 专用）
- [ ] 总时长（4–15 秒）与所有切镜时间戳、`S.SS` 两位小数完全一致；
- [ ] 所有 `<Picture N>` / `<Subject N>` / `(S1)` 标签前后一致，无悬空未定义标签；
- [ ] 正文为自然具体英文，对白 `<d>` 与画面文字 `"..."` 保留原文未误翻；
- [ ] 空间站位、180° 轴线与人物左右手 ↔ 画面左右映射已完全承接调度方案；
- [ ] `overall_soundscape` 与画面时间轴严格同序，无前文剧透音效词，瞬态音均对齐物理接触完成瞬间（未完成动作未写完成态音效）。
