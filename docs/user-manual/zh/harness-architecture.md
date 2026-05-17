# Harness 架构完全指南 🎯

> **让小学生也能快速上手的 Agent 架构说明书**

## 📚 目录

1. [什么是 Harness？](#什么是-harness)
2. [核心概念](#核心概念)
3. [架构详解](#架构详解)
4. [实战案例](#实战案例)
5. [常见问题](#常见问题)

---

## 什么是 Harness？

### 🎭 简单比喻

想象你是一个厨师（Model），而 **Harness** 就是你的厨房！

- **Loop（循环）** = 你的工作流程：看菜谱 → 拿食材 → 做菜 → 尝味道 → 继续改进
- **Tools（工具）** = 你的厨具：刀、锅、铲子、烤箱
- **Memory（记忆）** = 你眼前的菜谱和你能看到的所有信息

**核心公式：**
```
Harness = Loop + Tools + Memory
Agent   = Harness(Model)
```

用大白话说：
- **Harness** = 一个完整的工作环境（厨房）
- **Model** = 你的大脑（会思考和做决定）
- **Agent** = 在厨房里工作的厨师（大脑 + 厨房）

---

## 核心概念

### 1️⃣ Model（模型）- 大脑

```
Model = Parametric(权重, 只读)
```

**简单理解：**
- 这是 AI 的"大脑"，存储了所有学到的知识
- **只读**：不能修改大脑的知识，只能用它来思考
- **Reasoning（推理）**：大脑看到信息后的思考过程

**例子：**
```
你学会了数学 → 这是你的"权重"
看到 1+1=? → 你的大脑开始"推理"
得出答案 2 → 这是推理的结果
```

### 2️⃣ Context（上下文）- 眼前的信息

```
Context = 当前上下文窗口（你能看到的所有东西）
Memory  = Context  ← Harness 可以直接访问
```

**简单理解：**
- Context 就像你眼前的书桌，上面放着你当前能看到的所有东西
- 受限制：书桌大小有限，放不下太多东西
- Memory 就是 Context，它们是同一个东西

**例子：**
```
你正在做作业
书桌上有：
  ✓ 数学课本（当前任务）
  ✓ 笔记本（历史记录）
  ✓ 作业题目（Spec）
  ✓ 之前的答案（历史观察）
```

### 3️⃣ Resources（资源）- 外部仓库

```
Resources = Store + ExternalAPIs + ...
  Store = 外部持久存储（数据库、文件系统等）
```

**简单理解：**
- Resources 是书桌外的东西：书柜、抽屉、图书馆
- 你**不能直接看到**这些东西，需要通过 Tools 去拿
- 和 Memory 的区别：Memory 在眼前，Resources 需要去找

**例子：**
```
Memory（眼前）     vs     Resources（需要去找）
  课本在书桌上              字典在书柜里
  作业在面前                参考书在图书馆
  笔记摊开着                往年试卷在抽屉里
```

### 4️⃣ Tools（工具）- 做事的方法

```
Tools = { f : Input → Output × Effect? }    ← 原子 Tool（基础工具）
      ∪ { Agent }                            ← 递归 Tool（高级工具）
```

**简单理解：**
- **原子 Tool**：基础工具，如计算器、字典、铅笔
  - `Input`：你给它的东西（如数字"5+3"）
  - `Output`：它返回的结果（如"8"）
  - `Effect`：可能产生的影响（如写字会用掉铅笔芯）

- **递归 Tool**：可以是另一个 Agent（会调用其他工具的高级工具）

**例子：**
```
原子 Tool：
  查字典(word="apple") → "苹果" (无副作用)
  写文字(text="你好") → "已写入" (副作用：纸上多了字)

递归 Tool（Agent as Tool）：
  翻译助手(text="Hello")
    → 内部调用：查字典 + 理解语法 + 组装句子
    → 返回："你好"
```

---

## 架构详解

### 🔄 Loop（循环）- 工作流程

这是 Harness 最核心的部分！

```
Loop = Spec ──(建立授权)──▶
       Reason                    ← 至少一次（必须思考）
     → [ Act ──(验证权限)──▶
         Observe
       → Reason ]*               ← 零次或多次（可能重复）
     → Return
```

#### 第一步：Spec（任务说明书）

**简单理解：**
- 这是你收到的任务或问题
- 建立授权：确认你有权限做这件事

**例子：**
```
老师：请做第 5 页的数学题
  ↓
Spec = "完成第 5 页数学题"
授权检查：这是老师布置的作业 ✓
```

#### 第二步：Reason（思考）

```
Reason: Model 读取 Context（含 Spec + 历史）→ 决策
```

**简单理解：**
- 看眼前的所有信息（书桌上的东西）
- 用大脑思考下一步该做什么
- 必须至少思考一次！

**例子：**
```
Context 包含：
  - 任务：做第 5 页数学题
  - 历史：我刚才已经做完第 1-3 题
  - 工具：我有计算器

Reason 结果：
  "我应该做第 4 题，它是一个乘法，我需要用计算器"
```

#### 第三步：Act（行动）

```
Act: 调用 Tools → 作用于 Environment / Resources
权限检查：确认 Tool 允许你使用
```

**简单理解：**
- 使用工具来执行任务
- 在用工具前，要检查权限（你能用这个工具吗？）

**例子：**
```
决定：使用计算器算 25 × 4
权限检查：计算器允许使用 ✓
Act：输入 "25 × 4" 到计算器
```

#### 第四步：Observe（观察结果）

```
Observe: Tool 返回值 + Environment 反馈 → 追加到 Context
```

**简单理解：**
- 看工具给你的结果
- 把结果记在笔记本上（追加到 Context）

**例子：**
```
计算器显示：100
写在笔记本：第 4 题答案是 100
Context 更新：书桌上多了这条记录
```

#### 第五步：重复 or 结束

```
→ Reason 再次思考是否继续
→ 或 Return 返回最终结果
```

**简单理解：**
- 再次思考：还有其他题要做吗？
- 如果有：重复 Act → Observe → Reason
- 如果没有或达到限制：Return（结束）

**返回条件：**
```
✓ 完成：所有题都做完了
✓ max_steps：做了太多步，强制停止
✓ 错误：出错了，无法继续
✓ 人工中断：老师让你停下
```

---

### 🎯 完整 Loop 示例

**任务：做一道数学题 "计算 (5 + 3) × 2 = ?"**

```
┌─────────────────────────────────────────┐
│ Spec: "计算 (5 + 3) × 2"                 │
│ 授权：用户允许使用计算器 ✓                │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Reason #1:                               │
│ Context = [任务, 无历史]                  │
│ 思考：需要先算 5+3，再乘以 2              │
│ 决定：使用计算器算 5+3                    │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Act #1:                                  │
│ Tool: 计算器                              │
│ Input: "5 + 3"                           │
│ 权限检查：✓                               │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Observe #1:                              │
│ Output: 8                                │
│ Context += "5+3=8"                       │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Reason #2:                               │
│ Context = [任务, "5+3=8"]                │
│ 思考：现在需要算 8 × 2                    │
│ 决定：使用计算器算 8×2                    │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Act #2:                                  │
│ Tool: 计算器                              │
│ Input: "8 × 2"                           │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Observe #2:                              │
│ Output: 16                               │
│ Context += "8×2=16"                      │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Reason #3:                               │
│ Context = [任务, "5+3=8", "8×2=16"]      │
│ 思考：任务完成！答案是 16                 │
│ 决定：返回结果                            │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Return: 16                               │
└─────────────────────────────────────────┘
```

---

### 🏗️ Harness = Loop + Tools + Memory

```
Harness = Loop + Tools + Memory  ← 三者耦合，缺一不可
```

**简单理解：**
- 这三个组件紧密配合，不能分开
- Loop 需要 Tools 来执行任务
- Loop 需要 Memory 来记住信息
- Tools 的结果写入 Memory
- Memory 的内容影响 Loop 的决策

**用厨房比喻：**
```
Loop（流程）    = 看菜谱 → 切菜 → 炒菜 → 尝味 → 调整
Tools（工具）   = 刀、锅、铲子、调料
Memory（记忆）  = 眼前的菜谱 + 已完成的步骤

三者关系：
  流程告诉你用什么工具
  工具的结果记在记忆里
  记忆帮助流程做决定
```

---

### 🤖 Agent = Harness(Model)

```
Agent = Harness(Model)
  Model 注入 Loop.Reason 作为执行引擎
  Model 拥有：Parametric（知识权重）
  Harness 拥有：Context + Loop + Tools
  两者职责不重叠
```

**简单理解：**
- Agent = 在厨房工作的厨师
- Model = 厨师的大脑（知识和思考能力）
- Harness = 整个厨房（工作环境）
- Model 的"思考能力"被放进 Harness 的"Reason 步骤"

**职责分工：**
```
Model（大脑）负责：
  ✓ 理解任务
  ✓ 分析情况
  ✓ 做出决策
  ✓ 判断结果

Harness（环境）负责：
  ✓ 提供工具
  ✓ 管理记忆
  ✓ 控制流程
  ✓ 执行操作
```

---

## 实战案例

### 案例 1：查天气 Agent 🌤️

**需求：**
用户问："明天北京天气怎么样？"

#### 步骤 1：设计 Harness

```python
# ========== 定义 Tools ==========
def get_weather(city: str, date: str) -> dict:
    """查询天气的工具"""
    # 调用天气 API
    return {
        "city": city,
        "date": date,
        "weather": "晴",
        "temperature": "25°C"
    }

tools = {
    "get_weather": get_weather
}

# ========== 定义 Memory ==========
memory = {
    "context": [],  # 存储对话历史
    "max_tokens": 4096  # 上下文窗口大小
}

# ========== 定义 Loop ==========
def loop(spec: str, model, tools, memory):
    # 步骤 1: Spec
    memory["context"].append({"role": "user", "content": spec})

    max_steps = 10
    for step in range(max_steps):
        # 步骤 2: Reason
        decision = model.think(memory["context"])

        # 检查是否完成
        if decision["action"] == "return":
            return decision["result"]

        # 步骤 3: Act
        if decision["action"] == "call_tool":
            tool_name = decision["tool"]
            tool_input = decision["input"]

            # 权限检查
            if tool_name not in tools:
                return "错误：工具不存在"

            tool_output = tools[tool_name](**tool_input)

            # 步骤 4: Observe
            memory["context"].append({
                "role": "tool",
                "content": f"Tool: {tool_name}, Output: {tool_output}"
            })

    return "达到最大步数"

# ========== 组装 Harness ==========
harness = {
    "loop": loop,
    "tools": tools,
    "memory": memory
}

# ========== 创建 Agent ==========
# 假设我们有一个 Model（如 GPT-4）
agent = lambda spec: harness["loop"](
    spec,
    model=your_model,  # 注入 Model
    tools=harness["tools"],
    memory=harness["memory"]
)
```

#### 步骤 2：运行 Agent

```python
# 用户提问
user_question = "明天北京天气怎么样？"

# Agent 开始工作
result = agent(user_question)

# 内部流程（简化）：
"""
Reason #1:
  Context = ["用户问：明天北京天气怎么样？"]
  Model 思考：需要调用 get_weather 工具
  决定：call_tool("get_weather", {"city": "北京", "date": "明天"})

Act #1:
  调用 get_weather("北京", "明天")

Observe #1:
  结果：{"weather": "晴", "temperature": "25°C"}
  Context += "天气工具返回：晴，25°C"

Reason #2:
  Context = ["用户问...", "天气工具返回..."]
  Model 思考：已经获得答案，可以回复用户
  决定：return "明天北京天气晴，温度25°C"
"""

print(result)
# 输出：明天北京天气晴，温度25°C
```

---

### 案例 2：翻译 Agent 📝

**需求：**
把英文翻译成中文，并解释难词

#### 设计

```python
# ========== Tools ==========
tools = {
    # 原子 Tool
    "translate": lambda text: "翻译结果...",
    "explain_word": lambda word: "单词解释...",

    # 递归 Tool（另一个 Agent）
    "grammar_agent": another_agent  # 可以调用其他 Agent！
}

# ========== Loop 流程 ==========
"""
用户：Translate "The cat sat on the mat"

Reason #1:
  → 决定：先用 translate 工具翻译整句

Act #1 → Observe #1:
  → translate("The cat sat on the mat")
  → 结果："猫坐在垫子上"

Reason #2:
  → 发现有生词 "mat"
  → 决定：用 explain_word 解释 "mat"

Act #2 → Observe #2:
  → explain_word("mat")
  → 结果："mat = 垫子、地毯"

Reason #3:
  → 所有任务完成
  → Return: "翻译：猫坐在垫子上。注释：mat=垫子"
"""
```

---

### 案例 3：递归 Agent（Agent 调用 Agent）🔁

**需求：**
写一篇文章，需要调用研究 Agent 和写作 Agent

```
主 Agent（写文章）
  ├─ 研究 Agent（查资料）
  │   ├─ 搜索 Tool
  │   └─ 总结 Tool
  └─ 写作 Agent（写内容）
      ├─ 语法检查 Tool
      └─ 润色 Tool
```

**代码框架：**

```python
# ========== 研究 Agent ==========
research_agent = Agent(
    model=model,
    tools={
        "search": search_tool,
        "summarize": summarize_tool
    }
)

# ========== 写作 Agent ==========
writing_agent = Agent(
    model=model,
    tools={
        "check_grammar": grammar_tool,
        "polish": polish_tool
    }
)

# ========== 主 Agent ==========
main_agent = Agent(
    model=model,
    tools={
        "research": research_agent,  # Agent 作为 Tool！
        "write": writing_agent       # Agent 作为 Tool！
    }
)

# ========== 使用 ==========
result = main_agent("写一篇关于 AI 的文章")

# 内部流程：
"""
主 Agent:
  Reason: 需要先研究 AI
  Act: 调用 research_agent("AI 相关资料")
    ↓
    研究 Agent:
      Reason: 需要搜索
      Act: 调用 search("AI")
      Observe: 搜索结果...
      Return: 研究报告
    ↑
  Observe: 收到研究报告

  Reason: 现在可以写作了
  Act: 调用 writing_agent("根据报告写文章")
    ↓
    写作 Agent:
      Reason: 先写草稿
      Act: 生成草稿
      Reason: 需要检查语法
      Act: 调用 check_grammar
      Return: 最终文章
    ↑
  Observe: 收到最终文章

  Return: 完成的文章
"""
```

---

## 常见问题

### Q1: Memory 和 Resources 有什么区别？

**答：**

| 特性 | Memory (Context) | Resources |
|------|------------------|-----------|
| **位置** | 眼前（书桌上） | 外部（书柜、图书馆） |
| **访问方式** | 直接看到 | 需要通过 Tools 去拿 |
| **Model 可见性** | Model 可以直接读 | Model 看不到，必须 Act |
| **大小限制** | 有限（窗口大小） | 可以很大 |
| **例子** | 当前对话、笔记 | 数据库、文件系统、API |

**用代码说明：**

```python
# Memory（Context）
context = ["用户说：你好", "我回复：你好"]
# Model 可以直接读取
decision = model.think(context)  # ✓ 直接访问

# Resources
database = {"user_info": "..."}
# Model 看不到，需要 Tool
decision = model.think(context)  # Model 不知道 database 的存在
tool_call = "query_database"     # 必须通过 Tool
result = tools[tool_call](database)  # 用 Tool 去获取
```

### Q2: 为什么 Reasoning 不是独立组件？

**答：**

Reasoning（推理）是 Model 看到 Context 后**自然产生**的行为，不是一个单独的组件。

**比喻：**
```
你（Model）看到一道数学题（Context）
↓
你的大脑自动开始思考（Reasoning）
↓
这是你大脑的能力，不是一个独立的工具
```

**代码说明：**

```python
# 错误理解（把 Reasoning 当成独立组件）
reasoning_component = ReasoningEngine()
result = reasoning_component.process(context)  # ✗

# 正确理解（Reasoning 是 Model 的行为）
class Model:
    def __init__(self, weights):
        self.weights = weights  # 知识

    def think(self, context):
        # Reasoning 是 weights 作用于 context 的涌现行为
        # 不是独立的组件，而是 Model 的能力
        return self._process(context, self.weights)  # ✓
```

### Q3: Effect 是什么？

**答：**

Effect（副作用）是 Tool 对环境产生的影响。

**分类：**

```
无副作用（纯读操作）：
  ✓ 查字典：只是看，不改变任何东西
  ✓ 搜索：只是获取信息
  ✓ 读文件：只是读，不修改

有副作用：
  ✓ 写文件：文件内容改变了
  ✓ 发邮件：邮件被发送了（不可撤回）
  ✓ 删除数据：数据被删除了
```

**代码示例：**

```python
# 无副作用
def search(query: str) -> list:
    results = database.find(query)  # 只读
    return results  # Effect = None

# 有副作用
def send_email(to: str, content: str) -> str:
    email_server.send(to, content)  # Effect！环境改变了
    return "邮件已发送"

# 多副作用
def buy_product(product_id: str) -> dict:
    # Effect 1: 扣款
    account.deduct_money(price)
    # Effect 2: 减库存
    inventory.decrease(product_id)
    # Effect 3: 创建订单
    order = create_order(product_id)
    return order
```

### Q4: Agent 可以调用 Agent 吗？

**答：**

可以！这叫**递归 Tool**。

```
Tools = { 原子 Tool }  ← 基础工具（叶子节点）
      ∪ { Agent }      ← Agent 也是 Tool（可以有子节点）
```

**递归终止条件：**
到达**叶 Agent**（不再调用其他 Agent 的 Agent）。

**示例：**

```
总经理 Agent
  ├─ 财务 Agent（叶 Agent，只用基础 Tool）
  ├─ 技术 Agent
  │   ├─ 前端 Agent（叶 Agent）
  │   └─ 后端 Agent（叶 Agent）
  └─ 市场 Agent（叶 Agent）
```

**代码：**

```python
# 叶 Agent（递归终止）
frontend_agent = Agent(
    model=model,
    tools={"code": code_tool}  # 只有原子 Tool
)

# 递归 Agent
tech_agent = Agent(
    model=model,
    tools={
        "frontend": frontend_agent,  # Agent 作为 Tool
        "backend": backend_agent
    }
)

# 顶层 Agent
ceo_agent = Agent(
    model=model,
    tools={
        "tech": tech_agent,      # 递归
        "finance": finance_agent
    }
)
```

### Q5: 授权（Auth）是怎么工作的？

**答：**

授权确保 Agent 只能做它被允许做的事。

```
Spec ──(建立 Auth(self) ⊆ Auth(caller))──▶ Loop
             ↓
      自己的权限 ≤ 调用者的权限

Act ──(验证 Auth(self) ⊇ Tool.required)──▶ 执行
            ↓
      自己的权限 ≥ 工具要求的权限
```

**简单理解：**

1. **接收任务时**：你的权限不能超过给你任务的人
2. **使用工具时**：你的权限必须满足工具的要求

**例子：**

```python
# 场景 1：权限继承
用户权限 = ["读文件", "写文件"]
Agent 权限 = ["读文件"]  # ✓ 子集，合法

# 场景 2：工具权限检查
Agent 权限 = ["读文件"]
Tool 要求 = ["读文件"]  # ✓ 满足，可以使用

Agent 权限 = ["读文件"]
Tool 要求 = ["写文件"]  # ✗ 不满足，拒绝使用

# 代码实现
def act(tool_name, agent_auth, tools):
    tool = tools[tool_name]
    required_auth = tool.required_permissions

    if agent_auth >= required_auth:  # 权限检查
        return tool.execute()
    else:
        raise PermissionError("权限不足")
```

### Q6: 为什么 Context 有大小限制？

**答：**

Context 受 Model 的**上下文窗口**限制。

**比喻：**
```
你的书桌（Context）只有 1 米宽
↓
放不下 100 本书
↓
只能放最重要的几本
```

**技术原因：**
```
Model 处理信息的能力有限
↓
一次只能"看到"有限的 tokens
↓
GPT-4: 8k, 32k, 128k tokens
Claude: 100k, 200k tokens
```

**解决方案：**

```python
# 方案 1：总结压缩
if len(context) > max_tokens:
    context = summarize(context)

# 方案 2：只保留最近的
context = context[-max_tokens:]

# 方案 3：使用外部 Memory（Resources）
if len(history) > max_tokens:
    save_to_database(old_history)  # 存到 Resources
    context = recent_history + [pointer_to_database]
```

### Q7: 什么时候 Loop 会停止？

**答：**

```
Return 条件 ∈ { 完成 | max_steps | 错误 | 人工中断 }
```

**详细说明：**

1. **完成**：任务做完了
   ```python
   if task_completed:
       return result
   ```

2. **max_steps**：执行步数太多，强制停止
   ```python
   for step in range(max_steps):
       # ...
   return "超过最大步数"
   ```

3. **错误**：出错了，无法继续
   ```python
   try:
       # ...
   except Exception as e:
       return f"错误：{e}"
   ```

4. **人工中断**：用户手动停止
   ```python
   if user_pressed_stop:
       return "用户中断"
   ```

---

## 🎓 知识总结

### 核心公式

```
1. Harness = Loop + Tools + Memory
2. Agent   = Harness(Model)
3. Tools   = 原子 Tool ∪ Agent（递归）
4. Memory  = Context（有限大小）
5. Resources ≠ Memory（需要 Tools 访问）
```

### 关键理解

1. **Model vs Harness**
   - Model = 大脑（思考）
   - Harness = 环境（执行）

2. **Memory vs Resources**
   - Memory = 眼前可见
   - Resources = 需要去取

3. **Loop 循环**
   - Reason（思考）→ Act（行动）→ Observe（观察）→ 重复

4. **授权机制**
   - 接收任务：权限 ⊆ 调用者
   - 使用工具：权限 ⊇ 工具要求

5. **递归**
   - Agent 可以是 Tool
   - 递归在叶 Agent 终止

---

## 🚀 快速开始模板

### 最简单的 Agent

```python
# 1. 定义工具
tools = {
    "calculator": lambda expr: eval(expr)
}

# 2. 定义循环
def simple_loop(task, model, tools):
    context = [task]

    for _ in range(10):
        # 思考
        decision = model.think(context)

        # 完成？
        if decision["done"]:
            return decision["result"]

        # 行动
        tool = tools[decision["tool"]]
        result = tool(decision["input"])

        # 观察
        context.append(result)

    return "超时"

# 3. 创建 Agent
agent = lambda task: simple_loop(task, my_model, tools)

# 4. 使用
answer = agent("计算 25 * 4")
print(answer)  # 100
```

---

## 📖 延伸阅读

### 推荐资料

1. **ReAct 论文**：介绍 Reason + Act 模式
2. **LangChain 文档**：实践 Agent 框架
3. **AutoGPT 源码**：完整的 Agent 实现

### 进阶主题

1. **Multi-Agent 系统**：多个 Agent 协作
2. **Memory 管理**：长期记忆、短期记忆
3. **Tool 设计模式**：如何设计好的 Tool
4. **错误处理**：Agent 出错了怎么办

---

## ✅ 检查清单

在实现自己的 Agent 之前，确保你理解：

- [ ] Harness 三要素：Loop、Tools、Memory
- [ ] Loop 流程：Reason → Act → Observe
- [ ] Memory 和 Resources 的区别
- [ ] Agent 如何递归调用
- [ ] 授权机制如何工作
- [ ] 何时 Loop 会停止

---

## 💡 总结

**Harness = 工作环境**
- Loop = 工作流程
- Tools = 可用工具
- Memory = 可见信息

**Agent = 智能体**
- Model = 大脑
- Harness = 环境
- Agent = Model + Harness

**核心理念**
- 结构清晰
- 职责分明
- 可递归扩展
- 权限可控

---

**恭喜！🎉 你已经掌握了 Harness 架构的核心概念！**

现在你可以：
1. 设计自己的 Agent
2. 实现 Loop 流程
3. 创建自定义 Tools
4. 管理 Memory 和 Resources

**开始你的 Agent 之旅吧！** 🚀
