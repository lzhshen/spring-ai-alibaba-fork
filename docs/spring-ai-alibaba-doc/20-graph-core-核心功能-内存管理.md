AI 应用程序需要支持在同一轮会话的多条消息间共享上下文，或者在不同的会话场景先共享上下文。在 Spring AI Alibaba Graph 中，您可以添加两种类型的内存：

```
// 创建内存检查点器  
MemorySaver checkpointer = new MemorySaver();  
  
SaverConfig saverConfig = SaverConfig.builder()  
  .register(checkpointer)  
  .build();  
  
// 定义状态策略  
KeyStrategyFactory keyStrategyFactory = () -> {  
  Map<String, KeyStrategy> keyStrategyMap = new HashMap<>();  
  keyStrategyMap.put("messages", new AppendStrategy());  
  return keyStrategyMap;  
};  
  
// 构建图  
StateGraph stateGraph = new StateGraph(keyStrategyFactory)  
  .addNode("agent", agentNode)  
  .addEdge(START, "agent")  
  .addEdge("agent", END);  
  
// 使用检查点器编译图  
CompiledGraph graph = stateGraph.compile(  
  CompileConfig.builder()  
      .saverConfig(saverConfig)  
      .build()  
);  
  
// 使用会话 ID 调用图  
RunnableConfig config = RunnableConfig.builder()  
  .threadId("user-session-1")  
  .build();  
  
Map<String, Object> input = Map.of(  
  "messages", List.of(  
      Map.of("role", "user", "content", "你好！我是 Bob")  
  )  
);  
  
graph.invoke(input, config);
```

```
import com.alibaba.cloud.ai.graph.CompileConfig;  
import com.alibaba.cloud.ai.graph.CompiledGraph;  
import com.alibaba.cloud.ai.graph.KeyStrategy;  
import com.alibaba.cloud.ai.graph.KeyStrategyFactory;  
import com.alibaba.cloud.ai.graph.RunnableConfig;  
import com.alibaba.cloud.ai.graph.StateGraph;  
import com.alibaba.cloud.ai.graph.checkpoint.config.SaverConfig;  
import com.alibaba.cloud.ai.graph.checkpoint.savers.MemorySaver;  
import com.alibaba.cloud.ai.graph.state.strategy.AppendStrategy;  
  
import org.springframework.ai.chat.client.ChatClient;  
  
import java.util.HashMap;  
import java.util.List;  
import java.util.Map;  
  
import static com.alibaba.cloud.ai.graph.StateGraph.END;  
import
<truncated 12462 bytes>
的提示  
  String userPrompt = messages.get(messages.size() - 1).get("content");  
  String enhancedPrompt = "用户偏好: " + preferences + "  
用户问题: " + userPrompt;  
  
  // 调用 AI  
  ChatClient chatClient = chatClientBuilder.build();  
  String response = chatClient.prompt()  
      .user(enhancedPrompt)  
      .call()  
      .content();  
  
  return Map.of("messages", List.of(  
      Map.of("role", "assistant", "content", response)  
  ));  
});  
  
// 构建图  
StateGraph stateGraph = new StateGraph(keyStrategyFactory)  
  .addNode("load_preferences", loadUserPreferences)  
  .addNode("chat", chatNode)  
  .addEdge(START, "load_preferences")  
  .addEdge("load_preferences", "chat")  
  .addEdge("chat", END);  
  
// 配置检查点（短期内存）  
SaverConfig saverConfig = SaverConfig.builder()  
      .register(new MemorySaver())  
  .build();  
  
// 编译图  
CompiledGraph graph = stateGraph.compile(  
  CompileConfig.builder()  
      .saverConfig(saverConfig)  
      .build()  
);  
  
// 创建长期记忆存储并预填充用户偏好  
MemoryStore memoryStore = new MemoryStore();  
Map<String, Object> preferencesData = new HashMap<>();  
preferencesData.put("theme", "dark");  
preferencesData.put("language", "zh");  
preferencesData.put("timezone", "Asia/Shanghai");  
StoreItem preferencesItem = StoreItem.of(List.of("user_preferences"), "user_002", preferencesData);  
memoryStore.putItem(preferencesItem);  
  
// 运行图  
RunnableConfig config = RunnableConfig.builder()  
      .threadId("combined_thread")  
      .store(memoryStore)  
  .build();  
  
// 第一轮对话（加载偏好并开始对话）  
graph.invoke(Map.of(  
      "userId", "user_002",  
      "messages", List.of(Map.of("role", "user", "content", "你好"))  
), config);  
  
// 第二轮对话（使用短期和长期记忆）  
graph.invoke(Map.of(  
      "userId", "user_002",  
      "messages", List.of(Map.of("role", "user", "content", "根据我的偏好给我一些建议"))  
), config);
```
