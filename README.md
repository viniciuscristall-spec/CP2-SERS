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

> **PENDENTE:** preencher após concluir a Tarefa 1 (três classificadores, Accuracy/Precision/Recall/F1, média usada, matriz de confusão e interpretação).

| Algoritmo | Accuracy | Precision | Recall | F1 |
|---|---|---|---|---|
| _(classificador 1)_ | | | | |
| _(classificador 2)_ | | | | |
| _(classificador 3)_ | | | | |

**Conclusão:** _(modelo escolhido, classes mais confundidas e por que potência/localização podem não bastar)_

---

## Tarefa 2 — Regressão (Open-Meteo)

**Configuração:**

- **Entradas (`X`):** `temperatura_c`, `umidade_pct`, `nuvens_pct`, `vento_kmh`, `hora`.
- **Alvo (`y`):** `radiacao_w_m2`. A coluna `data_hora` serve apenas para ordenar e dividir; nenhuma transformação do alvo entra em `X`.
- **Divisão temporal (sem embaralhar):** primeiras 80% das horas para treino, últimas 20% para teste. A padronização (`StandardScaler`) é ajustada somente no treino, dentro de um `Pipeline`.
- **Algoritmos:** Regressão Linear (referência simples), Random Forest (*bagging* de árvores) e Gradient Boosting (*boosting* de árvores), com hiperparâmetros fixos definidos sem consultar o conjunto de teste.
- **Avaliação:** todos os modelos usam exatamente a mesma divisão e o mesmo conjunto de teste.

**Resultados (conjunto de teste):**

> Preencher com a tabela gerada pelo notebook (`resultados_regressao.csv`).

| Algoritmo | MAE (W/m²) | MSE ((W/m²)²) | R² |
|---|---|---|---|
| Regressão Linear | | | |
| Random Forest | | | |
| Gradient Boosting | | | |

**Conclusões:**

- **Melhor modelo:** _(preencher com o modelo de menor MAE e seu R²)_.
- **Papel da hora do dia:** a `hora` funciona como substituta da posição do Sol; a radiação tem formato de sino ao longo do dia, relação não linear que a regressão linear capta mal. O experimento de ablação do notebook mostra o efeito de remover a variável: _(preencher com o MAE com e sem `hora`)_.
- **Onde há mais erro:** _(preencher com as horas de maior MAE, conforme o gráfico "MAE por hora")_.
- **Limitações:** apenas 3 meses de um único local; o teste cobre o final de junho, o que pode diferir de abril/maio; dados estimados por modelo, não medidos.
- **Radiação não é geração elétrica:** W/m² é potência por área numa superfície horizontal. A energia de um sistema fotovoltaico (kWh) depende também da integração no tempo, da inclinação e orientação dos painéis, da eficiência e da temperatura das células, das perdas (inversor, sujeira, sombreamento) e da operação da rede. O modelo estima o **recurso solar**, não a **energia gerada**.

## Observações

- Nenhuma senha, token ou chave de API é utilizada ou publicada neste repositório.
- A tarefa trata da radiação estimada e do cadastro de empreendimentos, não da energia efetivamente produzida.
