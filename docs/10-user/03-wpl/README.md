# WPL 规则语言

WPL (Warp Processing Language) 是 Warp Parse 解析子系统（warp-parse）使用的规则语言，主要用于描述字段抽取、协议解析与简单判定逻辑。该目录文档与 `crates/wp-lang` 的解析实现保持一致。

## 内容概览

- [WPL 语言基础](./01-wpl_basics.md)
- [WPL 示例](./02-wpl_example.md)
- [WPL 管道函数](./03-wpl_pipe_functions.md)
- [WPL 语法（EBNF）](./04-wpl_grammar.md)

## 快速示例

```wpl
package demo {
  rule http_access {
    |decode/base64|unquote/unescape|
    (
      digit:status,
      time_3339:recv_time,
      ip:client_ip,
      http/request<[,]>,
      http/agent"
    )
  }
}
```

## 管道函数速览

| 类型 | 函数名 | 说明 |
|------|--------|------|
| **选择器** | `take(<name>)` | 选择指定字段为活跃字段 |
| | `last()` | 选择最后一个字段为活跃字段 |
| **字段集合操作** | `f_has(<name>)` | 检查指定字段是否存在 |
| | `f_chars_has(<name>, <value>)` | 检查指定字段值是否等于字符串 |
| | `f_chars_not_has(<name>, <value>)` | 检查指定字段值是否不等于字符串 |
| | `f_chars_in(<name>, [v1, v2, ...])` | 检查指定字段值是否在列表中 |
| | `f_digit_has(<name>, <num>)` | 检查指定字段数值是否相等 |
| | `f_digit_in(<name>, [n1, n2, ...])` | 检查指定字段数值是否在列表中 |
| | `f_ip_in(<name>, [ip1, ip2, ...])` | 检查指定 IP 字段是否在列表中 |
| **活跃字段操作** | `has()` | 检查活跃字段是否存在 |
| | `chars_has(<value>)` | 检查活跃字段值是否等于字符串 |
| | `chars_not_has(<value>)` | 检查活跃字段值是否不等于字符串 |
| | `chars_in([v1, v2, ...])` | 检查活跃字段值是否在列表中 |
| | `digit_has(<num>)` | 检查活跃字段数值是否相等 |
| | `digit_in([n1, n2, ...])` | 检查活跃字段数值是否在列表中 |
| | `ip_in([ip1, ip2, ...])` | 检查活跃 IP 字段是否在列表中 |
| **转换函数** | `json_unescape()` | 对 chars 字段进行 JSON 反转义 |
| | `base64_decode()` | 对 chars 字段进行 Base64 解码 |

## 相关文档

- 语法实现参考：`crates/wp-lang/src/parser/`
- 管道函数实现：`crates/wp-lang/src/parser/wpl_fun.rs`
- 数据类型定义：外部 crate `wp-model-core`
