优先级 P0（高收益、低迁移成本）

  - 量词语法糖
      - 动机：现在支持 N*、^N，但不直观；opt 语义仅作用第一个字段。
      - 建议：引入标准量词/可选糖
          - 字段：field?（0..1）、field+（1..n）、field{n}（n 次）、field{n,m}（区间）
          - 组：repeat(group){n,m} 或 some_of(elem){n,m}
      - 迁移：保持旧写法继续有效，新增糖语法转译到现有语义
  - 原始字符串与更直观的转义
      - 动机：目前 quoted_string 需要反斜杠转义；复杂 JSON/路径容易“转义地狱”
      - 建议：支持 r"..." 原始字符串（内部不处理 \ 转义），用于注解值、格式参数等
      - 迁移：r"..." 是新增能力，不破坏旧写法
  - 错误上下文与诊断
      - 动机：定位难；我们已为 array 元素补了 array/[idx]，建议横向推广
      - 建议：所有协议解析器（json/kv/http 等）在错误栈带“字段路径 + 组/字段索引 + 预期类型”：
          - 例：group[2]/field[5] <json> $.items/[3]/name expected chars
      - 迁移：仅改错误包装；不影响语义
  - 预处理步骤命名空间化（已做标准名，进一步完善）
      - 动机：现在名字统一为 base64_decode/unquote_unescape/hex_decode；后续扩展时建议分类
      - 建议：采用 verb_对象 的命名策略并保留命名空间前缀（可选）
          - decode/base64、decode/hex、unquote/unescape
      - 迁移：保持我们已上线的标准名，新增同义名以备后续迁移（再择机统一输出 name）

  优先级 P1（收益高、中等成本）

  - 严格/宽松模式（字段/组/协议级）
      - 动机：生产场景里数据常脏；严格失败 vs 宽松跳过/保留原值
      - 建议：
          - 字段级：strict(true|false) 或 #[strict] 注解；宽松时失败元素可保留 chars 或跳过并记录
          - 协议级：json/exact_json/array 等支持 strict/lenient 参数
      - 迁移：默认严格，新增可选开关，向后兼容
  - 复用与模块化
      - 动机：大规则复用差，复制粘贴多
      - 建议：
          - 宏/模板：macro name(args) { body }；在 rule 中展开
          - import/include：import "/pkg/rule" as r; 或 include "path.wpl"
      - 迁移：新增能力；先做 include（最小）、后做 macro（参数化）
  - 统一组合子语义（组合子=order/alt/some_of/opt）
      - 动机：语义已清晰，但可以统一“范式”
      - 建议：用一致的组合子族和参数表达
          - choice(...)（原 alt），repeat(elem, min=?, max=?）替代 some_of/量词组合，optional(elem) 替代 opt
      - 迁移：保留现名，新增别名，文档统一“范式”优先
  - Array 自定义范围/分隔（按需）
      - 动机：有些非标准列表不是 [,,,]
      - 建议：在 array 上支持 array<range,sep> 最小地开放自定义分隔
          - 例如 array<[,]>（现状）、array<{,}>（自定义括号）、或 array<;]>
      - 迁移：默认不写仍等价现状；仅新增能力

  优先级 P2（中期优化、需要设计验证）

  - 联合类型/自动判定（弱类型友好）
      - 动机：数组或字段里可能混合类型（ip 或 digit）
      - 建议：
          - 联合：array/(ip|digit) 或 array/alt(ip,digit)（每元素尝试 alt）
          - 自动：auto 尝试有限白名单类型（ip/digit/bool），成功即用
      - 风险：性能与歧义；需限白名单、可控
  - 插件接口增强
      - 动机：@plugin 块形态已落地；可补充参数化、安全、版本协商
      - 建议：
          - 参数：@plugin(id: x) { with param(key="v") (group...) }
          - 安全沙箱/能力声明：plugin manifest（支持哪些 meta/type）
          - 版本：@plugin(id: x, ver: "1.2")，升级可兼容
      - 迁移：先做参数透传（最小），再做能力/版本
  - DSL 版本化与弃用策略
      - 动机：本次已做不兼容；需要机制托底未来变更
      - 建议：
          - 文件头加 version: 2 或 #![wpl(version="2")]
          - 变更文档常驻，提供 rg/脚本迁移
      - 迁移：文档/工具先行，后落地版本标识解析
  - 工具链支持
      - 动机：规模规则开发需要更好的反馈
      - 建议：
          - 代码格式化（基于 AST 的 pretty printer）
          - 语言服务器（诊断、补全、跳转）
          - 快速模拟器（预处理/字段试跑）

  可立即落地的小改动（推荐先做）

  - 文档：在 EBNF 门面加上
      - array 的“允许尾随逗号/空数组”，已在基础文档补充；建议门面同步一句
      - 与 json 的组合姿势（先 json，再 array/subtype），减少误解
  - 错误上下文：把我们在 array 做的“array/[idx]”拓展到 json/kv（例如 json 路径 $.a/[0]/b）
  - patten_gen：已为 array 做了最小实现；可以继续为 kv/json 增一条简单样例（演示生成效果）

  如果你同意，我可以：

  1. 在 EBNF 门面补上 array 的简述（允许尾随逗号/空数组），并在基础文档加“与 json 组合”的示例；
  2. 将错误上下文增强推广到 kv/json（保持非侵入式，先加标签，不改语义）；
  3. 提出一版“量词语法糖”的语法草案（字段与组）和转译规则，供你评审。
