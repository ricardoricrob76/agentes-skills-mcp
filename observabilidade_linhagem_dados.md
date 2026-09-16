# Observabilidade e Linhagem de Dados em Aplicações de Agentic AI

> **Tutorial técnico para Engenheiros de IA, Arquitetos de Dados e Arquitetos de Soluções Integradas com IA Generativa**

![Agentic AI Observability](https://img.shields.io/badge/Agentic%20AI-Observability-blue)
![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-OTLP-orange)
![GenAI](https://img.shields.io/badge/GenAI-Tracing-purple)
![License](https://img.shields.io/badge/license-MIT-green)

## 1. Visão geral

Aplicações de **Agentic AI** introduzem uma cadeia de execução muito mais dinâmica do que uma aplicação tradicional:

```text
Usuário
   |
   v
[Orquestrador / Agente]
   |
   +--> LLM
   |
   +--> Memória
   |
   +--> RAG --> Embeddings --> Vector DB
   |
   +--> Tools --> APIs / Banco / MCP
   |
   +--> Agente especialista
   |
   +--> Validação / Guardrails
   |
   v
Resposta
```

Em produção, registrar somente `request`, `response` e `latency` não é suficiente. É necessário reconstruir **o caminho completo da decisão**, incluindo modelos utilizados, prompts, ferramentas, recuperação de documentos, contexto, tokens, custos, erros, avaliações e relações com os dados corporativos.

Este tutorial apresenta uma arquitetura de referência e compara alternativas atuais para observabilidade de aplicações LLM, RAG e agentes.

---

## 2. O problema: observar um agente é observar uma cadeia

Em uma API convencional, uma requisição normalmente possui um fluxo relativamente determinístico:

```text
HTTP Request
   |
   v
Service
   |
   v
Database
   |
   v
HTTP Response
```

Em um agente:

```text
Trace: atendimento-123
|
+-- Agent Run
|   |
|   +-- Planner
|   |
|   +-- LLM Call #1
|   |
|   +-- Tool: consultar_cliente
|   |     |
|   |     +-- PostgreSQL
|   |
|   +-- RAG Retrieval
|   |     |
|   |     +-- Embedding
|   |     +-- Vector DB
|   |     +-- documentos [A,B,C]
|   |
|   +-- LLM Call #2
|   |
|   +-- Guardrail
|
+-- Final Response
```

O objetivo da observabilidade é permitir responder:

- Qual agente executou?
- Qual modelo foi chamado?
- Qual prompt/contexto foi enviado?
- Quais ferramentas foram utilizadas?
- Qual documento originou a resposta?
- Qual consulta ao banco foi executada?
- Qual foi o tempo de cada etapa?
- Quantos tokens foram consumidos?
- Qual foi o custo?
- Onde ocorreu a falha?
- Qual versão do prompt, modelo e código estava em produção?
- Qual dado corporativo influenciou a resposta?
- A resposta foi avaliada como correta?
- Existe informação sensível no trace?

---

# 3. Observabilidade x Logging x Tracing x Linhagem

## 3.1 Logging

Registra eventos discretos:

```text
2026-09-16 08:30:12 INFO Tool=consulta_cliente
2026-09-16 08:30:13 INFO rows=4
```

É útil, mas perde parte do contexto da execução distribuída.

## 3.2 Metrics

Métricas agregadas:

```text
agent_requests_total
agent_errors_total
llm_tokens_total
llm_latency_seconds
rag_retrieval_latency
tool_calls_total
```

Excelente para dashboards e alertas.

## 3.3 Distributed Tracing

Relaciona todas as operações por uma mesma execução:

```text
Trace
 ├── Agent
 │    ├── LLM
 │    ├── Tool
 │    ├── Retrieval
 │    └── LLM
 └── Guardrail
```

## 3.4 Linhagem de dados

A linhagem responde **de onde veio o dado e por quais transformações passou**.

Exemplo:

```text
MTE.CAGED
   |
   v
ETL
   |
   v
Lakehouse / Iceberg
   |
   v
Feature / Dataset
   |
   v
RAG Index
   |
   v
Retriever
   |
   v
Agent
   |
   v
LLM
   |
   v
Resposta
```

Para Agentic AI, é interessante combinar:

```text
Observabilidade da execução
            +
Linhagem dos dados
            +
Observabilidade de IA
            +
Avaliação de qualidade
```

---

# 4. OpenTelemetry como camada de interoperabilidade

Uma arquitetura recomendada é separar **instrumentação** de **backend de observabilidade**.

```text
                 +-----------------------+
                 | Agentic AI Application|
                 +-----------+-----------+
                             |
                     OpenTelemetry
                             |
                             v
                 +-----------------------+
                 | OTel Collector        |
                 +-----------+-----------+
                             |
             +---------------+---------------+
             |               |               |
             v               v               v
          Phoenix         Langfuse        MLflow
             |               |               |
             +---------------+---------------+
                             |
                     Data / AI Platform
```

O **OpenTelemetry Collector** é uma camada vendor-neutral para receber, processar e exportar telemetria. Isso permite evitar acoplamento excessivo a uma única ferramenta de observabilidade.

Em uma arquitetura corporativa, o Collector pode atuar como:

- receiver;
- processor;
- sampler;
- filter;
- enrichment;
- router;
- exporter.

---

# 5. O que deve ser observado em um Agent

## 5.1 Identidade da execução

Cada execução deve possuir:

```text
trace_id
span_id
parent_span_id
session_id
user_id/hash
tenant_id
environment
application
agent_name
agent_version
```

## 5.2 LLM

Registrar, quando permitido:

```text
provider
model
model_version
temperature
input_tokens
output_tokens
latency
finish_reason
request_id
cost
```

Evite armazenar prompts/respostas com dados pessoais ou segredos sem uma política explícita.

## 5.3 Tool Calling

Exemplo:

```json
{
  "tool": "consultar_contrato",
  "operation": "get_contract",
  "latency_ms": 82,
  "status": "success"
}
```

## 5.4 RAG

Registrar:

```text
query
retriever
embedding_model
vector_database
top_k
document_ids
scores
reranker
retrieval_latency
```

A linhagem pode ser representada como:

```text
Pergunta
   |
   v
Embedding
   |
   v
Vector Search
   |
   +--> Documento 123
   +--> Documento 456
   +--> Documento 789
   |
   v
Reranking
   |
   v
Context
   |
   v
LLM
```

## 5.5 Agent-to-Agent

Para sistemas multiagentes:

```text
Supervisor
 |
 +--> Agent Financeiro
 |      |
 |      +--> Tool SQL
 |
 +--> Agent Jurídico
 |      |
 |      +--> RAG
 |
 +--> Agent Auditor
```

Cada agente deve ser representado por spans relacionados ao mesmo trace ou por traces correlacionados.

---

# 6. Semântica de telemetria para GenAI

Uma das principais tendências é utilizar convenções semânticas específicas para GenAI sobre OpenTelemetry.

A ideia é padronizar atributos como:

```text
gen_ai.system
gen_ai.request.model
gen_ai.response.id
gen_ai.response.finish_reasons
gen_ai.usage.input_tokens
gen_ai.usage.output_tokens
```

Para aplicações agentic, é interessante adicionar atributos de domínio:

```text
agent.name
agent.version
agent.role
agent.task
tool.name
tool.version
rag.query
rag.top_k
rag.document_ids
rag.retriever
evaluation.score
dataset.version
prompt.version
```

> **Recomendação:** use atributos padronizados sempre que existirem e atributos próprios com namespace claro para informações específicas do domínio.

---

# 7. Arquitetura de referência

```text
                         +----------------------+
                         |      Usuário         |
                         +----------+-----------+
                                    |
                                    v
                         +----------------------+
                         | API / Gateway        |
                         +----------+-----------+
                                    |
                                    v
                  +--------------------------------------+
                  |         Agentic AI Runtime            |
                  |                                      |
                  | Agent / Workflow / Graph             |
                  |       |                              |
                  |       +---- LLM                      |
                  |       +---- Memory                   |
                  |       +---- RAG                      |
                  |       +---- Tools                    |
                  |       +---- MCP                      |
                  |       +---- Guardrails               |
                  +------------------+-------------------+
                                     |
                           OpenTelemetry SDK
                                     |
                                     v
                         +----------------------+
                         | OTel Collector       |
                         +----------+-----------+
                                    |
             +----------------------+---------------------+
             |                      |                     |
             v                      v                     v
        Trace Backend          Metrics Backend       Log Backend
             |                      |                     |
             +----------------------+---------------------+
                                    |
                                    v
                         +----------------------+
                         | AI Observability     |
                         | Evaluation           |
                         | Lineage              |
                         +----------------------+
```

---

# 8. Principais alternativas

## 8.1 OpenTelemetry

**Categoria:** padrão/protocolo de observabilidade.

### Pontos positivos

- Vendor-neutral.
- Suporta traces, metrics e logs.
- Integração com múltiplos backends.
- Excelente para ambientes corporativos.
- Facilita arquitetura híbrida/multicloud.
- Reduz lock-in.

### Pontos negativos

- Não é uma plataforma completa de AI observability.
- Requer configuração e arquitetura.
- A semântica GenAI ainda exige atenção às versões e integrações.
- Avaliação de qualidade normalmente precisa de ferramentas adicionais.

**Indicação:** camada de interoperabilidade.

---

## 8.2 Arize Phoenix

**Categoria:** AI observability open source.

O Phoenix utiliza OpenTelemetry e OpenInference para observar aplicações LLM e agentes, incluindo chamadas de modelo, retrieval, ferramentas e embeddings.

### Pontos positivos

- Open source.
- Pode ser executado localmente.
- Forte foco em AI/LLM.
- Tracing + evaluation.
- Bom suporte para RAG.
- Integrações com diversos frameworks.

### Pontos negativos

- Mais especializado em AI observability do que em observabilidade corporativa generalista.
- Pode exigir arquitetura complementar para métricas/logs corporativos.
- Em ambientes grandes, é necessário avaliar operação, retenção e governança.

**Indicação:** laboratório, engenharia de IA, RAG e avaliação.

---

## 8.3 Langfuse

**Categoria:** LLM/Agent observability open source.

O Langfuse permite capturar traces, prompts, tokens, latência, custos e etapas intermediárias de aplicações LLM.

### Pontos positivos

- Open source.
- Self-hosting.
- Interface orientada a LLM.
- Bom suporte a prompts e tracing.
- Métricas de custo e tokens.
- Adequado a aplicações RAG e agentes.

### Pontos negativos

- Mais focado em LLM applications do que em observabilidade completa de infraestrutura.
- Pode demandar integração adicional com OpenTelemetry e plataformas corporativas.
- Governança e operação ficam sob responsabilidade do time quando self-hosted.

**Indicação:** equipes que desejam controle da plataforma de observabilidade de GenAI.

---

## 8.4 LangSmith

**Categoria:** plataforma de observabilidade e engenharia de aplicações LangChain.

### Pontos positivos

- Excelente integração com LangChain/LangGraph.
- Tracing muito simples.
- Debug de agentes.
- Avaliação.
- Monitoramento.
- Dashboards e automações.
- Forte integração com o ecossistema LangChain.

### Pontos negativos

- Forte associação ao ecossistema LangChain.
- Dependência de serviço/plataforma pode ser uma consideração em ambientes com restrições de dados.
- Para arquitetura totalmente vendor-neutral, OpenTelemetry tende a ser uma camada mais adequada.

**Indicação:** projetos LangChain/LangGraph e equipes que priorizam produtividade.

---

## 8.5 MLflow Tracing

**Categoria:** plataforma de ML/GenAI + tracing.

MLflow Tracing é compatível com OpenTelemetry e possui integrações para diversos frameworks de agentes e LLMs.

### Pontos positivos

- Open source.
- Ecossistema consolidado de ML.
- Tracking + tracing + evaluation.
- Integração com OpenTelemetry.
- Bom encaixe em ambientes MLOps.
- Amplo conjunto de integrações.

### Pontos negativos

- Plataforma relativamente ampla.
- Pode ser mais complexa que uma solução exclusivamente focada em tracing.
- Requer arquitetura adequada para separar experimentação, tracking e observabilidade operacional.

**Indicação:** organizações que já utilizam MLflow/MLOps.

---

## 8.6 OpenLIT

**Categoria:** AI observability OpenTelemetry-native.

### Pontos positivos

- Open source.
- Self-hostable.
- Construído sobre OpenTelemetry.
- Tracing, metrics, logs e exceptions.
- Foco em LLMs e agentes.
- Integração com diversos frameworks.

### Pontos negativos

- Ecossistema menor que plataformas mais maduras.
- Deve ser avaliado quanto à escala e integração com o stack corporativo.
- Pode exigir componentes adicionais para data lineage empresarial.

**Indicação:** equipes que querem uma solução OpenTelemetry-native para AI observability.

---

# 9. Comparativo

| Solução | Open Source | OTel | AI Tracing | RAG | Agents | Evaluation | Self-host | Foco |
|---|---:|---:|---:|---:|---:|---:|---:|---|
| OpenTelemetry | Sim | Nativo | Base | Indireto | Indireto | Não | Sim | Observabilidade |
| Phoenix | Sim | Sim | Excelente | Excelente | Excelente | Excelente | Sim | AI Observability |
| Langfuse | Sim | Sim | Excelente | Sim | Sim | Sim | Sim | LLM Observability |
| LangSmith | Parcial/serviço | Sim | Excelente | Excelente | Excelente | Sim | Limitado | LangChain |
| MLflow | Sim | Sim | Excelente | Sim | Excelente | Excelente | Sim | MLOps/GenAI |
| OpenLIT | Sim | Nativo | Excelente | Sim | Sim | Sim | Sim | AI Observability |

> A tabela é uma comparação arquitetural qualitativa, não um ranking. Capacidades e integrações mudam rapidamente; valide versões e licenciamento antes de uma decisão de produção.

---

# 10. Linhagem de dados em Agentic AI

A observabilidade da execução não substitui Data Lineage.

Uma arquitetura corporativa pode combinar:

```text
                   DATA LINEAGE
                       |
                       v
+---------+      +-----------+      +------------+
| Sources | ---> | Lakehouse | ---> | RAG Index  |
+---------+      +-----------+      +------------+
                                          |
                                          v
                                    +-----------+
                                    | Retriever |
                                    +-----+-----+
                                          |
                         TRACE            |
                           |              v
                           +--------> Agent
                                      |
                          +-----------+-----------+
                          |           |           |
                         LLM        Tool        Memory
                          |           |           |
                          +-----------+-----------+
                                      |
                                      v
                                  Response
```

O ideal é conseguir navegar:

```text
Resposta
   |
   v
Trace
   |
   v
Agent Run
   |
   v
RAG Retrieval
   |
   v
Document ID
   |
   v
Dataset
   |
   v
Tabela
   |
   v
Fonte original
```

Esse modelo permite investigação de incidentes e auditoria.

---

# 11. Exemplo de implementação com OpenTelemetry

## Instalação

```bash
pip install opentelemetry-api
pip install opentelemetry-sdk
pip install opentelemetry-exporter-otlp
```

## Instrumentação básica

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter

provider = TracerProvider()

exporter = OTLPSpanExporter(
    endpoint="http://localhost:4318/v1/traces"
)

provider.add_span_processor(
    BatchSpanProcessor(exporter)
)

trace.set_tracer_provider(provider)

tracer = trace.get_tracer("agentic-ai")

with tracer.start_as_current_span("agent.run") as span:

    span.set_attribute("agent.name", "assistente-corporativo")
    span.set_attribute("agent.version", "1.0.0")
    span.set_attribute("agent.task", "consulta")

    with tracer.start_as_current_span("tool.consultar_cliente") as tool_span:

        tool_span.set_attribute(
            "tool.name",
            "consultar_cliente"
        )

        # execução da ferramenta
```

---

# 12. Trace hierárquico para RAG

Uma implementação conceitual:

```python
with tracer.start_as_current_span("agent.run"):

    with tracer.start_as_current_span("rag.retrieve"):

        query = "Qual o status do benefício?"

        with tracer.start_as_current_span("embedding"):

            # gerar embedding
            pass

        with tracer.start_as_current_span("vector.search"):

            # buscar documentos
            documents = ["doc-123", "doc-456"]

        with tracer.start_as_current_span("reranking"):

            # reranking
            pass

    with tracer.start_as_current_span("llm.generate"):

        # chamada ao modelo
        pass
```

Visualmente:

```text
agent.run
|
+-- rag.retrieve
|   |
|   +-- embedding
|   +-- vector.search
|   +-- reranking
|
+-- llm.generate
```

---

# 13. OpenTelemetry Collector

Exemplo conceitual de `otel-collector.yaml`:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  batch:

exporters:
  debug:
    verbosity: normal

service:
  pipelines:
    traces:
      receivers:
        - otlp
      processors:
        - batch
      exporters:
        - debug
```

Execução:

```bash
otelcol-contrib --config otel-collector.yaml
```

Arquitetura:

```text
Application
     |
     | OTLP
     v
OTel Collector
     |
     +----> Phoenix
     |
     +----> Langfuse
     |
     +----> MLflow
     |
     +----> Grafana/Tempo
     |
     +----> outro backend
```

---

# 14. Observabilidade em ambiente Kubernetes

Em produção:

```text
                 Kubernetes
+------------------------------------------+
|                                          |
|  Agent Pod                               |
|      |                                   |
|      +-- OTel SDK                        |
|                                          |
|  RAG Service                             |
|      |                                   |
|      +-- OTel SDK                        |
|                                          |
|  Tool Service                            |
|      |                                   |
|      +-- OTel SDK                        |
|                                          |
+-------------------+----------------------+
                    |
                    v
             OTel Collector
                    |
                    +----> Trace Backend
                    +----> Metrics
                    +----> Logs
```

É recomendável utilizar:

- OpenTelemetry Operator;
- Collector Gateway;
- Kubernetes metadata enrichment;
- sampling;
- centralized secrets;
- TLS/mTLS;
- RBAC;
- network policies.

---

# 15. Observabilidade de MCP

Em arquiteturas com MCP, o trace deve capturar:

```text
Agent
 |
 +-- MCP Client
       |
       +-- MCP Server
             |
             +-- Tool
             |
             +-- Resource
             |
             +-- Database/API
```

Atributos recomendados:

```text
mcp.server
mcp.tool
mcp.resource
mcp.operation
mcp.request.id
mcp.latency
mcp.status
```

Nunca registre automaticamente:

```text
API Keys
Tokens
Passwords
Secrets
Credentials
Dados pessoais desnecessários
```

---

# 16. Observabilidade Multi-Agent

Uma arquitetura:

```text
                    Supervisor
                        |
          +-------------+-------------+
          |             |             |
          v             v             v
       Agent A       Agent B       Agent C
          |             |             |
        Tools         RAG          APIs
          |             |             |
          +-------------+-------------+
                        |
                        v
                     Final
```

Use:

```text
trace_id = execução completa

span_id = operação

agent.id = agente

agent.role = função

parent_span_id = relacionamento
```

Isso permite reconstruir a árvore de execução.

---

# 17. Métricas recomendadas

## Runtime

```text
request_count
error_rate
p50_latency
p95_latency
p99_latency
```

## LLM

```text
input_tokens
output_tokens
total_tokens
cost
model_latency
model_error_rate
```

## RAG

```text
retrieval_latency
top_k
retrieval_score
reranking_latency
context_size
```

## Agent

```text
tool_calls
agent_steps
agent_retries
handoffs
loop_count
max_iterations
```

## Qualidade

```text
faithfulness
answer_relevance
context_relevance
tool_accuracy
human_feedback
guardrail_failures
```

---

# 18. Golden Signals para Agentic AI

Adapte os Golden Signals tradicionais:

| Sinal | Agentic AI |
|---|---|
| Latency | Latência total + LLM + tools + RAG |
| Traffic | Requests + agent runs |
| Errors | HTTP + LLM + tools + guardrails |
| Saturation | CPU + GPU + tokens + context window |
| Cost | tokens + chamadas + infraestrutura |
| Quality | avaliações e feedback |
| Retrieval | relevância + recall + scores |
| Agent | loops + retries + tool success |

---

# 19. Segurança e privacidade

Observabilidade pode criar um novo vetor de risco.

Exemplo:

```text
LLM Input
   |
   +--> CPF
   +--> Nome
   +--> Salário
   +--> Contrato
   |
   v
Trace
```

Se o trace armazenar o payload integralmente:

```text
DADO SENSÍVEL
      |
      v
OBSERVABILIDADE
      |
      v
BACKEND
```

Portanto, implemente:

### Redaction

```python
def redact(value: str) -> str:
    return "[REDACTED]"
```

### Hashing

```text
user_id
   |
   v
SHA-256
   |
   v
trace attribute
```

### Filtering

Não envie para o backend:

```text
password
authorization
api_key
cookie
secret
token
```

### Sampling

Nem todo trace precisa ser armazenado integralmente.

```text
Production
   |
   +-- 100% errors
   +-- 100% critical agents
   +-- 10% success
   +-- 1% low-value traffic
```

---

# 20. Estratégia de adoção

## Nível 1 — Basic

```text
Application
   |
   +-- Logs
   +-- Metrics
```

## Nível 2 — Distributed Tracing

```text
Application
   |
   +-- OpenTelemetry
         |
         +-- Collector
```

## Nível 3 — AI Observability

```text
OTel
 |
 +-- LLM traces
 +-- RAG
 +-- Tools
 +-- Agents
 +-- Tokens
```

## Nível 4 — Evaluation

```text
Trace
 |
 +-- Quality
 +-- RAG evaluation
 +-- Agent evaluation
 +-- Human feedback
```

## Nível 5 — Data Lineage

```text
Source
  |
  v
Lakehouse
  |
  v
Dataset
  |
  v
RAG
  |
  v
Agent
  |
  v
Response
```

## Nível 6 — Governance

```text
Observability
+
Lineage
+
Security
+
Evaluation
+
Cost
+
Audit
```

---

# 21. Arquitetura corporativa recomendada

Para ambientes corporativos, uma arquitetura desacoplada pode ser:

```text
                       +--------------------+
                       | Agentic AI Apps    |
                       +---------+----------+
                                 |
                       OpenTelemetry SDK
                                 |
                                 v
                       +--------------------+
                       | OTel Collector     |
                       +---------+----------+
                                 |
          +----------------------+----------------------+
          |                      |                      |
          v                      v                      v
     AI Tracing             Infra Metrics          Logs
          |                      |                      |
          v                      v                      v
   Phoenix/Langfuse        Prometheus             Loki
   /MLflow/etc.            /Grafana                /ELK
          |
          v
   +-------------------+
   | Evaluation        |
   | Quality           |
   +-------------------+
          |
          v
   +-------------------+
   | Data Lineage      |
   | Catalog           |
   +-------------------+
```

O principal princípio arquitetural é:

> **Instrumentar uma vez, transportar por padrões abertos e permitir múltiplos consumidores de telemetria.**

---

# 22. Como escolher a ferramenta

Não existe uma ferramenta universalmente superior.

Use critérios:

| Critério | Pergunta |
|---|---|
| OpenTelemetry | Precisa ser vendor-neutral? |
| Self-host | Dados não podem sair da organização? |
| Framework | LangChain/LangGraph ou stack heterogênea? |
| RAG | Precisa visualizar retrieval em detalhe? |
| Agents | Precisa reconstruir workflows multiagentes? |
| Evaluation | Precisa avaliar qualidade automaticamente? |
| MLOps | Já existe MLflow? |
| Kubernetes | Precisa integrar ao cluster? |
| Data lineage | Precisa rastrear origem corporativa? |
| Compliance | Existe LGPD/política interna? |
| Cost | Qual volume de traces será armazenado? |

---

# 23. Recomendação arquitetural por cenário

### Stack LangChain/LangGraph

```text
LangGraph
   |
   v
LangSmith
```

ou

```text
LangGraph
   |
   v
OpenTelemetry
   |
   v
Phoenix / Langfuse / MLflow
```

### Stack corporativo multi-framework

```text
Agents
 |
OpenTelemetry
 |
OTel Collector
 |
+-- Phoenix
+-- MLflow
+-- Grafana
+-- SIEM
```

### Ambiente MLOps

```text
MLflow
+
OpenTelemetry
+
Data Lineage
+
Evaluation
```

### Ambiente altamente self-hosted

```text
OpenTelemetry
+
Collector
+
Phoenix/Langfuse/OpenLIT
+
Kubernetes
+
PostgreSQL/ClickHouse/etc.
```

---

# 24. Checklist de produção

## Instrumentação

- [ ] Trace ID
- [ ] Span ID
- [ ] Correlation ID
- [ ] Agent ID
- [ ] Agent version
- [ ] Model
- [ ] Prompt version
- [ ] Tool name
- [ ] RAG metadata
- [ ] Token usage

## Dados

- [ ] Document ID
- [ ] Dataset version
- [ ] Source system
- [ ] Table
- [ ] Partition
- [ ] Data product
- [ ] Lineage ID

## Segurança

- [ ] PII masking
- [ ] Secret filtering
- [ ] Encryption
- [ ] RBAC
- [ ] Retention policy
- [ ] Audit

## Operação

- [ ] Dashboards
- [ ] Alerts
- [ ] SLOs
- [ ] Sampling
- [ ] Cost monitoring
- [ ] Error tracking

## Qualidade

- [ ] Evaluation
- [ ] Golden dataset
- [ ] Regression tests
- [ ] Human feedback
- [ ] RAG evaluation
- [ ] Agent evaluation

---

# 25. Estrutura sugerida para um projeto GitHub

```text
agentic-ai-observability/
|
+-- README.md
|
+-- docs/
|   +-- architecture.md
|   +-- data-lineage.md
|   +-- security.md
|   +-- evaluation.md
|
+-- otel/
|   +-- collector.yaml
|
+-- app/
|   +-- agent.py
|   +-- rag.py
|   +-- tools.py
|   +-- telemetry.py
|
+-- dashboards/
|   +-- grafana/
|
+-- docker/
|   +-- docker-compose.yml
|
+-- examples/
|   +-- phoenix/
|   +-- langfuse/
|   +-- mlflow/
|
+-- tests/
|
+-- requirements.txt
```

---

# 26. Stack tecnológico de referência

Uma stack de laboratório pode ser:

```text
Python
FastAPI
LangGraph
OpenTelemetry
OTel Collector
Phoenix
PostgreSQL
Qdrant
Prometheus
Grafana
Docker Compose
```

Para produção:

```text
Kubernetes
+
OpenTelemetry Operator
+
OTel Collector Gateway
+
AI Observability Backend
+
Prometheus/Grafana
+
Data Catalog / Lineage
+
SIEM
+
Secrets Manager
```

---

# 27. Conclusão

Observabilidade em Agentic AI deve ser tratada como uma **capacidade arquitetural**, e não apenas como logging.

O modelo mais robusto é combinar:

```text
                 AGENTIC AI
                     |
       +-------------+-------------+
       |             |             |
       v             v             v
   Tracing        Metrics        Logs
       |             |             |
       +-------------+-------------+
                     |
              OpenTelemetry
                     |
       +-------------+-------------+
       |             |             |
       v             v             v
   AI Quality     Security      Cost
       |
       v
Data Lineage
       |
       v
Governance
```

Para ambientes corporativos heterogêneos, **OpenTelemetry como camada de interoperabilidade**, complementado por uma plataforma especializada em AI observability e por uma solução de Data Lineage, é um padrão arquitetural particularmente flexível.

Ferramentas como **Phoenix, Langfuse, LangSmith, MLflow e OpenLIT** podem ocupar papéis diferentes dependendo do framework, requisitos de self-hosting, governança, avaliação e integração com o ecossistema existente.

---

# 28. Referências

- OpenTelemetry — documentação oficial: https://opentelemetry.io/
- OpenTelemetry Collector: https://opentelemetry.io/docs/collector/
- Arize Phoenix: https://phoenix.arize.com/
- Phoenix GitHub: https://github.com/Arize-ai/phoenix
- Langfuse: https://langfuse.com/
- LangSmith: https://docs.langchain.com/langsmith/
- MLflow: https://mlflow.org/
- OpenLIT: https://github.com/openlit/openlit
- OpenInference: https://github.com/Arize-ai/openinference

---

## Autor

**Ricardo Roberto de Lima**

Mestre em Engenharia de Software | Engenheiro de IA | Arquiteto de Dados | Professor

Conteúdos e projetos relacionados a:

- Arquitetura de Dados
- Arquitetura de Soluções
- IA Generativa
- RAG
- Agentic AI
- MCP
- Data Engineering
- Data Governance
- Observabilidade
- Cloud e Multicloud

---
