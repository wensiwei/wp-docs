# WPL Parsing Examples

This document compiles various data parsing examples in the WPL language, including test data, parsing rules, and expected results for learning and reference.

## Table of Contents

1. [Basic Type Parsing](#basic-type-parsing)
2. [Time Format Parsing](#time-format-parsing)
3. [Network Data Parsing](#network-data-parsing)
4. [JSON Data Parsing](#json-data-parsing)
5. [Protocol Parsing](#protocol-parsing)
6. [Field Pipes Examples](#field-pipes-examples)
7. [Complex Combination Examples](#complex-combination-examples)

## Basic Type Parsing

### 1. Number Parsing

**Parsing Rule**:
```wpl
digit
```

**Number Parsing with Names**:
```wpl
digit:length
digit:id
digit:port
```

**Test Data**:
```
200
368
190
34616
```

**Expected Results**:
```
length: 200
port: 8080
id: 12345
```

### 2. String Parsing

**Parsing Rule**:
```wpl
chars
```

**String Parsing with Names**:
```wpl
chars:dev-name
chars:http/referer
chars:user-agent
```

**Test Data**:
```
nginx-server
https://www.example.com
curl/7.68.0
```

**Expected Results**:
```
dev-name: "nginx-server"
http/referer: "https://www.example.com"
user-agent: "curl/7.68.0"
```

### 3. Separator Parsing

**Underscore Separator**:
```wpl
_        # Single underscore
_^2      # Repeat 2 times
```

**Comma Separator**:
```wpl
<[,]>    # Comma separated
```

**Quote Separator**:
```wpl
"        # Quote
```

**Space Separator**:
```wpl
' '      # Space character
```

**Example Usage**:
```wpl
(ip, _^2, time, chars)
```

## Time Format Parsing

### 1. CLF (Common Log Format) Time Parsing

**Parsing Rule**:
```wpl
time/clf
```

**Test Data**:
```
06/Aug/2019:12:12:19 +0800
```

**Expected Result**:
```
2019-08-06 12:12:19  # Converted to standard time format
```

**Time with Brackets**:
```
[06/Aug/2019:12:12:19 +0800]
```

**Test Example**:
```wpl
rule test {
    (time/clf)
}
```

### 2. Standard Time Format Parsing

**Parsing Rule**:
```wpl
time
```

**Test Data Examples**:
```
2023-05-15 07:09:12
2023/5/15 15:09:12
```

**Expected Results**:
```
time: "2023-05-15 07:09:12"
time: "2023-05-15 15:09:12"
```

## Network Data Parsing

### 1. HTTP Request Parsing

**Parsing Rule**:
```wpl
http/request
```

**Test Data**:
```
GET /nginx-logo.png HTTP/1.1
```

**Expected Result**:
```
http/request: "GET /nginx-logo.png HTTP/1.1"
```

### 2. HTTP Status Code Parsing

**Parsing Rule**:
```wpl
http/status
```

**Test Data**:
```
200
```

**Expected Result**:
```
http/status: 200
```

### 3. HTTP User-Agent Parsing

**Parsing Rule**:
```wpl
http/agent
```

**Test Data**:
```
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36
```

**Expected Result**:
```
http/agent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36"
```

### 4. URL Parsing

**Parsing Rule**:
```wpl
chars:url  # Or use chars directly
```

**Test Data**:
```
http://119.122.1.4/
```

**Expected Result**:
```
url: "http://119.122.1.4/"
```

### 5. IP Address Parsing

**Parsing Rule**:
```wpl
ip
```

**Test Data**:
```
192.168.1.2
```

**Expected Result**:
```
ip: 192.168.1.2
```

### 6. Email Parsing

**Parsing Rule**:
```wpl
email
```

**Test Data Examples**:
```
johnjoke@example.com
user+tag@example-domain.com
first_last@example.com
foo-bar@example.co
```

**Expected Result**:
```
email: "johnjoke@example.com"
```

## JSON Data Parsing

### 1. Basic JSON Parsing

**Parsing Rule**:
```wpl
rule test {
    (json)
}
```

**Test Data**:
```json
{ "age": 18}
```

**Expected Result**:
```
age: 18
```

### 2. Specified Field Parsing

**Parsing Rule**:
```wpl
rule test {
    (json(chars@path, chars@txt) |json_unescape())
}
```

**Test Data**:
```json
{"path":"c:\\users\\fc\\file","txt":"line1\nline2"}
```

**Expected Results**:
```
path: "c:\\users\\fc\\file"
txt: "line1\nline2"  # Contains actual newline character
```

### 3. JSON Field Existence Check

**Parsing Rule**:
```wpl
rule test {
    (json |f_has(age))
}
```

**Test Data**:
```json
{ "age": 18}
```

**Expected Result**:
```
age: 18
```

**Failure Example**:
```wpl
rule test {
    (json |f_has(age1))  # Field does not exist
}
```

### 4. JSON Numeric Value Check

**Parsing Rule**:
```wpl
rule test {
    (json |f_digit_has(age, 18))
}
```

**Test Data**:
```json
{ "name": "china", "age": 18}
```

**Expected Results**:
```
name: "china"
age: 18
```

### 5. JSON Numeric List Check

**Parsing Rule**:
```wpl
rule test {
    (json |f_digit_in(age, [18, 19]))
}
```

**Test Data**:
```json
{ "name": "china", "age": 18}
```

**Expected Results**:
```
name: "china"
age: 18
```

## Protocol Parsing

### 1. Base64 Decoding

**Parsing Rule**:
```wpl
|decode/base64|
# Or
|base64|
```

**Complete Example**:
```wpl
|decode/base64|(digit:id<<,>>,time,sn,chars\:),opt(kv\;), (*kv\,)
```

**Test Data**:
Base64 encoded Huawei firewall log

**Expected Result**:
Decoded text format log

### 2. KV Key-Value Pair Parsing

**Basic KV Parsing**:
```wpl
kv
```

**KV Parsing with Field Name**:
```wpl
kv(@CID)
```

**Test Data**:
```
CID=0x814f041e;vsys=CSG_Security
```

**Expected Results**:
```
CID: "0x814f041e"
vsys: "CSG_Security"
```

### 3. Repeat Pattern Parsing

**Fixed Repeat Count**:
```wpl
12*kv    # Repeat 12 KV pairs
2*_      # Repeat 2 underscores
7*kv     # Repeat 7 KV pairs
```

**Arbitrary Repeat**:
```wpl
*kv      # Repeat any number of KVs
```

### 4. Optional Field Parsing

**Optional Field**:
```wpl
opt(kv\;)  # Optional KV pair (ending with semicolon)
```

### 5. Escape and Quote Handling

**String Decode Mode**:
```wpl
|str_mode(decoded)|
```

**Unquote/Unescape**:
```wpl
|unquote/unescape|
```

## Field Pipes Examples

Field Pipes are a powerful feature of WPL that allow further processing, validation, and transformation of parsed fields. The pipe operator `|` is used to chain multiple processing steps.

### 1. Encoding/Decoding Pipes

#### Base64 Decoding

**Parsing Rule**:
```wpl
rule test {
    (|decode/base64| (digit:id, time, chars:message))
}
```

**Test Data**:
```
SGVsbG8gV29ybGQxMjM0NTY3OjAwOjAwOjAw
```

**Expected Results**:
```
id: 1234567
time: "00:00:00"
message: "Hello World"
```

#### Hex Decoding

**Parsing Rule**:
```wpl
rule test {
    (|decode/hex| (chars:data))
}
```

**Test Data**:
```
48656c6c6f20576f726c64
```

**Expected Result**:
```
data: "Hello World"
```

### 2. String Processing Pipes

#### Unescape/Unquote Processing

**Parsing Rule**:
```wpl
rule test {
    (|unquote/unescape| (json(chars@path, chars@txt)))
}
```

**Test Data**:
```json
{"path":"c:\\users\\fc\\file","txt":"line1\nline2"}
```

**Expected Results**:
```
path: "c:\users\fc\file"  # Backslash properly processed
txt: "line1\nline2"       # Newline preserved
```

#### String Mode Switch

**Parsing Rule**:
```wpl
rule test {
    (json(chars@path, chars@txt) |json_unescape())
}
```

**Test Data**:
```json
{"path":"c:\\users\\fc\\file","txt":"line1\nline2"}
```

**Expected Results**:
```
path: "c:\\users\\fc\\file"
txt: "line1\nline2"  # Contains actual newline character
```

### 3. Field Validation Pipes

#### Field Existence Check

**Parsing Rule**:
```wpl
rule test {
    (json |f_has(name))
}
```

**Test Data**:
```json
{"name": "Alice", "age": 25}
```

**Expected Results**:
```
name: "Alice"
age: 25
```

**Failure Example**:
```wpl
rule test {
    (json |f_has(email))  # email field does not exist
}
```

#### String Value Check

**Parsing Rule**:
```wpl
rule test {
    (json |f_chars_has(status, success))
}
```

**Test Data**:
```json
{"status": "success", "message": "Operation completed"}
```

**Expected Results**:
```
status: "success"
message: "Operation completed"
```

#### String Not In List Check

**Parsing Rule**:
```wpl
rule test {
    (json |f_chars_not_has(level, error))
}
```

**Test Data**:
```json
{"level": "info", "msg": "Normal operation"}
```

**Expected Results**:
```
level: "info"
msg: "Normal operation"
```

#### String Value Range Check

**Parsing Rule**:
```wpl
rule test {
    (json |f_chars_in(priority, [high, medium]))
}
```

**Test Data**:
```json
{"priority": "high", "task": "Backup"}
```

**Expected Results**:
```
priority: "high"
task: "Backup"
```

### 4. Numeric Validation Pipes

#### Numeric Equality Check

**Parsing Rule**:
```wpl
rule test {
    (json | digit_exists(age, 25))
}
```

**Test Data**:
```json
{"name": "Bob", "age": 25}
```

**Expected Results**:
```
name: "Bob"
age: 25
```

#### Numeric Range Check

**Parsing Rule**:
```wpl
rule test {
    (json | digit_in(score, [80, 85, 90, 95, 100]))
}
```

**Test Data**:
```json
{"student": "Tom", "score": 90}
```

**Expected Results**:
```
student: "Tom"
score: 90
```

### 5. IP Address Validation Pipes

**Parsing Rule**:
```wpl
rule test {
    (json | ip_in(client_ip, ["192.168.1.0/24", "10.0.0.0/8"]))
}
```

**Test Data**:
```json
{"client_ip": "192.168.1.100", "action": "login"}
```

**Expected Results**:
```
client_ip: "192.168.1.100"
action: "login"
```

### 6. Multi-Step Pipe Processing

**Parsing Rule**:
```wpl
rule test {
    (|decode/base64| (json | exists_digit(age, 25)) | str_mode(decoded))
}
```

**Data Flow**:
1. Base64 decode raw data
2. Parse JSON
3. Validate age field equals 25
4. Apply string decode mode

**Test Data** (Base64 encoded):
```
eyJuYW1lIjogIkFsaWNlIiwgImFnZSI6IDI1fQ==
```

**Decoded Data**:
```json
{"name": "Alice", "age": 25}
```

**Expected Results**:
```
name: "Alice"
age: 25
```

### 7. Pipes Combined with Groups

**Parsing Rule**:
```wpl
rule test {
    (|decode/base64| (digit:id, chars:name) | (exists(id) & chars_exists(name, "admin")))
}
```

**Function Description**:
- First perform Base64 decoding
- Parse numeric ID and character name
- Validate ID field exists and name equals "admin"

### 8. Practical Application Examples

#### Huawei Firewall Log Processing

**Parsing Rule**:
```wpl
rule huawei_firewall {
    |decode/base64|
    (digit:id<<,>>, time, sn, chars\:,),
    opt(kv\;),
    (*kv\,)
}
```

**Processing Flow**:
1. Base64 decode log data
2. Parse fixed format header (ID, time, serial number, etc.)
3. Parse optional key-value pairs
4. Parse multiple key-value pairs

#### JSON Parsing with Validation

**Parsing Rule**:
```wpl
rule api_response {
    (json |
        exists(status_code) &
        digit_in(status_code, [200, 201, 202]) &
        chars_in(result_type, ["success", "partial"])
    )
}
```

**Function Description**:
- Parse JSON response
- Validate status_code field exists
- Validate status_code is in success range
- Validate result_type is success type

## Complex Combination Examples

### 1. Nginx Access Log Parsing

**Complete Parsing Rule**:
```wpl
rule nginx_log {
    (ip, _^2, time/clf<[,]>, http/request", http/status, digit:length, chars", http/agent", _")
}
```

**Test Data**:
```
192.168.1.2 - - [06/Aug/2019:12:12:19 +0800] "GET /nginx-logo.png HTTP/1.1" 200 368 "http://119.122.1.4/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36" "-"
```

**Expected Results**:
```
ip: 192.168.1.2
time: 2019-08-06 12:12:19
http/request: "GET /nginx-logo.png HTTP/1.1"
http/status: 200
length: 368
chars: "http://119.122.1.4/"
http/agent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36"
```

### 2. Huawei Firewall Log Parsing

**Parsing Rule**:
```wpl
|decode/base64|(digit:id<<,>>,time,sn,chars\:),opt(kv\;), (*kv\,)
```

**Data Flow**:
1. Base64 decode
2. Parse numeric ID
3. Parse time
4. Parse serial number
5. Parse characters
6. Parse optional KV pairs
7. Parse multiple KV pairs

### 3. JSON Geo-Location Enhancement

**Parsing Rule**:
```wpl
rule geo_enhance {
    (json(chars@src-ip +geo(city_name), @dst-ip +zone(zone_name), @dev-name +device(device_val)))
}
```

**Function Description**:
- Parse src-ip, dst-ip, dev-name fields from JSON
- Perform geo-location lookup on IP addresses
- Perform device type identification on device names

### 4. Complex Parsing with Pipe Processing

**Multi-Step Processing**:
```wpl
rule complex_parse {
    (|decode/base64| (json | exists_digit(age, 18)) | str_mode(decoded))
}
```

## WPL Syntax Key Points

### 1. Basic Syntax Structure
```wpl
package package_name {
    rule rule_name {
        (parsing expression)
    }
}
```

### 2. Pipe Operators
```wpl
|operator|  # Apply pipe operation
```
Common operations:
- `|decode/base64|` - Base64 decode
- `|str_mode(decoded)|` - String decode mode
- `|exists(field)|` - Check field existence
- `|exists_digit(field, value)|` - Check numeric field value
- `|unquote/unescape|` - Unquote/unescape

### 3. Field Naming
```wpl
type:field_name    # digit:length
type@JSON_path     # json(chars@path)
type+enhancement   # ip+geo(city_name)
```

### 4. Groups and Separators
```wpl
(...)         # Group
<separator>   # Specify separator
"separator"   # String separator
_             # Underscore
^N            # Repeat N times
```

### 5. Optional and Repeat
```wpl
opt(expression)   # Optional match
N*expression      # Repeat N times
*expression       # Repeat any number of times
```

### 6. Special Character Escaping
```wpl
\;            # Escape semicolon
\:            # Escape colon
\,            # Escape comma
\\            # Escape backslash
```

## Summary

WPL provides powerful data parsing capabilities, supporting:

1. **Multiple data formats**: JSON, KV, time, network protocols, etc.
2. **Flexible syntax**: Supports naming, pipes, groups, repetition, etc.
3. **Data enhancement**: Supports geo-location, device identification, and other enhancement features
4. **Combined parsing**: Multiple parsers can be combined
5. **Encoding processing**: Supports Base64, escape characters, and other encoding/decoding

Learning recommendations:
- Start with basic types, understand number, string, and separator parsing
- Master time format and network protocol parsing methods
- Learn JSON parsing and field operations
- Understand advanced usage of pipe operations and combined parsing
- Practice complex scenarios through actual log parsing cases

Through the above examples, you can quickly master the data parsing methods and techniques of WPL.
