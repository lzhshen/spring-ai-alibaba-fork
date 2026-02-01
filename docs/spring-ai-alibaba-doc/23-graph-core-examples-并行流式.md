```
import com.alibaba.cloud.ai.graph.CompileConfig;  
import com.alibaba.cloud.ai.graph.CompiledGraph;  
import com.alibaba.cloud.ai.graph.KeyStrategy;  
import com.alibaba.cloud.ai.graph.KeyStrategyFactory;  
import com.alibaba.cloud.ai.graph.RunnableConfig;  
import com.alibaba.cloud.ai.graph.StateGraph;  
import com.alibaba.cloud.ai.graph.action.AsyncNodeAction;  
import com.alibaba.cloud.ai.graph.exception.GraphStateException;  
import com.alibaba.cloud.ai.graph.state.strategy.AppendStrategy;  
import com.alibaba.cloud.ai.graph.streaming.StreamingOutput;  
  
import java.time.Duration;  
import java.util.HashMap;  
import java.util.Map;  
import java.util.concurrent.CompletableFuture;  
import java.util.concurrent.atomic.AtomicInteger;  
  
import reactor.core.publisher.Flux;  
  
import static com.alibaba.cloud.ai.graph.StateGraph.END;  
import static com.alibaba.cloud.ai.graph.StateGraph.START;  
  
/**  
* 并行流式输出示例  
* 演示如何在并行分支中使用 Flux 实现流式输出  
* 每个并行节点可以独立产生流式输出，并保持各自的节点 ID  
*/  
public class ParallelStreamingExample {  
  
  /**  
   * 示例 1: 并行节点流式输出 - 每个节点保持独立的节点 ID  
   *  
   * 演示如何创建多个并行节点，每个节点返回 Flux 流式输出  
   * 流式输出会保持各自的节点 ID，便于区分不同节点的输出  
   */  
  public static void parallelStreamingWithNodeIdPreservation() throws GraphStateException {  
      // 定义状态策略  
      KeyStrategyFactory keyStrategyFactory = () -> {  
          Map<String, KeyStrategy> keyStrategyMap = new HashMap<>();  
          keyStrategyMap.put("messages", new AppendStrategy());  
          keyStrategyMap.put("parallel_results", new AppendStrategy());  
          return keyStrategyMap;  
      };  
  
      // 并行节点 1 - 返回 Flux 流式输出  
      AsyncNodeAction node1 = state -> {  
 
<truncated 4971 bytes>
ap.put("stream_result", new AppendStrategy());  
      return keyStrategyMap;  
  };  
  
  // 单个流式节点  
  AsyncNodeAction streamingNode = state -> {  
      // 创建流式数据  
      Flux<String> dataStream = Flux.just("块1", "块2", "块3", "块4", "块5")  
              .delayElements(Duration.ofMillis(100));  
  
  
      return CompletableFuture.completedFuture(Map.of("stream_output", dataStream));  
  };  
  
  // 构建图  
  StateGraph stateGraph = new StateGraph(keyStrategyFactory)  
          .addNode("streaming_node", streamingNode)  
          .addEdge(START, "streaming_node")  
          .addEdge("streaming_node", END);  
  
  // 编译图  
  CompiledGraph graph = stateGraph.compile(  
          CompileConfig.builder()  
                  .build()  
  );  
  
  // 创建配置  
  RunnableConfig config = RunnableConfig.builder()  
          .threadId("single_streaming_thread")  
          .build();  
  
  System.out.println("开始单节点流式输出...  
");  
  
  AtomicInteger streamCount = new AtomicInteger(0);  
  String[] lastNodeId = new String[1];  
  
  // 执行流式图  
  graph.stream(Map.of("input", "test"), config)  
          .filter(output -> output instanceof StreamingOutput)  
          .map(output -> (StreamingOutput<?>) output)  
          .doOnNext(streamingOutput -> {  
              streamCount.incrementAndGet();  
              lastNodeId[0] = streamingOutput.node();  
              System.out.println("[流式输出] 节点: " + streamingOutput.node() +  
                      ", 内容: " + streamingOutput.chunk());  
          })  
          .doOnComplete(() -> {  
              System.out.println("  
=== 单节点流式输出完成 ===");  
              System.out.println("节点 ID: " + lastNodeId[0]);  
              System.out.println("流式块数: " + streamCount.get());  
          })  
          .doOnError(error -> {  
              System.err.println("流式输出错误: " + error.getMessage());  
          })  
          .blockLast();  
}
```
