# Avaliação — APIs, energias renováveis e aprendizado de máquina

## Integrantes

| RM | Nome |
|---|---|
| 572718 | Kaique da Silva Assis |
| 569062 | André Debiazzi |
| 572049 | Vinicius Cristal |

## Objetivo

Consultar duas APIs públicas e aplicar aprendizado de máquina em duas tarefas independentes, comparando **três algoritmos em cada uma**:

1. **Tarefa 1 — Classificação:** classificar a fonte de um empreendimento (Solar, Eólica ou Hidráulica) a partir de potência e localização.
2. **Tarefa 2 — Regressão:** estimar a radiação solar horizontal média (W/m²) em Petrolina (PE) a partir de variáveis meteorológicas e da hora local.

## Fontes e período dos dados

| Tarefa | Fonte | Detalhes |
|---|---|---|
| 1 | [ANEEL — SIGA](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel) (API CKAN/DataStore) | Empreendimentos `UFV`, `EOL`, `UHE`, `PCH`, `CGH`; as três últimas formam a classe *Hidráulica* |
| 2 | [Open-Meteo — API histórica](https://open-meteo.com/en/docs/historical-weather-api) | Petrolina (−9,39; −40,50), **01/04/2025 a 30/06/2025**, fuso `America/Recife`, horas de 7h a 17h |

Ambas as consultas são públicas e **não exigem token ou chave de API**. Os dados do Open-Meteo são estimados por modelos/reanálise, não medidos por sensor.

## Arquivos do repositório

| Arquivo | Descrição |
|---|---|
| `Checkpoint_SERS_ANEEL_PETROLINA.ipynb` | Notebook completo (Tarefas 1 e 2) |
| `aneel_classificacao_orange.csv` | Dados da Tarefa 1 gerados pela API |
| `meteo_regressao_orange.csv` | Dados da Tarefa 2 gerados pela API |
| `resultados_regressao.csv` | Tabela de métricas da Tarefa 2 |
| `fig_*.png` | Figuras geradas pelo notebook |

## Como executar

1. Instale as dependências (Python 3.9+):
   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn jupyter
   ```
2. Abra o notebook (`jupyter notebook`) ou envie-o ao Google Colab.
3. Execute **todas as células na ordem** (`Kernel > Restart & Run All`). É necessária conexão com a internet nas células de consulta às APIs, que também geram os CSVs.
4. Para reproduzir apenas a Tarefa 2 sem internet, use o `meteo_regressao_orange.csv` já incluído e execute a partir da seção "Carregamento e exploração dos dados".

Todas as sementes aleatórias estão fixadas em `42`.

---

## Tarefa 1 — Classificação (ANEEL)

**Configuração:**

- **Entradas (`X`):** `potencia_kw`, `latitude`, `longitude`. **Alvo (`y`):** `fonte` (Solar, Eólica ou Hidráulica). `SigTipoGeracao`, nomes, CEG e descrições da fonte **não** são usados como entrada.
- **Dados:** 3.876 empreendimentos válidos (Hidráulica 1.476, Solar 1.200, Eólica 1.200).
- **Divisão:** 80% treino / 20% teste (776 exemplos), **estratificada**, `random_state=42`.
- **Algoritmos:** KNN (`k=5`, com `StandardScaler` dentro de um `Pipeline`), Árvore de Decisão (`max_depth=6`) e Random Forest (100 árvores).
- **Média das métricas por classe:** `macro` (Precision, Recall e F1).

**Resultados (conjunto de teste):**

| Algoritmo | Accuracy | Precision (macro) | Recall (macro) | F1 (macro) |
|---|---|---|---|---|
| KNN | 0,965 | 0,966 | 0,964 | 0,965 |
| Árvore de Decisão | 0,912 | 0,910 | 0,908 | 0,908 |
| Random Forest | 0,976 | 0,977 | 0,974 | 0,975 |

**Matrizes de confusão (teste; linhas = classe real, colunas = prevista, ordem Eólica / Hidráulica / Solar):**

| Modelo | Eólica | Hidráulica | Solar |
|---|---|---|---|
| KNN | 231 / 6 / 3 | 1 / 292 / 3 | 6 / 8 / 226 |
| Árvore de Decisão | 220 / 4 / 16 | 6 / 287 / 3 | 29 / 10 / 201 |
| Random Forest | 235 / 4 / 1 | 2 / 294 / 0 | 5 / 7 / 228 |

**Conclusão:**

- **Modelo escolhido:** Random Forest, com o melhor resultado em todas as métricas (accuracy 0,976 e F1 macro 0,975). O KNN ficou próximo e a Árvore de Decisão foi a mais fraca.
- **Classes mais confundidas:** a classe **Solar** é a mais difícil (no Random Forest, 12 dos 240 exemplos solares foram errados: 7 previstos como Hidráulica e 5 como Eólica). Na Árvore, a confusão entre Solar e Eólica é a maior (29 solares previstos como eólicos e 16 eólicos como solares).
- **Por que potência e localização podem não bastar:** a potência de um empreendimento não identifica sozinha a fonte (usinas pequenas de fontes diferentes têm potências parecidas) e as coordenadas só indicam onde as usinas se concentram, não qual tecnologia é usada. O resultado depende também do recorte da amostra: os dados têm limite por tipo e não representam a participação real de cada fonte na matriz brasileira. Para uma aplicação real seriam necessárias informações adicionais, como tecnologia, data de operação e situação regulatória.

---

## Tarefa 2 — Regressão (Open-Meteo)

**Configuração:**

- **Entradas (`X`):** `temperatura_c`, `umidade_pct`, `nuvens_pct`, `vento_kmh`, `hora`.
- **Alvo (`y`):** `radiacao_w_m2`. A coluna `data_hora` serve apenas para ordenar e dividir; nenhuma transformação do alvo entra em `X`.
- **Divisão temporal (sem embaralhar):** primeiras 80% das horas para treino, últimas 20% para teste. A padronização (`StandardScaler`) é ajustada somente no treino, dentro de um `Pipeline`.
- **Algoritmos:** Regressão Linear (referência simples), Random Forest (*bagging* de árvores) e Gradient Boosting (*boosting* de árvores), com hiperparâmetros fixos definidos sem consultar o conjunto de teste.
- **Avaliação:** todos os modelos usam exatamente a mesma divisão e o mesmo conjunto de teste.

**Resultados (conjunto de teste):**

Divisão temporal: **treino** com 800 linhas (01/04/2025 07h a 12/06/2025 14h) e **teste** com 201 linhas (12/06/2025 15h a 30/06/2025 17h). Tabela gerada pelo notebook (`resultados_regressao.csv`):

| Algoritmo | MAE (W/m²) | MSE ((W/m²)²) | R² |
|---|---|---|---|
| Regressão Linear | 145,205 | 30.034,201 | 0,360 |
| Random Forest | 69,277 | 7.859,748 | 0,832 |
| Gradient Boosting | 64,945 | 7.233,587 | 0,846 |

A Regressão Linear gerou 19 previsões negativas (fisicamente impossíveis) em 201; Random Forest e Gradient Boosting não geraram nenhuma.

**Conclusões:**

- **Melhor modelo:** Gradient Boosting, com o menor MAE (64,9 W/m², cerca de 17,4% da radiação média do teste, 373,4 W/m²) e R² de 0,846. O Random Forest ficou próximo (MAE 69,3; R² 0,832) e a Regressão Linear foi a pior (MAE 145,2; R² 0,360).
- **Papel da hora do dia:** a `hora` funciona como substituta da posição do Sol; a radiação tem formato de sino ao longo do dia, relação não linear que a regressão linear capta mal. O experimento de ablação do notebook mostra o efeito de remover a variável: no Random Forest, o MAE passa de 69,3 W/m² (com `hora`) para 131,1 W/m² (sem `hora`), um aumento de 89,2%, e o R² cai de 0,832 para 0,341. Na importância por permutação, a `hora` também é a variável que mais aumenta o erro quando embaralhada.
- **Onde há mais erro:** nas horas centrais do dia. No Gradient Boosting, o maior MAE é às 10h (100,3 W/m²), seguido de 9h a 14h com valores entre cerca de 70 e 81 W/m²; as bordas (7h e 17h) têm os menores erros (24,7 e 48,7 W/m²).
- **Limitações:** apenas 3 meses de um único local; o teste cobre o final de junho, o que pode diferir de abril/maio; dados estimados por modelo, não medidos.
- **Radiação não é geração elétrica:** W/m² é potência por área numa superfície horizontal. A energia de um sistema fotovoltaico (kWh) depende também da integração no tempo, da inclinação e orientação dos painéis, da eficiência e da temperatura das células, das perdas (inversor, sujeira, sombreamento) e da operação da rede. O modelo estima o **recurso solar**, não a **energia gerada**.

## Observações

- Nenhuma senha, token ou chave de API é utilizada ou publicada neste repositório.
- A tarefa trata da radiação estimada e do cadastro de empreendimentos, não da energia efetivamente produzida.
