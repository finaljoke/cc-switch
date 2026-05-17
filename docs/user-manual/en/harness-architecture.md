# Complete Guide to Harness Architecture 🎯

> **An Agent architecture manual that even elementary school students can quickly master**

## 📚 Table of Contents

1. [What is Harness?](#what-is-harness)
2. [Core Concepts](#core-concepts)
3. [Architecture Deep Dive](#architecture-deep-dive)
4. [Practical Examples](#practical-examples)
5. [Frequently Asked Questions](#frequently-asked-questions)

---

## What is Harness?

### 🎭 Simple Analogy

Imagine you're a chef (Model), and **Harness** is your kitchen!

- **Loop** = Your workflow: read recipe → get ingredients → cook → taste → adjust
- **Tools** = Your utensils: knife, pan, spatula, oven
- **Memory** = The recipe in front of you and all the information you can see

**Core Formula:**
```
Harness = Loop + Tools + Memory
Agent   = Harness(Model)
```

In plain English:
- **Harness** = A complete working environment (the kitchen)
- **Model** = Your brain (capable of thinking and making decisions)
- **Agent** = A chef working in the kitchen (brain + kitchen)

---

## Core Concepts

### 1️⃣ Model - The Brain

```
Model = Parametric(weights, read-only)
```

**Simple Understanding:**
- This is the AI's "brain," storing all learned knowledge
- **Read-only**: Can't modify the brain's knowledge, only use it to think
- **Reasoning**: The thought process when the brain sees information

**Example:**
```
You learned math → This is your "weights"
See 1+1=? → Your brain starts "reasoning"
Get answer 2 → This is the reasoning result
```

### 2️⃣ Context - Information at Hand

```
Context = Current context window (everything you can see)
Memory  = Context  ← Harness can directly access
```

**Simple Understanding:**
- Context is like the desk in front of you, holding everything you can currently see
- Limited: Desk size is finite, can't hold too much
- Memory is Context, they're the same thing

**Example:**
```
You're doing homework
On the desk:
  ✓ Math textbook (current task)
  ✓ Notebook (history)
  ✓ Homework questions (Spec)
  ✓ Previous answers (past observations)
```

### 3️⃣ Resources - External Storage

```
Resources = Store + ExternalAPIs + ...
  Store = External persistent storage (databases, file systems, etc.)
```

**Simple Understanding:**
- Resources are things outside the desk: bookshelf, drawers, library
- You **can't see** them directly, need to use Tools to get them
- Difference from Memory: Memory is in front of you, Resources need to be fetched

**Example:**
```
Memory (in front of you)    vs    Resources (need to fetch)
  Textbook on desk                  Dictionary on shelf
  Homework in front                 Reference books in library
  Notes opened                      Old exams in drawer
```

### 4️⃣ Tools - Methods to Do Things

```
Tools = { f : Input → Output × Effect? }    ← Atomic Tool (basic tools)
      ∪ { Agent }                            ← Recursive Tool (advanced tools)
```

**Simple Understanding:**
- **Atomic Tool**: Basic tools like calculator, dictionary, pencil
  - `Input`: What you give it (e.g., numbers "5+3")
  - `Output`: The result it returns (e.g., "8")
  - `Effect`: Possible impact (e.g., writing uses up pencil lead)

- **Recursive Tool**: Can be another Agent (advanced tool that calls other tools)

**Example:**
```
Atomic Tool:
  lookup_dictionary(word="apple") → "a fruit" (no side effect)
  write_text(text="Hello") → "Written" (side effect: text added to paper)

Recursive Tool (Agent as Tool):
  translation_assistant(text="Hello")
    → Internally calls: lookup_dictionary + understand grammar + assemble sentence
    → Returns: "你好"
```

---

## Architecture Deep Dive

### 🔄 Loop - The Workflow

This is the most core part of Harness!

```
Loop = Spec ──(establish Auth(self) ⊆ Auth(caller))──▶
       Reason                    ← At least once (must think)
     → [ Act ──(verify Auth(self) ⊇ Tool.required)──▶
         Observe
       → Reason ]*               ← Zero or more times (may repeat)
     → Return
```

#### Step 1: Spec (Task Specification)

**Simple Understanding:**
- This is the task or question you received
- Establish authorization: Confirm you have permission to do this

**Example:**
```
Teacher: Please do the math problems on page 5
  ↓
Spec = "Complete page 5 math problems"
Auth check: This is homework assigned by teacher ✓
```

#### Step 2: Reason (Think)

```
Reason: Model reads Context (includes Spec + history) → Decision
```

**Simple Understanding:**
- Look at all information in front of you (things on desk)
- Use brain to think about what to do next
- Must think at least once!

**Example:**
```
Context contains:
  - Task: Do page 5 math problems
  - History: I just finished problems 1-3
  - Tools: I have a calculator

Reason result:
  "I should do problem 4, it's multiplication, I need the calculator"
```

#### Step 3: Act (Take Action)

```
Act: Call Tools → Affect Environment / Resources
Permission check: Verify Tool allows you to use it
```

**Simple Understanding:**
- Use tools to execute the task
- Before using tools, check permissions (can you use this tool?)

**Example:**
```
Decision: Use calculator to compute 25 × 4
Permission check: Calculator allowed ✓
Act: Input "25 × 4" into calculator
```

#### Step 4: Observe (Observe Results)

```
Observe: Tool return value + Environment feedback → Append to Context
```

**Simple Understanding:**
- Look at the result the tool gives you
- Write the result in notebook (append to Context)

**Example:**
```
Calculator shows: 100
Write in notebook: Problem 4 answer is 100
Context updated: This record added to desk
```

#### Step 5: Repeat or Finish

```
→ Reason again to think whether to continue
→ Or Return final result
```

**Simple Understanding:**
- Think again: Are there more problems to do?
- If yes: Repeat Act → Observe → Reason
- If no or limit reached: Return (finish)

**Return conditions:**
```
✓ Complete: All problems finished
✓ max_steps: Too many steps, forced stop
✓ Error: Something went wrong, can't continue
✓ Manual interrupt: Teacher told you to stop
```

---

### 🎯 Complete Loop Example

**Task: Solve a math problem "Calculate (5 + 3) × 2 = ?"**

```
┌─────────────────────────────────────────┐
│ Spec: "Calculate (5 + 3) × 2"           │
│ Auth: User allows calculator use ✓       │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Reason #1:                               │
│ Context = [task, no history]            │
│ Think: Need to calculate 5+3 first,     │
│        then multiply by 2                │
│ Decision: Use calculator for 5+3        │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Act #1:                                  │
│ Tool: Calculator                         │
│ Input: "5 + 3"                           │
│ Permission check: ✓                      │
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
│ Context = [task, "5+3=8"]               │
│ Think: Now need to calculate 8 × 2      │
│ Decision: Use calculator for 8×2        │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Act #2:                                  │
│ Tool: Calculator                         │
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
│ Context = [task, "5+3=8", "8×2=16"]     │
│ Think: Task complete! Answer is 16      │
│ Decision: Return result                 │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Return: 16                               │
└─────────────────────────────────────────┘
```

---

### 🏗️ Harness = Loop + Tools + Memory

```
Harness = Loop + Tools + Memory  ← Three components tightly coupled
```

**Simple Understanding:**
- These three components work closely together, inseparable
- Loop needs Tools to execute tasks
- Loop needs Memory to remember information
- Tools' results are written to Memory
- Memory's content influences Loop's decisions

**Kitchen Analogy:**
```
Loop (process)    = Read recipe → Chop → Stir-fry → Taste → Adjust
Tools             = Knife, pan, spatula, seasonings
Memory            = Recipe in front + completed steps

Relationship:
  Process tells you which tool to use
  Tool results are recorded in memory
  Memory helps process make decisions
```

---

### 🤖 Agent = Harness(Model)

```
Agent = Harness(Model)
  Model is injected into Loop.Reason as execution engine
  Model owns: Parametric (knowledge weights)
  Harness owns: Context + Loop + Tools
  No overlap in responsibilities
```

**Simple Understanding:**
- Agent = Chef working in the kitchen
- Model = Chef's brain (knowledge and thinking ability)
- Harness = The entire kitchen (work environment)
- Model's "thinking ability" is placed into Harness's "Reason step"

**Division of Responsibilities:**
```
Model (brain) is responsible for:
  ✓ Understanding tasks
  ✓ Analyzing situations
  ✓ Making decisions
  ✓ Judging results

Harness (environment) is responsible for:
  ✓ Providing tools
  ✓ Managing memory
  ✓ Controlling workflow
  ✓ Executing operations
```

---

## Practical Examples

### Example 1: Weather Agent 🌤️

**Requirement:**
User asks: "What's the weather tomorrow in Beijing?"

#### Step 1: Design Harness

```python
# ========== Define Tools ==========
def get_weather(city: str, date: str) -> dict:
    """Tool to query weather"""
    # Call weather API
    return {
        "city": city,
        "date": date,
        "weather": "Sunny",
        "temperature": "25°C"
    }

tools = {
    "get_weather": get_weather
}

# ========== Define Memory ==========
memory = {
    "context": [],  # Store conversation history
    "max_tokens": 4096  # Context window size
}

# ========== Define Loop ==========
def loop(spec: str, model, tools, memory):
    # Step 1: Spec
    memory["context"].append({"role": "user", "content": spec})

    max_steps = 10
    for step in range(max_steps):
        # Step 2: Reason
        decision = model.think(memory["context"])

        # Check if complete
        if decision["action"] == "return":
            return decision["result"]

        # Step 3: Act
        if decision["action"] == "call_tool":
            tool_name = decision["tool"]
            tool_input = decision["input"]

            # Permission check
            if tool_name not in tools:
                return "Error: Tool doesn't exist"

            tool_output = tools[tool_name](**tool_input)

            # Step 4: Observe
            memory["context"].append({
                "role": "tool",
                "content": f"Tool: {tool_name}, Output: {tool_output}"
            })

    return "Reached max steps"

# ========== Assemble Harness ==========
harness = {
    "loop": loop,
    "tools": tools,
    "memory": memory
}

# ========== Create Agent ==========
# Assume we have a Model (like GPT-4)
agent = lambda spec: harness["loop"](
    spec,
    model=your_model,  # Inject Model
    tools=harness["tools"],
    memory=harness["memory"]
)
```

#### Step 2: Run Agent

```python
# User question
user_question = "What's the weather tomorrow in Beijing?"

# Agent starts working
result = agent(user_question)

# Internal process (simplified):
"""
Reason #1:
  Context = ["User asks: What's the weather tomorrow in Beijing?"]
  Model thinks: Need to call get_weather tool
  Decision: call_tool("get_weather", {"city": "Beijing", "date": "tomorrow"})

Act #1:
  Call get_weather("Beijing", "tomorrow")

Observe #1:
  Result: {"weather": "Sunny", "temperature": "25°C"}
  Context += "Weather tool returned: Sunny, 25°C"

Reason #2:
  Context = ["User asks...", "Weather tool returned..."]
  Model thinks: Already got answer, can reply to user
  Decision: return "Tomorrow Beijing weather: Sunny, 25°C"
"""

print(result)
# Output: Tomorrow Beijing weather: Sunny, 25°C
```

---

### Example 2: Translation Agent 📝

**Requirement:**
Translate English to Chinese and explain difficult words

#### Design

```python
# ========== Tools ==========
tools = {
    # Atomic Tool
    "translate": lambda text: "Translation result...",
    "explain_word": lambda word: "Word explanation...",

    # Recursive Tool (another Agent)
    "grammar_agent": another_agent  # Can call other Agents!
}

# ========== Loop Process ==========
"""
User: Translate "The cat sat on the mat"

Reason #1:
  → Decision: First use translate tool for entire sentence

Act #1 → Observe #1:
  → translate("The cat sat on the mat")
  → Result: "猫坐在垫子上"

Reason #2:
  → Found difficult word "mat"
  → Decision: Use explain_word to explain "mat"

Act #2 → Observe #2:
  → explain_word("mat")
  → Result: "mat = cushion, carpet"

Reason #3:
  → All tasks complete
  → Return: "Translation: 猫坐在垫子上. Note: mat=cushion"
"""
```

---

### Example 3: Recursive Agent (Agent Calling Agent) 🔁

**Requirement:**
Write an article, need to call research Agent and writing Agent

```
Main Agent (write article)
  ├─ Research Agent (find info)
  │   ├─ Search Tool
  │   └─ Summarize Tool
  └─ Writing Agent (write content)
      ├─ Grammar Check Tool
      └─ Polish Tool
```

**Code Framework:**

```python
# ========== Research Agent ==========
research_agent = Agent(
    model=model,
    tools={
        "search": search_tool,
        "summarize": summarize_tool
    }
)

# ========== Writing Agent ==========
writing_agent = Agent(
    model=model,
    tools={
        "check_grammar": grammar_tool,
        "polish": polish_tool
    }
)

# ========== Main Agent ==========
main_agent = Agent(
    model=model,
    tools={
        "research": research_agent,  # Agent as Tool!
        "write": writing_agent       # Agent as Tool!
    }
)

# ========== Usage ==========
result = main_agent("Write an article about AI")

# Internal process:
"""
Main Agent:
  Reason: Need to research AI first
  Act: Call research_agent("AI related info")
    ↓
    Research Agent:
      Reason: Need to search
      Act: Call search("AI")
      Observe: Search results...
      Return: Research report
    ↑
  Observe: Received research report

  Reason: Now can write
  Act: Call writing_agent("Write article based on report")
    ↓
    Writing Agent:
      Reason: First write draft
      Act: Generate draft
      Reason: Need to check grammar
      Act: Call check_grammar
      Return: Final article
    ↑
  Observe: Received final article

  Return: Completed article
"""
```

---

## Frequently Asked Questions

### Q1: What's the difference between Memory and Resources?

**Answer:**

| Feature | Memory (Context) | Resources |
|---------|------------------|-----------|
| **Location** | In front (on desk) | External (shelf, library) |
| **Access Method** | Directly visible | Need Tools to fetch |
| **Model Visibility** | Model can read directly | Model can't see, must Act |
| **Size Limit** | Limited (window size) | Can be very large |
| **Example** | Current conversation, notes | Database, file system, APIs |

**Code Illustration:**

```python
# Memory (Context)
context = ["User said: Hello", "I replied: Hello"]
# Model can read directly
decision = model.think(context)  # ✓ Direct access

# Resources
database = {"user_info": "..."}
# Model can't see, needs Tool
decision = model.think(context)  # Model doesn't know database exists
tool_call = "query_database"     # Must use Tool
result = tools[tool_call](database)  # Use Tool to fetch
```

### Q2: Why isn't Reasoning an independent component?

**Answer:**

Reasoning is a behavior that **emerges naturally** when Model sees Context, not a separate component.

**Analogy:**
```
You (Model) see a math problem (Context)
↓
Your brain automatically starts thinking (Reasoning)
↓
This is your brain's capability, not a separate tool
```

**Code Illustration:**

```python
# Wrong understanding (treating Reasoning as independent component)
reasoning_component = ReasoningEngine()
result = reasoning_component.process(context)  # ✗

# Correct understanding (Reasoning is Model's behavior)
class Model:
    def __init__(self, weights):
        self.weights = weights  # Knowledge

    def think(self, context):
        # Reasoning is emergent behavior of weights acting on context
        # Not an independent component, but Model's capability
        return self._process(context, self.weights)  # ✓
```

### Q3: What is Effect?

**Answer:**

Effect (side effect) is the impact a Tool has on the environment.

**Classification:**

```
No side effect (pure read operations):
  ✓ Lookup dictionary: Just looking, doesn't change anything
  ✓ Search: Just getting information
  ✓ Read file: Just reading, not modifying

Has side effect:
  ✓ Write file: File content changed
  ✓ Send email: Email sent (irreversible)
  ✓ Delete data: Data deleted
```

**Code Example:**

```python
# No side effect
def search(query: str) -> list:
    results = database.find(query)  # Read only
    return results  # Effect = None

# Has side effect
def send_email(to: str, content: str) -> str:
    email_server.send(to, content)  # Effect! Environment changed
    return "Email sent"

# Multiple side effects
def buy_product(product_id: str) -> dict:
    # Effect 1: Deduct money
    account.deduct_money(price)
    # Effect 2: Decrease inventory
    inventory.decrease(product_id)
    # Effect 3: Create order
    order = create_order(product_id)
    return order
```

### Q4: Can Agents call other Agents?

**Answer:**

Yes! This is called **recursive Tool**.

```
Tools = { Atomic Tool }  ← Basic tools (leaf nodes)
      ∪ { Agent }        ← Agent is also Tool (can have children)
```

**Recursion Termination Condition:**
Reaches **leaf Agent** (Agent that no longer calls other Agents).

**Example:**

```
CEO Agent
  ├─ Finance Agent (leaf Agent, only uses basic Tools)
  ├─ Tech Agent
  │   ├─ Frontend Agent (leaf Agent)
  │   └─ Backend Agent (leaf Agent)
  └─ Marketing Agent (leaf Agent)
```

**Code:**

```python
# Leaf Agent (recursion terminates)
frontend_agent = Agent(
    model=model,
    tools={"code": code_tool}  # Only atomic Tools
)

# Recursive Agent
tech_agent = Agent(
    model=model,
    tools={
        "frontend": frontend_agent,  # Agent as Tool
        "backend": backend_agent
    }
)

# Top-level Agent
ceo_agent = Agent(
    model=model,
    tools={
        "tech": tech_agent,      # Recursive
        "finance": finance_agent
    }
)
```

### Q5: How does authorization (Auth) work?

**Answer:**

Authorization ensures Agent can only do what it's allowed to do.

```
Spec ──(establish Auth(self) ⊆ Auth(caller))──▶ Loop
             ↓
      Your permissions ≤ Caller's permissions

Act ──(verify Auth(self) ⊇ Tool.required)──▶ Execute
            ↓
      Your permissions ≥ Tool's requirements
```

**Simple Understanding:**

1. **When receiving task**: Your permissions can't exceed task giver's
2. **When using tool**: Your permissions must meet tool's requirements

**Example:**

```python
# Scenario 1: Permission inheritance
User permissions = ["read_file", "write_file"]
Agent permissions = ["read_file"]  # ✓ Subset, legal

# Scenario 2: Tool permission check
Agent permissions = ["read_file"]
Tool requires = ["read_file"]  # ✓ Satisfied, can use

Agent permissions = ["read_file"]
Tool requires = ["write_file"]  # ✗ Not satisfied, denied

# Code implementation
def act(tool_name, agent_auth, tools):
    tool = tools[tool_name]
    required_auth = tool.required_permissions

    if agent_auth >= required_auth:  # Permission check
        return tool.execute()
    else:
        raise PermissionError("Insufficient permissions")
```

### Q6: Why does Context have size limits?

**Answer:**

Context is limited by Model's **context window**.

**Analogy:**
```
Your desk (Context) is only 1 meter wide
↓
Can't fit 100 books
↓
Can only place most important few
```

**Technical Reason:**
```
Model's information processing capability is limited
↓
Can only "see" limited tokens at once
↓
GPT-4: 8k, 32k, 128k tokens
Claude: 100k, 200k tokens
```

**Solutions:**

```python
# Solution 1: Summarize and compress
if len(context) > max_tokens:
    context = summarize(context)

# Solution 2: Keep only recent
context = context[-max_tokens:]

# Solution 3: Use external Memory (Resources)
if len(history) > max_tokens:
    save_to_database(old_history)  # Store in Resources
    context = recent_history + [pointer_to_database]
```

### Q7: When does Loop stop?

**Answer:**

```
Return conditions ∈ { complete | max_steps | error | manual interrupt }
```

**Detailed Explanation:**

1. **Complete**: Task finished
   ```python
   if task_completed:
       return result
   ```

2. **max_steps**: Too many execution steps, forced stop
   ```python
   for step in range(max_steps):
       # ...
   return "Exceeded max steps"
   ```

3. **Error**: Something went wrong, can't continue
   ```python
   try:
       # ...
   except Exception as e:
       return f"Error: {e}"
   ```

4. **Manual interrupt**: User manually stopped
   ```python
   if user_pressed_stop:
       return "User interrupted"
   ```

---

## 🎓 Knowledge Summary

### Core Formulas

```
1. Harness = Loop + Tools + Memory
2. Agent   = Harness(Model)
3. Tools   = Atomic Tool ∪ Agent (recursive)
4. Memory  = Context (limited size)
5. Resources ≠ Memory (needs Tools to access)
```

### Key Understanding

1. **Model vs Harness**
   - Model = Brain (thinking)
   - Harness = Environment (execution)

2. **Memory vs Resources**
   - Memory = Visible in front
   - Resources = Need to fetch

3. **Loop Cycle**
   - Reason (think) → Act (action) → Observe (observe) → Repeat

4. **Authorization Mechanism**
   - Receive task: permissions ⊆ caller
   - Use tool: permissions ⊇ tool requirements

5. **Recursion**
   - Agent can be Tool
   - Recursion terminates at leaf Agent

---

## 🚀 Quick Start Template

### Simplest Agent

```python
# 1. Define tools
tools = {
    "calculator": lambda expr: eval(expr)
}

# 2. Define loop
def simple_loop(task, model, tools):
    context = [task]

    for _ in range(10):
        # Think
        decision = model.think(context)

        # Done?
        if decision["done"]:
            return decision["result"]

        # Act
        tool = tools[decision["tool"]]
        result = tool(decision["input"])

        # Observe
        context.append(result)

    return "Timeout"

# 3. Create Agent
agent = lambda task: simple_loop(task, my_model, tools)

# 4. Use
answer = agent("Calculate 25 * 4")
print(answer)  # 100
```

---

## 📖 Further Reading

### Recommended Resources

1. **ReAct Paper**: Introduces Reason + Act pattern
2. **LangChain Documentation**: Practical Agent framework
3. **AutoGPT Source Code**: Complete Agent implementation

### Advanced Topics

1. **Multi-Agent Systems**: Multiple Agents collaborating
2. **Memory Management**: Long-term memory, short-term memory
3. **Tool Design Patterns**: How to design good Tools
4. **Error Handling**: What to do when Agent fails

---

## ✅ Checklist

Before implementing your own Agent, make sure you understand:

- [ ] Harness three elements: Loop, Tools, Memory
- [ ] Loop process: Reason → Act → Observe
- [ ] Difference between Memory and Resources
- [ ] How Agents recursively call each other
- [ ] How authorization mechanism works
- [ ] When Loop stops

---

## 💡 Summary

**Harness = Working Environment**
- Loop = Workflow
- Tools = Available tools
- Memory = Visible information

**Agent = Intelligent Entity**
- Model = Brain
- Harness = Environment
- Agent = Model + Harness

**Core Philosophy**
- Clear structure
- Well-defined responsibilities
- Recursively extensible
- Controllable permissions

---

**Congratulations! 🎉 You've mastered the core concepts of Harness architecture!**

Now you can:
1. Design your own Agent
2. Implement Loop workflows
3. Create custom Tools
4. Manage Memory and Resources

**Start your Agent journey!** 🚀
