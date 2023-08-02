# Flink Dynamic CEP

## 动态CEP做了啥

在 Flink CEP 中经常遇到需要更换匹配模式的情况，一般需要关闭更换和重启程序进行处理，flink 的动态CEP提供了一种模式，可以对支持动态策略更改的方法。

## 新增接口说明

下面是，Flink复杂事件动态匹配内容可以参考[FLIP-200](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=195730308)内容的说明。

 FLIP（Flink Improvement Proposal）中，新增了几个公共接口来增强 Flink CEP（复杂事件处理）的功能。这些接口用于定义模式、匹配规则和处理匹配结果的方式，并实现动态模式处理器的更新。

- PatternProcessor 接口和相关类：PatternProcessor 接口用于表示一个特定的模式处理器，它包含了定义模式、匹配规则和处理匹配结果的方法。通过实现 PatternProcessor 接口，用户可以自定义不同的模式处理器，然后在 CEP 中使用这些处理器来匹配和处理事件流。

```java
/**
 * Base class for a pattern processor definition.
 *
 * <p>A pattern processor defines a {@link Pattern}, how to match the pattern, and how to process
 * the found matches.
 *
 * @param <IN> Base type of the elements appearing in the pattern.
 * @param <OUT> Type of produced elements based on found matches.
 */
@PublicEvolving
public interface PatternProcessor<IN, OUT> extends Serializable, Versioned {
    /**
     * Returns the ID of the pattern processor.
     *
     * @return The ID of the pattern processor.
     */
    String getId();
 
    /**
     * Returns the scheduled time at which the pattern processor should come into effective.
     *
     * <p>If the scheduled time is earlier than current event/processing time, the pattern processor
     * will immediately become effective.
     *
     * <p>If the pattern processor should always become effective immediately, the scheduled time
     * can be set to {@code Long.MIN_VALUE}: {@value Long#MIN_VALUE}.
     *
     * @return The scheduled time.
     */
    default Long getTimestamp() {
        return Long.MIN_VALUE;
    }
 
    /**
     * Returns the {@link Pattern} to be matched.
     *
     * @return The pattern of the pattern processor.
     */
    Pattern<IN, ?> getPattern();
 
    /**
     * Returns the {@link KeySelector} to extract the key of the elements appearing in the pattern.
     *
     * <p>Only data with the same key can be chained to match a pattern. If extracted key is null,
     * the behavior is to reuse current key if is {@link KeyedStream}, or allocate the same key for
     * all input data if is not {@link KeyedStream}.
     *
     * @return The key selector of the pattern processor.
     */
    @Nullable
    KeySelector<IN, ?> getKeySelector();
 
    /**
     * Get the {@link PatternProcessFunction} to process the found matches for the pattern.
     *
     * @return The pattern process function of the pattern processor.
     */
    PatternProcessFunction<IN, OUT> getPatternProcessFunction();
}
```

- PatternProcessorManager 接口：PatternProcessorManager 接口负责处理模式处理器的更新，确保所有子任务都得到一致的模式处理器，并提供当前活动的模式处理器信息。该接口由 OperatorCoordinator 实现，用于控制 CEP 操作符的动态模式处理。

```java
/**
 * This manager handles updated pattern processors and manages current pattern processors.
 *
 * @param <IN> Base type of the elements appearing in the pattern.
 * @param <OUT> Type of produced elements based on found matches.
 */
@PublicEvolving
public interface PatternProcessorManager<IN, OUT> {
    /**
     * Deal with the notification that pattern processors are updated.
     *
     * @param patternProcessors A list of all latest pattern processors.
     */
    void onPatternProcessorsUpdated(List<PatternProcessor<IN, OUT>> patternProcessors);
 
    /**
     * Returns the current pattern processors managed.
     *
     * @return The current pattern processors.
     */
    List<PatternProcessor<IN, OUT>> getCurrentPatternProcessors();
}
```

- PatternProcessorDiscoverer 接口：PatternProcessorDiscoverer 接口用于发现模式处理器的变化并通知 PatternProcessorManager。用户可以根据实际需求，在实现中定义如何获取模式处理器的更新，例如从配置文件、数据库或其他数据源中获取模式处理器信息。通过 PatternProcessorDiscoverer，Flink CEP 可以支持动态地获取模式处理器的更新，实现在运行时动态地添加、删除或更改模式处理器，而无需重新启动作业。

```java
/**
 * Interface that discovers pattern processor updates, notifies {@link PatternProcessorManager} of
 * pattern processor updates and provides the initial pattern processors.
 *
 * @param <IN> Base type of the elements appearing in the pattern.
 * @param <OUT> Type of produced elements based on found matches.
 */
@PublicEvolving
public interface PatternProcessorDiscoverer<IN, OUT> {
    /**
     * Discover the pattern processor updates.
     *
     * <p>In dynamic pattern processor update case, this function should be a continuous process to
     * check pattern processor updates and notify the {@link PatternProcessorManager}.
     */
    void discoverPatternProcessorUpdates(PatternProcessorManager<IN, OUT> manager) throws Exception;
 
    /**
     * Returns the pattern processors an operator should be set up with.
     *
     * @return The initial pattern processors.
     */
    List<PatternProcessor<IN, OUT>> getInitialPatternProcessors();
}
```

- CEP.patternProcess() 方法：CEP.patternProcess() 方法是一个便捷的 API，用于将动态的模式处理器应用于输入的 DataStream，并创建一个新的 DataStream，其中包含匹配到的复杂事件模式处理器的结果。通过该方法，用户可以在运行时动态更新模式处理器，并在同一个 DataStream 上同时处理多个不同的复杂事件模式。

```java
public class CEP {
    /**
     * Creates a {@link DataStream} containing the processed results of matched CEP pattern
     * processors.
     *
     * @param input DataStream containing the input events
     * @param factory Pattern processor discoverer factory to create the {@link
     *     PatternProcessorDiscoverer}
     * @param <IN> Type of the elements appearing in the pattern
     * @param <OUT> Type of produced elements based on found matches
     * @return Resulting data stream
     */
    public static <IN, OUT> DataStream<OUT> patternProcessors(
            DataStream<IN> input, PatternProcessorDiscovererFactory<IN, OUT> factory) {...}
}
```

通过引入这些公共接口和方法，Flink CEP 可以更加灵活地处理复杂的事件处理场景，并实现动态地调整处理逻辑，适应不断变化的业务需求。

![img](https://intranetproxy.alipay.com/skylark/lark/__mermaid_v3/57e05464869ac1a651ad6a881cba4639.svg)

## 新增接口例子

解读一下动态变化应用的一个方法 `JDBCPeriodicPatternProcessorDiscoverer`，

```java
// table 名称
private final String tableName;

private final List<PatternProcessor<T>> initialPatternProcessors;
private final ClassLoader userCodeClassLoader;

// sql.Statement, Statement是Java 执行数据库操作的一个重要方法，用于在已经建立数据库连接的基础上，向数据库发送要执行的SQL语句。
private Statement statement;
private ResultSet resultSet;


private Map<String, Tuple4<String, Integer, String, String>> latestPatternProcessors;
```

- arePatternProcessorsUpdated() 方法

```java
    @Override
    public boolean arePatternProcessorsUpdated() {
        // latestPatternProcessors 为空或者 initialPatternProcessors不为空 返回true
        if (latestPatternProcessors == null
                && !CollectionUtil.isNullOrEmpty(initialPatternProcessors)) {
            return true;
        }

        // statement为空返回false
        if (statement == null) {
            return false;
        }

        // 重新查看数据库 tableName 中的数据
        try {
            resultSet = statement.executeQuery("SELECT * FROM " + tableName);
            Map<String, Tuple4<String, Integer, String, String>> currentPatternProcessors =
                    new HashMap<>();
            while (resultSet.next()) {
                String id = resultSet.getString("id");
    // 如果currentPatternProcessors的id.f1 大雨等于 结果中的version值，则不更新
                if (currentPatternProcessors.containsKey(id)
                        && currentPatternProcessors.get(id).f1 >= resultSet.getInt("version")) {
                    continue;
                }
    // 将id，对应的信息放入 currentPatternProcessors 中
                currentPatternProcessors.put(
                        id,
                        new Tuple4<>(
                                requireNonNull(resultSet.getString("id")),
                                resultSet.getInt("version"),
                                requireNonNull(resultSet.getString("pattern")),
                                resultSet.getString("function")));
            }
    // isPatternProcessorUpdated判断currentPatternProcessors和latestPatternProcessors是否相等，不等返回true
            if (latestPatternProcessors == null
                    || isPatternProcessorUpdated(currentPatternProcessors)) {
                latestPatternProcessors = currentPatternProcessors;
                return true;
            } else {
                return false;
            }
        } catch (SQLException e) {
            LOG.warn("Pattern processor discoverer checks rule changes - " + e.getMessage());
        }
        return false;
    }
```

- getLatestPatternProcessors() 方法

 ```java
    @SuppressWarnings("unchecked")
    @Override
    public List<PatternProcessor<T>> getLatestPatternProcessors() throws Exception {
        ObjectMapper objectMapper =
                new ObjectMapper()
                        .registerModule(
                                new SimpleModule()
                                        .addDeserializer(
                                                ConditionSpec.class,
                                                ConditionSpecStdDeserializer.INSTANCE)
                                        .addDeserializer(Time.class, TimeStdDeserializer.INSTANCE)
                                        .addDeserializer(
                                                NodeSpec.class, NodeSpecStdDeserializer.INSTANCE));
    // 将 latestPatternProcessors 中值以流的形式创建 map 映射成 DefaultPatternProcessor<T>
        return latestPatternProcessors.values().stream()
                .map(
                        patternProcessor -> {
                            try {
                                String patternStr = patternProcessor.f2;
                                // 将 patternStr 转换成 GraphSpec
                                GraphSpec graphSpec =
                                        objectMapper.readValue(patternStr, GraphSpec.class);
                                objectMapper.enable(SerializationFeature.INDENT_OUTPUT);
                                System.out.println(
                                        objectMapper
                                                .writerWithDefaultPrettyPrinter()
                                                .writeValueAsString(graphSpec));
                                PatternProcessFunction<T, ?> patternProcessFunction = null;
                                if (!StringUtils.isNullOrWhitespaceOnly(patternProcessor.f3)) {
                                    patternProcessFunction =
                                            (PatternProcessFunction<T, ?>)
                                                    this.userCodeClassLoader
                                                            .loadClass(patternProcessor.f3)
                                                            .newInstance();
                                }
                                LOG.warn(
                                        objectMapper
                                                .writerWithDefaultPrettyPrinter()
                                                .writeValueAsString(patternProcessor.f2));

                                // map 出 DefaultPatternProcessor<> 出来， ClassLoader 加入了
                                return new DefaultPatternProcessor<>(
                                        patternProcessor.f0,
                                        patternProcessor.f1,
                                        patternStr,
                                        patternProcessFunction,
                                        this.userCodeClassLoader);
                            } catch (Exception e) {

                                LOG.error(
                                        "Get the latest pattern processors of the discoverer failure. - "
                                                + e.getMessage());
                                e.printStackTrace();
                            }
                            return null;
                        })
                .collect(Collectors.toList());
    }
 ```

## 动态CEP中规则的JSON格式定义

### JSON格式定义

对于一个事件序列（Event Sequence）中的模式（Pattern），我们可以将其看作一个图（Graph），图中节点（Node）为针对某些事件（Event）的模式，节点之间的边（Edge）为事件选择策略（Event Selection Strategy），即如何从一类模式的匹配转移到另一类模式的匹配。每个图也可以看作一个更大的图的子节点，从而允许模式的嵌套。基于以上考虑，Flink定义了一套基于JSON的规范来描述CEP中的规则，进而方便规则的存储与修改，该规范中各个字段的含义如下。

节点（Node）定义

一个节点（Node）即一个完整的模式（Pattern），它包含如下属性。

| 字段名     | 描述                                              | 类型         | 是否必填 | 备注                                                         |
| :--------- | :------------------------------------------------ | :----------- | :------- | :----------------------------------------------------------- |
| name       | Pattern名称。                                     | string       | 是       | 一个唯一的字符串。**说明** 不同节点的名称不能重复。          |
| type       | 该Node类型。                                      | enum(string) | 是       | 对于包含子Pattern的节点，该字段值为COMPOSITE。对于无子Pattern的节点，该字段值为ATOMIC。 |
| quantifier | 量词，用于描述如何匹配该Pattern，例如只匹配一次。 | dict         | 是       | 请参见本文量词（Quantifier）定义。                           |
| condition  | 条件。                                            | dict         | 否       | 请参见本文条件（Condition）定义。                            |

### 量词（Quantifier）定义

量词的作用是描述对于满足该Pattern的事件要如何匹配。例如模式` "A*" ` 对应的量词properties为LOOPING，该Pattern内部的事件选择策略为SKIP_TILL_ANY。

| 字段名            | 描述                                                      | 类型                | 是否必填 | 备注                                                         |
| :---------------- | :-------------------------------------------------------- | :------------------ | :------- | :----------------------------------------------------------- |
| consumingStrategy | 事件选择策略。                                            | enum(string)        | 是       | 仅支持以下取值：STRICTSKIP_TILL_NEXTSKIP_TILL_ANY取值及含义请参见本文连续性定义。 |
| times             | 用于描述该Pattern需要匹配多少次。                         | dict                | 否       | 取值示例如下。`"times": {          "from": 3,          "to": 3,          "windowTime": {          "unit": "MINUTES",          "size": 12          }        },`其中from和to的数据类型均为integer，windowTime的单位可以为DAYS、HOURS、MINUTES、SECONDS和MILLISECONDS。**说明** windowTime可以设为null，即`"windowTime": null`。 |
| properties        | 描述该量词所具有的属性。                                  | array of enumString | 是       | 取值及含义请参见本文量词属性含义。                           |
| untilCondition    | 停止条件。**说明** 仅可在LOOPING量词修饰的Pattern后使用。 | dict                | 否       | 取值及含义请参见本文条件（Condition）定义。                  |

#### 使用Aviator

- 基于Aviator表达式的Condition

Aviator是一个表达式求值引擎，可以动态地将表达式编译成字节码（详情请参见[aviatorscript](https://github.com/killme2008/aviatorscript)）。因此我们可以在作业中使用基于Aviator表达式的Condition，使得条件的阈值也可以动态修改，而无需修改Java代码重新编译运行。

| 字段名     | 描述           | 类型   | 是否必填 | 备注                                                         |
| :--------- | :------------- | :----- | :------- | :----------------------------------------------------------- |
| type       | 类名。         | string | 是       | 固定值为AVIATOR。                                            |
| expression | 表达式字符串。 | string | 是       | 形如**price > 10**这样的表达式字符串（**price**变量名来自于Java代码中定义的字段）。您可以将该字符串在数据库中的值进行修改。例如修改为**price > 20**，Flink CEP作业会动态加载**price > 20**构造新的AviatorCondition来处理之后的事件。 |
参考：https://www.alibabacloud.com/help/zh/flink/developer-reference/definitions-of-rules-in-the-json-format-in-dynamic-flink-cep#concept-2258817