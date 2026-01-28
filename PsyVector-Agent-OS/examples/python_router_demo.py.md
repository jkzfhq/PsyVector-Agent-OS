
import json
import os
import yaml  # pip install pyyaml

# 假设文件结构：
# ./manifest.json
# ./skills/heaven/pv_01_the_sovereign.skill.md
# ...

class PsyVectorOS:
    def __init__(self, manifest_path="../manifest.json", skills_root="../"):
        self.manifest_path = manifest_path
        self.skills_root = skills_root
        self.skills_db = self._load_manifest()
        print(f"✅ PsyVector OS Initialized. Loaded {len(self.skills_db)} skills.")

    def _load_manifest(self):
        """加载技能索引表"""
        try:
            with open(self.manifest_path, 'r', encoding='utf-8') as f:
                data = json.load(f)
                # 将列表转换为以 ID 为键的字典，方便检索
                return {skill['id']: skill for skill in data['skills']}
        except FileNotFoundError:
            print("❌ Error: manifest.json not found.")
            return {}

    def _read_skill_content(self, relative_path):
        """读取具体的 .skill.md 文件内容"""
        full_path = os.path.join(self.skills_root, relative_path)
        try:
            with open(full_path, 'r', encoding='utf-8') as f:
                content = f.read()
                # 分离 YAML 头和 System Instruction 正文
                parts = content.split("---")
                if len(parts) >= 3:
                    system_prompt = parts[2].strip()
                    return system_prompt
                return content
        except Exception as e:
            return f"Error loading skill file: {str(e)}"

    def route(self, user_input):
        """
        [模拟路由层]
        在生产环境中，这里应该调用 LLM 或向量数据库 (Vector DB) 
        来计算 user_input 与 manifest 中 'activation_rules' 的相似度。
        
        此处为了演示，使用简单的关键词规则匹配。
        """
        print(f"\n🔍 Analyzing Intent: '{user_input}'...")
        
        # --- 模拟路由逻辑 (Mock Router) ---
        if "start" in user_input or "create" in user_input:
            return "pv_01_the_sovereign"
        elif "stuck" in user_input or "lazy" in user_input:
            return "pv_51_shock_awakener" 
        elif "anxious" in user_input or "worry" in user_input:
            return "pv_52_mindful_anchor"
        elif "contract" in user_input or "argue" in user_input:
            return "pv_06_strategic_negotiator"
        else:
            # 默认兜底：大地母亲 (Sustainer)
            return "pv_02_the_sustainer"

    def execute(self, user_input):
        """执行 Agent 流程"""
        # 1. 路由
        skill_id = self.route(user_input)
        skill_info = self.skills_db.get(skill_id)
        
        if not skill_info:
            print("⚠️ Skill not found.")
            return

        print(f"👉 Matched Archetype: [{skill_info['display_name']}]")
        print(f"   (Core Drive: {skill_info['core_drive']})")

        # 2. 加载 System Prompt
        system_instruction = self._read_skill_content(skill_info['file_path'])
        
        # 3. 模拟 LLM 生成 (实际项目中这里调用 OpenAI API)
        print("\n📝 [Injecting System Prompt into LLM]...")
        print("-" * 40)
        print(f"SYSTEM: {system_instruction[:150]}...\n(Truncated for view)...")
        print("-" * 40)
        print(f"🤖 AI Response ({skill_info['display_name']} Mode):")
        print("   [Here the LLM would generate a response based on the loaded persona...]")

# --- Main Execution ---
if __name__ == "__main__":
    agent = PsyVectorOS()
    
    # 测试案例 1: 创业/启动
    agent.execute("I want to start a new company but I'm scared.")
    
    # 测试案例 2: 焦虑/内耗
    agent.execute("I am so anxious about the deadline, I can't breathe.")