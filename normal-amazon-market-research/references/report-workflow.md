# Bruce Amazon Market Research Workflow

## 1. 输入解析

触发语通常是：

- `请帮我调研 magnetic drawing board 市场`
- `请帮我调研 xxx（亚马逊产品英文关键词）市场`
- `分析 xxx 亚马逊市场`

解析字段：

| 字段 | 默认 |
|---|---|
| keyword | 从用户句子中提取英文产品关键词 |
| site | US |
| report_date | 当前日期 |
| output | `reports/dtc/{keyword_slug}_amazon_market_research_{SITE}_{YYYYMMDD}.html` |

如果用户只给中文品类，应先让用户确认英文关键词，或自行给出英文关键词假设并标注。

## 2. 数据口径

优先口径：

1. `keyword_search_results(keyword, US, page=1..5)` 获取关键词前五页自然位产品。
2. 聚合 ASIN，去掉明显无关产品。
3. 按产品形态、场景、技术路线、价格带、用户需求聚类，形成细分市场。
4. 细分市场容量 = 该细分内 ASIN 的月销量/月销额汇总。
5. 代表 ASIN = 该细分销量、排名、典型性综合最强的产品。

辅助数据：

- `keyword_detail`：关键词搜索量、CPC、热度。
- `keyword_trend`：关键词趋势和季节性。
- `product_detail`：代表 ASIN 和竞品详情、价格、评分、规格、图片。
- `product_reviews`：VOC，尤其负评。
- `competitor_product_keywords` / `product_traffic_terms`：流量关键词和自然曝光能力。
- Reddit、论坛、TikTok、YouTube、品牌官网：用户语言与场景补充。需要引用外部网页时必须浏览核验。

## 3. 报告结构

### 3.1 报告标题

每次触发本 Skill 生成报告时，报告主标题必须使用：

`《{产品关键词}市场调研报告》`

其中 `{产品关键词}` 使用用户输入的原始产品关键词，保留用户提供的语言、大小写和核心词序。例如用户输入 `bean bag cover`，标题应为 `《bean bag cover市场调研报告》`。HTML 的 `<title>`、页面 H1、报告封面/主标题必须保持一致。

### 3.2 核心结论

必须回答：

- 这个关键词市场的真实构成是什么？
- 哪些细分看似有量但不适合进入？
- 推荐进入哪个细分，为什么？
- 当前结论到哪个阶段，是否等待用户选择细分？

### 3.3 关键词市场总览

建议包含：

- 关键词月搜索量
- 前五页自然位样本 ASIN 数
- 样本月销量
- 样本月销额
- 均价
- 明显剔除的无关细分

### 3.4 细分市场分类

按关键词自然位样本聚类。每个细分至少包含：

| 字段 | 说明 |
|---|---|
| 细分市场 | 按用户能理解的英文/中文名称 |
| 代表 ASIN | 最典型或销量最高 |
| 产品主图 | HTML 里展示图片 |
| 月销量 | 细分 ASIN 汇总 |
| 月销额 | 细分 ASIN 汇总 |
| 均价 | 细分均价 |
| 样本 ASIN 数 | 该细分覆盖的产品数 |
| 用户需求 | 为什么用户买这类产品 |
| 与品牌/项目的关联 | 是否适合进入 |

### 3.5 销售季节性分析

使用关键词趋势、代表 ASIN 趋势、历史月销量代理判断：

- 旺季月份
- 淡旺季波动指数
- 是否 Q4 礼品驱动
- 上新和备货节奏建议

### 3.6 细分市场增长率与趋势动能

建议字段：

| 细分市场 | 趋势代表 ASIN | 近月销量 | 近 3 个月环比 | 同比 | 季节峰值 | 动能评分 | 决策含义 |

若数据不足，使用“代表 ASIN 趋势代理”，并标注为估算口径。

### 3.7 进入优先级排序

根据排序结果，系统将自动选取排名最高的细分市场继续深研。

在评分前必须补充品牌相关性口径：

- 默认品牌为 `JoyCat`，除非用户明确指定其他品牌。
- 使用 `product_search` 按品牌名检索 Amazon US 现有产品；至少提炼现有产品的人群、年龄段、使用场景、价格带、品类能力、渠道心智和合规能力。
- 将每个细分市场与 JoyCat/指定品牌现有产品线做相关性判断，尤其评估是否能复用现有人群、礼品属性、儿童学习/玩具/感官场景、供应链能力、合规经验和品牌视觉语言。
- 若品牌现有产品数据不足，必须在报告中写明“品牌相关性为假设，待用户确认/待数据验证”，不得省略该维度。

评分维度必须包含：

| 维度 | 含义 |
|---|---|
| 市场容量 | 月销量/月销额 |
| 竞争环境 | 集中度、评论壁垒、头部垄断 |
| 增长动能 | 环比、同比、季节性 |
| 创新难易度 | 是否能做出真实差异 |
| JoyCat/品牌相关性 | 是否符合 JoyCat 或用户指定品牌的现有人群、场景、价格带、品类能力、渠道心智和品牌视觉 |
| 合规/供应链复杂度 | 玩具、电子、磁体等风险 |

建议权重：

| 分数 | 权重 | 说明 |
|---|---:|---|
| 市场综合分 | 70% | 由市场容量、竞争环境、增长动能、创新难易度、合规/供应链复杂度综合得出 |
| JoyCat/品牌相关性分 | 30% | 由现有产品线协同、人群复用、渠道复用、品牌心智延展、供应链/合规能力匹配度综合得出 |
| 品牌适配后分数 | 100% | `市场综合分 * 0.7 + JoyCat/品牌相关性分 * 0.3` |

输出：

必须先输出 2-3 张卡片展示 Top 3 细分市场，每张卡片至少包含：

- 市场综合分
- JoyCat/品牌相关性分
- 品牌适配后分数
- 一句话说明为什么与品牌相关或不相关

然后输出完整排序表：

| 排名 | 细分市场 | 市场综合分 | JoyCat/品牌相关性 | 品牌适配后分数 | 建议 | 核心理由 | 主要风险 |

核心理由必须同时覆盖：

- 市场机会为什么成立或不成立。
- 与 JoyCat/指定品牌现有产品线的协同点。
- 不能协同的原因，例如偏成人、偏户外、偏家具整椅、偏耗材、偏低价工具、偏离儿童学习/玩具/礼品/感官场景等。

写完本节必须停止，向用户询问：

```text
请问你希望进入哪个细分市场？
```

如果用户回答“不需要了/不用继续”，直接生成阶段性最终报告，结尾说明“报告停留在细分市场选择阶段，未展开竞品深拆和产品策略”。

## 4. 用户选择细分市场后的后续报告

### 4.1 重点细分市场概览

围绕用户选择的细分市场输出：

- 细分月销量
- 细分销量占比
- 细分月销额
- 细分均价
- 价格段与销量关系
- 主要竞品月销量
- 细分判断

### 4.2 主要竞品调研

建议选 5-8 个竞品。必须有产品主图。竞品列必须包含品牌名和 ASIN，且 ASIN 必须嵌入 Amazon 商品页超链接，US 站默认格式为 `https://www.amazon.com/dp/{ASIN}`。HTML 链接建议使用 `target="_blank"` 和 `rel="noopener noreferrer"`，方便用户从报告直接跳转核验。

| 竞品 | 主图 | 形态 | 价格/月销 | 规格参数 | 核心卖点 | 主要短板 | 对新品启发 |

`规格参数` 是固定字段单元格，不允许写成自由摘要。每个竞品必须按顺序包含：

- 产品规格
- 产品重量
- 产品材质
- FBA费用
- 产品评分
- 评论数量
- 上架时间

优先用 `product_detail` 补齐这些字段。若某个字段无法采集，仍必须保留字段名，并写 `待验证`，不得省略。

### 4.3 用户画像：购买者与使用者分离

如果品类存在购买者和使用者不同，必须分开画像。

画像口径：角色、场景、目标、痛点、行为、决策因素。

儿童玩具推荐结构：

| 角色 | 典型画像 | 场景/目标 | 决策或体验需求 |
|---|---|---|---|
| 购买者 1：出行控场型父母 | 年龄、家庭、焦虑、购买触发 | 车程/餐厅/飞机/等待 | 安静、无屏、无散件、安全 |
| 购买者 2：学习价值型父母 | Montessori/早教/fine motor | 家庭早教/亲子短任务 | 学习任务、分级、安全背书 |
| 购买者 3：礼物购买者 | 亲友/祖父母/老师 | 生日/圣诞/返校/课堂奖励 | 包装、年龄标识、评价、价格 |
| 使用者 1：低龄探索儿童 | 3-4 岁 | 自由玩、触觉反馈 | 低挫败、易吸附、不卡顿 |
| 使用者 2：任务挑战儿童 | 4-6 岁 | 图案卡/路径/颜色任务 | 成就感、难度递进、可独立完成 |
| 使用者 3：特殊场景儿童 | 感统/专注力需求 | 等待/过渡/安静活动 | 低噪、重复反馈、安全封闭 |

### 4.4 Kano 模型

输出：

| Kano 类型 | 用户需求 | 证据来源 | 产品含义 |

至少包含：

- Must-be：安全、质量、无危险、基础可用性。
- Performance：越好越能提升转化的体验。
- Attractive：差异化和惊喜点。
- Reverse：做了反而降低体验的因素。

### 4.5 VOC 与机会点

需要包含：

- 正向 VOC
- 负向 VOC
- Reddit/论坛/社媒补充反馈
- 共性信号与机会点

### 4.6 新品卖点排序

排序维度：

- 卖点吸引力
- 实现成本友好度
- 可落地性

输出：

| 排名 | 卖点 | 综合分 | 吸引力 | 成本友好度 | 可落地性 | 排序理由 |

写完本节必须停止并询问用户。HTML 报告必须在本节之后插入可直接填写的“产品规划表单”，不得只输出静态空表。优先复用 `assets/product-plan-form.html` 的结构和交互。

表单要求：

- 横向支持 1-3 个 SKU/型号方案，第一列为字段名，后三列为 SKU 1、SKU 2（选填）、SKU 3（选填）。
- 字段必须包含：SKU/型号、产品定价（USD）、核心卖点、次要卖点、产品配置&性能（选填）、销售渠道、预估毛利率、上市时间。
- HTML 中必须使用可编辑控件（`input` / `textarea`），而不是空白单元格。
- 表单应包含“复制已填写内容”和“清空表单”按钮；优先加入 localStorage 自动保存，避免刷新丢失。
- 复制内容建议转成 Markdown 表格，便于用户直接粘贴回对话。

对话中询问用户：

```text
请在报告末尾的产品规划表单中填写 1-3 个 SKU 方案，填写完成后点击“复制已填写内容”并发给我，以便我输出最终产品定义和产品策略。
```

如果当前不是 HTML 输出环境，使用以下 Markdown 兜底表格：

```markdown
| 字段 | SKU 1 | SKU 2（选填） | SKU 3（选填） |
|---|---|---|---|
| SKU/型号 |  |  |  |
| 产品定价（USD） |  |  |  |
| 核心卖点 |  |  |  |
| 次要卖点 |  |  |  |
| 产品配置&性能（选填） |  |  |  |
| 销售渠道 |  |  |  |
| 预估毛利率 |  |  |  |
| 上市时间 |  |  |  |
```

## 5. 用户补充产品信息后的最终报告

进入本章时必须叠加 `product-strategy` Skill 的框架执行。若当前会话可读取该 Skill，先读取 `product-strategy/references/product-strategy-framework.md`；若无法读取，也必须按以下结构补齐：策略诊断、目标用户、竞争逻辑、产品屋、产品矩阵、生命周期、定价传播、输出物和下一步动作。若用户只填写“预估毛利率”而未提供采购价、FBA、头程等详细成本，利润成本核算先按用户毛利率做策略级估算，并标注详细费用待验证。

### 5.1 最终产品定义

必须输出：

- 英文产品命名：好记、贴合卖点、能延展 SKU 系列。
- 一句英文 slogan。
- 产品定位。
- 目标购买者与目标使用者。
- 使用场景。
- 核心卖点和次要卖点。
- 规格和基础参数。
- 主要竞品锚点。

SKU 命名建议：

- 简短：2-4 个英文词。
- 可系列化：能扩展 Basic/Plus/Pro 或场景名。
- 直接表达核心利益：Travel、Quiet、CardLock、Go、Dots、Focus、Safe 等。
- 避免过度泛化和难拼写词。
- 最终报告每一次都必须使用 `5.1C SKU 规划与产品定义固定格式`，不要改成散文段落、普通清单或普通 SKU 矩阵表。产品矩阵可作为辅助表存在，但不能替代 5.1C 固定表。

### 5.1A Product Strategy 策略诊断

必须输出：

- Executive conclusion：一句话说明 Go / Conditional Go / Watch / No-Go。
- 关键假设与缺失证据：哪些数据来自 Sorftime，哪些来自用户输入，哪些仍需验证。
- 策略诊断：当前产品定义中清晰、薄弱、矛盾或未验证的地方。
- 目标用户：生活方式、信念、购买逻辑、使用场景、痛点/痒点/惊喜点。
- 竞争逻辑：用户想要什么、竞品没满足什么、我们能真实交付什么。

当用户提供产品文档、飞书文档、规格书、图片或卖点表时，必须重新理解产品定义，并修正原关键词市场口径。不要机械沿用前面选择的细分市场；如果新产品实际跨越相邻市场，应增加“市场口径修正”章节。

### 5.1B 产品屋与产品矩阵

必须输出 Product House：

| 层级 | 内容 |
|---|---|
| Positioning | 产品面向谁、在哪个品类、解决什么问题 |
| Mindshare | 用户应该记住的一句话心智 |
| Functional Value | 功能价值 |
| Emotional Value | 情绪价值 |
| Identity Value | 身份价值 |
| POP | 品类基础入场项 |
| POD | 真正差异化 |
| RTB / Proof | 支撑证据：测试、认证、结构、材料、评论、供应链等 |
| Visual Direction | 主图、A+、视频和包装表达方向 |

必须输出产品矩阵：

- SKU 角色：traffic / profit / flagship / defense / test。
- 每个 SKU 的渠道角色、价格带、颜色/规格、上市优先级。
- 是否存在内耗、库存复杂度或品牌定位稀释风险。

### 5.1C SKU 规划与产品定义固定格式

当用户在第二次交互中提供产品型号、定价、卖点、渠道等信息后，最终报告每一次都必须包含一个标题为 `SKU 规划与产品定义` 的板块，并固定采用三列表格：

| 产品规划 | 基础款 | 升级款 |
|---|---|---|

该三列表格是强制主格式。无论用户提供几个 SKU，都不得改成横向多 SKU 矩阵、普通表格、散文段落或清单。多 SKU 映射规则如下：

- 用户只提供 1 个 SKU：基础款列填写用户 SKU；升级款列不得自动生成、命名或假设任何升级款。升级款列统一填写“用户未提供升级款 / 暂不规划”，并在各固定行中保持该含义；不得基于市场机会自行补充可选升级 SKU、后续升级方向、升级款卖点、升级款定价或升级款名称。
- 用户提供 2 个 SKU：基础款列填写入门/低价/流量/测试 SKU；升级款列填写高价/利润/旗舰/差异化 SKU。
- 用户提供 3 个或更多 SKU：基础款列填写最低门槛或首发流量 SKU；升级款列填写主推升级 SKU，并在同一列中说明旗舰/批量/延展 SKU 的承接关系。必要时在固定表后追加“产品矩阵辅助说明”，但辅助表不能替代固定表。
- 如果用户提供 2 个或更多 SKU 但没有明确基础/升级关系，按价格、配置、渠道角色和上市优先级推断，并在“命名逻辑”或“角色”中说明假设。该规则不适用于用户只提供 1 个 SKU 的情况；单 SKU 不得被扩展成升级款。

固定行顺序如下，不要随意删除：

| 行名 | 基础款填写要求 | 升级款填写要求 |
|---|---|---|
| SKU / 英文命名 | `产品型号 · 英文产品名`，下一行写“命名逻辑”。英文名要好记、贴合卖点、可系列化。 | 若用户提供升级 SKU：`产品型号 · 英文产品名`，下一行写“命名逻辑”。若用户未提供升级 SKU：只写“用户未提供升级款 / 暂不规划”，不得生成升级款名称。 |
| 一句 Slogan | 英文 slogan 加粗，下一行中文解释。 | 若用户提供升级 SKU：英文 slogan 加粗，下一行中文解释。若用户未提供升级 SKU：写“用户未提供升级款 / 暂不规划”，不得生成升级款 slogan。 |
| 角色 | 说明该 SKU 是流量切入款、搜索转化款、利润款、礼品承接款、旗舰款、防御款或测试款。 | 若用户提供升级 SKU：说明升级款如何承担利润、差异化、礼品、内容化或高客单角色。若用户未提供升级 SKU：写“用户未提供升级款 / 暂不规划”，不得补充升级款角色。 |
| 核心卖点 | 必须使用 `sell-grid` / `sell-item` 卡片，呈现 ICON + 英文短句 + 中文解释。建议 4 个卖点。 | 若用户提供升级 SKU：同样使用 `sell-grid` / `sell-item` 卡片。若用户未提供升级 SKU：写“用户未提供升级款 / 暂不规划”，不得生成升级款卖点卡片。 |
| 其他卖点 | 使用英文短语串联，分号分隔；中文解释可放在短语后。 | 若用户提供升级 SKU：使用英文短语串联，分号分隔；中文解释可放在短语后。若用户未提供升级 SKU：写“用户未提供升级款 / 暂不规划”。 |
| 定价 | `$xx.xx` 加粗，并解释该价格承担的战略角色。 | 若用户提供升级 SKU：`$xx.xx` 加粗，并解释该价格承担的战略角色。若用户未提供升级 SKU：写“用户未提供升级款 / 暂不规划”，不得生成升级款价格区间。 |
| 主要竞品 | 可用 `colspan="2"` 合并，列出 3-5 个核心 ASIN / 品牌。 | 同左。 |
| 销售渠道 | 可用 `colspan="2"` 合并，说明 Amazon、TikTok、官网、线下/批发等角色。 | 同左。 |

HTML 结构必须使用如下样式类，保证版式一致：

```html
<h3 style="margin-top:22px">SKU 规划与产品定义</h3>
<table>
  <thead>
    <tr>
      <th>产品规划</th>
      <th>基础款</th>
      <th>升级款</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>SKU / 英文命名</strong></td>
      <td><strong>{MODEL_BASIC} · {ENGLISH_NAME_BASIC}</strong><br><span class="small">命名逻辑：{naming_logic_basic}</span></td>
      <td><strong>{MODEL_UPGRADE} · {ENGLISH_NAME_UPGRADE}</strong><br><span class="small">命名逻辑：{naming_logic_upgrade}</span></td>
    </tr>
    <tr>
      <td><strong>一句 Slogan</strong></td>
      <td><strong>{slogan_basic}</strong><br><span class="small">{中文解释}</span></td>
      <td><strong>{slogan_upgrade}</strong><br><span class="small">{中文解释}</span></td>
    </tr>
    <tr>
      <td><strong>角色</strong></td>
      <td>{基础款战略角色}</td>
      <td>{升级款战略角色}</td>
    </tr>
    <tr>
      <td><strong>核心卖点</strong></td>
      <td>
        <div class="sell-grid">
          <div class="sell-item"><div class="sell-icon">{ICON}</div><div><div class="sell-title">{English Benefit}</div><div class="sell-cn">{中文解释}</div></div></div>
        </div>
      </td>
      <td>
        <div class="sell-grid">
          <div class="sell-item"><div class="sell-icon">{ICON}</div><div><div class="sell-title">{English Benefit}</div><div class="sell-cn">{中文解释}</div></div></div>
        </div>
      </td>
    </tr>
    <tr>
      <td><strong>其他卖点</strong></td>
      <td>{English Phrase 1}；{English Phrase 2}；{English Phrase 3}。</td>
      <td>{English Phrase 1}；{English Phrase 2}；{English Phrase 3}。</td>
    </tr>
    <tr>
      <td><strong>定价</strong></td>
      <td><strong>${price_basic}</strong>，{定价策略说明}</td>
      <td><strong>${price_upgrade}</strong>，{定价策略说明}</td>
    </tr>
    <tr>
      <td><strong>主要竞品</strong></td>
      <td colspan="2">{competitor_asins_or_brands}</td>
    </tr>
    <tr>
      <td><strong>销售渠道</strong></td>
      <td colspan="2">{channels_and_roles}</td>
    </tr>
  </tbody>
</table>
```

同时必须在 HTML CSS 中包含以下样式。如果报告已有同名样式，复用即可：

```css
.sell-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; }
.sell-item { display: grid; grid-template-columns: 34px 1fr; gap: 8px; align-items: start; border: 1px solid var(--line); background: #faf8f4; padding: 9px; min-height: 64px; }
.sell-icon { width: 30px; height: 30px; display: grid; place-items: center; border-radius: 50%; background: #ebe5dc; font-size: 17px; }
.sell-title { font-weight: 800; color: var(--ink); line-height: 1.25; }
.sell-cn { color: var(--muted); font-size: 12px; margin-top: 2px; }
```

ICON 规则：

- HTML 报告可用 emoji 或短英文标签；飞书画板版优先使用短英文标签（如 FLY、LOCK、MAG、QUIET、CARD、TASK、GIFT、FOCUS），避免图标兼容问题。
- 英文短句必须是用户利益，不要写内部功能名。例如写 `Travel-Light Size`，不要只写 `8.8 x 7 inch`。
- 中文解释必须补充规格、场景或购买理由。

交付前强制自检：

- HTML 必须存在标题 `SKU 规划与产品定义`。
- 该标题下第一张主表必须是三列表头：`产品规划`、`基础款`、`升级款`。
- 主表必须包含这些固定行：`SKU / 英文命名`、`一句 Slogan`、`角色`、`核心卖点`、`其他卖点`、`定价`、`主要竞品`、`销售渠道`。
- `核心卖点` 行中，用户实际提供的 SKU 必须使用 `sell-grid` / `sell-item` 卡片，并包含英文短句和中文解释；若用户未提供升级 SKU，升级款列不得使用卖点卡片或生成升级卖点，只写“用户未提供升级款 / 暂不规划”。
- 如果还有普通 SKU 矩阵表，只能放在固定表之后，并命名为“产品矩阵辅助说明”或类似辅助标题。

### 5.2 卖点表达

使用“ICON + 英文短句 + 中文解释”的形式，并优先嵌入 `SKU 规划与产品定义` 表格的“核心卖点”行。

示例：

| ICON | 英文短句 | 中文解释 |
|---|---|---|
| FLY | Travel-Light Size | 轻薄便携，适合外出 |
| LOCK | Sealed Magnet Safety | 封闭磁体与不可拆磁笔 |
| MAG | Easy-Pull Magnetic Dots | 强磁易吸附，不卡珠 |

### 5.3 利润成本核算

根据用户填写数据计算。

基础公式：

```text
平台佣金 = 售价 * 15%（除非用户给出其他比例）
退货预估 = 售价 * 退货率
广告预估 = 售价 * 广告占比
总成本 = 采购价 + FBA配送费用 + 头程费用 + 平台佣金 + 退货预估 + 广告预估 + 其他费用
净利润 = 售价 - 总成本
净利润率 = 净利润 / 售价
成本占比 = 总成本 / 售价
```

如果用户给出多种头程方式，分别计算快递、空运、海运。

必须给出结论：

- 是否可上市。
- 哪种履约方式适合首批。
- 哪种履约方式适合规模化。
- 广告空间是否足够。
- 是否需要提价、降本或砍功能。

### 5.4 产品策略

包含：

- 产品屋：功能价值、情绪价值、身份价值、POP、POD、RTB。
- 定价策略。
- 渠道策略：Amazon、TikTok、官网或用户指定渠道。
- Listing 主图/A+/视频信息层级。
- 30/60/90 天落地动作。
- Go/No-Go 红线。

必须额外输出：

- 竞品对比表：自家产品 vs 2-3 个关键竞品，包含定位、POP/POD、价格、用户好评/差评、证明资产和战略含义。
- 生命周期计划：当前是 introduction / growth / maturity / renewal 中哪一阶段，应采取什么投资动作。
- 传播计划：核心信息、平台语言、内容形式、KOL/KOC/专家/用户证言逻辑、KPI。
- 一页式策略全景图：Insight / Strategy / Marketing / Compliance / Decision / 30-60-90 天。

### 5.5 合规风险分析

最终完整报告必须包含合规风险分析。不要只写“注意合规”，必须固定拆成三段，并输出可以执行的风险表：

1. 侵权风险分析
2. 安全风险分析
3. 其他风险

开头必须说明：本节是基于官方数据源和公开信息的上市前初筛，不替代律师 FTO 意见、正式商标检索报告或第三方实验室测试。

#### 5.5A 侵权风险分析

侵权风险必须覆盖：

- 核心关键词商标侵权风险：产品关键词、品牌拟命名、材料命名、系列命名、Slogan、常用英文卖点词。
- 产品专利/外观设计风险：产品结构、材料夹层、配件收纳、边缘包覆、外观造型、包装组合方式。
- 营销素材风险：竞品图片、图标、A+ 结构、对比表达、说明书和包装视觉。

必须调用或引用这些官方数据源：

- USPTO Trademark Search：`https://tmsearch.uspto.gov/search/search`
- USPTO Patent Public Search：`https://www.uspto.gov/patents/search/patent-public-search`
- EUIPO eSearch plus / Designs：`https://www.euipo.europa.eu/en/search-ip`
- 如涉及欧盟专利初筛，可补充 EPO Espacenet：`https://www.epo.org/en/searching-for-patents/technical/espacenet`

推荐表格：

| 核验对象 | 官方数据源 | 当前初筛判断 | 风险等级 | 上市前动作 |
|---|---|---|---|---|
| 核心关键词商标 | USPTO / EUIPO | 是否为通用描述词、是否有近似活跃商标、是否与同类商品冲突 | 高/中/低 | 正式检索、改名、律师确认 |
| 产品专利/外观设计 | USPTO Patent / EUIPO Designs / Espacenet | 是否涉及结构、收纳、外观、材料组合专利或外观设计 | 高/中/低 | FTO 检索、竞品权利人反查、设计绕开 |

#### 5.5B 安全风险分析

安全风险必须结合产品使用者、材料、结构、配件、渠道和销售地区调用或引用法规文件数据源。优先使用官方监管或标准入口，不要只写经验判断。

常用数据源示例：

- CPSC Children's Product Certificate：`https://www.cpsc.gov/Business--Manufacturing/Testing-Certification/Childrens-Product-Certificate`
- CPSC Toy Safety：`https://www.cpsc.gov/toysafety`
- CPSC ASTM F963 Requirements：`https://www.cpsc.gov/Business--Manufacturing/Business-Education/Toy-Safety/ASTM-F-963-Chart`
- CPSC Art Materials / LHAMA：`https://www.cpsc.gov/business--manufacturing/business-education/business-guidance/art-materials`
- EU Toy Safety Directive 2009/48/EC：`https://health.ec.europa.eu/publications/directive-200948ec_en`

推荐表格：

| 风险项 | 法规/数据源 | 触发原因 | 等级 | 上市前证据清单 |
|---|---|---|---|---|
| 儿童产品/玩具属性 | CPSC / ASTM F963 / EU Toy Safety | 面向儿童、课堂、玩具、学习用品 | 高 | CPC、CPSIA、ASTM F963、EN 71、年龄标签 |
| 材料与配件 | CPSC Art Materials / SDS | 油墨、颜料、胶水、塑料、皮肤接触 | 高/中 | ASTM D-4236、SDS、重金属/邻苯测试 |

#### 5.5C 其他风险

其他风险至少考虑：

- 平台审核：Amazon、TikTok Shop、官网广告对儿童、安全、环保、功效和竞品对比宣称的审核。
- 营销宣称：绝对化、环保、无毒、治疗/改善、行业领先、永不损坏等需要证据的表达。
- 包装物流：大件、易碎、重货、配件缺失、说明书和标签。
- 售后与评价：气味、破损、缺件、儿童误用、材料变形。

需要主动检查的触发因素：

- 儿童/婴幼儿/宠物使用。
- 食品接触、入口、皮肤长时间接触、化妆品或个护。
- 电子、电池、磁体、发热、无线、灯光、声音。
- 承重家具、睡眠、泳池/户外、防撞/保护、安全防护。
- 医疗、健康、治疗、感官、ASD/ADHD/SPD、改善症状等宣称。
- Amazon US / CA / EU / UK 等不同站点合规文件差异。

推荐表格：

| 风险 | 触发原因 | 严重度 | 建议动作 | 上市前证据 |
|---|---|---|---|---|
| 示例：儿童窒息/缠绕/困住 | 产品可进入、可封闭、可接触填充物 | 高 | 结构改良、警示、第三方测试 | 测试报告、CPC、标签图 |

按品类选择合规要点：

- 儿童产品：CPC、tracking label、CPSC-accepted lab、年龄分级、警示标签。
- Bean bag / 软体家具 / cover：ASTM F1912、child-resistant zipper、填充物接触风险、可进入空间、警示语。
- 感官/健康/治疗相关产品：避免 treat / cure / relieve symptoms / improve autism or ADHD 等医疗化承诺；需要 claim substantiation 和平台广告词审查。
- 磁体/电子/电池：磁体吞咽、电池、FCC/UL/Prop 65 等。
- CA / EU 等站点：语言标签、进口商信息、当地儿童产品法规和平台文件要求。

必须给出合规决策：

- `Go`：合规文件和测试路径清晰，风险可控。
- `Conditional Go`：产品机会成立，但必须通过指定测试/文件/结构验证后再下大货。
- `No-Go until redesign`：存在结构性安全风险，需改设计再评估。

## 6. HTML 报告要求

HTML 应包含：

- 清晰的标题和执行摘要。
- KPI 卡片。
- ECharts 图表：细分销量、销额、优先级、趋势、竞品销量、价格段销量、Kano、新品卖点。
- 表格：细分市场、竞品、用户画像、Kano、VOC、卖点排序、产品定义、成本核算、策略。
- 产品主图。
- 风险提示。

样式要求：

- 商业报告风格，适合产品经理和管理层阅读。
- 信息密度高但不要杂乱。
- 移动端可读。
- 所有关键结论必须出现在 HTML 中，聊天窗口只给摘要和文件路径。

## 7. 交付话术

最终交付示例：

```text
报告已完成：{path}
核心建议：...
下一步建议：...
```
