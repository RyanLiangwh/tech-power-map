# 数据维护说明

所有人物与关系数据都在 **`data.js`**（与渲染逻辑 `index.html` 分离）。改数据不用动代码，改完刷新页面即可。

## 数据结构

```js
window.TPM = { PEOPLE:[...], RELATIONS:[...], INSIGHTS:[...] }
```

### PEOPLE 人物
| 字段 | 说明 |
|---|---|
| `id` | 唯一英文短 id（关系、洞察都靠它引用，**不可重复**） |
| `en` / `zh` | 英文名 / 中文名 |
| `org` | 所属组织 |
| `title` | 头衔/一句话简介 |
| `g` | 阵营 1–7（1 科技巨头 / 2 AI / 3 半导体硬件 / 4 中国科技 / 5 投资人 / 6 平台新贵 / 7 先驱传奇） |
| `era` | 时代：`ind` 工业 / `elec` 晶体管 / `pc` PC / `web` 互联网 / `mob` 移动云 / `ai` AI |
| `w`（可选） | Wikipedia 页面标题覆盖（头像/查证抓不准时手动指定） |

### RELATIONS 关系
| 字段 | 说明 |
|---|---|
| `s` / `t` | 两端人物 id |
| `k` | 关系类型：`cofounder` `work` `invest` `mentor` `rival` `ally` `friend` `family` `feud` |
| `d` | 关系描述 |
| `c`（可选） | 可信度 `1`/`2`/`3`，**不填则按类型自动分级**（见下） |
| `src`（可选） | 来源链接数组，如 `["https://...","https://..."]`；不填详情页显示「来源待补」 |

### INSIGHTS 隐藏关系洞察
`{ t:标题, p:[节点 id 链路], n:说明 }`，`p` 里相邻两点必须在 RELATIONS 中存在边。

## 可信度分级（来源机制）

| 档 | 标签 | 含义 |
|---|---|---|
| 1 | 公开可查 | 工商/任职/股权/亲属等可直接核实的结构性事实 |
| 2 | 广泛报道 | 主流媒体多方报道、业内公认 |
| 3 | 含主观概括 | 含坊间说法、未证实传闻或带主观判断的概括 |

**类型默认档**（`KIND_CONF`，在 `index.html` 顶部，可调）：
`family/cofounder/work/invest → 1`，`mentor/rival/ally/feud → 2`，`friend → 3`。
单条关系想覆盖默认，加 `c:` 即可（例：苹果↔英特尔「洽谈入股」是传闻，已设 `c:3`）。

## 逐条补来源（推荐做法）
找到对应关系，加 `src` 数组即可，例：
```js
{s:"jensen",t:"lisasu",k:"family",d:"...",src:["https://en.wikipedia.org/wiki/Jensen_Huang"]},
```
补了来源后，建议把该条 `c` 同步调整为更高可信度。

## 季度校对清单
- [ ] 接班/离职变动：CEO 交棒、高管跳槽（标题、`org`、关系是否过期）
- [ ] 新增重要人物与关系；删除已失效/证伪的条目
- [ ] 给「来源待补」的高热度关系补 `src`，并复核 `c`
- [ ] 把当年新发生的恩怨/合作/投资补进去
- [ ] 跑一遍 `node check.js`（见下）确认无悬空边、无重复 id

## 自检
```
node -e 'global.window={};eval(require("fs").readFileSync("data.js","utf8"));const t=window.TPM,ids=new Set(t.PEOPLE.map(p=>p.id));console.log("people",t.PEOPLE.length,"rel",t.RELATIONS.length,"dangling",t.RELATIONS.filter(r=>!ids.has(r.s)||!ids.has(r.t)).length)'
```
