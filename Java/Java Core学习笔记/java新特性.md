# Java 新特性

## 1. record 记录类（Java 16 正式特性）

### 1.1 对应的 Java 版本

| 版本 | 状态 |
| --- | --- |
| Java 14 | 首次预览，JEP 359 |
| Java 15 | 第二次预览，JEP 384 |
| Java 16 | 正式发布，JEP 395 |
| Java 17 | 可以直接使用，不需要开启预览功能 |

**record 和紧凑构造器都属于 Java 16 已正式提供的能力，不是 Java 17 才新增的语法。** Java 14、15 的预览版本需要对应版本的预览选项。

版本依据：[Oracle Java 15 Record Classes](https://docs.oracle.com/en/java/javase/15/language/records.html)、[Oracle Java 17 迁移指南：JDK 16 的变化](https://docs.oracle.com/en/java/javase/17/migrate/significant-changes-jdk-release.html)。

### 1.2 record 解决什么问题？

1. 有些类主要负责保存一组相关数据，例如生产者身份、查找条件、登记配置、处理结果。
2. 普通类需要手动编写字段、构造方法、访问方法，以及 `equals()`、`hashCode()`、`toString()`。
3. record 用简洁的声明表达这组数据，并默认生成上述成员。**record 仍然是一种类，可以通过 new 创建对象。**

```java
public record Producer(String system, String service) {
}
```

这里的 `system` 和 `service` 称为 record 的组件（record components）。

```java
Producer producer = new Producer("gfiot", "dgforkliftjsgfsvr");

System.out.println(producer.system());  // gfiot
System.out.println(producer.service()); // dgforkliftjsgfsvr
```

**访问方法与组件同名：是 system()、service()，不是 getSystem()、getService()。**

### 1.3 Java 默认生成哪些内容？

| 内容 | Producer 示例 |
| --- | --- |
| 每个组件对应的 private final 实例字段 | system、service |
| 与组件列表对应的规范构造器 | Producer(String system, String service) |
| public 访问方法 | system()、service() |
| equals() | 同一种 record 类型，按组件值判断是否相等 |
| hashCode() | 与相等比较配套，用于 Map、Set 等集合 |
| toString() | 输出类型名称、组件名称及对应值，方便查看对象 |

对于本例的 String 组件，按组件值比较就是比较两个字符串的内容。

下面用普通类帮助理解字段、构造器和访问方法。**示意省略了 equals()、hashCode()、toString()，不能直接作为完整等价替换。**

```java
public final class ProducerClass {
    private final String system;
    private final String service;

    public ProducerClass(String system, String service) {
        this.system = system;
        this.service = service;
    }

    public String system() {
        return system;
    }

    public String service() {
        return service;
    }
}
```

record 类隐式为 final，不能被继承；可以实现接口，也可以定义自己的方法。因此它适合表达数据，不意味着类体只能为空。

自动成员与限制可查：[Oracle Record Classes](https://docs.oracle.com/en/java/javase/16/language/records.html)。

### 1.4 public Producer { ... } 是什么写法？

阅读 AdapterRegistry 时遇到的代码：

```java
public record Producer(String system, String service) {
    public Producer {
        requireText(system, "system");
        requireText(service, "service");
    }
}
```

1. `public Producer { ... }` 叫作**紧凑规范构造器（compact canonical constructor）**，通常简称紧凑构造器。
2. 它的参数从 record 的组件列表推导，因此不用重复写 `(String system, String service)`。
3. 构造器体执行完毕后，Java 自动将参数赋给对应组件字段。
4. 它不是无参构造器，也不是普通方法；创建时仍需要提供两个参数。
5. `requireText()` 是项目自己的校验方法，不是 record 自带的功能。当前项目用它拒绝 null、空串及纯空白字符串，并抛出登记配置异常。

```java
Producer producer = new Producer("gfiot", "dgforkliftjsgfsvr");
```

执行顺序：

```text
构造参数 system、service 得到传入值
    ↓
执行紧凑构造器中的两次 requireText()
    ↓
校验通过，自动完成两个组件字段的赋值
    ↓
得到 Producer 对象
```

**校验失败并抛出异常时，本次构造失败，不会返回 Producer 对象。record 本身不会自动拒绝 null 或空白字符串。**

### 1.5 与显式规范构造器对照

下面是同一套项目校验逻辑的两种写法。两种方式二选一，不能同时为同一个 record 声明这两个规范构造器。

**写法一：显式写出完整参数列表和字段赋值。**

```java
public record Producer(String system, String service) {
    public Producer(String system, String service) {
        requireText(system, "system");
        requireText(service, "service");

        this.system = system;
        this.service = service;
    }
}
```

**写法二：使用紧凑构造器，把参数声明和最终赋值交给 Java。**

```java
public record Producer(String system, String service) {
    public Producer {
        requireText(system, "system");
        requireText(service, "service");
    }
}
```

这些片段中的 `requireText()` 沿用项目提供的方法。紧凑构造器里不能再手动写 `this.system = system` 这种组件字段赋值。

紧凑构造器还可以调整参数，自动赋值采用调整后的值：

```java
public record Label(String text) {
    public Label {
        if (text == null) {
            throw new IllegalArgumentException("text 不能为空");
        }

        // 修改的是构造参数；构造器结束时将整理后的值保存到字段。
        text = text.strip();
    }
}
```

这里用 Label 演示语言能力；项目 Producer 的 requireText() 只校验，不会自动去掉空格或改写来源身份。

构造器规则依据：[Java 17 语言规范 §8.10.4.2](https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.10.4.2)。

### 1.6 为什么 record 适合做 Map 的键或 Set 的元素？

```java
Producer first = new Producer("gfiot", "dgforkliftjsgfsvr");
Producer second = new Producer("gfiot", "dgforkliftjsgfsvr");

System.out.println(first == second);      // false：两个不同对象
System.out.println(first.equals(second)); // true：类型相同，组件值相等
```

登记时保存一个对象，消息到来时根据字段重新创建另一个对象，仍然可以匹配：

```java
Set<Producer> admittedProducers = new HashSet<>();
admittedProducers.add(first);

System.out.println(admittedProducers.contains(second)); // true
```

这里需要导入 `java.util.Set` 和 `java.util.HashSet`。

1. `==` 比较引用是否指向同一个对象。
2. record 默认生成的 `equals()`、`hashCode()` 支持按组件值匹配。
3. 普通类如果没有重写这两个方法，继承自 Object 的默认相等判断是引用相等，两个字段相同的新对象也可能匹配不上。
4. **record 的价值不只是少写 getter；正确、配套的相等比较对于复合查找键很重要。**

### 1.7 AdapterRegistry 中的多个 record 分别干什么？

它们使用同一种语言特性，但业务职责不同，不能全部理解成查找键。

**第一组：身份或查找条件。**

| record | 保存的组件 | 用途 |
| --- | --- | --- |
| Producer | system、service | 检查生产者是否在准入集合中 |
| ProtocolKey | sourceType、protocolCode | 查找来源和协议登记 |
| MessageKey | protocol、messageType | 判断消息是否支持或属于范围外 |
| AdapterKey | message、format | 查找实际适配器 |

```java
ProtocolKey protocol = new ProtocolKey("forklift", "forkliftJsgf");
MessageKey message = new MessageKey(protocol, "aabb_realtime");
AdapterKey adapterKey = new AdapterKey(message, "json");
```

这几个键逐层组合，最终表达：

```text
AdapterKey
  message
    protocol
      sourceType   = forklift
      protocolCode = forkliftJsgf
    messageType    = aabb_realtime
  format           = json
```

==> 展开就是 `source_type + protocol_code + message_type + format` 四个字段。嵌套 record 的默认相等比较会继续比较内部组件，因此重新创建的同值键也能查到同一项。

**第二组：登记配置。**

| record | 保存的内容 |
| --- | --- |
| MappingRegistration | 消息类型、格式、实际适配器对象 |
| SourceRegistration | 来源协议、允许的生产者集合、映射登记、范围外消息类型 |

```java
new MappingRegistration(
        "aabb_realtime",
        "json",
        new ForkliftRealtimeJsonAdapter());
```

这项配置把消息类型、格式与具体适配器对象绑定。Producer 的生产者二元组负责准入，不负责暗中选择另一套映射。

**第三组：运行时选择结果。**

| record | 保存的内容 |
| --- | --- |
| Resolution.Selected | 选中的适配器对象 |
| Resolution.Rejected | 拒绝原因 reason 和说明 detail |

这里的 Selected、Rejected 是结果数据；它们共同实现 Resolution，供 resolve() 用同一个返回类型表达两个分支。限制这两个实现的 sealed 接口是另一项语言特性，不是 record 本身的功能。

本节项目代码片段用于联系已有类名理解用法，依赖 AdapterRegistry 内的类型及项目依赖，不是可以整体复制到一个独立文件的完整程序。

### 1.8 record 不等于深度不可变

1. 组件对应的字段是 final，构造后不能重新赋值，默认也没有 setter。
2. 如果组件保存的是可变对象引用，该对象内部仍然可能变化。
3. 例如组件是 List 时，record 并不会自动把列表复制成不可修改集合。需要按需求进行防御性复制，并考虑元素本身是否可变。
4. 组件内容如果会影响 equals()、hashCode()，把对象放进 Map、Set 后再修改这些内容，可能破坏后续查找。

==> **record 固定了组件字段的赋值，不会自动保证整个对象图不可变。**

### 1.9 什么时候使用 record？

| 场景 | 判断 |
| --- | --- |
| 保存一组确定的数据，如身份、查找条件、配置声明、结果快照 | 适合考虑 record |
| 需要按值比较的复合键 | record 能减少样板和相等比较遗漏 |
| 需要频繁通过 setter 改变实例状态 | 普通类通常更直观 |
| 需要继承某个业务父类 | record 不能继承另一个业务类，其直接父类固定为 java.lang.Record |
| 负责组织流程、访问基础设施或维护复杂状态 | 通常仍使用普通类，按实际职责设计 |

在本项目中，Producer、AdapterKey、MappingRegistration、Selected 等主要承载数据，因此使用 record；AdapterRegistry 自己需要组织登记和执行选择流程，使用普通 class。
