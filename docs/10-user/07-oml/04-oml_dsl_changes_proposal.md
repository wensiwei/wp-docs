# OML DSL 改进建议（草案，征求评审）

目的：提升 OML 的一致性、可读性与可维护性；在保证向后兼容的前提下，逐步优化关键字与语义。本文仅为“提案”，用于多人评审，不代表已实现。

注：文内涉及的“隐私段”或“隐私短名”仅为语法建议；当前引擎默认不启用运行期隐私/脱敏处理。

相关实现参考：`crates/wp-oml/src/parser/*`

## 1. 已完成的文档层调整

- EBNF 命名统一为 unix-like（全部小写、下划线分隔）。
  - 例如：`OML -> oml`、`Header -> header`、`TakeExpr -> take_expr` 等。
- 新增《OML 使用示例》文档，覆盖常见能力与易错点。

## 2. 关键词与语义改进（第一阶段，建议采纳）

这些改动建议先“文档主推 + 解析兼容”，不立即破坏老写法。

1) 明确并保留 `read`/`take` 的差异；删除 `crate`
- 现状：`read` 为非破坏性读取（可反复读取）；`take` 为破坏性读取（取走后从 src 移除，不可再取）
- 建议：
  - 保留 `read` 与 `take` 的差异并在文档中明确定位；`crate` 不再文档曝光（解析保留兼容）。
- 示例（行为差异）：
```oml
name : diff
---
X1 = read(A1) ;   # 多次读取仍可得到值
X2 = read(A1) ;
Y1 = take(B1) ;   # 第一次取走
Y2 = take(B1) ;   # 第二次将取不到（B1 已从 src 移除）
```

2) 收敛参数键，避免 `in` 歧义
- 现状：`in` 同时用于 match 条件与 read/take 参数；易混淆。
- 建议：增加 ` keys:` 作为 `in:` 的同义键；文档主推 `keys:`，解析兼容 `in:`。
- 示例：
```oml
# 旧
arr = collect read(in:[sport,dport]) ;
# 新（推荐）
arr = collect read(keys:[sport,dport]) ;
```


4) 规范 match 分支分隔
- 建议：规范为“每个分支以分号 `;` 结尾”；解析继续兼容逗号 `,`。
- 示例：
```oml
X = match read(k) {
  chars(A) => chars(bj);
  _ => chars(sz);
};
```

5) 强化错误上下文
- 建议：在参数键未知、pipe 函数参数非法、SQL 主体不合法等位置，增加 `Expected: ...` 提示与可用枚举值列表。
 - 示例（期望错误文案）：
   - 参数键未知：
```
read(foo:bar)
        ^ unknown arg key 'foo'. Expected one of: get | keys | option | <json_path>
```
   - pipe 参数非法：
```
pipe read(x) | url_get(hosts)
                      ^ invalid url_get arg 'hosts'. Expected: domain | host | uri | path | params | default
```
   - SQL 主体不合法（严格模式）：
```
select a,b from table-1 where x = read(k);
                 ^ invalid identifier 'table-1'. Expected: [A-Za-z0-9_.] or '*'
```

## 3. 关键词与语义改进（第二阶段，可选项）

这些改动建议需要更多评审，默认不立即落地。

1) 管道语法去掉前缀 `pipe`
- 现状：`pipe read(x) | to_json` 冗余。
 - 建议：允许 `read(x) | to_json`，`pipe` 可选。解析需在 `read/take` 后前瞻 `|` 分支。
- 示例：
```oml
# 现有
out = pipe read(ports) | to_json ;
# 提案
out = read(ports) | to_json ;
```

2) `map` 增加直观别名
- 建议：新增 `object { ... }` ，语义与 `map` 相同；文档主推新名，删除 `map` 。


4) 隐私段短名
- 建议：`privacy_ip` -> `ip`、`privacy_keymsg` -> `keymsg` 等短名；或引入 `privacy { key: ip; ... }` 块体语法。


6) pipe 函数家族命名一致性（引入别名）
- 建议：`arr_get`->`nth`；`obj_get`->`get`；`path_get`->`path(...)`；`url_get`->`url(...)` 等；先加别名，逐步替换示例。

7) 内置时间函数别名
- 建议：新增 `now.date()` / `now.time()` / `now.hour()` 等更自然的写法，保留 `Time::now_*`。

8) header `rule` 更直观
- 建议：允许 `rules:`（复数）或 `use:`/`import:` 作为同义词。

## 4. 示例对照（旧/新）

1) 读取与参数键：
```oml
# 旧
name : ex
---
ports : array = collect read(in:[sport,dport]) ;
user  = take(user) ;

# 新（推荐）
name : ex
---
ports : array = collect read(keys:[sport,dport]) ;
user  = read(user) ;
```

2) match 分隔与 `!=` 提案：
```oml
# 现（兼容逗号/分号）
X = match read(k) {
  chars(A) => chars(bj),
  _ => chars(sz)
};

# 建议（统一分号；可选支持 !=）
X = match read(k) {
  chars(A) => chars(bj);
  !(chars(B)) => chars(hk);
  _ => chars(sz);
};
```

3) 管道省略 `pipe`（提案）：
```oml
# 现
json = pipe read(ports) | to_json ;
# 提案
json = read(ports) | to_json ;
```

4) map 别名（提案）：
```oml
# 现
values : obj = map { a:digit = read(); };
# 提案
values : obj = object { a:digit = read(); };
```

## 5. 兼容与迁移建议
- 文档先行，解析层逐步“加别名、保留兼容”，并在日志/诊断中提示新写法。
- 统一在 `wparse` 或格式化工具中加入风格检查（如强制分号）。
- 给出迁移窗口与版本化策略（例如：3 个次版本内保留兼容，之后移除或仅告警）。

## 6. 开放问题（待评审）
- 是否需要完全弃用 `take`（还是永久保留为 read 别名）？
- 参数键是否统一采用 `keys:`（与 match 中的 `in` 区分清晰）？
- 是否需要 pipeline 无 `pipe` 的语法（解析歧义与实现复杂度评估）？
- 隐私短名的枚举边界（是否与数据类型短名冲突）？
- SQL 严格模式的默认行为与回退策略（是否提供全局/每条目级开关）。

> 评审通过后，我可以按“第一阶段”方案提交解析与错误信息的兼容性实现，并同步更新示例与指南。
