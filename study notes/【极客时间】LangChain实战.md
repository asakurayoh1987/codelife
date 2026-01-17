# LangChain实战课

## 00、Architecture

The LangChain framework consists of multiple open-source libraries. Read more in the [Architecture](https://python.langchain.com/docs/concepts/architecture/) page.

- **`langchain-core`**: Base abstractions for chat models and other components.
- **Integration packages** (e.g. `langchain-openai`, `langchain-anthropic`, etc.): Important integrations have been split into lightweight packages that are co-maintained by the LangChain team and the integration developers.
- **`langchain`**: Chains, agents, and retrieval strategies that make up an application's cognitive architecture.
- **`langchain-community`**: Third-party integrations that are community maintained.
- **`langgraph`**: Orchestration framework for combining LangChain components into production-ready applications with persistence, streaming, and other key features. See [LangGraph documentation](https://langchain-ai.github.io/langgraph/).

## 01、LangChain Demo

[langchain github](https://github.com/langchain-ai)

```python
from dotenv import load_dotenv
import os
from langchain_openai.chat_models import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

load_dotenv()

api_key = os.getenv("API_KEY")
api_base_url = os.getenv("API_BASE_URL")
api_model = os.getenv("API_MODEL")

# 使用正确的配置方式初始化模型
chat = ChatOpenAI(
    model=api_model,
    temperature=0.6,
    base_url=api_base_url,
    api_key=api_key,
    streaming=True,
)

system_template = "Translate the following from English into {language}"
prompt_template = ChatPromptTemplate.from_messages(
    [("system", system_template), ("user", "{text}")]
)

prompt = prompt_template.invoke({"language": "Chinese", "text": "hello, world!"})

try:
    # 使用流式处理
    for chunk in chat.stream(prompt):
        print(chunk.content, end="", flush=True)
    print()  # 添加换行

except Exception as e:
    print(f"发生错误: {e}")

```



## 02、 基于LangChain实现的问答系统（RAG）

