# WPL Rule Language

WPL (Warp Processing Language) is the rule language used by the Warp Parse parsing subsystem (warp-parse), primarily for describing field extraction, protocol parsing, and simple decision logic. The documentation in this directory is consistent with the `crates/wp-lang` parser implementation.

## Content Overview

- [WPL Language Basics](./01-wpl_basics.md)
- [WPL Examples](./02-wpl_example.md)
- [WPL Pipe Functions](./03-wpl_pipe_functions.md)
- [WPL Grammar (EBNF)](./04-wpl_grammar.md)

## Quick Example

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

## Pipe Functions Quick Reference

| Category | Function | Description |
|----------|----------|-------------|
| **Selectors** | `take(<name>)` | Select specified field as active field |
| | `last()` | Select last field as active field |
| **Field Set Operations** | `f_has(<name>)` | Check if specified field exists |
| | `f_chars_has(<name>, <value>)` | Check if specified field value equals string |
| | `f_chars_not_has(<name>, <value>)` | Check if specified field value does not equal string |
| | `f_chars_in(<name>, [v1, v2, ...])` | Check if specified field value is in list |
| | `f_digit_has(<name>, <num>)` | Check if specified field number equals value |
| | `f_digit_in(<name>, [n1, n2, ...])` | Check if specified field number is in list |
| | `f_ip_in(<name>, [ip1, ip2, ...])` | Check if specified IP field is in list |
| **Active Field Operations** | `has()` | Check if active field exists |
| | `chars_has(<value>)` | Check if active field value equals string |
| | `chars_not_has(<value>)` | Check if active field value does not equal string |
| | `chars_in([v1, v2, ...])` | Check if active field value is in list |
| | `digit_has(<num>)` | Check if active field number equals value |
| | `digit_in([n1, n2, ...])` | Check if active field number is in list |
| | `ip_in([ip1, ip2, ...])` | Check if active IP field is in list |
| **Conversion Functions** | `json_unescape()` | JSON unescape for chars field |
| | `base64_decode()` | Base64 decode for chars field |

## Related Documentation

- Grammar implementation reference: `crates/wp-lang/src/parser/`
- Pipe functions implementation: `crates/wp-lang/src/parser/wpl_fun.rs`
- Data type definitions: external crate `wp-model-core`
