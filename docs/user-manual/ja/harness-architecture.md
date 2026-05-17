# Harness アーキテクチャ完全ガイド 🎯

> **小学生でも素早くマスターできるAgentアーキテクチャマニュアル**

## 📚 目次

1. [Harnessとは？](#harnessとは)
2. [コアコンセプト](#コアコンセプト)
3. [アーキテクチャ詳解](#アーキテクチャ詳解)
4. [実践例](#実践例)
5. [よくある質問](#よくある質問)

---

## Harnessとは？

### 🎭 シンプルな例え

あなたがシェフ(Model)で、**Harness**はあなたのキッチンだと想像してください！

- **Loop(ループ)** = あなたのワークフロー: レシピを見る → 材料を取る → 調理する → 味見する → 調整する
- **Tools(ツール)** = あなたの調理器具: 包丁、フライパン、お玉、オーブン
- **Memory(メモリ)** = 目の前のレシピとあなたが見ることができるすべての情報

**コア公式:**
```
Harness = Loop + Tools + Memory
Agent   = Harness(Model)
```

簡単に言うと:
- **Harness** = 完全な作業環境(キッチン)
- **Model** = あなたの脳(思考と意思決定が可能)
- **Agent** = キッチンで働くシェフ(脳 + キッチン)

---

## コアコンセプト

### 1️⃣ Model(モデル) - 脳

```
Model = Parametric(重み, 読み取り専用)
```

**シンプルな理解:**
- これはAIの「脳」で、学習したすべての知識を保存しています
- **読み取り専用**: 脳の知識を変更できず、思考にのみ使用できます
- **Reasoning(推論)**: 脳が情報を見たときの思考プロセス

**例:**
```
あなたは数学を学びました → これがあなたの「重み」です
1+1=?を見る → あなたの脳が「推論」を始めます
答え2を得る → これが推論の結果です
```

### 2️⃣ Context(コンテキスト) - 手元の情報

```
Context = 現在のコンテキストウィンドウ(見えるすべてのもの)
Memory  = Context  ← Harnessが直接アクセス可能
```

**シンプルな理解:**
- Contextは目の前の机のようなもので、現在見ることができるすべてのものが置かれています
- 制限あり: 机のサイズは有限で、あまり多くのものを置けません
- MemoryはContextです。これらは同じものです

**例:**
```
あなたは宿題をしています
机の上には:
  ✓ 数学の教科書(現在のタスク)
  ✓ ノート(履歴)
  ✓ 宿題の問題(Spec)
  ✓ 以前の答え(過去の観察)
```

### 3️⃣ Resources(リソース) - 外部ストレージ

```
Resources = Store + ExternalAPIs + ...
  Store = 外部永続ストレージ(データベース、ファイルシステムなど)
```

**シンプルな理解:**
- Resourcesは机の外にあるもの: 本棚、引き出し、図書館
- それらを**直接見ることはできず**、Toolsを使って取得する必要があります
- Memoryとの違い: Memoryは目の前にあり、Resourcesは取りに行く必要があります

**例:**
```
Memory(目の前)           vs    Resources(取りに行く必要がある)
  机の上の教科書                  本棚の辞書
  目の前の宿題                    図書館の参考書
  開いているノート                引き出しの過去の試験問題
```

### 4️⃣ Tools(ツール) - 物事を行う方法

```
Tools = { f : Input → Output × Effect? }    ← Atomic Tool(基本ツール)
      ∪ { Agent }                            ← Recursive Tool(高度なツール)
```

**シンプルな理解:**
- **Atomic Tool**: 計算機、辞書、鉛筆などの基本ツール
  - `Input`: 与えるもの(例: 数字「5+3」)
  - `Output`: 返される結果(例: 「8」)
  - `Effect`: 可能な影響(例: 文字を書くと鉛筆の芯が減る)

- **Recursive Tool**: 別のAgentになることができます(他のツールを呼び出す高度なツール)

**例:**
```
Atomic Tool:
  lookup_dictionary(word="apple") → "りんご" (副作用なし)
  write_text(text="こんにちは") → "書き込み完了" (副作用: 紙に文字が追加)

Recursive Tool (ToolとしてのAgent):
  translation_assistant(text="Hello")
    → 内部で呼び出し: lookup_dictionary + 文法理解 + 文章組み立て
    → 返す: "こんにちは"
```

---

## アーキテクチャ詳解

### 🔄 Loop(ループ) - ワークフロー

これはHarnessの最も中心的な部分です！

```
Loop = Spec ──(Auth(self) ⊆ Auth(caller)を確立)──▶
       Reason                    ← 少なくとも1回(必ず思考)
     → [ Act ──(Auth(self) ⊇ Tool.requiredを検証)──▶
         Observe
       → Reason ]*               ← 0回以上(繰り返し可能)
     → Return
```

#### ステップ1: Spec(タスク仕様)

**シンプルな理解:**
- これはあなたが受け取ったタスクまたは質問です
- 認可の確立: これを行う権限があることを確認します

**例:**
```
先生: 5ページの数学問題を解いてください
  ↓
Spec = "5ページの数学問題を完了する"
認可チェック: これは先生から与えられた宿題です ✓
```

#### ステップ2: Reason(思考)

```
Reason: ModelがContextを読む(Spec + 履歴を含む) → 意思決定
```

**シンプルな理解:**
- 目の前のすべての情報を見る(机の上のもの)
- 脳を使って次に何をすべきか考える
- 少なくとも1回は考えなければなりません！

**例:**
```
Contextに含まれるもの:
  - タスク: 5ページの数学問題を解く
  - 履歴: 問題1-3を解き終わったばかり
  - ツール: 計算機がある

Reasonの結果:
  「問題4をすべきだ。掛け算なので計算機が必要」
```

#### ステップ3: Act(行動)

```
Act: Toolsを呼び出す → Environment / Resourcesに作用
権限チェック: Toolが使用を許可していることを確認
```

**シンプルな理解:**
- タスクを実行するためにツールを使用する
- ツールを使う前に権限をチェックする(このツールを使えるか？)

**例:**
```
決定: 計算機を使って25 × 4を計算
権限チェック: 計算機は使用許可 ✓
Act: 計算機に「25 × 4」を入力
```

#### ステップ4: Observe(観察)

```
Observe: Toolの戻り値 + Environmentからのフィードバック → Contextに追加
```

**シンプルな理解:**
- ツールが与える結果を見る
- 結果をノートに書く(Contextに追加)

**例:**
```
計算機の表示: 100
ノートに書く: 問題4の答えは100
Context更新: この記録が机に追加される
```

#### ステップ5: 繰り返しまたは終了

```
→ Reasonで再度考える 続けるべきか？
→ またはReturn 最終結果を返す
```

**シンプルな理解:**
- もう一度考える: 他に解くべき問題はあるか？
- あれば: Act → Observe → Reasonを繰り返す
- なければまたは制限に達した場合: Return(終了)

**返却条件:**
```
✓ 完了: すべての問題が終わった
✓ max_steps: ステップ数が多すぎる、強制停止
✓ エラー: 何か問題が発生、続行不可
✓ 手動中断: 先生が止めるよう指示
```

---

### 🎯 完全なLoopの例

**タスク: 数学問題を解く「(5 + 3) × 2 = ?」を計算**

```
┌─────────────────────────────────────────┐
│ Spec: "(5 + 3) × 2を計算"                │
│ 認可: ユーザーが計算機の使用を許可 ✓       │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Reason #1:                               │
│ Context = [タスク, 履歴なし]              │
│ 思考: まず5+3を計算し、次に2を掛ける      │
│ 決定: 計算機で5+3を計算                   │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Act #1:                                  │
│ Tool: 計算機                              │
│ Input: "5 + 3"                           │
│ 権限チェック: ✓                           │
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
│ Context = [タスク, "5+3=8"]              │
│ 思考: 次は8 × 2を計算する必要がある       │
│ 決定: 計算機で8×2を計算                  │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Act #2:                                  │
│ Tool: 計算機                              │
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
│ Context = [タスク, "5+3=8", "8×2=16"]   │
│ 思考: タスク完了！答えは16                │
│ 決定: 結果を返す                          │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Return: 16                               │
└─────────────────────────────────────────┘
```

---

### 🏗️ Harness = Loop + Tools + Memory

```
Harness = Loop + Tools + Memory  ← 3つのコンポーネントが密結合
```

**シンプルな理解:**
- これら3つのコンポーネントは密接に連携し、分離できません
- LoopはTasksを実行するためにToolsが必要
- Loopは情報を記憶するためにMemoryが必要
- Toolsの結果はMemoryに書き込まれる
- Memoryの内容がLoopの意思決定に影響

**キッチンの例え:**
```
Loop(プロセス)    = レシピを見る → 切る → 炒める → 味見 → 調整
Tools(ツール)     = 包丁、フライパン、お玉、調味料
Memory(メモリ)    = 目の前のレシピ + 完了したステップ

関係:
  プロセスがどのツールを使うか指示
  ツールの結果はメモリに記録
  メモリがプロセスの意思決定を支援
```

---

### 🤖 Agent = Harness(Model)

```
Agent = Harness(Model)
  ModelはLoop.Reasonに実行エンジンとして注入される
  Modelが所有: Parametric(知識の重み)
  Harnessが所有: Context + Loop + Tools
  責任の重複なし
```

**シンプルな理解:**
- Agent = キッチンで働くシェフ
- Model = シェフの脳(知識と思考能力)
- Harness = キッチン全体(作業環境)
- Modelの「思考能力」がHarnessの「Reasonステップ」に配置される

**責任分担:**
```
Model(脳)が担当:
  ✓ タスクの理解
  ✓ 状況の分析
  ✓ 意思決定
  ✓ 結果の判断

Harness(環境)が担当:
  ✓ ツールの提供
  ✓ メモリの管理
  ✓ ワークフローの制御
  ✓ 操作の実行
```

---

## 実践例

### 例1: 天気Agent 🌤️

**要件:**
ユーザーが尋ねる: 「明日の北京の天気は？」

#### ステップ1: Harnessの設計

```python
# ========== Toolsの定義 ==========
def get_weather(city: str, date: str) -> dict:
    """天気を照会するツール"""
    # 天気APIを呼び出す
    return {
        "city": city,
        "date": date,
        "weather": "晴れ",
        "temperature": "25°C"
    }

tools = {
    "get_weather": get_weather
}

# ========== Memoryの定義 ==========
memory = {
    "context": [],  # 会話履歴を保存
    "max_tokens": 4096  # コンテキストウィンドウのサイズ
}

# ========== Loopの定義 ==========
def loop(spec: str, model, tools, memory):
    # ステップ1: Spec
    memory["context"].append({"role": "user", "content": spec})

    max_steps = 10
    for step in range(max_steps):
        # ステップ2: Reason
        decision = model.think(memory["context"])

        # 完了チェック
        if decision["action"] == "return":
            return decision["result"]

        # ステップ3: Act
        if decision["action"] == "call_tool":
            tool_name = decision["tool"]
            tool_input = decision["input"]

            # 権限チェック
            if tool_name not in tools:
                return "エラー: ツールが存在しません"

            tool_output = tools[tool_name](**tool_input)

            # ステップ4: Observe
            memory["context"].append({
                "role": "tool",
                "content": f"Tool: {tool_name}, Output: {tool_output}"
            })

    return "最大ステップ数に到達"

# ========== Harnessの組み立て ==========
harness = {
    "loop": loop,
    "tools": tools,
    "memory": memory
}

# ========== Agentの作成 ==========
# Modelがあると仮定(GPT-4など)
agent = lambda spec: harness["loop"](
    spec,
    model=your_model,  # Modelを注入
    tools=harness["tools"],
    memory=harness["memory"]
)
```

#### ステップ2: Agentの実行

```python
# ユーザーの質問
user_question = "明日の北京の天気は？"

# Agentが動き始める
result = agent(user_question)

# 内部プロセス(簡略化):
"""
Reason #1:
  Context = ["ユーザーが尋ねる: 明日の北京の天気は？"]
  Modelの思考: get_weatherツールを呼び出す必要がある
  決定: call_tool("get_weather", {"city": "北京", "date": "明日"})

Act #1:
  get_weather("北京", "明日")を呼び出す

Observe #1:
  結果: {"weather": "晴れ", "temperature": "25°C"}
  Context += "天気ツールが返す: 晴れ、25°C"

Reason #2:
  Context = ["ユーザーが尋ねる...", "天気ツールが返す..."]
  Modelの思考: すでに答えを得た、ユーザーに返信できる
  決定: return "明日の北京の天気: 晴れ、25°C"
"""

print(result)
# 出力: 明日の北京の天気: 晴れ、25°C
```

---

### 例2: 翻訳Agent 📝

**要件:**
英語を中国語に翻訳し、難しい単語を説明する

#### 設計

```python
# ========== Tools ==========
tools = {
    # Atomic Tool
    "translate": lambda text: "翻訳結果...",
    "explain_word": lambda word: "単語の説明...",

    # Recursive Tool(別のAgent)
    "grammar_agent": another_agent  # 他のAgentを呼び出せる！
}

# ========== Loopプロセス ==========
"""
ユーザー: "The cat sat on the mat"を翻訳

Reason #1:
  → 決定: まずtranslateツールで文全体を翻訳

Act #1 → Observe #1:
  → translate("The cat sat on the mat")
  → 結果: "猫はマットの上に座った"

Reason #2:
  → 難しい単語"mat"を発見
  → 決定: explain_wordで"mat"を説明

Act #2 → Observe #2:
  → explain_word("mat")
  → 結果: "mat = マット、敷物"

Reason #3:
  → すべてのタスク完了
  → Return: "翻訳: 猫はマットの上に座った。注: mat=マット"
"""
```

---

### 例3: 再帰的Agent(AgentがAgentを呼び出す) 🔁

**要件:**
記事を書く、研究Agentと執筆Agentを呼び出す必要がある

```
メインAgent(記事を書く)
  ├─ 研究Agent(情報を探す)
  │   ├─ 検索Tool
  │   └─ 要約Tool
  └─ 執筆Agent(コンテンツを書く)
      ├─ 文法チェックTool
      └─ 推敲Tool
```

**コードフレームワーク:**

```python
# ========== 研究Agent ==========
research_agent = Agent(
    model=model,
    tools={
        "search": search_tool,
        "summarize": summarize_tool
    }
)

# ========== 執筆Agent ==========
writing_agent = Agent(
    model=model,
    tools={
        "check_grammar": grammar_tool,
        "polish": polish_tool
    }
)

# ========== メインAgent ==========
main_agent = Agent(
    model=model,
    tools={
        "research": research_agent,  # ToolとしてのAgent！
        "write": writing_agent       # ToolとしてのAgent！
    }
)

# ========== 使用 ==========
result = main_agent("AIについての記事を書く")

# 内部プロセス:
"""
メインAgent:
  Reason: まずAIを研究する必要がある
  Act: research_agent("AI関連情報")を呼び出す
    ↓
    研究Agent:
      Reason: 検索が必要
      Act: search("AI")を呼び出す
      Observe: 検索結果...
      Return: 研究レポート
    ↑
  Observe: 研究レポートを受け取る

  Reason: 今は執筆できる
  Act: writing_agent("レポートに基づいて記事を書く")を呼び出す
    ↓
    執筆Agent:
      Reason: まず下書きを書く
      Act: 下書きを生成
      Reason: 文法チェックが必要
      Act: check_grammarを呼び出す
      Return: 最終記事
    ↑
  Observe: 最終記事を受け取る

  Return: 完成した記事
"""
```

---

## よくある質問

### Q1: MemoryとResourcesの違いは？

**答え:**

| 特徴 | Memory (Context) | Resources |
|------|------------------|-----------|
| **場所** | 目の前(机の上) | 外部(本棚、図書館) |
| **アクセス方法** | 直接見える | Toolsで取得が必要 |
| **Modelの可視性** | Modelが直接読める | Modelには見えない、Actが必要 |
| **サイズ制限** | 制限あり(ウィンドウサイズ) | 非常に大きくできる |
| **例** | 現在の会話、ノート | データベース、ファイルシステム、API |

**コードで説明:**

```python
# Memory(Context)
context = ["ユーザーが言った: こんにちは", "私が返信: こんにちは"]
# Modelは直接読める
decision = model.think(context)  # ✓ 直接アクセス

# Resources
database = {"user_info": "..."}
# Modelには見えない、Toolが必要
decision = model.think(context)  # Modelはdatabaseの存在を知らない
tool_call = "query_database"     # Toolを使う必要がある
result = tools[tool_call](database)  # Toolで取得
```

### Q2: なぜReasoningは独立したコンポーネントではないのか？

**答え:**

Reasoning(推論)は、ModelがContextを見たときに**自然に発生する**振る舞いであり、独立したコンポーネントではありません。

**例え:**
```
あなた(Model)が数学問題(Context)を見る
↓
あなたの脳が自動的に考え始める(Reasoning)
↓
これはあなたの脳の能力であり、独立したツールではない
```

**コードで説明:**

```python
# 間違った理解(ReasoningIndependent componentとして扱う)
reasoning_component = ReasoningEngine()
result = reasoning_component.process(context)  # ✗

# 正しい理解(ReasoningはModelの振る舞い)
class Model:
    def __init__(self, weights):
        self.weights = weights  # 知識

    def think(self, context):
        # Reasoningは重みがcontextに作用する創発的振る舞い
        # 独立したコンポーネントではなく、Modelの能力
        return self._process(context, self.weights)  # ✓
```

### Q3: Effectとは？

**答え:**

Effect(副作用)は、Toolが環境に与える影響です。

**分類:**

```
副作用なし(純粋な読み取り操作):
  ✓ 辞書を引く: 見るだけで、何も変更しない
  ✓ 検索: 情報を取得するだけ
  ✓ ファイルを読む: 読むだけで、変更しない

副作用あり:
  ✓ ファイルに書き込む: ファイルの内容が変わった
  ✓ メールを送る: メールが送信された(取り消せない)
  ✓ データを削除: データが削除された
```

**コード例:**

```python
# 副作用なし
def search(query: str) -> list:
    results = database.find(query)  # 読み取りのみ
    return results  # Effect = None

# 副作用あり
def send_email(to: str, content: str) -> str:
    email_server.send(to, content)  # Effect！環境が変わった
    return "メール送信完了"

# 複数の副作用
def buy_product(product_id: str) -> dict:
    # Effect 1: お金を引く
    account.deduct_money(price)
    # Effect 2: 在庫を減らす
    inventory.decrease(product_id)
    # Effect 3: 注文を作成
    order = create_order(product_id)
    return order
```

### Q4: AgentがAgentを呼び出せるか？

**答え:**

はい！これを**再帰的Tool**と呼びます。

```
Tools = { Atomic Tool }  ← 基本ツール(葉ノード)
      ∪ { Agent }        ← AgentもTool(子を持てる)
```

**再帰の終了条件:**
**葉Agent**(他のAgentを呼び出さないAgent)に到達。

**例:**

```
CEOAgent
  ├─ 財務Agent(葉Agent、基本Toolのみ使用)
  ├─ 技術Agent
  │   ├─ フロントエンドAgent(葉Agent)
  │   └─ バックエンドAgent(葉Agent)
  └─ マーケティングAgent(葉Agent)
```

**コード:**

```python
# 葉Agent(再帰終了)
frontend_agent = Agent(
    model=model,
    tools={"code": code_tool}  # Atomic Toolのみ
)

# 再帰的Agent
tech_agent = Agent(
    model=model,
    tools={
        "frontend": frontend_agent,  # ToolとしてのAgent
        "backend": backend_agent
    }
)

# トップレベルAgent
ceo_agent = Agent(
    model=model,
    tools={
        "tech": tech_agent,      # 再帰
        "finance": finance_agent
    }
)
```

### Q5: 認可(Auth)はどのように機能するか？

**答え:**

認可は、Agentが許可されていることだけを行えるようにします。

```
Spec ──(Auth(self) ⊆ Auth(caller)を確立)──▶ Loop
             ↓
      自分の権限 ≤ 呼び出し元の権限

Act ──(Auth(self) ⊇ Tool.requiredを検証)──▶ 実行
            ↓
      自分の権限 ≥ ツールの要件
```

**シンプルな理解:**

1. **タスクを受け取るとき**: 自分の権限はタスクを与えた人を超えられない
2. **ツールを使うとき**: 自分の権限はツールの要件を満たす必要がある

**例:**

```python
# シナリオ1: 権限の継承
ユーザー権限 = ["read_file", "write_file"]
Agent権限 = ["read_file"]  # ✓ サブセット、合法

# シナリオ2: ツール権限チェック
Agent権限 = ["read_file"]
Tool要件 = ["read_file"]  # ✓ 満たしている、使用可能

Agent権限 = ["read_file"]
Tool要件 = ["write_file"]  # ✗ 満たしていない、拒否

# コード実装
def act(tool_name, agent_auth, tools):
    tool = tools[tool_name]
    required_auth = tool.required_permissions

    if agent_auth >= required_auth:  # 権限チェック
        return tool.execute()
    else:
        raise PermissionError("権限不足")
```

### Q6: なぜContextにサイズ制限があるのか？

**答え:**

ContextはModelの**コンテキストウィンドウ**によって制限されます。

**例え:**
```
あなたの机(Context)は幅1メートルしかない
↓
100冊の本を置けない
↓
最も重要な数冊しか置けない
```

**技術的理由:**
```
Modelの情報処理能力には限界がある
↓
一度に「見る」ことができるトークンは限られている
↓
GPT-4: 8k, 32k, 128k tokens
Claude: 100k, 200k tokens
```

**解決策:**

```python
# 解決策1: 要約圧縮
if len(context) > max_tokens:
    context = summarize(context)

# 解決策2: 最近のものだけ保持
context = context[-max_tokens:]

# 解決策3: 外部Memory(Resources)を使用
if len(history) > max_tokens:
    save_to_database(old_history)  # Resourcesに保存
    context = recent_history + [pointer_to_database]
```

### Q7: いつLoopは停止するか？

**答え:**

```
返却条件 ∈ { 完了 | max_steps | エラー | 手動中断 }
```

**詳細説明:**

1. **完了**: タスクが終わった
   ```python
   if task_completed:
       return result
   ```

2. **max_steps**: 実行ステップが多すぎる、強制停止
   ```python
   for step in range(max_steps):
       # ...
   return "最大ステップ数を超えた"
   ```

3. **エラー**: 何か問題が発生、続行不可
   ```python
   try:
       # ...
   except Exception as e:
       return f"エラー: {e}"
   ```

4. **手動中断**: ユーザーが手動で停止
   ```python
   if user_pressed_stop:
       return "ユーザーが中断"
   ```

---

## 🎓 知識のまとめ

### コア公式

```
1. Harness = Loop + Tools + Memory
2. Agent   = Harness(Model)
3. Tools   = Atomic Tool ∪ Agent(再帰)
4. Memory  = Context(制限されたサイズ)
5. Resources ≠ Memory(Toolsでアクセスが必要)
```

### 重要な理解

1. **Model vs Harness**
   - Model = 脳(思考)
   - Harness = 環境(実行)

2. **Memory vs Resources**
   - Memory = 目の前に見える
   - Resources = 取りに行く必要がある

3. **Loopサイクル**
   - Reason(思考) → Act(行動) → Observe(観察) → 繰り返し

4. **認可メカニズム**
   - タスクを受け取る: 権限 ⊆ 呼び出し元
   - ツールを使う: 権限 ⊇ ツール要件

5. **再帰**
   - AgentはToolになれる
   - 再帰は葉Agentで終了

---

## 🚀 クイックスタートテンプレート

### 最もシンプルなAgent

```python
# 1. ツールを定義
tools = {
    "calculator": lambda expr: eval(expr)
}

# 2. ループを定義
def simple_loop(task, model, tools):
    context = [task]

    for _ in range(10):
        # 思考
        decision = model.think(context)

        # 完了？
        if decision["done"]:
            return decision["result"]

        # 行動
        tool = tools[decision["tool"]]
        result = tool(decision["input"])

        # 観察
        context.append(result)

    return "タイムアウト"

# 3. Agentを作成
agent = lambda task: simple_loop(task, my_model, tools)

# 4. 使用
answer = agent("25 * 4を計算")
print(answer)  # 100
```

---

## 📖 さらに読む

### 推奨資料

1. **ReAct論文**: Reason + Actパターンを紹介
2. **LangChainドキュメント**: 実践的なAgentフレームワーク
3. **AutoGPTソースコード**: 完全なAgent実装

### 高度なトピック

1. **マルチAgentシステム**: 複数のAgentが協力
2. **Memory管理**: 長期記憶、短期記憶
3. **Tool設計パターン**: 良いToolの設計方法
4. **エラー処理**: Agentが失敗したらどうするか

---

## ✅ チェックリスト

自分のAgentを実装する前に、理解していることを確認:

- [ ] Harnessの3要素: Loop、Tools、Memory
- [ ] Loopプロセス: Reason → Act → Observe
- [ ] MemoryとResourcesの違い
- [ ] Agentが再帰的に呼び出す方法
- [ ] 認可メカニズムの動作
- [ ] Loopが停止するタイミング

---

## 💡 まとめ

**Harness = 作業環境**
- Loop = ワークフロー
- Tools = 利用可能なツール
- Memory = 見える情報

**Agent = 知的エンティティ**
- Model = 脳
- Harness = 環境
- Agent = Model + Harness

**コア哲学**
- 明確な構造
- 明確に定義された責任
- 再帰的に拡張可能
- 制御可能な権限

---

**おめでとうございます！🎉 Harnessアーキテクチャの核心概念をマスターしました！**

これからできること:
1. 独自のAgentを設計
2. Loopワークフローを実装
3. カスタムToolsを作成
4. MemoryとResourcesを管理

**Agentの旅を始めましょう！** 🚀
