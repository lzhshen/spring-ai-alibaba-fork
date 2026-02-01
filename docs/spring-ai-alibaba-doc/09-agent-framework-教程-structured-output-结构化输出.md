## Structured Output 结构化输出

结构化输出允许 Agent 以特定的、可预测的格式返回数据。相比于解析自然语言响应，您可以直接获得 JSON 对象或 Java POJO 形式的结构化数据，应用程序可以直接使用。

Spring AI Alibaba 的 `ReactAgent.Builder` 通过 `outputSchema` 和 `outputType` 方法处理结构化输出。当您设置所需的结构化输出模式时，Agent 会自动在用户消息中增加模式指令，模型会根据指定的格式生成数据。

```
ReactAgent agent = ReactAgent.builder()  
  .name("agent")  
  .model(chatModel)  
  .outputSchema(schemaString)  // Custom JSON schema as String  
  // OR  
  .outputType(MyClass.class)   // Java class - auto-converted to schema  
  .build();
```

## 输出格式选项[​](#输出格式选项 "输出格式选项的直接链接")

Spring AI Alibaba 支持两种方式控制结构化输出：

* **`outputSchema(String schema)`**: 提供 JSON schema 字符串。推荐使用 `BeanOutputConverter` 从 Java 类自动生成 schema，也可以手动提供自定义的 schema 字符串
* **`outputType(Class<?> type)`**: 提供 Java 类 - 使用 `BeanOutputConverter` 自动转换为 JSON schema（推荐方式，类型安全）
* **不指定**: 返回非结构化的自然语言响应

**推荐做法**：使用 `BeanOutputConverter` 生成 schema，既保证了类型安全，又实现了自动 schema 生成，代码更易维护。

结构化响应在 Agent 的 `AssistantMessage` 中作为 JSON 文本返回，可以解析为您需要的格式。

## 输出 Schema 策略[​](#输出-schema-策略 "输出 Schema 策略的直接链接")

您可以使用 `BeanOutputConverter` 从 Java 类自动生成 JSON schema，或者直接提供 JSON schema 字符串。推荐使用 `BeanOutputConverter` 以获得类型安全和自动 schema 生成。

### 方法签名[​](#方法签名 "方法签名的直接链接")

```
Builder output
<truncated 10482 bytes>
t(), DataOutput.class);  
  // 处理数据  
} catch (JsonProcessingException e) {  
  System.err.println("JSON解析失败: " + e.getMessage());  
  System.err.println("原始输出: " + result.getText());  
  // 回退处理  
}
```

### 验证模式[​](#验证模式 "验证模式的直接链接")

```
public class ValidatedOutput {  
  private String title;  
  private Integer rating;  
  
  public void validate() throws IllegalArgumentException {  
      if (title == null || title.isEmpty()) {  
          throw new IllegalArgumentException("标题不能为空");  
      }  
      if (rating != null && (rating < 1 || rating > 5)) {  
          throw new IllegalArgumentException("评分必须在1-5之间");  
      }  
  }  
  
  // Getter 和 Setter 方法...  
}  
  
AssistantMessage result = agent.call("生成评价");  
ValidatedOutput output = mapper.readValue(result.getText(), ValidatedOutput.class);  
output.validate();  // 如果无效则抛出异常
```

### 重试模式[​](#重试模式 "重试模式的直接链接")

```
int maxRetries = 3;  
DataOutput data = null;  
  
for (int i = 0; i < maxRetries; i++) {  
  try {  
      AssistantMessage result = agent.call("提取数据");  
      data = mapper.readValue(result.getText(), DataOutput.class);  
      break;  // 成功  
  } catch (Exception e) {  
      if (i == maxRetries - 1) {  
          throw new RuntimeException("多次尝试后仍然失败", e);  
      }  
      System.out.println("第" + (i + 1) + "次尝试失败，重试中...");  
  }  
}
```

Spring AI Alibaba 专注于简单性和灵活性，允许开发者在显式 schema 字符串（最大控制）和 Java 类（类型安全）之间进行选择。

## 相关文档[​](#相关文档 "相关文档的直接链接")

* [Agents](/docs/frameworks/agent-framework/tutorials/agents) - 了解 ReactAgent 功能
* [Models](/docs/frameworks/agent-framework/tutorials/models) - 支持的聊天模型
* [Messages](/docs/frameworks/agent-framework/tutorials/messages) - 消息类型和处理
