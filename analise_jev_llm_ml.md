# JEV Classificador e LLMs Tradicionais

> Artigo técnico para publicação no GitHub.

## 1. Introdução

Aplicações modernas de Inteligência Artificial combinam regras, classificadores especializados e Large Language Models (LLMs) para transformar dados em decisões. Este artigo apresenta o **JEV Classificador** como uma abordagem didática orientada à decisão e compara seu uso com classificadores tradicionais de Machine Learning e LLMs.

A questão central é simples: **quando uma decisão deve ser resolvida por regras, por um modelo especializado ou por um LLM?**

A resposta depende de dados disponíveis, complexidade semântica, custo, latência, explicabilidade, privacidade e requisitos operacionais.

---

## 2. O problema de classificação

Um classificador associa uma entrada `X` a uma classe `Y`:

```text
X → Modelo → Y
```

Formalmente:

```text
D = {(x₁,y₁), (x₂,y₂), ..., (xₙ,yₙ)}
f(x) → y
```

Em classificação binária:

```text
y ∈ {0, 1}
```

Em classificação multiclasse:

```text
y ∈ {A, B, C, D}
```

---

## 3. O que é o JEV Classificador?

Neste artigo, **JEV Classificador** é tratado como uma abordagem simples e didática de classificação orientada à decisão. O foco é demonstrar o fluxo:

```text
Entrada → Inferência → Classe/Decisão
```

O JEV pode ser implementado inicialmente com regras determinísticas e posteriormente evoluído para uma arquitetura híbrida envolvendo Machine Learning e LLMs.

```text
                    JEV
                     |
       +-------------+-------------+
       |             |             |
    Regras       ML clássico   Modelo híbrido
       |             |             |
   if/else       árvore/ML    features + LLM
```

---

## 4. Exemplo simples com regras

Considere um sistema de classificação de chamados:

- `BAIXA`
- `MÉDIA`
- `ALTA`
- `CRÍTICA`

```python
def jev_classificador(prioridade, impacto, usuarios):
    if prioridade == "alta" and impacto == "alto" and usuarios > 100:
        return "CRITICA"

    if prioridade == "alta" or impacto == "alto":
        return "ALTA"

    if usuarios > 20:
        return "MEDIA"

    return "BAIXA"
```

Exemplo:

```python
resultado = jev_classificador("alta", "alto", 500)
print(resultado)
```

Saída:

```text
CRITICA
```

Nesse caso não há treinamento estatístico: o conhecimento está explícito nas regras.

---

## 5. JEV versus Machine Learning

Um classificador tradicional aprende padrões a partir de dados. Por exemplo, uma árvore de decisão pode receber idade, renda e score e produzir uma classe.

```python
from sklearn.tree import DecisionTreeClassifier

X = [
    [25, 2500, 450],
    [32, 5000, 650],
    [41, 8000, 780],
    [22, 1800, 390],
    [55, 9000, 820]
]

y = [
    "Reprovado",
    "Aprovado",
    "Aprovado",
    "Reprovado",
    "Aprovado"
]

modelo = DecisionTreeClassifier(random_state=42)
modelo.fit(X, y)

resultado = modelo.predict([[35, 7000, 720]])
print(resultado[0])
```

Comparação conceitual:

| Característica | JEV/Regras | ML tradicional |
|---|---|---|
| Conhecimento | Regras explícitas | Dados |
| Treinamento | Não obrigatório | Sim |
| Determinismo | Alto | Depende do modelo |
| Explicabilidade | Alta | Varia |
| Atualização | Alteração das regras | Novo treinamento |
| Custo de inferência | Muito baixo | Baixo |

---

## 6. Onde entram os LLMs?

LLMs permitem trabalhar diretamente com linguagem natural. Em vez de receber apenas atributos estruturados, o modelo pode interpretar uma descrição textual.

Exemplo:

```text
Classifique o cliente como APROVADO ou REPROVADO.

Cliente:
35 anos
renda: R$ 7.000
score: 720

Responda somente com a classe.
```

Resposta esperada:

```text
APROVADO
```

A principal vantagem é a flexibilidade semântica. A principal desvantagem, dependendo do cenário, pode envolver custo, latência, variabilidade e maior complexidade de governança.

---

## 7. Estudo de caso: classificação de chamados

Considere:

```text
1. Sistema está fora do ar para todos os usuários.
2. Usuário não consegue alterar sua senha.
3. Relatório apresenta lentidão.
```

Uma regra simples poderia ser:

```python
def classificar_chamado(texto):
    texto = texto.lower()

    if "fora do ar" in texto:
        return "CRITICO"

    if "senha" in texto:
        return "MEDIO"

    return "BAIXO"
```

O problema aparece quando o texto muda semanticamente:

```text
Todos os usuários estão impossibilitados de acessar a aplicação.
```

A expressão `fora do ar` não aparece, mas um modelo com maior capacidade semântica pode reconhecer a relação entre as frases.

---

## 8. LLM para classificação

Uma chamada conceitual pode ser estruturada assim:

```python
prompt = """
Classifique o chamado em:
CRITICO, MEDIO ou BAIXO.

Chamado:
Todos os usuários estão impossibilitados de acessar a aplicação.

Responda somente com a categoria.
"""
```

O resultado esperado seria uma categoria como:

```text
CRITICO
```

Em produção, recomenda-se impor saída estruturada, validação da resposta e observabilidade.

---

## 9. Estudo de caso: classificação documental

Considere documentos classificados como:

```text
Contrato
Nota Fiscal
Relatório
Ofício
Processo
Parecer
```

Com muitos exemplos rotulados, um classificador tradicional pode ser treinado. Para textos mais variados e tarefas que exigem interpretação contextual, um LLM pode ser incorporado.

Uma arquitetura RAG pode ser:

```text
Documento
    ↓
Extração de texto
    ↓
Chunking
    ↓
Embeddings
    ↓
Vector Database
    ↓
Recuperação
    ↓
LLM
    ↓
Classificação
```

---

## 10. Estudo de caso: sentimento

Entrada:

```text
Estou muito satisfeito com o atendimento.
```

Classes:

```text
POSITIVO
NEUTRO
NEGATIVO
```

O problema pode ser resolvido por um modelo supervisionado ou por um LLM. A escolha deve considerar volume, custo, latência, precisão, privacidade e manutenção.

---

## 11. Métricas de avaliação

As principais métricas de classificação são:

### Accuracy

```text
Accuracy = predições corretas / total de exemplos
```

### Precision

```text
Precision = VP / (VP + FP)
```

### Recall

```text
Recall = VP / (VP + FN)
```

### F1-Score

```text
F1 = 2 × Precision × Recall / (Precision + Recall)
```

Para LLMs, também podem ser avaliados:

- aderência ao formato;
- consistência;
- robustez;
- factualidade;
- segurança;
- custo por inferência;
- latência.

---

## 12. Experimento comparativo

Um laboratório pode utilizar 1.000 exemplos de chamados:

```text
70% → treinamento
15% → validação
15% → teste
```

Comparar:

```text
Modelo A → JEV/Regras
Modelo B → Decision Tree
Modelo C → Random Forest
Modelo D → LLM
Modelo E → Híbrido
```

Tabela sugerida:

| Modelo | Accuracy | F1 | Latência | Custo |
|---|---:|---:|---:|---:|
| JEV/Regras | medir | medir | medir | medir |
| Decision Tree | medir | medir | medir | medir |
| Random Forest | medir | medir | medir | medir |
| LLM | medir | medir | medir | medir |
| Híbrido | medir | medir | medir | medir |

**Importante:** os resultados devem ser preenchidos a partir de experimentos reais; números hipotéticos não devem ser apresentados como resultados científicos.

---

## 13. Arquitetura híbrida

Uma arquitetura prática pode utilizar um classificador especializado primeiro e encaminhar apenas casos ambíguos para um LLM:

```text
                         +------------------+
                         |      Usuário     |
                         +--------+---------+
                                  |
                                  v
                         +------------------+
                         | Pré-processamento|
                         +--------+---------+
                                  |
                                  v
                         +------------------+
                         | JEV / Regras     |
                         +--------+---------+
                                  |
                       +----------+----------+
                       |                     |
                 Alta confiança         Baixa confiança
                       |                     |
                       v                     v
                  Classificação             LLM
                       |                     |
                       +----------+----------+
                                  |
                                  v
                         +------------------+
                         | Validação        |
                         +--------+---------+
                                  |
                                  v
                            Decisão final
```

Essa estratégia pode reduzir chamadas ao LLM e preservar regras determinísticas para casos simples.

---

## 14. Aplicação Streamlit

Um laboratório simples pode ser construído com Streamlit:

```python
import streamlit as st

st.title("JEV Classificador")

texto = st.text_area(
    "Informe o problema:",
    "O sistema está fora do ar para todos os usuários."
)

if st.button("Classificar"):
    texto = texto.lower()

    if "fora do ar" in texto:
        classe = "CRÍTICO"
    elif "senha" in texto:
        classe = "MÉDIO"
    else:
        classe = "BAIXO"

    st.success(f"Classificação: {classe}")
```

Executar com:

```bash
streamlit run app.py
```

---

## 15. Evolução para uma arquitetura de IA

```text
             Streamlit
                 |
                 v
             FastAPI
                 |
       +---------+---------+
       |                   |
       v                   v
 JEV Classificador        LLM
       |                   |
       +---------+---------+
                 |
                 v
          Decisão Final
```

A aplicação pode evoluir com banco de dados, autenticação, logs, métricas, observabilidade, RAG, embeddings, guardrails, IA Gateway e roteamento entre modelos.

---

## 16. JEV + LLM + RAG

```text
                    Documento
                        |
                        v
                 Classificador JEV
                        |
                        v
                    Categoria
                        |
                        v
                     Busca/RAG
                        |
                        v
                 Base de conhecimento
                        |
                        v
                       LLM
                        |
                        v
                  Resposta final
```

Nesse desenho, o JEV pode determinar o caminho da execução, o RAG fornece contexto e o LLM interpreta e gera a resposta.

---

## 17. Governança e segurança

Uma solução corporativa deve considerar:

### Dados

- qualidade;
- origem;
- versionamento;
- sensibilidade;
- retenção.

### Modelos

- versão;
- treinamento;
- avaliação;
- drift;
- monitoramento.

### LLMs

- prompt injection;
- vazamento de dados;
- alucinação;
- controle de acesso;
- auditoria;
- limites de contexto.

Uma trilha mínima de auditoria pode registrar:

```text
Entrada → Modelo → Versão → Resultado → Confiança → Decisão
```

---

## 18. Comparação geral

| Dimensão | JEV/Regras | ML Tradicional | LLM |
|---|---|---|---|
| Entrada | Estruturada | Estruturada/textual | Linguagem/contexto |
| Treinamento | Não obrigatório | Sim | Pré-treinado |
| Contexto semântico | Baixo | Médio | Alto |
| Determinismo | Alto | Variável | Variável |
| Custo | Muito baixo | Baixo | Pode ser alto |
| Latência | Muito baixa | Baixa | Média/alta |
| Dados rotulados | Não obrigatório | Geralmente necessários | Pode usar poucos exemplos |
| Explicabilidade | Alta | Variável | Mais complexa |
| Flexibilidade | Baixa/média | Média | Alta |

---

## 19. Quando usar cada abordagem?

### JEV/Regras

Adequado quando as regras são conhecidas, a decisão precisa ser determinística e a latência/custo devem ser mínimos.

### Machine Learning

Adequado quando existem dados históricos e padrões que podem ser aprendidos de forma estatística.

### LLM

Adequado quando a entrada é predominantemente textual, há necessidade de compreensão semântica e o sistema pode justificar o custo e a complexidade operacional.

### Híbrido

Adequado quando diferentes tipos de decisão coexistem e é possível encaminhar apenas casos complexos para modelos mais sofisticados.

---

## 20. Laboratório proposto

Construir uma aplicação que classifique chamados em:

```text
BAIXO
MÉDIO
ALTO
CRÍTICO
```

### Etapa 1 — JEV

Implementar regras determinísticas.

### Etapa 2 — Machine Learning

Criar um dataset com pelo menos 100 exemplos e treinar um classificador.

### Etapa 3 — LLM

Utilizar um LLM para classificação por prompt.

### Etapa 4 — Avaliação

Medir:

```text
Accuracy
Precision
Recall
F1
Latência
Custo
```

### Etapa 5 — Híbrido

Implementar:

```text
JEV
 ↓
Confiança
 ↓
+----------------+
|                |
Alta             Baixa
|                |
v                v
Classe           LLM
```

---

## 21. Conclusão

O principal aprendizado não é escolher genericamente entre JEV, Machine Learning ou LLM, mas compreender que cada tecnologia atende a diferentes classes de problemas.

Uma regra simples pode resolver uma decisão simples. Um classificador tradicional pode ser apropriado quando existem dados históricos e uma tarefa bem definida. Um LLM pode ser útil quando linguagem, contexto e interpretação semântica são centrais.

Em sistemas corporativos, uma arquitetura híbrida pode combinar:

```text
JEV + Machine Learning + LLM + RAG + Observabilidade
```

A decisão arquitetural deve considerar natureza dos dados, complexidade, volume de inferências, latência, custo, explicabilidade, privacidade, disponibilidade de dados, segurança e necessidade de geração de linguagem.

Assim, o JEV Classificador pode funcionar como uma excelente porta de entrada para ensinar classificação e tomada de decisão, enquanto a comparação com Machine Learning e LLMs demonstra a evolução dos sistemas inteligentes.

---

## 22. Referências recomendadas

- **Scikit-learn** — classificação, treinamento e avaliação de modelos.
- **Hugging Face** — Transformers, modelos de linguagem e classificação.
- **OpenAI** — modelos de linguagem e APIs.
- **Google Cloud** — Machine Learning e avaliação.
- **Microsoft Azure AI** — modelos de linguagem e soluções de IA.

---

## Autor

**Ricardo Roberto de Lima**  
Professor | Engenheiro de IA | Arquiteto de Dados

Material destinado a fins educacionais, acadêmicos e de laboratório.

---

## Licença

Este material pode ser utilizado para fins educacionais e acadêmicos, mantendo-se a referência ao autor original.
