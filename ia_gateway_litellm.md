# LiteLLM como AI Gateway para LLMs, Agents e MCP

> Uma abordagem técnica para arquitetura, roteamento, governança, observabilidade e integração de aplicações de IA Generativa com múltiplos modelos e provedores.

## Sumário

- [1. Introdução](#1-introdução)
- [2. O problema do acesso direto aos LLMs](#2-o-problema-do-acesso-direto-aos-llms)
- [3. O que é um AI Gateway](#3-o-que-é-um-ai-gateway)
- [4. O que é o LiteLLM](#4-o-que-é-o-litellm)
- [5. Arquitetura](#5-arquitetura)
- [6. Principais recursos](#6-principais-recursos)
- [7. LLM Routing](#7-llm-routing)
- [8. Fallback e resiliência](#8-fallback-e-resiliência)
- [9. Multi-Cloud e Multi-Model](#9-multi-cloud-e-multi-model)
- [10. Governança e controle de acesso](#10-governança-e-controle-de-acesso)
- [11. Gestão de custos](#11-gestão-de-custos)
- [12. Observabilidade](#12-observabilidade)
- [13. LiteLLM + RAG](#13-litellm--rag)
- [14. LiteLLM + Agents](#14-litellm--agents)
- [15. LiteLLM + MCP](#15-litellm--mcp)
- [16. LiteLLM + frameworks de IA](#16-litellm--frameworks-de-ia)
- [17. Implantação](#17-implantação)
- [18. Exemplo de configuração](#18-exemplo-de-configuração)
- [19. Exemplo de cliente Python](#19-exemplo-de-cliente-python)
- [20. Boas práticas](#20-boas-práticas)
- [21. Arquitetura de referência](#21-arquitetura-de-referência)
- [22. Roadmap de implementação](#22-roadmap-de-implementação)
- [23. Conclusão](#23-conclusão)
- [24. Referências](#24-referências)

---

## 1. Introdução

A adoção de IA Generativa em ambientes corporativos está evoluindo de experimentos isolados para plataformas compostas por **LLMs, RAG, agentes, ferramentas, APIs e serviços de dados**.

Nesse cenário, conectar cada aplicação diretamente a um fornecedor de LLM cria acoplamento e dificulta:

- substituição de modelos;
- uso de múltiplos provedores;
- controle de custos;
- aplicação de políticas;
- rate limiting;
- observabilidade;
- auditoria;
- alta disponibilidade;
- operação de agentes em escala.

O conceito de **AI Gateway** surge para resolver parte desse problema.

O [LiteLLM](https://www.litellm.ai/) pode atuar como uma camada central entre aplicações e modelos, oferecendo uma interface unificada e recursos de **routing, fallback, autenticação, governança, controle de gastos, logging e observabilidade**.

A proposta arquitetural pode ser resumida em:

```text
Aplicações / Agents
        |
        v
+-------------------------+
|       LiteLLM           |
|       AI Gateway        |
+-------------------------+
        |
        +--------+--------+--------+
        |        |        |        |
        v        v        v        v
     OpenAI   Anthropic Google   Local
```

---

## 2. O problema do acesso direto aos LLMs

Em uma arquitetura simples, uma aplicação pode chamar diretamente um provedor:

```text
Aplicação
    |
    v
OpenAI API
```

Quando surgem novos modelos:

```text
Aplicação
   |
   +---- OpenAI
   |
   +---- Anthropic
   |
   +---- Google
   |
   +---- Azure OpenAI
   |
   +---- AWS Bedrock
   |
   +---- Ollama
```

A aplicação passa a incorporar detalhes específicos de cada fornecedor.

### Principais problemas

#### Acoplamento

O código depende de APIs e configurações específicas.

#### Gestão de credenciais

As aplicações precisam lidar com múltiplas chaves e secrets.

#### Falta de governança central

Cada aplicação pode utilizar modelos, limites e budgets diferentes.

#### Observabilidade fragmentada

Os dados de consumo ficam distribuídos em vários sistemas.

#### Falta de resiliência

A indisponibilidade de um provedor pode impactar diretamente a aplicação.

#### Complexidade operacional

Quanto maior o número de aplicações, agentes e modelos, maior a complexidade.

---

## 3. O que é um AI Gateway

Um **AI Gateway** é uma camada intermediária especializada em controlar o tráfego entre consumidores de IA e provedores/modelos.

Conceitualmente:

```text
                  +----------------+
                  | Applications   |
                  | RAG / Agents   |
                  +-------+--------+
                          |
                          v
                  +---------------+
                  |   AI Gateway  |
                  +-------+-------+
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
           Provider A  Provider B  Provider C
```

O Gateway pode concentrar:

- autenticação;
- autorização;
- roteamento;
- retries;
- fallback;
- rate limiting;
- budgets;
- tracking de consumo;
- logging;
- observabilidade;
- políticas operacionais.

### AI Gateway não substitui o LLM

O Gateway é uma **camada de infraestrutura e controle**.

Ele não deve ser confundido com:

- modelo fundacional;
- banco vetorial;
- framework RAG;
- framework de agentes;
- ferramenta de treinamento de modelos.

---

## 4. O que é o LiteLLM

O LiteLLM é um projeto open source que oferece uma interface unificada para trabalhar com diferentes provedores e modelos de LLM.

Ele pode ser utilizado principalmente de duas formas.

### 4.1 LiteLLM SDK

A aplicação importa a biblioteca e realiza chamadas aos modelos.

```python
from litellm import completion

response = completion(
    model="openai/gpt-5",
    messages=[
        {
            "role": "user",
            "content": "Explique o conceito de RAG."
        }
    ]
)

print(response)
```

### 4.2 LiteLLM Proxy / AI Gateway

O LiteLLM é executado como um serviço independente.

```text
+-------------+
| Application |
+------+------+
       |
       v
+-------------+
|   LiteLLM   |
|   Gateway   |
+------+------+
       |
       +---- OpenAI
       +---- Gemini
       +---- Anthropic
       +---- Azure
       +---- Bedrock
       +---- Local LLM
```

Essa segunda abordagem é especialmente interessante em ambientes corporativos porque cria uma **camada compartilhada de infraestrutura de IA**.

---

## 5. Arquitetura

Uma arquitetura básica:

```text
                           CLIENTS
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
           Web App          RAG            Agents
              |               |               |
              +---------------+---------------+
                              |
                              v
                    +-------------------+
                    |     LiteLLM       |
                    |    AI Gateway     |
                    +---------+---------+
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
          OpenAI          Anthropic         Google
             |                |                |
             v                v                v
           GPT              Claude           Gemini
```

Uma arquitetura corporativa pode adicionar:

```text
                   Identity / SSO
                         |
                         v
Applications --> API Gateway --> LiteLLM
                                  |
                +-----------------+----------------+
                |                 |                |
                v                 v                v
             Routing          Governance       Observability
                |                 |                |
                +-----------------+----------------+
                                  |
                         Model Providers
```

---

## 6. Principais recursos

O LiteLLM pode ser utilizado para centralizar diversos aspectos da operação de LLMs.

| Categoria | Exemplos |
|---|---|
| Integração | Interface unificada |
| Routing | Seleção de deployments/modelos |
| Resiliência | Retry e fallback |
| Segurança | API Keys, autenticação e autorização |
| Governança | Budgets, equipes e projetos |
| Controle | Rate limits |
| Custos | Tracking de consumo |
| Observabilidade | Logs, métricas e callbacks |
| Operação | Proxy Server |
| Integração | Frameworks, ferramentas e aplicações |

A disponibilidade exata de recursos depende da versão instalada e da configuração adotada. Consulte sempre a documentação oficial.

---

## 7. LLM Routing

O **routing** é um dos conceitos centrais.

Em vez de determinar rigidamente:

```text
Aplicação -> GPT
```

podemos utilizar:

```text
Aplicação
    |
    v
 LiteLLM Router
    |
    +---- Modelo A
    +---- Modelo B
    +---- Modelo C
```

O roteamento pode considerar fatores como:

- modelo solicitado;
- deployment;
- disponibilidade;
- prioridade;
- capacidade;
- latência;
- custo;
- limites de utilização.

### Exemplo conceitual

```text
Request
   |
   v
+------------------+
| Routing Policy   |
+--------+---------+
         |
   +-----+-----+
   |     |     |
   v     v     v
  GPT  Gemini Claude
```

### Estratégia por tarefa

```text
Pergunta simples
      |
      v
Modelo econômico

Pergunta complexa
      |
      v
Modelo avançado

Tarefa de código
      |
      v
Modelo especializado
```

O objetivo é evitar a utilização indiscriminada do modelo mais caro.

---

## 8. Fallback e resiliência

Uma arquitetura sem fallback:

```text
Application
     |
     v
 Provider A
     X
  FAILURE
```

Com fallback:

```text
Application
     |
     v
  LiteLLM
     |
     v
 Provider A
     X
     |
     v
 Provider B
     |
     v
 Response
```

Isso permite desenvolver estratégias para situações como:

- indisponibilidade;
- timeout;
- erro de provider;
- limite de utilização;
- falhas transitórias.

### Importante

Fallback não significa simplesmente trocar um modelo por outro.

É necessário avaliar:

- compatibilidade;
- capacidade;
- contexto;
- qualidade;
- custo;
- requisitos funcionais.

---

## 9. Multi-Cloud e Multi-Model

O Gateway pode funcionar como uma camada de abstração entre aplicações e diferentes ambientes.

```text
                         LiteLLM
                            |
          +-----------------+-----------------+
          |                 |                 |
          v                 v                 v
       Cloud A           Cloud B           On-Prem
          |                 |                 |
       Models            Models            Models
```

Isso é particularmente relevante para organizações que possuem:

- estratégia multi-cloud;
- requisitos de soberania;
- modelos privados;
- workloads locais;
- ambientes híbridos.

### Exemplo

```text
                     Application
                          |
                       LiteLLM
                          |
             +------------+------------+
             |            |            |
             v            v            v
          Azure         AWS         Local GPU
             |            |            |
          Models        Models        Models
```

---

## 10. Governança e controle de acesso

Uma plataforma corporativa precisa estabelecer políticas.

### Perguntas fundamentais

- Quem pode utilizar o Gateway?
- Qual equipe pode acessar determinado modelo?
- Qual é o budget?
- Qual é o limite de requisições?
- Quais aplicações podem utilizar modelos premium?
- Como auditar o consumo?

Uma estrutura conceitual:

```text
Organization
     |
     +---- Team A
     |       |
     |       +---- API Keys
     |       +---- Budget
     |
     +---- Team B
     |       |
     |       +---- API Keys
     |       +---- Budget
     |
     +---- Team C
             |
             +---- API Keys
             +---- Budget
```

### Princípio recomendado

> A aplicação não deveria possuir liberdade irrestrita para utilizar qualquer modelo em produção.

As políticas devem ser centralizadas sempre que possível.

---

## 11. Gestão de custos

O custo de IA depende de fatores como:

- modelo;
- tokens de entrada;
- tokens de saída;
- volume de requisições;
- contexto enviado;
- retries;
- chamadas de ferramentas.

Exemplo:

```text
Projeto A
   |
   +-- GPT       $300
   +-- Gemini    $100
   +-- Claude    $150
              -------
                $550
```

O Gateway pode ajudar a estruturar a visibilidade do consumo por:

- usuário;
- equipe;
- projeto;
- modelo;
- aplicação.

### FinOps para IA

Uma estratégia madura deve acompanhar:

```text
Custo
  |
  +---- por modelo
  +---- por aplicação
  +---- por usuário
  +---- por equipe
  +---- por projeto
```

---

## 12. Observabilidade

Observabilidade é essencial para aplicações de IA.

Uma plataforma GenAI deve responder:

```text
Quem chamou?
Qual modelo?
Qual provider?
Quanto tempo demorou?
Quantos tokens?
Quanto custou?
Houve erro?
Houve retry?
Houve fallback?
```

### Arquitetura

```text
                       LiteLLM
                          |
          +---------------+---------------+
          |               |               |
          v               v               v
        Logs           Metrics          Traces
          |               |               |
          +---------------+---------------+
                          |
                          v
                 Observability Stack
```

O ecossistema LiteLLM possui integrações/callbacks para diferentes ferramentas de observabilidade, incluindo soluções como Langfuse, MLflow, Helicone e Traceloop.

### OpenTelemetry

Em arquiteturas maiores, OpenTelemetry pode ser utilizado como parte da estratégia de telemetria:

```text
Application
     |
LiteLLM
     |
OpenTelemetry
     |
+----+---------+
|              |
v              v
Metrics       Traces
```

---

## 13. LiteLLM + RAG

RAG significa **Retrieval-Augmented Generation**.

Uma arquitetura típica:

```text
              User Query
                   |
                   v
              Retriever
                   |
          +--------+--------+
          |                 |
          v                 v
     Vector Search       BM25
          |                 |
          +--------+--------+
                   |
                   v
                Context
                   |
                   v
                LiteLLM
                   |
                   v
                  LLM
                   |
                   v
               Response
```

O LiteLLM não substitui:

- vector database;
- mecanismo de busca;
- chunking;
- embeddings;
- reranking.

Ele atua principalmente na camada de acesso aos modelos.

### Exemplo de stack

```text
RAG Application
      |
LangChain / LlamaIndex
      |
Retriever
      |
PGVector / FAISS / Chroma
      |
Context
      |
LiteLLM
      |
LLM
```

---

## 14. LiteLLM + Agents

Um agente normalmente combina:

```text
LLM
+
Tools
+
Memory
+
Planning / Reasoning
+
Context
```

Uma arquitetura:

```text
                 User
                   |
                   v
                Agent
                   |
                   v
                LiteLLM
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
     GPT         Claude      Gemini
```

O Gateway cria uma camada de abstração entre o agente e os modelos.

### Benefício

O Agent pode permanecer relativamente desacoplado do fornecedor.

Isso facilita:

- troca de modelo;
- testes A/B;
- fallback;
- controle de custos;
- políticas centralizadas.

---

## 15. LiteLLM + MCP

**Model Context Protocol (MCP)** é um protocolo para integração de modelos/agentes com ferramentas e fontes de contexto.

Um agente pode utilizar:

```text
Agent
 |
 +---- GitHub
 |
 +---- Jira
 |
 +---- GitLab
 |
 +---- Database
 |
 +---- APIs
 |
 +---- Sistemas corporativos
```

A página de AI Gateway do LiteLLM posiciona o produto também em cenários envolvendo **MCP e Agents**.

Arquitetura conceitual:

```text
                         Agent
                           |
                           v
                       LiteLLM
                           |
                      MCP Gateway
                           |
            +--------------+--------------+
            |              |              |
            v              v              v
         GitHub          Jira          Database
```

### Governança

A camada de Gateway pode ser utilizada para organizar:

- autenticação;
- autorização;
- políticas;
- acesso às ferramentas;
- observabilidade.

---

## 16. LiteLLM + frameworks de IA

O LiteLLM pode ser integrado a arquiteturas que utilizam frameworks como:

- LangChain;
- LangGraph;
- LlamaIndex;
- aplicações Python;
- aplicações via APIs compatíveis.

Uma arquitetura de agentes:

```text
                 LangGraph
                     |
          +----------+----------+
          |                     |
          v                     v
       Agent A               Agent B
          |                     |
          +----------+----------+
                     |
                     v
                  LiteLLM
                     |
        +------------+------------+
        |            |            |
        v            v            v
      OpenAI      Anthropic     Google
```

O princípio arquitetural é separar:

```text
Orquestração
     |
     v
AI Gateway
     |
     v
Modelos
```

---

## 17. Implantação

O LiteLLM pode ser utilizado em diferentes estratégias de implantação.

### Desenvolvimento local

```text
Developer
    |
localhost
    |
LiteLLM
    |
LLM
```

### Docker

```text
Docker
  |
  +---- LiteLLM
  |
  +---- PostgreSQL
  |
  +---- Redis
```

### Kubernetes

```text
                Kubernetes
                     |
        +------------+------------+
        |            |            |
        v            v            v
     LiteLLM      LiteLLM      LiteLLM
        |            |            |
        +------------+------------+
                     |
                Load Balancer
                     |
        +------------+------------+
        |            |            |
        v            v            v
     OpenAI       Azure          AWS
```

### Ambientes corporativos

Pode ser considerado em:

- on-premises;
- cloud;
- multi-cloud;
- Kubernetes;
- ambientes híbridos;
- ambientes com requisitos específicos de segurança.

---

## 18. Exemplo de configuração

Um `config.yaml` pode definir modelos/deployments.

Exemplo didático:

```yaml
model_list:

  - model_name: gpt
    litellm_params:
      model: openai/gpt-5
      api_key: os.environ/OPENAI_API_KEY

  - model_name: gemini
    litellm_params:
      model: gemini/gemini-2.5-pro
      api_key: os.environ/GEMINI_API_KEY

  - model_name: local
    litellm_params:
      model: ollama/llama3
      api_base: http://localhost:11434
```

A ideia é expor nomes lógicos:

```text
gpt
gemini
local
```

enquanto o Gateway mantém os detalhes de configuração dos providers.

> **Nota:** nomes de modelos, parâmetros, provedores e versões mudam com o tempo. Valide a sintaxe na documentação oficial da versão instalada.

---

## 19. Exemplo de cliente Python

Uma aplicação pode consumir o Gateway utilizando uma interface compatível com o padrão OpenAI.

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-demo",
    base_url="http://localhost:4000"
)

response = client.chat.completions.create(
    model="gpt",
    messages=[
        {
            "role": "user",
            "content": "Explique o conceito de AI Gateway."
        }
    ]
)

print(response.choices[0].message.content)
```

A arquitetura passa a ser:

```text
Python Application
        |
        v
OpenAI-compatible API
        |
        v
     LiteLLM
        |
        v
       LLM
```

### Benefício arquitetural

A aplicação conhece:

```text
http://localhost:4000
```

e um modelo lógico:

```text
gpt
```

O Gateway abstrai parte da complexidade de infraestrutura.

---

## 20. Boas práticas

### 20.1 Não exponha API Keys

Utilize:

- environment variables;
- secret managers;
- Kubernetes Secrets;
- vaults.

Evite:

```python
api_key="sk-real-key"
```

em código versionado.

---

### 20.2 Defina modelos lógicos

Prefira:

```text
model="production-chat"
```

a acoplar todas as aplicações a um deployment específico.

---

### 20.3 Crie políticas de fallback

Defina:

```text
Primary
   |
Failure
   |
Secondary
   |
Failure
   |
Tertiary
```

---

### 20.4 Monitore custos

Acompanhe:

- tokens;
- requests;
- custo;
- modelo;
- projeto;
- usuário.

---

### 20.5 Monitore qualidade

Não basta monitorar infraestrutura.

Avalie também:

- relevância;
- factualidade;
- taxa de erro;
- respostas inadequadas;
- qualidade do RAG;
- qualidade das ferramentas dos agentes.

---

### 20.6 Separe ambientes

Recomendação:

```text
Development
     |
Staging
     |
Production
```

Cada ambiente deve possuir configurações e credenciais apropriadas.

---

### 20.7 Trate o Gateway como infraestrutura

O Gateway deve ser incorporado ao ciclo de:

- DevOps;
- segurança;
- observabilidade;
- gestão de mudanças;
- backup;
- alta disponibilidade.

---

## 21. Arquitetura de referência

Uma arquitetura mais completa:

```text
                              USERS
                                |
                                v
                    +----------------------+
                    |   AI Applications     |
                    | Chat | RAG | Agents  |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |      API Gateway     |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |       LiteLLM        |
                    |      AI Gateway      |
                    +----------+-----------+
                               |
        +----------------------+----------------------+
        |                      |                      |
        v                      v                      v
   Authentication          Routing              Governance
        |                      |                      |
        +----------------------+----------------------+
                               |
                 +-------------+-------------+
                 |             |             |
                 v             v             v
              OpenAI       Anthropic       Google
                 |             |             |
                 v             v             v
               Models        Models        Models

                               |
                               v
                    +----------------------+
                    |    Observability     |
                    | Logs / Metrics /     |
                    | Traces / Cost        |
                    +----------------------+
```

### Camadas

| Camada | Responsabilidade |
|---|---|
| Application | Experiência e regras de negócio |
| RAG | Recuperação de contexto |
| Agent | Raciocínio e execução |
| API Gateway | Entrada externa |
| LiteLLM | Gateway de IA |
| Providers | Inferência |
| Observability | Monitoramento |
| Governance | Políticas e controle |

---

## 22. Roadmap de implementação

Uma adoção gradual pode seguir estas etapas.

### Fase 1 — PoC

```text
LiteLLM
   |
1 ou 2 modelos
```

Objetivos:

- validar integração;
- testar API;
- testar configuração.

### Fase 2 — Multi-Model

```text
LiteLLM
 |--- Provider A
 |--- Provider B
 |--- Local
```

Objetivos:

- routing;
- fallback;
- comparação de modelos.

### Fase 3 — Governança

Adicionar:

- API Keys;
- equipes;
- budgets;
- rate limits;
- autenticação.

### Fase 4 — Observabilidade

Adicionar:

- métricas;
- logs;
- traces;
- tracking de custos;
- Langfuse/OpenTelemetry ou solução equivalente.

### Fase 5 — Agents + MCP

Adicionar:

```text
Agents
  |
LiteLLM
  |
MCP
  |
Tools
```

### Fase 6 — Produção

Evoluir para:

- Kubernetes;
- alta disponibilidade;
- secrets management;
- CI/CD;
- observabilidade;
- segurança;
- disaster recovery;
- políticas de governança.

---

## 23. Conclusão

A adoção de múltiplos modelos de IA cria um novo problema arquitetural:

> **Como permitir que aplicações e agentes utilizem diferentes LLMs sem criar dependência excessiva de um único fornecedor?**

O conceito de AI Gateway responde a esse desafio introduzindo uma camada de abstração e controle.

O LiteLLM pode ocupar essa posição oferecendo recursos para:

- integração multi-modelo;
- routing;
- fallback;
- autenticação;
- governança;
- rate limiting;
- gestão de custos;
- observabilidade;
- integração com Agents;
- integração com MCP.

A arquitetura resultante:

```text
Application
     |
     v
    RAG
     |
     v
   Agent
     |
     v
 LiteLLM
     |
     +--------+---------+---------+
     |        |         |         |
     v        v         v         v
  OpenAI  Anthropic   Google    Local
```

permite separar claramente:

```text
Aplicação
    ≠
Orquestração
    ≠
AI Gateway
    ≠
Modelo
    ≠
Infraestrutura
```

Essa separação é fundamental para construir plataformas de IA Generativa **escaláveis, observáveis, governáveis e independentes de fornecedor**.

---

## 24. Referências

### LiteLLM

- **AI Gateway:** https://www.litellm.ai/ai-gateway
- **Documentação oficial:** https://docs.litellm.ai/
- **Engineering Blog:** https://docs.litellm.ai/blog
- **GitHub:** https://github.com/BerriAI/litellm

### Tecnologias relacionadas

- OpenAI API: https://platform.openai.com/docs/
- Anthropic: https://docs.anthropic.com/
- Google Gemini API: https://ai.google.dev/
- Model Context Protocol: https://modelcontextprotocol.io/
- LangChain: https://python.langchain.com/
- LangGraph: https://langchain-ai.github.io/langgraph/
- LlamaIndex: https://docs.llamaindex.ai/
- OpenTelemetry: https://opentelemetry.io/
- Langfuse: https://langfuse.com/

---

## Licença

Este material pode ser utilizado para fins educacionais e de estudo.

Ao reutilizar ou adaptar este conteúdo em projetos públicos, recomenda-se manter as referências às documentações oficiais e aos projetos open source utilizados como base.

---

**Tema:** IA Generativa • AI Gateway • LiteLLM • LLM Routing • Agents • MCP • RAG • Observability • Governance

**Material técnico e educacional**
