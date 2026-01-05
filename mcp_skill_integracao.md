# Uso de Skills com IA Generativa e MCP
## Arquitetura, Conceitos e Exemplo Prático em Python

## 1. Introdução

Com a evolução das aplicações de Inteligência Artificial Generativa, torna-se insuficiente depender apenas da capacidade textual dos modelos de linguagem (LLMs). Para resolver problemas reais — corporativos, científicos ou operacionais — é necessário conectar esses modelos a **capacidades externas**, como cálculos, bancos de dados, APIs e regras de negócio.

Nesse contexto, destacam-se dois conceitos fundamentais:

- **Skills (Habilidades)**: funções especializadas responsáveis por executar ações concretas.
- **MCP (Model Context Protocol)**: protocolo que padroniza como essas habilidades são disponibilizadas e utilizadas pelos modelos de IA.

Este documento apresenta uma visão técnica sobre o uso conjunto de **Skills + IA Generativa + MCP**, com um exemplo prático em Python, simples e direto ao ponto.

---

## 2. O que são Skills em IA Generativa?

### 2.1 Definição Técnica

**Skills** são capacidades explícitas, executáveis e determinísticas que permitem à IA realizar tarefas fora do escopo puramente linguístico, tais como:

- Consultar bancos de dados
- Executar cálculos
- Validar regras de negócio
- Integrar APIs externas
- Automatizar processos

Arquiteturalmente, uma Skill possui as seguintes características:

- Determinística  
- Versionável  
- Auditável  
- Reutilizável  

O modelo de IA **decide quando usar a Skill**, mas **não executa diretamente a lógica**.

---

### 2.2 Exemplos Comuns de Skills

- `calcular_total_vendas(vendas)`
- `validar_documento(cpf)`
- `consultar_score_credito(cliente_id)`
- `buscar_exame_laboratorial(paciente_id)`

---

## 3. MCP – Model Context Protocol aplicado a Skills

### 3.1 Papel do MCP

O **Model Context Protocol (MCP)** atua como uma camada intermediária entre:

- Modelos de IA Generativa (LLMs)
- Skills disponíveis
- Fontes de dados e serviços externos

O MCP define **como as Skills são descritas, descobertas e invocadas**, sem que o modelo precise conhecer detalhes técnicos de implementação.

Principais responsabilidades do MCP:

- Padronizar chamadas de Skills  
- Isolar o modelo da infraestrutura  
- Controlar permissões e acessos  
- Garantir governança e auditoria  
- Permitir evolução independente de modelos e capacidades  

---

## 4. Arquitetura: IA Generativa + Skills + MCP

### 4.1 Visão em Camadas

1. **Camada Cognitiva**
   - Modelo de Linguagem (LLM)
   - Prompt e contexto

2. **Camada MCP**
   - Registro de Skills
   - Injeção de contexto
   - Mediação de chamadas

3. **Camada de Skills**
   - Funções Python
   - APIs e serviços externos

4. **Camada de Dados**
   - Bancos de dados
   - Sistemas corporativos

---

### 4.2 Fluxo de Execução

1. Usuário envia uma pergunta ou solicitação
2. O modelo identifica a necessidade de usar uma Skill
3. O MCP fornece a descrição da Skill disponível
4. O modelo solicita a execução da Skill
5. A Skill é executada
6. O resultado retorna ao modelo
7. O modelo gera a resposta final contextualizada

---

## 5. Exemplo Prático em Python

### 5.1 Definição de uma Skill

```python
# skill_calculo_vendas.py

def calcular_total_vendas(vendas: list[float]) -> float:
    """
    Skill responsável por calcular o total de vendas.
    """
    return sum(vendas)


### 5.2 Registro de Skills (Simulação do MCP)
# mcp_registry.py

from skill_calculo_vendas import calcular_total_vendas

SKILLS_REGISTRY = {
    "calcular_total_vendas": {
        "description": "Calcula o total de vendas a partir de uma lista de valores.",
        "function": calcular_total_vendas,
        "input_schema": {
            "vendas": "list[float]"
        },
        "output": "float"
    }
}

### 5.3 Executor MCP
# mcp_executor.py

from mcp_registry import SKILLS_REGISTRY

def execute_skill(skill_name: str, payload: dict):
    skill = SKILLS_REGISTRY.get(skill_name)

    if not skill:
        raise ValueError("Skill não encontrada")

    func = skill["function"]
    return func(**payload)

###  5.4 Uso da Skill pela IA Generativa (Simulação)
app.py

from mcp_executor import execute_skill

Pergunta do usuário (simulada)
user_question = "Qual foi o total de vendas do mês?"

Decisão da IA: usar a Skill
skill_name = "calcular_total_vendas"
payload = {
    "vendas": [1200.50, 899.90, 450.00, 2300.00]
}

# MCP executa a Skill
resultado = execute_skill(skill_name, payload)

# Resposta final gerada pela IA
resposta_final = f"O total de vendas no período foi de R$ {resultado:,.2f}"

print(resposta_final)

---

Saída Esperada
O total de vendas no período foi de R$ 4.850,40
---

## 6. Aplicações Práticas

Engenharia de Dados
Skills para validação, profiling e qualidade de dados
MCP controlando acesso a catálogos e DW

Saúde
Skills para análise de exames
MCP integrando padrões HL7/FHIR

Finanças
Skills de risco, fraude e compliance
MCP garantindo governança e auditoria

Educação
Skills para avaliação automática
MCP conectando rubricas e histórico acadêmico

---

## 7. Benefícios do Uso de Skills com MCP

Separação clara entre decisão e execução
Segurança e governança
Reuso de capacidades
Escalabilidade

Auditoria e rastreabilidade

Facilidade de manutenção

---

## 8. Conclusão

O uso de Skills integradas à IA Generativa por meio do MCP representa um avanço significativo em arquiteturas modernas de IA.
A IA decide
A Skill executa

O MCP governa
Essa abordagem permite construir aplicações inteligentes, confiáveis e prontas para produção, mantendo controle técnico e organizacional.

---
Autor:
Ricardo Roberto de Lima
Arquiteto de Dados | Engenheiro de IA | Especialista em IA Generativa
