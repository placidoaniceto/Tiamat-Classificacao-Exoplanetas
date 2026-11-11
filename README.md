# Tiamat — Classificação de candidatos a exoplanetas
Gustavo Placido Aniceto da Silva

Trabalho de aprendizado de máquina desenvolvido a partir dos cadernos da disciplina ATP-203. O objetivo é comparar modelos capazes de distinguir **candidatos a exoplanetas** de **falsos positivos**, usando atributos do catálogo Kepler Q1–Q17 DR25, disponibilizado pelo NASA Exoplanet Archive.

A tarefa reproduz a classificação registrada no catálogo. Um objeto classificado como candidato não é, necessariamente, um planeta confirmado.

## Arquivos

- (Tiamat_Exoplanetas_Extras.ipynb): código, contextualização científica, gráficos, resultados e conclusões.
- (Dataset Exoplanetas.csv): dados utilizados pelo notebook.
- `README.md`: apresentação do projeto e instruções de execução.

## Como executar

1. Baixe o notebook e o CSV e coloque os dois na mesma pasta, mantendo os nomes dos arquivos.

2. Abra `Tiamat_Exoplanetas_Extras.ipynb` e execute todas as células, de cima para baixo.

O notebook lê o CSV local, realiza a análise e apresenta os resultados nas próprias células. Não baixa dados durante a execução nem exporta arquivos auxiliares. As buscas do Optuna são a etapa mais demorada; o tempo depende do computador. Os resultados da execução realizada já estão incluídos no notebook.

A execução de referência utilizou Python 3.12, scikit-learn 1.8.0 e Optuna 4.5.0. Mudanças de versão podem alterar os resultados.

## Dataset

A cópia utilizada contém **6.148 registros e 30 colunas**, provenientes da tabela `q1_q17_dr25_koi`, com o recorte original `koi_count = 1`.

O código seleciona **25 atributos numéricos** e a coluna-alvo `koi_pdisposition`. Os atributos incluem medidas de brilho, características do trânsito, parâmetros orbitais ajustados e propriedades estelares. Suas definições e unidades estão no notebook.

O alvo é representado por:

- `0`: `FALSE POSITIVE` — falso positivo do catálogo.
- `1`: `CANDIDATE` — candidato do catálogo.

Após remover linhas com valores ausentes nas colunas selecionadas, restam **6.041 exemplos**. Identificadores e campos que resumem diretamente a decisão de classificação não são usados como preditores.

## Metodologia

1. Leitura do CSV, seleção das colunas e remoção de registros incompletos.
2. Divisão estratificada em **80% para treino** (4.832 exemplos) e **20% para teste** (1.209 exemplos), com semente 42, usando índices e `.loc`.
3. Estatísticas descritivas e gráficos do conjunto de treino.
4. Comparação de cinco algoritmos com um baseline que sempre prevê a classe mais frequente.
5. Busca com **100 configurações por algoritmo**, usando Optuna e **validação cruzada de cinco folds somente dentro do treino**.
6. Seleção pelo maior **F1 macro médio de validação cruzada**.
7. Ajuste no treino completo e avaliação no teste, incluindo matriz de confusão.
8. Interpretação do vencedor por **permutation importance**, com 20 repetições por atributo.

Os algoritmos comparados são regressão logística, k-NN, árvore de decisão, Random Forest e HistGradientBoosting. A logística utiliza um pipeline com `StandardScaler`. O k-NN utiliza `StandardScaler`, **PCA** e classificador, com o número de componentes escolhido pelo Optuna entre 2 e 24.

As transformações dos pipelines são aprendidas dentro de cada fold. O F1 macro atribui o mesmo peso ao F1 de cada classe.

## Resultados da execução incluída

| Modelo | F1 treino | F1 médio de CV | F1 teste |
|---|---:|---:|---:|
| Baseline | 0,3796 | 0,3796 | 0,3797 |
| Regressão logística | 0,8199 | 0,8178 | 0,7952 |
| k-NN + PCA | 1,0000 | 0,8049 | 0,7973 |
| Árvore de decisão | 0,8417 | 0,8242 | 0,8022 |
| Random Forest | 0,9840 | 0,8534 | 0,8316 |
| HistGradientBoosting | 1,0000 | 0,8633 | 0,8459 |

O **HistGradientBoosting** foi selecionado pela validação cruzada, antes da avaliação no teste nesta execução. O k-NN utilizou **22 componentes principais**. Foram concluídas 100 configurações distintas por algoritmo, totalizando 500 tentativas.

## Limitações

Os resultados se referem a um recorte do catálogo e aos registros completos. A remoção de valores ausentes pode alterar a população estudada. Os rótulos representam a triagem histórica do catálogo, sem confirmar novos planetas.

O teste já havia sido consultado em versões anteriores do trabalho; portanto, é uma avaliação histórica. As médias de validação cruzada foram usadas na seleção de hiperparâmetros e não constituem uma avaliação independente dessa busca. O efeito do PCA não foi isolado por uma comparação controlada com e sem redução. A importância por permutação depende do modelo e das correlações entre atributos e não demonstra causalidade.
