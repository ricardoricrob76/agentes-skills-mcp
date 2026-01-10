# 🧠 Agentes de IA com MCP na Prática  
## Implementação Simples com Python, FastAPI e Streamlit

---

## 1. Introdução

Com a evolução dos **Large Language Models (LLMs)**, surgiu a necessidade de arquiteturas mais organizadas, escaláveis e seguras para aplicações inteligentes. Nesse contexto, **Agentes de IA** ganham destaque ao permitir autonomia, tomada de decisão e uso de ferramentas externas.

O **MCP – Model Context Protocol** surge como um padrão arquitetural que organiza como o contexto é construído, controlado e entregue ao modelo, garantindo:

- Separação clara de responsabilidades  
- Governança do contexto  
- Escalabilidade  
- Observabilidade  
- Integração com múltiplas fontes de dados  

Neste artigo, você irá construir um **Agente de IA simples**, utilizando:

- **FastAPI** como backend (MCP Server)  
- **Streamlit** como frontend  
- **Python puro**, sem frameworks complexos  

---

## 2. O que é MCP (Model Context Protocol)?

O **MCP** define uma forma estruturada de organizar o contexto fornecido ao modelo de IA, normalmente dividido em:

| Camada MCP        | Responsabilidade                                |
|-------------------|------------------------------------------------|
| System Context    | Regras, papel do agente, limites                |
| User Context      | Entrada do usuário                              |
| Memory Context    | Histórico ou estado                             |
| Tool Context      | Funções, APIs e dados externos                  |
| Output Context    | Resposta final controlada                      |

> 👉 O **MCP não é uma biblioteca**, mas sim um **padrão arquitetural**.

---

## 3. Arquitetura da Solução

### Visão Geral

[ Streamlit ]
|
| Prompt do Usuário
v
[ FastAPI - MCP Server ]
|
| Contexto Estruturado (MCP)
v
[ Agente de IA ]
|
v
[ Resposta Controlada ]

yaml
Copiar código

---

## 4. Prática Guiada – Agente MCP em Python

### 4.1 Estrutura do Projeto

ai-agent-mcp/
│
├── backend/
│ ├── main.py
│ ├── agent.py
│ └── mcp_context.py
│
├── frontend/
│ └── app.py
│
└── requirements.txt

python
Copiar código

---

### 4.2 MCP Context – Organização do Contexto

📄 **mcp_context.py**

```python
def build_mcp_context(user_input: str):
    system_context = """
    Você é um Agente de IA especialista em tecnologia.
    Responda de forma clara, objetiva e didática.
    """

    memory_context = "Nenhuma memória anterior disponível."

    tool_context = "Ferramentas disponíveis: nenhuma."

    return {
        "system": system_context,
        "memory": memory_context,
        "tools": tool_context,
        "user": user_input
    }
4.3 Agente de IA (Simples e Didático)
📄 agent.py

python
Copiar código
def ai_agent(mcp_context: dict):
    prompt = f"""
    {mcp_context['system']}

    Histórico:
    {mcp_context['memory']}

    Pergunta do usuário:
    {mcp_context['user']}
    """

    # Simulação de resposta do LLM
    response = f"🤖 Resposta do Agente:\n\nCom base na pergunta '{mcp_context['user']}', este é um exemplo de resposta estruturada usando MCP."

    return response
🔹 Observação didática: aqui simulamos o LLM.
Em produção, você conectaria uma API como OpenAI, Azure OpenAI ou um LLM local.

4.4 FastAPI – MCP Server
📄 main.py

python
Copiar código
from fastapi import FastAPI
from pydantic import BaseModel
from mcp_context import build_mcp_context
from agent import ai_agent

app = FastAPI(title="Agente de IA com MCP")

class UserRequest(BaseModel):
    question: str

@app.post("/agent")
def run_agent(request: UserRequest):
    mcp_context = build_mcp_context(request.question)
    response = ai_agent(mcp_context)
    return {"response": response}
▶️ Executar o backend

bash
Copiar código
uvicorn main:app --reload
4.5 Streamlit – Interface do Usuário
📄 app.py

python
Copiar código
import streamlit as st
import requests

st.set_page_config(page_title="Agente MCP", layout="centered")

st.title("🧠 Agente de IA com MCP")

question = st.text_input("Digite sua pergunta:")

if st.button("Consultar Agente"):
    if question:
        response = requests.post(
            "http://localhost:8000/agent",
            json={"question": question}
        )
        st.success(response.json()["response"])
    else:
        st.warning("Digite uma pergunta.")
▶️ Executar o frontend

bash
Copiar código
streamlit run app.py
5. O Que Você Aprendeu Nesta Prática
✔ Conceito de Agentes de IA
✔ Aplicação do MCP na prática
✔ Separação clara entre frontend e backend
✔ Uso de FastAPI como MCP Server
✔ Uso de Streamlit como interface interativa
✔ Arquitetura pronta para evoluir para:

RAG

Multi-agentes

Ferramentas externas

Memória persistente

Observabilidade

6. Extensões Sugeridas (Exercícios)
🔹 Adicionar memória (Redis ou banco relacional)

🔹 Integrar com um LLM real

🔹 Criar múltiplos agentes especializados

🔹 Implementar ferramentas (APIs externas)

🔹 Adicionar logs e métricas do agente

7. Conclusão
O uso de Agentes de IA com MCP permite criar soluções mais organizadas, seguras e escaláveis, mesmo em projetos simples.

Com Python, FastAPI e Streamlit, é possível sair rapidamente do conceito para uma aplicação funcional, ideal para ensino, prototipação e MVPs.
