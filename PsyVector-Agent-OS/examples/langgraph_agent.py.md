
import json
import operator
from typing import Annotated, TypedDict, Union

from langchain_openai import ChatOpenAI
from langchain_core.messages import SystemMessage, HumanMessage, BaseMessage
from langgraph.graph import StateGraph, END

# --- 1. 定义 Agent 状态 ---
class AgentState(TypedDict):
    messages: Annotated[list[BaseMessage], operator.add]
    current_skill_id: str
    skill_instruction: str

# --- 2. 加载 PsyVector 数据 ---
def load_manifest():
    with open("../manifest.json", "r", encoding="utf-8") as f:
        return json.load(f)

MANIFEST = load_manifest()
SKILL_MAP = {s['id']: s for s in MANIFEST['skills']}

# --- 3. 定义节点 (Nodes) ---

def router_node(state: AgentState):
    """
    路由节点：分析用户意图，选择 Skill ID。
    (实际可使用 LLM Function Calling 进行选择)
    """
    last_message = state["messages"][-1].content.lower()
    
    # 简单的模拟逻辑
    selected_id = "pv_02_the_sustainer" # Default
    
    if "fire" in last_message or "cut" in last_message:
        selected_id = "pv_43_decisive_reformer"
    elif "detail" in last_message or "check" in last_message:
        selected_id = "pv_63_quality_guardian"
        
    print(f"\n[Router] Selected Skill: {selected_id}")
    return {"current_skill_id": selected_id}

def skill_loader_node(state: AgentState):
    """
    加载节点：读取 .skill.md 文件，提取 System Instruction
    """
    skill_id = state["current_skill_id"]
    skill_meta = SKILL_MAP.get(skill_id)
    
    # 读取文件 (假设路径正确)
    file_path = f"../{skill_meta['file_path']}" 
    with open(file_path, "r", encoding="utf-8") as f:
        content = f.read()
        # 提取 --- 后的正文
        instruction = content.split("---")[2].strip()
    
    return {"skill_instruction": instruction}

def generation_node(state: AgentState):
    """
    生成节点：使用加载的 Skill 调用 LLM
    """
    llm = ChatOpenAI(model="gpt-4o") # 需要配置 OPENAI_API_KEY
    
    # 动态构建 Prompt：System (Skill) + User Input
    messages = [
        SystemMessage(content=state["skill_instruction"]),
    ] + state["messages"]
    
    response = llm.invoke(messages)
    return {"messages": [response]}

# --- 4. 构建图 (Graph) ---
workflow = StateGraph(AgentState)

# 添加节点
workflow.add_node("router", router_node)
workflow.add_node("loader", skill_loader_node)
workflow.add_node("generator", generation_node)

# 定义流程
workflow.set_entry_point("router")
workflow.add_edge("router", "loader")
workflow.add_edge("loader", "generator")
workflow.add_edge("generator", END)

# 编译图
app = workflow.compile()

# --- 5. 运行示例 ---
if __name__ == "__main__":
    print("🚀 Starting PsyVector x LangGraph Agent...")
    
    user_input = "I need to fire a toxic employee, but I feel guilty."
    inputs = {"messages": [HumanMessage(content=user_input)]}
    
    # 运行图
    # 注意：运行此代码需要安装 langgraph, langchain, langchain_openai 并设置 API Key
    # 这里仅演示流程结构
    print(f"User Input: {user_input}")
    
    # 模拟输出过程 (避免无 Key 报错)
    print("\n--- Graph Execution Trace ---")
    print("1. [Router] Analyzed intent: Needs decisiveness.")
    print("2. [Router] Selected: pv_43_decisive_reformer (Margaret Thatcher).")
    print("3. [Loader] Loaded instruction from skills/heaven/pv_43_decisive_reformer.skill.md.")
    print("4. [Generator] LLM is now acting as Margaret Thatcher...")
    print("5. [Output] 'Guilt helps no one. State the facts. Make the cut clean.'")