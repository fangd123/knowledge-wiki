---
title: "给AI Agent装上记忆：LangChain的记忆设计"
date: 2026-04-07
author: fangd123
license: CC BY-NC-SA 4.0
category: AI技术与工程
tags:
  - AI技术与工程
ai_assisted: true
status: published
---

在AI Agent越来越智能的今天,如何让它们"记住"与用户的互动,已经成为构建实用AI系统的关键。LangChain团队在长期与客户合作构建Agent应用的过程中,深入思考了这个问题,并提出了一套完整的记忆系统设计方案。这套方案不仅参考了人类记忆的心理学研究,还结合了实际工程实践,形成了一个既有理论基础又具备可操作性的框架。

## 为什么Agent需要记忆?

想象一下,如果你有一个人类助理,但Ta每次见到你都忘记之前的所有对话,你需要反复告诉Ta同样的信息。这该有多糟糕?AI Agent面临的正是这个问题。

LangChain团队的CEO Harrison Chase在2024年的博客中指出,记忆是Agent应用开发中仅次于Agent本身的第二大热词。但是,很多人误以为大语言模型(LLM)本身就具备记忆能力,因为它们表现得如此像人类,然而实际上LLM本身并不会自动记住任何东西,你必须主动为系统添加记忆功能。

记忆让Agent从一个"健忘症患者"变成了真正有用的助手。它使得Agent能够:
- 记住用户的偏好和个人信息
- 从过往的成功经验中学习
- 根据反馈不断优化自己的行为
- 提供个性化的服务体验

## 记忆的两个层次:短期与长期

LangChain团队将Agent的记忆分为两大类,这个分类方式借鉴了人类记忆系统的研究成果:

### 短期记忆:对话中的工作记忆

短期记忆,也称为线程范围(thread-scoped)记忆,它追踪当前会话中的持续对话,通过维护一个会话中的消息历史来工作。就像你在和朋友聊天时,能记住刚才说了什么一样,短期记忆让Agent能够理解对话的上下文。

**技术实现**

LangGraph通过线程范围的检查点(checkpoints)来管理短期记忆,将其作为Agent状态的一部分进行持久化。这个状态通常包含:
- 完整的对话历史(消息列表)
- 上传的文件
- 检索到的文档
- 生成的中间结果

代码示例:

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.prebuilt import create_react_agent
from langchain_openai import ChatOpenAI

# 初始化模型和记忆
llm = ChatOpenAI(model="gpt-4")
memory = MemorySaver()  # 短期记忆管理器

# 创建带记忆的Agent
agent = create_react_agent(llm, tools=[], checkpointer=memory)

# 使用thread_id来标识不同的对话
config = {"configurable": {"thread_id": "conversation-1"}}

# 第一轮对话
response1 = agent.invoke(
    {"messages": [{"role": "user", "content": "我叫张三"}]}, 
    config
)

# 第二轮对话 - Agent能记住之前说过的话
response2 = agent.invoke(
    {"messages": [{"role": "user", "content": "我刚才说我叫什么?"}]}, 
    config
)
# Agent会回答"你叫张三"
```

**短期记忆的挑战**

对话历史是短期记忆最常见的形式,而长对话给当今的LLM带来了挑战。完整的历史可能无法装入LLM的上下文窗口,导致不可恢复的错误。即使LLM支持完整的上下文长度,大多数LLM在处理长上下文时表现仍然很差,它们会被过时或无关的内容"分心",同时遭受更慢的响应时间和更高的成本。

因此,实际应用中需要采用一些策略来管理对话历史:

**1. 滑动窗口(Sliding Window)**
只保留最近的N条消息:

```python
from langchain_core.messages import trim_messages

# 只保留最近10条消息
trimmed_messages = trim_messages(
    messages,
    max_tokens=10,
    strategy="last"
)
```

**2. 对话总结(Summarization)**
通过使用LLM对之前的对话进行总结,记忆模块会随着新数据或经验的添加而增量式地更新和精炼总结:

```python
def summarize_conversation(messages):
    # 使用LLM总结对话
    summary_prompt = """
    请总结以下对话的关键信息:
    {messages}
    """
    summary = llm.invoke(summary_prompt.format(messages=messages))
    return summary
```

**3. 向量化检索**
向量化是现代AI记忆管理的核心,它将文本信息转换为能够封装单词和概念底层含义的数值表示。通过将记忆分割成离散的块并进行向量化,开发者可以利用向量搜索来精确高效地检索最相关的记忆。

### 长期记忆:跨会话的知识积累

长期记忆允许系统在不同对话或会话之间保留信息。与短期记忆不同,长期记忆是跨线程的,保存在自定义的"命名空间"中。

LangChain在2025年1月宣布,大多数AI应用今天就像金鱼一样,它们在对话之间会忘记一切。这不仅效率低下,而且从根本上限制了AI能做什么。为了解决这个问题,LangChain推出了跨线程的长期记忆支持。

LangChain团队借鉴了认知科学研究,特别是CoALA论文,该论文很好地将人类记忆类型映射到了Agent记忆,将长期记忆细分为三种类型:

## 三种长期记忆类型

### 1. 语义记忆(Semantic Memory):记住事实

语义记忆在人类中指的是对学校学到的知识、概念的含义及其相互关系的记忆。在Agent中,CoALA论文将语义记忆描述为关于世界的事实库。今天,这最常被Agent用于个性化应用。

**两种存储模式**

LangChain团队在实践中发现,语义记忆有两种主要的管理方式:

**模式一:用户画像(Profile)**

将所有信息集中在一个持续更新的JSON文档中:

```python
from langgraph.store.memory import InMemoryStore
import uuid

store = InMemoryStore()

# 存储用户画像
user_id = "user_123"
namespace = ("users", user_id)

profile = {
    "name": "张三",
    "preferences": {
        "language": "简体中文",
        "communication_style": "简洁直接",
        "technical_level": "中级开发者"
    },
    "dietary": {
        "restrictions": ["素食"],
        "favorite_foods": ["意大利面", "沙拉"]
    }
}

store.put(namespace, "profile", profile)

# 检索时很简单,直接GET
user_profile = store.get(namespace, "profile")
```

这种方式的优点是:
- 信息集中,易于查看
- 检索简单,只需要一个GET操作
- 适合信息结构明确的场景

缺点是:
- 随着信息增多,更新容易出错
- 可能在更新时丢失某些信息
- 需要预先定义好schema

**模式二:记忆集合(Collection)**

将每条事实作为独立文档存储:

```python
# 存储独立的记忆片段
memories_namespace = ("user_memories", user_id)

# 添加多个记忆
store.put(
    memories_namespace,
    str(uuid.uuid4()),
    {"type": "preference", "content": "用户喜欢简洁的回答"}
)

store.put(
    memories_namespace,
    str(uuid.uuid4()),
    {"type": "skill", "content": "用户熟悉Python编程"}
)

store.put(
    memories_namespace,
    str(uuid.uuid4()),
    {"type": "dietary", "content": "用户是素食主义者"}
)

# 检索时使用语义搜索
relevant_memories = store.search(
    memories_namespace,
    query="用户的编程能力",
    limit=5
)
```

这种方式的优点是:
- 每条记忆独立,不会相互影响
- 更容易管理和更新
- 可以无限扩展
- 对LLM来说,生成新对象比将新信息与现有配置文件进行协调更容易,因此文档集合往往导致更高的召回率

缺点是:
- 需要实现搜索和过滤逻辑
- 可能缺少整体上下文
- Agent需要决定何时插入、更新或删除记忆

**实际应用:个性化聊天机器人**

LangChain提供了一个长期记忆Agent的完整实现示例,该Agent可以存储、检索和使用记忆来增强与用户的互动。以下是简化的实现:

```python
from langchain_core.tools import tool
from langchain_core.runnables import RunnableConfig
from langgraph.config import get_store

@tool
def save_user_preference(
    preference_type: str,
    preference_value: str,
    config: RunnableConfig
) -> str:
    """保存用户偏好到长期记忆"""
    store = get_store()
    user_id = config["configurable"].get("user_id")
    
    memory = {
        "type": preference_type,
        "value": preference_value,
        "timestamp": datetime.now().isoformat()
    }
    
    store.put(("preferences", user_id), str(uuid.uuid4()), memory)
    return f"已保存{preference_type}偏好"

@tool
def recall_user_preferences(
    query: str,
    config: RunnableConfig
) -> str:
    """检索相关的用户偏好"""
    store = get_store()
    user_id = config["configurable"].get("user_id")
    
    # 语义搜索相关记忆
    memories = store.search(
        ("preferences", user_id),
        query=query,
        limit=5
    )
    
    return "\n".join([m.value["value"] for m in memories])

# Agent可以在对话中自动调用这些工具
agent = create_react_agent(
    llm,
    tools=[save_user_preference, recall_user_preferences],
    store=store
)
```

### 2. 情景记忆(Episodic Memory):记住经验

情景记忆指的是回忆特定的过去事件。在人类中,这是指一个人回忆过去经历的特定事件。在Agent中,CoALA论文将情景记忆定义为存储Agent过去行动的序列。

**实现方式:Few-Shot示例学习**

实际上,情景记忆通常通过few-shot示例提示来实现,Agent从过去的序列中学习正确执行任务。有时"展示"比"告诉"更容易,LLM很擅长从示例中学习。

举个实际例子,假设我们要训练一个邮件分类Agent:

```python
# 存储成功的分类示例
episodic_namespace = ("email_triage", "examples")

# 示例1:紧急邮件
store.put(
    episodic_namespace,
    "example_urgent_1",
    {
        "input": {
            "subject": "生产环境宕机!",
            "sender": "运维团队",
            "content": "线上服务器出现故障,需要立即处理"
        },
        "output": {
            "category": "urgent",
            "action": "立即通知",
            "reasoning": "包含'宕机'、'立即'等紧急关键词"
        }
    }
)

# 示例2:可以忽略的邮件
store.put(
    episodic_namespace,
    "example_ignore_1",
    {
        "input": {
            "subject": "每周新闻简报",
            "sender": "marketing@company.com",
            "content": "本周行业动态..."
        },
        "output": {
            "category": "ignore",
            "action": "归档",
            "reasoning": "营销邮件,非工作必需"
        }
    }
)

# 使用时检索最相似的示例
def get_few_shot_examples(email_content: str, k: int = 3):
    """根据当前邮件内容检索最相关的历史示例"""
    examples = store.search(
        episodic_namespace,
        query=email_content,
        limit=k
    )
    
    # 格式化为few-shot提示
    few_shot_prompt = "\n\n".join([
        f"示例 {i+1}:\n输入: {ex.value['input']}\n输出: {ex.value['output']}"
        for i, ex in enumerate(examples)
    ])
    
    return few_shot_prompt
```

**与LangSmith的集成**

值得注意的是,记忆商店只是存储few-shot示例数据的一种方式。如果你想要更多的开发者参与,或将few-shot示例与评估工具更紧密地结合,也可以使用LangSmith Dataset来存储数据。

LangSmith会为你索引数据集,并基于关键词相似性使用类似BM25的算法来检索与用户输入最相关的few-shot示例。

### 3. 程序记忆(Procedural Memory):记住规则

程序记忆指的是如何执行任务的长期记忆,类似于大脑的核心指令集。在人类中,这是指记住如何骑自行车。在Agent中,CoALA论文将程序记忆描述为LLM权重和Agent代码的组合,它们从根本上决定了Agent的工作方式。

**实现方式:反思与提示词优化**

实际上,Agent自动修改模型权重或重写代码的情况相当罕见。然而,Agent修改自己的提示词则更为常见。一个有效的方法是通过"反思"或元提示。

基本思路是:
1. 让Agent看到当前的系统提示词
2. 回顾最近的对话或用户反馈
3. 思考哪里可以改进
4. 生成新的提示词

代码实现:

```python
from langchain_core.prompts import ChatPromptTemplate

# 存储当前的系统提示词
procedural_namespace = ("agent_instructions",)

initial_prompt = """
你是一个邮件助手。你需要帮用户处理邮件。
对于每封邮件,你需要决定是忽略、回复还是提醒用户。
"""

store.put(procedural_namespace, "current_prompt", {
    "version": 1,
    "prompt": initial_prompt,
    "last_updated": datetime.now().isoformat()
})

def update_instructions_from_feedback(
    conversation_history: list,
    user_feedback: str,
    config: RunnableConfig
):
    """根据反馈更新系统指令"""
    store = get_store()
    
    # 获取当前提示词
    current_instruction = store.get(procedural_namespace, "current_prompt")
    
    # 构造反思提示
    reflection_prompt = f"""
    当前系统指令:
    {current_instruction.value['prompt']}
    
    最近的对话历史:
    {conversation_history}
    
    用户反馈:
    {user_feedback}
    
    基于上述信息,请生成改进后的系统指令。
    改进后的指令应该解决用户反馈中提到的问题。
    """
    
    # 使用LLM生成新指令
    new_prompt = llm.invoke(reflection_prompt)
    
    # 保存新版本
    store.put(procedural_namespace, "current_prompt", {
        "version": current_instruction.value['version'] + 1,
        "prompt": new_prompt.content,
        "last_updated": datetime.now().isoformat(),
        "reason": user_feedback
    })
    
    return "指令已更新"
```

**实际案例:Twitter摘要生成器**

LangChain团队构建了一个Tweet生成器,使用外部反馈和提示词重写来生成高质量的论文摘要推文。在这个案例中,具体的摘要提示词很难预先指定,但用户批评生成的推文并提供如何改进摘要过程的反馈则相对容易。

```python
# 用户反馈循环
while True:
    # 生成推文
    tweet = generate_tweet(paper_content)
    
    # 获取用户反馈
    feedback = input("对这条推文满意吗?(满意/不满意,原因): ")
    
    if feedback.startswith("不满意"):
        # 更新提示词
        update_instructions_from_feedback(
            conversation_history=[tweet],
            user_feedback=feedback
        )
        
        # 使用新提示词重新生成
        continue
    else:
        break
```

## 记忆的写入策略

除了思考在Agent中更新什么类型的记忆,开发者还会思考如何更新Agent记忆。LangChain团队总结了两种主要策略:

### 策略一:热路径写入(In the Hot Path)

主动记忆形成发生在对话期间,当关键上下文出现时能够立即更新。这种方法易于实现,让Agent自己选择如何存储和更新其记忆。

**实现示例**

```python
@tool
def save_important_info(
    info_type: str,
    content: str,
    config: RunnableConfig
) -> str:
    """Agent主动保存重要信息到记忆"""
    store = get_store()
    user_id = config["configurable"]["user_id"]
    
    store.put(
        ("memories", user_id),
        str(uuid.uuid4()),
        {
            "type": info_type,
            "content": content,
            "saved_at": datetime.now().isoformat()
        }
    )
    return "信息已保存到长期记忆"

# Agent可以在对话中主动调用
# 例如,当用户说"我喜欢披萨"时,
# Agent会调用 save_important_info("food_preference", "喜欢披萨", config)
```

**优点:**
- 记忆立即可用
- 透明度高,用户知道什么被记住了
- 实现简单

**缺点:**
- 会给用户交互增加可感知的延迟,并且为Agent满足用户需求的能力增加了一个障碍
- Agent需要多任务处理
- 可能影响主要任务的质量

### 策略二:后台写入(In the Background)

"潜意识"记忆形成是指在对话发生后(或在一段时间不活跃后)提示LLM反思对话,寻找模式并提取洞察,而不会减慢即时交互速度或增加Agent工具选择决策的复杂性。

**实现示例**

```python
import asyncio
from datetime import datetime, timedelta

class BackgroundMemoryService:
    def __init__(self, store, llm):
        self.store = store
        self.llm = llm
        self.pending_conversations = {}
    
    async def schedule_memory_extraction(
        self,
        user_id: str,
        conversation: list,
        delay_minutes: int = 5
    ):
        """延迟执行记忆提取"""
        # 如果有新消息,重新调度
        if user_id in self.pending_conversations:
            self.pending_conversations[user_id].cancel()
        
        # 创建新的延迟任务
        task = asyncio.create_task(
            self._extract_after_delay(user_id, conversation, delay_minutes)
        )
        self.pending_conversations[user_id] = task
    
    async def _extract_after_delay(
        self,
        user_id: str,
        conversation: list,
        delay_minutes: int
    ):
        """等待一段时间后提取记忆"""
        await asyncio.sleep(delay_minutes * 60)
        
        # 提取记忆
        extraction_prompt = f"""
        分析以下对话,提取重要的记忆:
        {conversation}
        
        请提取:
        1. 用户的偏好
        2. 重要的事实
        3. 用户的目标或需求
        
        以JSON格式返回。
        """
        
        memories = await self.llm.ainvoke(extraction_prompt)
        
        # 保存到记忆商店
        for memory in memories:
            self.store.put(
                ("extracted_memories", user_id),
                str(uuid.uuid4()),
                memory
            )
        
        # 清理
        del self.pending_conversations[user_id]

# 使用示例
memory_service = BackgroundMemoryService(store, llm)

# 对话结束时触发
await memory_service.schedule_memory_extraction(
    user_id="user_123",
    conversation=conversation_history,
    delay_minutes=5  # 5分钟不活跃后提取
)
```

**优点:**
- 不影响主流程响应速度
- Agent可以专注完成任务
- 可以批量处理,避免重复工作
- 记忆管理逻辑独立,易于维护

**缺点:**
- 记忆不是立即可用
- 需要决定触发时机
- 实现相对复杂

## 记忆存储:技术实现细节

在核心层面,跨线程记忆"只是"一个持久化的文档存储,让你可以放置、获取和搜索你保存的记忆。

### 基本存储API

```python
from langgraph.store.memory import InMemoryStore

# 初始化存储(生产环境应使用数据库后端)
store = InMemoryStore()

# 存储记忆 - 使用命名空间组织
namespace = ("user", "user_123", "preferences")
key = "food_preferences"
value = {
    "likes": ["意大利面", "寿司"],
    "dislikes": ["香菜"],
    "dietary_restrictions": ["素食"]
}

store.put(namespace, key, value)

# 检索记忆
memory = store.get(namespace, key)
print(memory.value)  # 获取存储的内容

# 搜索记忆 - 支持语义搜索和过滤
results = store.search(
    namespace,
    query="用户喜欢什么食物?",  # 语义搜索
    filter={"dietary_restrictions": "素食"},  # 精确过滤
    limit=10
)
```

### 生产级存储后端

在生产环境中,应该使用支持持久化的存储后端:

**1. MongoDB存储**

MongoDB Store for LangGraph提供了灵活且可扩展的长期记忆解决方案,原生支持JSON结构、向量搜索能力、异步操作以及TTL索引:

```python
from langgraph.store.mongodb import MongoDBStore

# 配置MongoDB
store = MongoDBStore(
    connection_string="mongodb://localhost:27017",
    database_name="agent_memory",
    collection_prefix="memories_"
)

# API使用方式相同
store.put(namespace, key, value)
```

MongoDB的优势包括:原生JSON结构、向量搜索用于语义检索、可扩展架构包括分布式架构和自动分片,以及细粒度的基于角色的访问控制。

**2. Redis存储**

Redis提供了高性能的内存存储,特别适合需要快速访问的场景,包括线程级持久化和跨线程记忆能力:

```python
from langgraph_checkpoint_redis import RedisSaver, RedisStore

# 短期记忆:使用Redis Checkpointer
checkpointer = RedisSaver.from_conn_info(
    host="localhost",
    port=6379,
    db=0
)

# 长期记忆:使用Redis Store
store = RedisStore.from_conn_info(
    host="localhost",
    port=6379,
    db=1
)

agent = create_react_agent(
    llm,
    tools,
    checkpointer=checkpointer,  # 短期记忆
    store=store  # 长期记忆
)
```

**3. 向量数据库存储**

对于需要大量语义搜索的场景,可以使用专门的向量数据库:

```python
from langchain_openai import OpenAIEmbeddings
from langchain_core.vectorstores import InMemoryVectorStore

# 初始化嵌入模型
embeddings = OpenAIEmbeddings()

# 创建向量存储
vector_store = InMemoryVectorStore(embeddings)

# 存储记忆(会自动向量化)
from langchain_core.documents import Document

doc = Document(
    page_content="用户喜欢吃意大利面",
    metadata={"user_id": "user_123", "type": "食物偏好"}
)
vector_store.add_documents([doc])

# 语义搜索
results = vector_store.similarity_search(
    "用户的饮食偏好是什么?",
    k=5
)
```

### 混合存储策略

最复杂的方法涉及主动式记忆管理,即AI本身决定记住什么、如何组织记忆以及何时检索它们。这种方法使Agent能够维护一个随经验发展的自适应、个性化的记忆系统。

```python
class HybridMemoryStore:
    """混合记忆存储,结合不同后端优势"""
    
    def __init__(self, vector_store, document_store):
        self.vector_store = vector_store  # 用于语义搜索
        self.document_store = document_store  # 用于结构化数据
    
    def put(self, namespace, key, value):
        """同时存储到两个后端"""
        # 向量存储:用于语义搜索
        if "content" in value:
            doc = Document(
                page_content=value["content"],
                metadata={"key": key, "namespace": namespace}
            )
            self.vector_store.add_documents([doc])
        
        # 文档存储:保存完整结构
        self.document_store.put(namespace, key, value)
    
    def search(self, namespace, query=None, filter=None, limit=10):
        """智能搜索:结合语义和精确匹配"""
        results = []
        
        # 如果有查询文本,使用向量搜索
        if query:
            semantic_results = self.vector_store.similarity_search(
                query, 
                k=limit,
                filter={"namespace": namespace}
            )
            results.extend(semantic_results)
        
        # 如果有过滤条件,使用文档存储
        if filter:
            filtered_results = self.document_store.search(
                namespace,
                filter=filter,
                limit=limit
            )
            results.extend(filtered_results)
        
        return results[:limit]
```

## 完整示例:带记忆的邮件助手

让我们通过一个完整的例子,展示如何构建一个具备三种记忆类型的邮件助手:

```python
from langgraph.prebuilt import create_react_agent
from langgraph.store.memory import InMemoryStore
from langgraph.checkpoint.memory import MemorySaver
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langchain_core.runnables import RunnableConfig
from langgraph.config import get_store
import uuid
from datetime import datetime

# 初始化
llm = ChatOpenAI(model="gpt-4")
store = InMemoryStore()
checkpointer = MemorySaver()

# ============ 语义记忆工具 ============
@tool
def save_user_preference(
    preference: str,
    config: RunnableConfig
) -> str:
    """保存用户偏好到长期记忆"""
    store = get_store()
    user_id = config["configurable"]["user_id"]
    
    memory = {
        "content": preference,
        "type": "preference",
        "timestamp": datetime.now().isoformat()
    }
    
    store.put(("preferences", user_id), str(uuid.uuid4()), memory)
    return "偏好已保存"

@tool
def recall_preferences(
    query: str,
    config: RunnableConfig
) -> str:
    """检索用户偏好"""
    store = get_store()
    user_id = config["configurable"]["user_id"]
    
    results = store.search(
        ("preferences", user_id),
        query=query,
        limit=5
    )
    
    if not results:
        return "未找到相关偏好"
    
    return "\n".join([r.value["content"] for r in results])

# ============ 情景记忆:Few-shot示例 ============
def load_triage_examples(user_id: str) -> str:
    """加载邮件分类的历史示例"""
    examples = store.search(
        ("triage_examples", user_id),
        limit=3
    )
    
    if not examples:
        return "暂无历史示例"
    
    prompt = "以下是一些邮件分类的示例:\n\n"
    for i, ex in enumerate(examples):
        prompt += f"示例 {i+1}:\n"
        prompt += f"邮件主题: {ex.value['subject']}\n"
        prompt += f"分类: {ex.value['category']}\n"
        prompt += f"原因: {ex.value['reason']}\n\n"
    
    return prompt

@tool
def save_triage_example(
    subject: str,
    category: str,
    reason: str,
    config: RunnableConfig
) -> str:
    """保存成功的分类示例作为情景记忆"""
    store = get_store()
    user_id = config["configurable"]["user_id"]
    
    example = {
        "subject": subject,
        "category": category,
        "reason": reason,
        "timestamp": datetime.now().isoformat()
    }
    
    store.put(
        ("triage_examples", user_id),
        str(uuid.uuid4()),
        example
    )
    return "示例已保存"

# ============ 程序记忆:系统提示词 ============
def get_system_prompt(user_id: str) -> str:
    """获取当前的系统提示词"""
    instruction = store.get(("instructions", user_id), "current")
    
    if instruction:
        return instruction.value["prompt"]
    
    # 默认提示词
    default_prompt = """
    你是一个智能邮件助手。你的任务是帮助用户处理邮件。
    
    对于每封邮件,你需要:
    1. 根据用户偏好和历史示例进行分类
    2. 决定是忽略、回复还是通知用户
    3. 如果需要回复,生成恰当的回复内容
    
    记得在处理过程中保存重要的用户偏好和成功的示例。
    """
    
    # 保存默认提示词
    store.put(
        ("instructions", user_id),
        "current",
        {
            "prompt": default_prompt,
            "version": 1,
            "updated_at": datetime.now().isoformat()
        }
    )
    
    return default_prompt

@tool
def update_system_prompt(
    feedback: str,
    config: RunnableConfig
) -> str:
    """基于反馈更新系统提示词"""
    store = get_store()
    user_id = config["configurable"]["user_id"]
    
    current = store.get(("instructions", user_id), "current")
    
    # 使用LLM生成改进后的提示词
    reflection_prompt = f"""
    当前指令:
    {current.value['prompt']}
    
    用户反馈:
    {feedback}
    
    请生成改进后的系统指令,解决用户反馈中提到的问题。
    """
    
    improved_prompt = llm.invoke(reflection_prompt).content
    
    # 保存新版本
    store.put(
        ("instructions", user_id),
        "current",
        {
            "prompt": improved_prompt,
            "version": current.value['version'] + 1,
            "updated_at": datetime.now().isoformat(),
            "reason": feedback
        }
    )
    
    return "系统指令已更新"

# ============ 构建Agent ============
tools = [
    save_user_preference,
    recall_preferences,
    save_triage_example,
    update_system_prompt
]

agent = create_react_agent(
    llm,
    tools,
    checkpointer=checkpointer,  # 短期记忆
    store=store  # 长期记忆
)

# ============ 使用Agent ============
def process_email(email, user_id, thread_id):
    """处理邮件"""
    config = {
        "configurable": {
            "user_id": user_id,
            "thread_id": thread_id
        }
    }
    
    # 加载系统提示词(程序记忆)
    system_prompt = get_system_prompt(user_id)
    
    # 加载历史示例(情景记忆)
    examples = load_triage_examples(user_id)
    
    # 加载用户偏好(语义记忆)
    preferences = recall_preferences("邮件处理偏好", config)
    
    # 构造完整消息
    message = f"""
    {system_prompt}
    
    用户偏好:
    {preferences}
    
    {examples}
    
    现在请处理这封邮件:
    主题: {email['subject']}
    发件人: {email['from']}
    内容: {email['content']}
    """
    
    # 调用Agent
    response = agent.invoke(
        {"messages": [{"role": "user", "content": message}]},
        config
    )
    
    return response

# 示例使用
email = {
    "subject": "紧急:服务器宕机",
    "from": "ops@company.com",
    "content": "生产环境服务器出现故障,需要立即处理"
}

result = process_email(
    email,
    user_id="user_123",
    thread_id="email_session_1"
)

print(result)
```

## 最佳实践与优化技巧

### 1. 记忆管理策略

**定期清理过期记忆**

```python
from datetime import datetime, timedelta

def cleanup_old_memories(user_id: str, days: int = 30):
    """清理超过指定天数的旧记忆"""
    cutoff_date = datetime.now() - timedelta(days=days)
    
    all_memories = store.search(("memories", user_id))
    
    for memory in all_memories:
        if datetime.fromisoformat(memory.value["timestamp"]) < cutoff_date:
            store.delete(("memories", user_id), memory.key)
```

**记忆重要性评分**

```python
def score_memory_importance(memory: dict) -> float:
    """评估记忆的重要性"""
    factors = {
        "recency": 0.3,  # 最近性
        "frequency": 0.3,  # 使用频率
        "relevance": 0.4  # 相关性
    }
    
    score = 0
    
    # 计算最近性得分
    days_old = (datetime.now() - datetime.fromisoformat(
        memory["timestamp"]
    )).days
    recency_score = max(0, 1 - days_old / 365)
    score += recency_score * factors["recency"]
    
    # 计算使用频率
    frequency_score = min(1, memory.get("access_count", 0) / 10)
    score += frequency_score * factors["frequency"]
    
    # 可以根据实际需求添加更多评分维度
    
    return score
```

### 2. 性能优化

**批量操作**

```python
def batch_save_memories(memories: list, namespace: tuple):
    """批量保存记忆,提高效率"""
    for memory in memories:
        store.put(namespace, str(uuid.uuid4()), memory)
```

**缓存常用记忆**

```python
from functools import lru_cache

@lru_cache(maxsize=100)
def get_user_profile(user_id: str):
    """缓存用户画像,减少数据库访问"""
    return store.get(("profiles", user_id), "main")
```

### 3. 隐私与安全

**命名空间隔离**

```python
# 不同用户的记忆完全隔离
user1_namespace = ("user_memories", "user_123")
user2_namespace = ("user_memories", "user_456")

# 团队共享记忆
team_namespace = ("team_memories", "team_alpha")

# 全局记忆(谨慎使用)
global_namespace = ("global_memories",)
```

**敏感信息加密**

```python
from cryptography.fernet import Fernet

def encrypt_sensitive_memory(content: str, key: bytes) -> str:
    """加密敏感记忆内容"""
    f = Fernet(key)
    return f.encrypt(content.encode()).decode()

def decrypt_sensitive_memory(encrypted_content: str, key: bytes) -> str:
    """解密记忆内容"""
    f = Fernet(key)
    return f.decrypt(encrypted_content.encode()).decode()
```

## LangMem:记忆管理工具包

2025年5月,LangChain发布了LangMem SDK,这是一个帮助Agent通过长期记忆学习和改进的库。它提供了从对话中提取信息、通过提示词更新优化Agent行为,以及维护关于行为、事实和事件的长期记忆的工具。

LangMem的核心API可以与任何存储系统一起使用,提供了记忆管理器来提取新记忆、更新或删除过时记忆,以及基于新对话信息整合和概括现有记忆;提示词优化器基于对话信息更新提示词规则和核心行为。

```python
from langmem import MemoryManager

# 初始化记忆管理器
memory_manager = MemoryManager(store=store)

# 自动从对话中提取记忆
conversation = [
    {"role": "user", "content": "我喜欢在早上喝咖啡"},
    {"role": "assistant", "content": "好的,我记住了"}
]

extracted_memories = memory_manager.extract_memories(
    conversation,
    user_id="user_123"
)

# 记忆会被自动分类和存储
print(extracted_memories)
# [
#   {"type": "preference", "content": "早上喝咖啡"},
#   {"category": "routine", "time": "morning"}
# ]
```

## 总结

Agent的记忆系统是构建智能、个性化AI应用的基础。LangChain团队通过深入研究人类记忆机制,结合大量工程实践,为我们提供了一套完整的解决方案:

**核心概念**:
- **短期记忆**通过检查点机制管理对话状态,确保上下文连贯性
- **长期记忆**跨越会话边界,让Agent能够积累知识和经验

**三种记忆类型**:
- **语义记忆**存储事实和知识,用于个性化服务
- **情景记忆**保存成功经验,通过few-shot学习提升能力
- **程序记忆**维护行为规则,通过反思不断优化

**实现策略**:
- **热路径写入**提供即时反馈,但会增加延迟
- **后台写入**保持流畅体验,需要精心设计触发机制

**技术栈**:
- LangGraph提供统一的记忆管理框架
- 支持多种存储后端(MongoDB、Redis、向量数据库)
- LangMem简化记忆提取和管理流程

正如LangChain团队所说,记忆的最佳实践对每个应用来说仍然包含非常特定于应用的逻辑。没有一种万能的解决方案,关键是要根据你的具体需求,合理选择和组合这些技术。

随着Agent应用变得越来越复杂,一个设计良好的记忆系统将成为不可或缺的基础设施。它让Agent从简单的"工具"进化为真正理解用户、能够学习成长的"助手"。

如果你正在构建自己的Agent应用,不妨从LangChain团队提供的这套框架开始,逐步为你的Agent赋予"记忆"的能力。记住,好的记忆不仅仅是存储数据,更重要的是知道存什么、什么时候存、以及如何在恰当的时机回忆起来。

