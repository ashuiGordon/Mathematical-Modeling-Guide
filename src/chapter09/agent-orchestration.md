# Agent 编排与自动化建模

## 引言

数学建模竞赛是一项综合性极强的智力活动，参赛者需要在有限时间内完成问题分析、模型构建、编程求解、结果验证和论文撰写等多个环节。传统的 AI 辅助方式——向 ChatBot 提问并获取回答——已经难以满足复杂建模任务的需求。我们需要的不是一个"问答机器"，而是一个能够自主感知任务、制定计划、调用工具并持续迭代的**智能体（Agent）**。

本章将系统介绍 AI Agent 的核心概念、架构设计与实现方法，并展示如何构建一个面向数学建模的自动化 Pipeline。

---

## 9.1 AI Agent 基础概念

### 9.1.1 什么是 AI Agent

AI Agent（智能体）是一个能够**自主感知环境、做出决策并采取行动**的系统。其核心运行机制是一个持续的循环：

```
感知(Perceive) → 决策(Decide) → 行动(Act) → 反馈 → 感知...
```

- **感知**：接收用户输入、环境状态、工具返回结果
- **决策**：基于大语言模型的推理能力，分析当前状态并规划下一步
- **行动**：调用工具、执行代码、生成文本或与其他 Agent 通信
- **反馈循环**：根据行动结果评估是否达成目标，决定是否继续迭代

### 9.1.2 Agent vs 普通 ChatBot

| 维度 | 普通 ChatBot | AI Agent |
|------|-------------|----------|
| 交互模式 | 单轮问答 | 多步自主执行 |
| 工具使用 | 无 | 可调用外部工具 |
| 任务规划 | 无 | 自主分解与规划 |
| 记忆能力 | 仅上下文窗口 | 短期+长期记忆 |
| 错误处理 | 无法自纠 | 可检测错误并重试 |

以建模为例：ChatBot 回答"如何用层次分析法"，Agent 则能自动分析赛题、选择模型、编码求解、验证结果。

### 9.1.3 工具调用（Tool Use / Function Calling）

工具调用是 Agent 的核心能力。大语言模型本身只能生成文本，但通过工具调用机制，它可以执行代码、搜索文献、读写文件、调用专业软件。

```python
# 工具定义示例
tools = [
    {
        "name": "python_executor",
        "description": "执行Python代码并返回结果",
        "parameters": {
            "type": "object",
            "properties": {
                "code": {"type": "string", "description": "要执行的Python代码"}
            },
            "required": ["code"]
        }
    },
    {
        "name": "web_search",
        "description": "搜索学术文献和建模资料",
        "parameters": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "搜索关键词"}
            },
            "required": ["query"]
        }
    }
]
```

### 9.1.4 记忆系统

Agent 的记忆分为两类：

- **短期记忆**：当前对话上下文和中间结果，容量有限但访问快
- **长期记忆**：持久化存储的知识（历届赛题模式、模型模板等），通过向量数据库检索

```python
class AgentMemory:
    def __init__(self):
        self.short_term = []       # 当前对话历史
        self.long_term = VectorStore()  # 向量数据库

    def recall(self, query, top_k=5):
        """从长期记忆中检索相关信息"""
        return self.long_term.search(query, top_k=top_k)
```

---

## 9.2 建模 Agent 架构设计

### 9.2.1 单 Agent 建模助手

最简单的架构是一个 Agent 完成所有任务，适合问题简单、流程较短的场景。

```
用户输入赛题 → [LLM] 分析问题 → [工具] 编写执行代码 → [LLM] 验证结果 → 生成报告
```

优点：实现简单，上下文连贯。缺点：角色混乱，上下文窗口有限。

### 9.2.2 多 Agent 协作系统

对于复杂任务，推荐多 Agent 协作，模拟团队分工：

```
审题Agent → 建模Agent → 编码Agent → 写作Agent
```

| Agent | 角色 | 核心能力 | 输出物 |
|-------|------|---------|--------|
| 审题 Agent | 问题分析专家 | 信息提取、问题分类 | 结构化问题描述 |
| 建模 Agent | 数学建模专家 | 模型选择、公式推导 | 数学模型文档 |
| 编码 Agent | 编程实现专家 | 代码编写、调试 | 可运行的代码 |
| 写作 Agent | 论文撰写专家 | 学术写作、排版 | 完整论文 |

### 9.2.3 Agent 间通信与协调

**流水线模式**：Agent 按固定顺序执行，前者输出作为后者输入。

```python
class ModelingPipeline:
    def run(self, problem_text):
        analysis = self.agents["analyst"].analyze(problem_text)
        model = self.agents["modeler"].design(analysis)
        results = self.agents["coder"].implement(model)
        paper = self.agents["writer"].compose(analysis, model, results)
        return paper
```

**中心调度模式**：主管 Agent 统一调度，动态决定执行顺序。

```python
class OrchestratorAgent:
    def run(self, problem_text):
        plan = self.create_plan(problem_text)
        for step in plan:
            agent = self.select_agent(step)
            result = agent.execute(step)
            if not self.validate(result):
                plan = self.revise_plan(plan, result)
        return self.compile_results()
```

---

## 9.3 关键技术

### 9.3.1 RAG：检索增强生成

RAG（Retrieval-Augmented Generation）使 Agent 能查阅外部知识库——历届优秀论文、模型理论、标准方法。

```python
from langchain.vectorstores import FAISS
from langchain.embeddings import OpenAIEmbeddings

class ModelingKnowledgeBase:
    def __init__(self):
        self.embeddings = OpenAIEmbeddings()
        self.vectorstore = FAISS.load_local("modeling_kb", self.embeddings)

    def query(self, question, top_k=3):
        docs = self.vectorstore.similarity_search(question, k=top_k)
        return "\n".join([doc.page_content for doc in docs])
```

构建知识库步骤：收集素材 → 文本切分（500-1000字/片段） → Embedding向量化 → 存入向量数据库 → 检索服务。

### 9.3.2 MCP：模型上下文协议

MCP（Model Context Protocol）是 Anthropic 提出的开放标准，为 Agent 与外部工具提供统一接口。

```python
from mcp import Server

server = Server("modeling-tools")

@server.tool("solve_linear_programming")
def solve_lp(objective: str, constraints: list[str]):
    """求解线性规划问题"""
    from scipy.optimize import linprog
    result = linprog(c, A_ub=A, b_ub=b)
    return {"optimal_value": result.fun, "solution": result.x.tolist()}
```

通过 MCP，可以为 Agent 配备专业工具（优化求解器、统计检验等），而无需修改 Agent 代码。

### 9.3.3 Code Interpreter

Code Interpreter 使 Agent 能即时执行代码验证结果：Agent 生成代码 → 沙箱执行 → 结果反馈 → Agent 据此调整。

### 9.3.4 Web Search

Web Search 使 Agent 获取最新学术文献、数据集和方法论，填补知识库的时效性空白。

---

## 9.4 自动化建模 Pipeline 设计

### 9.4.1 完整流程

```
输入赛题 → 问题分析 → 文献调研 → 模型推荐 → 代码生成 → 结果验证 → 报告生成
```

```python
class AutoModelingPipeline:
    def run(self, problem_text: str):
        self.state["analysis"] = self.analyze_problem(problem_text)
        self.state["literature"] = self.search_literature(
            self.state["analysis"]["keywords"])
        # 人工确认点
        candidates = self.recommend_models(self.state)
        self.state["model"] = self.human_confirm("请确认模型选择", candidates)
        self.state["code"] = self.generate_code(self.state)
        self.state["results"] = self.execute_and_verify(self.state["code"])
        # 人工确认点
        self.human_confirm("请检查计算结果", self.state["results"])
        return self.generate_report(self.state)
```

### 9.4.2 人机协作节点设计

全自动化并非最优。关键决策点需人工确认：

| 阶段 | 自动化程度 | 人工介入 |
|------|-----------|---------|
| 问题分析 | 全自动 | 确认理解是否正确 |
| 模型选择 | 半自动 | **确认模型方案** |
| 代码实现 | 全自动 | 报错时介入 |
| 结果验证 | 半自动 | **确认结果合理性** |
| 论文撰写 | 全自动 | **最终审阅修改** |

设计原则：关键决策由人确认；异常触发介入；渐进式放权；保留回溯能力。

---

## 9.5 实现工具与框架

### 9.5.1 Claude Agent SDK

```python
import anthropic

client = anthropic.Anthropic()

def modeling_agent(problem: str):
    messages = [{"role": "user", "content": problem}]
    system_prompt = "你是数学建模专家Agent，逐步分析问题、选择模型、编码求解。"

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=4096,
            system=system_prompt,
            tools=tools,
            messages=messages
        )
        if response.stop_reason == "tool_use":
            tool_results = process_tool_calls(response)
            messages.append({"role": "assistant", "content": response.content})
            messages.append({"role": "user", "content": tool_results})
        else:
            return extract_final_answer(response)
```

### 9.5.2 LangGraph

LangGraph 适合构建有状态的多步骤 Agent 工作流：

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class ModelingState(TypedDict):
    problem: str
    analysis: str
    model_type: str
    code: str
    results: str
    paper: str

def analyze_problem(state: ModelingState) -> ModelingState:
    analysis = llm.invoke(f"分析赛题：\n{state['problem']}")
    return {**state, "analysis": analysis}

def select_model(state: ModelingState) -> ModelingState:
    model_type = llm.invoke(f"推荐模型：\n{state['analysis']}")
    return {**state, "model_type": model_type}

def generate_code(state: ModelingState) -> ModelingState:
    code = llm.invoke(f"编写代码：\n{state['model_type']}")
    return {**state, "code": code}

def validate_results(state: ModelingState) -> ModelingState:
    results = execute_code(state["code"])
    return {**state, "results": results}

def write_paper(state: ModelingState) -> ModelingState:
    paper = llm.invoke(f"撰写论文：分析={state['analysis']}，结果={state['results']}")
    return {**state, "paper": paper}

workflow = StateGraph(ModelingState)
workflow.add_node("analyze", analyze_problem)
workflow.add_node("model", select_model)
workflow.add_node("code", generate_code)
workflow.add_node("validate", validate_results)
workflow.add_node("write", write_paper)
workflow.set_entry_point("analyze")
workflow.add_edge("analyze", "model")
workflow.add_edge("model", "code")
workflow.add_edge("code", "validate")
workflow.add_edge("validate", "write")
workflow.add_edge("write", END)

app = workflow.compile()
result = app.invoke({"problem": "赛题内容..."})
```

### 9.5.3 CrewAI

CrewAI 专注于多 Agent 角色协作：

```python
from crewai import Agent, Task, Crew

analyst = Agent(
    role="数学建模分析师",
    goal="深入分析赛题，识别问题类型和关键约束",
    tools=[web_search_tool, knowledge_base_tool]
)
modeler = Agent(
    role="建模专家",
    goal="设计最优的数学模型方案",
    tools=[symbolic_math_tool]
)
coder = Agent(
    role="算法工程师",
    goal="将模型转化为高效Python代码",
    tools=[code_executor_tool]
)
writer = Agent(
    role="论文撰写专家",
    goal="撰写逻辑清晰的建模论文",
    tools=[document_tool]
)

tasks = [
    Task(description="分析赛题", agent=analyst),
    Task(description="设计模型", agent=modeler),
    Task(description="实现代码", agent=coder),
    Task(description="撰写论文", agent=writer)
]
crew = Crew(agents=[analyst, modeler, coder, writer], tasks=tasks)
result = crew.kickoff(inputs={"problem": "赛题内容..."})
```

---

## 9.6 实战案例：构建数学建模 Agent

### 9.6.1 完整实现

```python
"""
modeling_agent.py — 一个完整的数学建模Agent
"""
import json
import subprocess
import tempfile
import anthropic

# ===== 工具实现 =====

def execute_python(code: str) -> str:
    """在安全环境中执行Python代码"""
    with tempfile.NamedTemporaryFile(mode='w', suffix='.py', delete=False) as f:
        f.write(code)
        f.flush()
        try:
            result = subprocess.run(
                ['python', f.name],
                capture_output=True, text=True, timeout=30
            )
            if result.returncode == 0:
                return f"执行成功:\n{result.stdout}"
            else:
                return f"执行错误:\n{result.stderr}"
        except subprocess.TimeoutExpired:
            return "错误: 代码执行超时(30秒)"

def query_knowledge_base(query: str) -> str:
    """查询建模知识库"""
    knowledge = {
        "优化": "线性规划、整数规划、非线性规划、动态规划",
        "预测": "时间序列(ARIMA)、回归分析、灰色预测、神经网络",
        "评价": "层次分析法(AHP)、TOPSIS、熵权法、模糊综合评价",
        "分类": "聚类分析、判别分析、支持向量机、决策树",
        "图论": "最短路径、最小生成树、网络流、旅行商问题",
    }
    results = [f"{k}: {v}" for k, v in knowledge.items() if k in query]
    return "\n".join(results) if results else "未找到直接匹配，请细化查询"

# ===== 工具定义 =====

TOOLS = [
    {
        "name": "execute_python",
        "description": "执行Python代码进行数学计算",
        "input_schema": {
            "type": "object",
            "properties": {
                "code": {"type": "string", "description": "要执行的Python代码"}
            },
            "required": ["code"]
        }
    },
    {
        "name": "query_knowledge_base",
        "description": "查询数学建模知识库",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "查询内容"}
            },
            "required": ["query"]
        }
    }
]

TOOL_MAP = {
    "execute_python": lambda inp: execute_python(inp["code"]),
    "query_knowledge_base": lambda inp: query_knowledge_base(inp["query"]),
}

# ===== Agent 主循环 =====

SYSTEM_PROMPT = """你是一个专业的数学建模Agent。工作流程：
1. 阅读题目，提取关键信息（已知条件、约束、目标）
2. 查询知识库，了解相关方法
3. 确定合适的数学模型
4. 编写Python代码求解
5. 执行代码验证结果
6. 总结解题过程和结论
每一步通过工具调用支撑分析，计算必须实际运行代码。"""

def run_modeling_agent(problem: str, max_iterations: int = 10):
    """运行建模Agent"""
    client = anthropic.Anthropic()
    messages = [{"role": "user", "content": problem}]

    print(f"{'='*50}\n数学建模Agent启动\n{'='*50}\n")

    for iteration in range(max_iterations):
        print(f"--- 迭代 {iteration + 1} ---")
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=4096,
            system=SYSTEM_PROMPT,
            tools=TOOLS,
            messages=messages
        )
        messages.append({"role": "assistant", "content": response.content})

        if response.stop_reason == "end_turn":
            print("[Agent完成任务]")
            return "".join(b.text for b in response.content if hasattr(b, "text"))

        if response.stop_reason == "tool_use":
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    print(f"  调用: {block.name}")
                    result = TOOL_MAP[block.name](block.input)
                    print(f"  结果: {result[:80]}...")
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result
                    })
            messages.append({"role": "user", "content": tool_results})

    return "达到最大迭代次数"

# ===== 使用示例 =====
if __name__ == "__main__":
    problem = """
    某城市有5个消防站和8个重点防火区域。已知各消防站到各区域的响应时间（分钟）：

    站\\区  A   B   C   D   E   F   G   H
    站1    5   8  12  15   7   9  11  14
    站2    9   4   7  10  12   6   8  11
    站3   11   7   3   6  14  10   5   9
    站4   14  11   8   4   9  12   7   3
    站5    7  13  10   8   5  11  13   6

    每个消防站最多覆盖3个区域。建立优化模型使所有区域最大响应时间最小化。
    """
    result = run_modeling_agent(problem)
    print(f"\n{'='*50}\n最终结果\n{'='*50}\n{result}")
```

### 9.6.2 运行效果

运行后 Agent 会自动完成：
1. **问题分析**：识别为 Minimax 分配优化问题
2. **知识检索**：查询优化模型相关方法
3. **代码生成**：编写 `pulp` 整数规划求解代码
4. **执行验证**：运行代码获取最优解
5. **结果汇报**：总结最优分配方案

```
==================================================
数学建模Agent启动
==================================================

--- 迭代 1 ---
  调用: query_knowledge_base
  结果: 优化: 线性规划、整数规划...
--- 迭代 2 ---
  调用: execute_python
  结果: 执行成功: 最优响应时间 = 9分钟...
[Agent完成任务]
```

---

## 9.7 注意事项与最佳实践

### 9.7.1 可靠性

```python
def robust_tool_call(tool_func, inputs, max_retries=3):
    for attempt in range(max_retries):
        try:
            result = tool_func(**inputs)
            if validate_result(result):
                return result
        except Exception as e:
            if attempt == max_retries - 1:
                raise
    return None
```

- 代码结果要做合理性检查（数量级、正负号、物理意义）
- 关键中间步骤设置断言

### 9.7.2 Prompt 工程

好的 System Prompt 应包含：角色定位、明确的工作步骤、约束条件、输出格式要求。避免模糊指令，给 Agent 清晰的行动边界。

### 9.7.3 成本控制

| 策略 | 说明 |
|------|------|
| 设置最大迭代次数 | 防止无限循环 |
| 使用缓存 | 相同查询不重复调用 |
| 分级模型 | 简单任务用小模型，复杂推理用大模型 |
| 精简上下文 | 及时总结，避免过长历史 |

### 9.7.4 安全性

- **沙箱执行**：Agent 生成的代码必须在隔离环境中运行
- **权限最小化**：只赋予完成任务所需的最小权限
- **输出审核**：最终输出必须人工审核

### 9.7.5 常见陷阱

1. **过度依赖**：Agent 输出必须经人工验证
2. **LLM 幻觉**：可能编造不存在的公式或定理
3. **调试困难**：多 Agent 错误传播难追踪，需完整日志
4. **上下文溢出**：长对话导致早期信息被"遗忘"

---

## 9.8 前沿展望

### 9.8.1 全自动建模的可能性与局限

当前已可实现：自动识别问题类型、选择模型框架、生成调试代码、生成论文初稿。

显著局限：
- **创新不足**：Agent 擅长应用已知方法，难以提出全新思路
- **领域深度有限**：专业性极强的问题理解不够深入
- **推理链脆弱**：多步推理中错误会累积放大

### 9.8.2 发展趋势

- **短期（1-2年）**：Agent 作为建模助手加速工作，人机协作成为主流
- **中期（3-5年）**：多 Agent 独立完成标准化建模任务，具备从反馈中学习的能力
- **长期**：Agent 能进行创造性建模，人类从"执行者"转变为"审核者"

### 9.8.3 给建模者的建议

1. **拥抱协作**：将 Agent 作为放大能力的工具
2. **提升不可替代性**：培养创新思维和跨学科整合能力
3. **掌握 Prompt 工程**：学会有效传达意图和约束
4. **保持批判思维**：始终验证 Agent 输出，不盲目信任

---

## 本章小结

本章系统介绍了 AI Agent 在数学建模中的应用——从感知-决策-行动循环的基础概念，到单 Agent 与多 Agent 协作架构，再到 RAG、MCP、Code Interpreter 等关键技术，最后通过完整实战案例展示了从零构建建模 Agent 的全过程。Agent 编排不是为了取代建模者，而是让你从重复性劳动中解放出来，将精力集中在最有价值的创造性工作上。
