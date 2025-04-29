---
title: "超越DeepSeek！阿里最新开源模型Qwen3"
date: 2025-04-20T13:54:44+08:00
draft: false
tags:
    - llm
    - Qwen3
    - deepseek
    - 大模型
---

---

> 近日，阿里巴巴重磅开源了新一代大语言模型——**Qwen3**！这款模型不仅性能全面超越Llama 4、DeepSeek-R1等顶尖模型，还首次引入“快慢脑”设计，支持灵活切换“深度思考”与“快速响应”模式，堪称AI领域的“瑞士军刀”。更重要的是，**Qwen3已通过Apache 2.0协议免费开源**，全球开发者和企业均可免费商用！  

---

## **一、Qwen3有哪些核心亮点？**  

1️⃣ **“快慢脑”混合推理模式，效率与质量兼得**  
   - **思考模式**：针对复杂问题（如数学推理、代码生成、逻辑分析），模型会逐步推导并验证答案，确保结果准确性，但耗时稍长。  
   - **非思考模式**：对简单任务（如日常对话、快速问答）直接输出答案，响应速度极快，几乎无延迟。  
   - *用户可通过指令（如 `/think` 或 `/no_think`）自由切换两种模式，甚至在同一次对话中动态调整！*  

2️⃣ **性能登顶，成本却大幅降低**  
   - **旗舰版Qwen3-235B-A22B**：总参数2350亿，激活仅需220亿，推理能力碾压Grok-3、Gemini 2.5-Pro，在AIME数学竞赛中斩获81.5分（开源模型最高纪录）。  
   - **小型MoE模型Qwen3-30B-A3B**：仅用300亿总参数、30亿激活参数，却超越Qwen2.5-72B-Instruct的性能，部署成本仅为竞品的1/3！  
   - **显存占用更低**：满血版仅需4张NVIDIA H20显卡即可部署，性价比极高。  

3️⃣ **多语言+多模态，覆盖全球需求**  
   - **119种语言支持**：从中文、英文到小语种（如越南语、泰米尔语），满足全球化业务需求。  
   - **集成MCP协议**：无缝对接外部工具（如数据库、API、插件），轻松构建智能体（Agent），完成复杂任务链（如数据分析、自动化编程）。  

4️⃣ **代码与数学能力爆表**  
   - **LiveCodeBench测试得分70+**，代码生成能力超越Grok-3；  
   - **数学推理能力**：在AIME25、MATH等基准测试中表现优异，堪比人类奥赛选手。  

   ![训练流程](https://qianwen-res.oss-accelerate.aliyuncs.com/assets/blog/qwen3/post-training.png)

---

## **二、如何快速上手Qwen3？**  

### **1. 开源平台与下载方式**  
Qwen3已在以下平台开放下载，全部采用**Apache 2.0协议**，可免费商用
- **魔搭社区**（[ModelScope](https://modelscope.cn/)）  
- **Hugging Face**（[Qwen3 Collection](https://huggingface.co/collections/Qwen/qwen3)）  
- **GitHub**（[Qwen3 GitHub](https://github.com/QwenLM/Qwen3)）  

### **2. 部署与调用方法**  
- **云端调用**：通过阿里云百炼平台（[PAI-BLADE](https://help.aliyun.com/blade)）直接调用Qwen3 API，无需本地部署。  
- **本地部署**：  
  - 推荐使用 **SGLang** 或 **vLLM** 框架加速推理；  
  - 小型模型（如Qwen3-4B）可用消费级GPU（如RTX 4090）运行；  
  - 开发者工具支持 **Ollama、LMStudio、MLX、llama.cpp** 等轻量化方案。  

### **3. 实战技巧**  
- **切换推理模式**：  
  - 在输入中添加 `/think` 触发深度思考模式（适合复杂任务）；  
  - 添加 `/no_think` 快速获取答案（适合日常对话）。  
- **调用工具（Function Calling）**：  
  - 使用 **Qwen-Agent** 框架，通过预定义模板调用外部API（如天气查询、数据库检索），实现自动化工作流。  
- **多语言交互**：  
  - 直接输入目标语言（如日语、西班牙语），模型自动适配翻译与生成能力。  

---

## **三、Qwen3适合哪些应用场景？**  

| **场景**          | **Qwen3解决方案**                                                                 |
|-------------------|----------------------------------------------------------------------------------|
| **智能客服**       | 多语言支持 + 快速响应，处理高频咨询；结合MCP协议调用订单系统，解决复杂问题。            |
| **代码辅助**       | 生成高质量代码片段，实时调试建议，支持Python、Java、C++等主流语言。                   |
| **教育领域**       | 数学题自动解析、个性化学习路径推荐，甚至模拟老师讲解知识点。                          |
| **内容创作**       | 创意写作、文案生成、多语言翻译，大幅提升生产效率。                                   |
| **企业智能化**     | 集成内部数据库与API，打造专属智能助手（如财务分析Bot、供应链优化Agent）。             |

---

## **四、本地使用Qwen3**

接下来我们实际操作一下, 在本地体验一个简单的对话：

``` python
import os

# 设置大陆镜像
os.environ['HF_ENDPOINT'] = 'https://hf-mirror.com'

from transformers import AutoModelForCausalLM, AutoTokenizer

# 使用一个最小的模型
model_name = "Qwen/Qwen3-0.6B"

# load the tokenizer and the model
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype="auto",
    device_map="auto"
)

# prepare the model input
prompt = "Give me a short introduction to large language model."
messages = [
    {"role": "user", "content": prompt}
]
text = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True,
    enable_thinking=True # Switches between thinking and non-thinking modes. Default is True.
)
model_inputs = tokenizer([text], return_tensors="pt").to(model.device)

# conduct text completion
generated_ids = model.generate(
    **model_inputs,
    max_new_tokens=32768
)
output_ids = generated_ids[0][len(model_inputs.input_ids[0]):].tolist() 

# parsing thinking content
try:
    # rindex finding 151668 (</think>)
    index = len(output_ids) - output_ids[::-1].index(151668)
except ValueError:
    index = 0

thinking_content = tokenizer.decode(output_ids[:index], skip_special_tokens=True).strip("\n")
content = tokenizer.decode(output_ids[index:], skip_special_tokens=True).strip("\n")

print("thinking content:", thinking_content)
print("content:", content)


```

尝试多轮对话, 并在过程中切换推理模式：
``` python
from transformers import AutoModelForCausalLM, AutoTokenizer

class QwenChatbot:
    def __init__(self, model_name="Qwen/Qwen3-0.6B"):
        self.tokenizer = AutoTokenizer.from_pretrained(model_name)
        self.model = AutoModelForCausalLM.from_pretrained(model_name)
        self.history = []

    def generate_response(self, user_input):
        messages = self.history + [{"role": "user", "content": user_input}]

        text = self.tokenizer.apply_chat_template(
            messages,
            tokenize=False,
            add_generation_prompt=True
        )

        inputs = self.tokenizer(text, return_tensors="pt")
        response_ids = self.model.generate(**inputs, max_new_tokens=32768)[0][len(inputs.input_ids[0]):].tolist()
        response = self.tokenizer.decode(response_ids, skip_special_tokens=True)

        # Update history
        self.history.append({"role": "user", "content": user_input})
        self.history.append({"role": "assistant", "content": response})

        return response

# Example Usage
if __name__ == "__main__":
    chatbot = QwenChatbot()

    # First input (without /think or /no_think tags, thinking mode is enabled by default)
    user_input_1 = "How many r's in strawberries?"
    print(f"User: {user_input_1}")
    response_1 = chatbot.generate_response(user_input_1)
    print(f"Bot: {response_1}")
    print("----------------------")

    # Second input with /no_think
    user_input_2 = "Then, how many r's in blueberries? /no_think"
    print(f"User: {user_input_2}")
    response_2 = chatbot.generate_response(user_input_2)
    print(f"Bot: {response_2}") 
    print("----------------------")

    # Third input with /think
    user_input_3 = "Really? /think"
    print(f"User: {user_input_3}")
    response_3 = chatbot.generate_response(user_input_3)
    print(f"Bot: {response_3}")

```

---

## **五、结语：Qwen3开启AI新纪元**  

Qwen3的发布标志着大模型进入“**灵活化、低成本、全球化**”的新阶段。无论是初创团队还是企业开发者，都能借助这一开源利器快速构建AI应用。  

**现在就行动吧！**  
- 访问 [Qwen官网](https://chat.qwen.ai/) 体验在线Demo；  
- 下载模型代码，探索更多可能性！  

> **关注我们，第一时间获取Qwen3实战教程与行业案例！**  
