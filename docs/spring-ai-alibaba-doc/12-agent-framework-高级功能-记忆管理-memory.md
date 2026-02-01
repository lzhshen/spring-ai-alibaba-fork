## 记忆管理（Memory）

## 概述[​](#概述 "概述的直接链接")

Spring AI Alibaba 的 Agent 使用持久化机制来实现长期记忆功能。这是一个高级主题，需要对 Spring AI Alibaba 的 Agent 框架有一定了解。

```
┌─────────────────────────────────────────────────────────┐  
│                     ReactAgent                          │  
├─────────────────────────────────────────────────────────┤  
│                                                         │  
│  ┌──────────────────┐        ┌────────────────────┐   │  
│  │  短期记忆         │        │   长期记忆          │   │  
│  │  Short-term      │        │   Long-term        │   │  
│  │  (MemorySaver)   │        │   (MemoryStore)    │   │  
│  └──────────────────┘        └────────────────────┘   │  
│         │                             │                │  
│         │ threadId                    │ namespace/key  │  
│         ↓                             ↓                │  
│  ┌──────────────────┐        ┌────────────────────┐   │  
│  │  对话历史         │        │  用户画像          │   │  
│  │  Conversation    │        │  User Profiles     │   │  
│  │  History         │        │  Preferences       │   │  
│  │  Message State   │        │  Persistent Data   │   │  
│  └─────────────
<truncated 23765 bytes>
/ 提取用户输入  
      List<Message> messages = (List<Message>) state.value("messages").orElse(new ArrayList<>());  
      if (messages.isEmpty()) {  
          return CompletableFuture.completedFuture(Map.of());  
      }  
  
      // 加载现有偏好  
      Optional<StoreItem> prefsOpt = memoryStore.getItem(List.of("user_data"), userId + "_preferences");  
      List<String> prefs = new ArrayList<>();  
      if (prefsOpt.isPresent()) {  
          Map<String, Object> prefsData = prefsOpt.get().getValue();  
          prefs = (List<String>) prefsData.getOrDefault("items", new ArrayList<>());  
      }  
  
      // 简单的偏好提取（实际应用中使用NLP）  
      for (Message msg : messages) {  
          String content = msg.getText().toLowerCase();  
          if (content.contains("喜欢") || content.contains("偏好")) {  
              prefs.add(msg.getText());  
  
              Map<String, Object> prefsData = new HashMap<>();  
              prefsData.put("items", prefs);  
              StoreItem item = StoreItem.of(List.of("user_data"), userId + "_preferences", prefsData);  
              memoryStore.putItem(item);  
  
              System.out.println("学习到用户偏好 " + userId + ": " + msg.getText());  
          }  
      }  
  
      return CompletableFuture.completedFuture(Map.of());  
  }  
};  
  
ReactAgent agent = ReactAgent.builder()  
  .name("learning_agent")  
  .model(chatModel)  
  .hooks(preferenceLearningHook)  
  .saver(new MemorySaver())  
  .build();  
  
RunnableConfig config = RunnableConfig.builder()  
  .threadId("learning_thread")  
  .addMetadata("user_id", "user_004")  
  .store(memoryStore)  
  .build();  
  
// 用户表达偏好  
agent.invoke("我喜欢喝绿茶。", config);  
agent.invoke("我偏好早上运动。", config);  
  
// 验证偏好已被存储  
Optional<StoreItem> savedPrefs = memoryStore.getItem(List.of("user_data"), "user_004_preferences");  
if (savedPrefs.isPresent()) {  
  // 偏好应该被保存到长期记忆中  
}
```
