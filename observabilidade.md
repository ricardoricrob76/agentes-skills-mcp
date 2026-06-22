# AI Observability em Produção: LangGraph + Langfuse + OpenTelemetry + Grafana para LLMOps

## Introdução

À medida que aplicações baseadas em Large Language Models (LLMs) evoluem para arquiteturas compostas por agentes, RAG, ferramentas externas e workflows complexos, torna-se fundamental implementar uma estratégia de **AI Observability**.

Em ambientes corporativos, a ausência de rastreabilidade pode comprometer:

* Governança dos modelos;
* Qualidade das respostas;
* Diagnóstico de falhas;
* Controle de custos;
* Auditoria;
* Escalabilidade.

Este artigo apresenta uma arquitetura moderna de **LLMOps**, combinando:

* LangGraph;
* Langfuse;
* OpenTelemetry;
* Prometheus;
* Grafana;
* PGVector;
* FastAPI;
* OpenAI.

---

# Arquitetura de Referência

```text
                                    Usuários
                                        │
                                        ▼
                              API Gateway / FastAPI
                                        │
                                        ▼
                                LangGraph Supervisor
                                        │
                ┌───────────────────────┼───────────────────────┐
                ▼                       ▼                       ▼
         Retrieval Agent          Search Agent             SQL Agent
                │                       │                       │
                ▼                       ▼                       ▼
            PGVector                  Tools                  PostgreSQL
                │                       │
                └──────────────┬────────┘
                               ▼
                           OpenAI GPT-4o
                               │
                               ▼
                          Langfuse Tracing
                               │
             ┌─────────────────┼──────────────────┐
             ▼                 ▼                  ▼
         OpenTelemetry      Metrics            Evaluations
             │                 │
             ▼                 ▼
        Prometheus         Langfuse DB
             │
             ▼
           Grafana
```

---

# O Problema

Em aplicações tradicionais monitoramos:

* CPU;
* Memória;
* Latência;
* Throughput.

Em aplicações baseadas em IA precisamos monitorar também:

* Prompt utilizado;
* Tokens consumidos;
* Modelo utilizado;
* Contexto recuperado;
* Hallucinations;
* Qualidade da resposta;
* Custos;
* Feedback dos usuários.

Nasce então o conceito de:

## AI Observability

---

# Stack Tecnológica

| Camada             | Tecnologia    |
| ------------------ | ------------- |
| API                | FastAPI       |
| Orquestração       | LangGraph     |
| Vetores            | PGVector      |
| LLM                | OpenAI GPT-4o |
| Observabilidade IA | Langfuse      |
| Tracing            | OpenTelemetry |
| Métricas           | Prometheus    |
| Dashboards         | Grafana       |
| Persistência       | PostgreSQL    |

---

# Fluxo Completo

```text
Pergunta usuário
        │
        ▼
Supervisor Agent
        │
        ▼
Retrieval Agent
        │
        ▼
PGVector
        │
        ▼
Documentos Recuperados
        │
        ▼
GPT-4o
        │
        ▼
Resposta
        │
        ▼
Langfuse Trace
        │
        ▼
OpenTelemetry
        │
        ▼
Prometheus
        │
        ▼
Grafana
```

---

# LangGraph: Orquestração de Agentes

O LangGraph permite modelar fluxos complexos.

```python
from langgraph.graph import StateGraph

workflow = StateGraph(State)

workflow.add_node("retrieval", retrieval_node)
workflow.add_node("generation", generation_node)

workflow.add_edge("retrieval","generation")

graph = workflow.compile()
```

Cada nó do grafo representa:

* Tool Calling;
* Agentes especializados;
* Memória;
* RAG;
* APIs externas.

---

# Observabilidade com Langfuse

Toda execução produz um Trace.

```text
Trace
│
├── Span: User Query
├── Span: Embeddings
├── Span: PGVector Search
├── Span: Retrieved Documents
├── Span: GPT-4o
└── Span: Final Response
```

Benefícios:

* Debug;
* Replay;
* Avaliação;
* Custos;
* Análise temporal.

---

# Integração OpenTelemetry

OpenTelemetry torna possível integrar IA ao ecossistema tradicional de observabilidade.

```python
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("retrieval"):
    docs = retriever.invoke(question)
```

Cada Span é enviado para:

* Langfuse;
* Grafana Tempo;
* Jaeger;
* Prometheus.

---

# Instrumentação Automática

```python
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor

FastAPIInstrumentor.instrument_app(app)
```

Também é possível instrumentar:

* Requests;
* PostgreSQL;
* Redis;
* OpenAI;
* LangChain.

---

# Monitorando RAG

Pipeline:

```text
Question
    │
    ▼
Embedding
    │
    ▼
PGVector
    │
    ▼
Top-K
    │
    ▼
Context
    │
    ▼
LLM
```

Métricas importantes:

### Latência da busca vetorial

```text
vector_search_latency_ms
```

### Quantidade de documentos recuperados

```text
retrieved_docs_total
```

### Similaridade média

```text
retrieval_score
```

### Tokens utilizados

```text
prompt_tokens
completion_tokens
```

---

# Exportando Métricas para Prometheus

```python
from prometheus_client import Counter

requests_counter = Counter(
    'llm_requests_total',
    'Total de requisições'
)

requests_counter.inc()
```

Outras métricas:

```python
llm_latency_seconds
llm_cost_usd
hallucination_rate
feedback_score
response_time_ms
```

---

# Dashboards no Grafana

## Performance

* Requisições por minuto;
* Latência média;
* Throughput.

---

## Custos

```text
GPT-4o Cost
Claude Cost
Gemini Cost
```

Visualizações:

* Custo diário;
* Custo por usuário;
* Custo por agente.

---

## Tokens

```text
Input Tokens
Output Tokens
Total Tokens
```

---

## Qualidade

```text
Feedback positivo
Taxa de erro
Faithfulness
Groundedness
```

---

# Avaliação Automática

LLM-as-Judge:

```python
trace.score(
    name="faithfulness",
    value=0.93
)
```

Indicadores:

| Métrica      | Objetivo               |
| ------------ | ---------------------- |
| Faithfulness | Reduzir hallucinations |
| Relevance    | Relevância             |
| Correctness  | Precisão               |
| Toxicity     | Segurança              |
| Completeness | Cobertura              |

---

# Feedback Humano

```python
trace.score(
    name="feedback",
    value=1
)
```

1 = positivo

0 = negativo

Permite criar ciclos contínuos de melhoria.

---

# AI FinOps

Monitorar custos por:

* Usuário;
* Aplicação;
* Modelo;
* Agente.

Exemplo:

```text
GPT-4o
US$ 42,00/dia

GPT-4o-mini
US$ 8,00/dia

Claude Sonnet
US$ 18,00/dia
```

---

# Arquitetura Multiagentes

```text
                    Supervisor
                         │
      ┌──────────────────┼───────────────────┐
      ▼                  ▼                   ▼
Retrieval Agent     SQL Agent         Search Agent
      │                  │                   │
      ▼                  ▼                   ▼
PGVector             PostgreSQL            APIs
      │                  │                   │
      └──────────────────┼───────────────────┘
                         ▼
                       GPT-4o
                         ▼
                      Langfuse
                         ▼
                  OpenTelemetry
                         ▼
                    Prometheus
                         ▼
                      Grafana
```

---

# Ambiente Docker Compose

```yaml
services:

  postgres:
    image: pgvector/pgvector:pg16

  langfuse:
    image: langfuse/langfuse

  prometheus:
    image: prom/prometheus

  grafana:
    image: grafana/grafana

  fastapi:
    build: .

  ollama:
    image: ollama/ollama
```

---

# Observabilidade em Produção

Uma aplicação corporativa baseada em IA deve monitorar:

## Infraestrutura

* CPU;
* Memória;
* Containers.

## Aplicação

* APIs;
* Banco de dados;
* Cache.

## LLMOps

* Tokens;
* Custos;
* Latência.

## AI Quality

* Hallucinations;
* Groundedness;
* Feedback.

## Agentes

* Tool Calls;
* Loops;
* Falhas.

---

# Arquitetura Enterprise

```text
                    Load Balancer
                           │
                           ▼
                       FastAPI
                           │
                           ▼
                      LangGraph
                           │
         ┌─────────────────┼────────────────┐
         ▼                 ▼                ▼
      OpenAI           PGVector          Redis
         │
         ▼
      Langfuse
         │
         ▼
  OpenTelemetry Collector
         │
 ┌───────┼────────┬─────────┐
 ▼       ▼        ▼         ▼
Tempo  Loki   Prometheus  Jaeger
 │
 ▼
Grafana
```

---

# Benefícios

✔ Observabilidade ponta a ponta

✔ Debug distribuído

✔ AI FinOps

✔ Governança de prompts

✔ Métricas de qualidade

✔ Avaliação automática

✔ Monitoramento de agentes

✔ Integração OpenTelemetry

✔ Dashboards corporativos

✔ Base para LLMOps em produção

---

# Conclusão

Aplicações modernas baseadas em IA exigem um novo paradigma operacional: **AI Observability**.

Da mesma forma que Prometheus e Grafana revolucionaram a observabilidade de microserviços, ferramentas como Langfuse e OpenTelemetry tornam-se componentes essenciais para sistemas baseados em agentes, RAG e LLMs.

A combinação:

```text
LangGraph
+ Langfuse
+ OpenTelemetry
+ Prometheus
+ Grafana
+ PGVector
+ FastAPI
```

forma uma arquitetura robusta e escalável para ambientes corporativos de IA Generativa, estabelecendo os pilares de uma plataforma completa de **LLMOps em produção**.

---

# Elaboração
* Autor: Ricardo Roberto de Lima - Engenheiro de IA.

# Referências

* https://langfuse.com
* https://github.com/langfuse/langfuse
* https://langchain-ai.github.io/langgraph
* https://opentelemetry.io
* https://grafana.com
* https://prometheus.io
* https://www.postgresql.org
* https://github.com/pgvector/pgvector
