# 引用事件字段详解

通过名称引用字段或字段集合通常很有用，您可以使用Logstash字段引用语法达成此目的。

访问字段的语法需指定字段的完整路径，使每个片段都包含在方括号中。

字段引用可以在管道配置中的条件语句中依次标明，作为管道插件的字符串参数，或者管道插件使用的sprintf语句中：

```yaml
filter {
  #  +----literal----+     +----literal----+
  #  |               |     |               |
  if [@metadata][date] and [@metadata][time] {
    mutate {
      add_field {
        "[@metadata][timestamp]" => "%{[@metadata][date]} %{[@metadata][time]}"
      # |                      |    |  |               |    |               | |
      # +----string-argument---+    |  +--field-ref----+    +--field-ref----+ |
      #                             +-------- sprintf format string ----------+
      }
    }
  }
}
```

### 正式语法
下面是Field Reference的正式语法，附有注释和示例。

#### 字段引用语法

字段引用语法是一个或多个路径片段的序列，可以直接在Logstash管道 [条件表达式](06-Configuring-Logstash/Accessing-Event-Data-and-Fields-in-the-Configuration.md#条件表达式) 中直接使用，而无需任何其他引用（例如 `[request]`，`[response][status]`）。

```js
fieldReferenceLiteral
  : ( pathFragment )+
  ;
```

#### 字段引用（事件API）

用于操作事件字段或使用sprintf语法的事件API，比它们作为字段引用所接受的管道语法更灵活。 顶级字段可以通过其字段名称直接引用，而无需使用方括号，并且对复合字段引用有一些支持，简化了以编程方式生成的字段引用的方法。

因此，通过事件API进行字段引用的场景如下：

- 单个字段引用语法; 
- 单个字段名称（引用顶级字段）; 
- 单个复合字段引用。

```js
eventApiFieldReference
  : fieldReferenceLiteral
  | fieldName
  | compositeFieldReference
  ;
```

#### 路径片段

路径片段是指用方括号包括的字段名（例如：`[request]`）

```js
pathFragment
  : '[' fieldName ']'
  ;
```

#### 字段名

字段名指的是一个字符串（字符串不可为 `[` 或 `]`）

```yaml
fieldName
  : ( ~( '[' | ']' ) )+
  ;
```

#### 复合字段引用

在某些情况下，可能需要以编程方式引用一个或多个字段，从而组成字段引用，例如在操作插件中的字段时，或使用Ruby Filter插件和Event API时。

```yaml
fieldReference = "[path][to][deep nested field]"
compositeFieldReference = "[@metadata][#{fieldReference}][size]"
# => "[@metadata][[path][to][deep nested field]][size]"
```

#### 复合字段引用的典型样例

| 可接受的复合字段引用         | 字段引用的典型样例         |
| ---------------------------- | -------------------------- |
| `+[[deep][nesting]][field]+` | `+[deep][nesting][field]+` |
| `+[foo][[bar]][bingo]+`      | `+[foo][bar][bingo]+`      |
| `+[[ok]]+`                   | `+[ok]+`                   |

复合字段引用是一个或多个路径片段，或嵌入字段引用的序列。

```js
compositeFieldReference
  : ( pathFragment | embeddedFieldReference )+
  ;
```

事件API支持复合字段引用，但管道配置中不支持复合字段引用。

#### 嵌入字段引用

```js
embeddedFieldReference
  : '[' fieldReference ']'
  ;
```

嵌入式字段引用是一个字段引用，它本身包含在方括号（ `[` 和 `]` ）中，可以是复合字段引用的一个组件。