<p align="center">
  <img src="IMAGENS/brasão p tiamat.png" alt="Brasão dos Observadores de Lumi — Ad Astra Ad Infinitum" width="250">
</p>

<h1 align="center">Tiamat — Classificação de Candidatos a Exoplanetas</h1>

<p align="center">
  <strong>Gustavo Placido Aniceto da Silva 
Ilum Escola de Ciência — Centro Nacional de Pesquisa em Energia e Materiais (CNPEM)
</p>

## Sobre o projeto

**Tiamat** é um projeto de classificação supervisionada que investiga a capacidade de diferentes algoritmos de aprendizado de máquina de reproduzir a disposição de objetos de interesse da missão *Kepler* (*Kepler Objects of Interest*, ou KOIs), utilizando **25 atributos astronômicos** presentes em uma base tabular.

O objetivo é distinguir registros classificados no catálogo como **`CANDIDATE`** (candidato a exoplaneta) ou **`FALSE POSITIVE`** (falso positivo). A classificação reproduz rótulos do catálogo: **um candidato não é, necessariamente, um planeta confirmado**, e um resultado preditivo não constitui descoberta ou confirmação astronômica.

O trabalho reúne contextualização científica, auditoria dos dados, análise exploratória, preparação de atributos, comparação de modelos, otimização de hiperparâmetros, avaliação de generalização e interpretação de resultados.

## Objetivos

- Comparar classificadores com um modelo de referência (*baseline*).
- Evitar vazamento de informações entre treino, validação e teste, mantendo objetos de uma mesma estrela na mesma partição.
- Investigar o efeito da **Análise de Componentes Principais (PCA)** sobre o desempenho do k-NN.
- Otimizar hiperparâmetros e selecionar um limiar de classificação com dados internos de treinamento.
- Quantificar desempenho, incerteza e importância preditiva dos atributos, com interpretação adequada aos limites científicos da base.

## Dados utilizados

O conjunto analisado nas saídas registradas do notebook contém **6.148 registros**, com **25 atributos preditivos** e duas classes:

| Característica | Descrição |
|---|---|
| Fonte representada | Recorte tabular de objetos de interesse da missão Kepler (KOIs) |
| Registros | 6.148 |
| Candidatos (`CANDIDATE`) | 2.353 |
| Falsos positivos (`FALSE POSITIVE`) | 3.795 |
| Atributos preditivos | 25 |
| Variável-alvo | `koi_pdisposition` |
| Identificador de estrela (agrupamento) | `kepid` |
| Identificador do objeto | `kepoi_name` |
| Desenvolvimento | 4.918 registros |
| Teste reservado | 1.230 registros |

Os atributos incluem magnitudes fotométricas, características do trânsito, parâmetros orbitais e estimativas de propriedades estelares. O caderno apresenta um **dicionário das 25 variáveis**, com descrições e unidades.



## Metodologia

O experimento segue estas etapas:

1. **Auditoria e limpeza:** validação do esquema, dos rótulos e dos identificadores; verificação de duplicatas, ausências e valores incompatíveis com regras definidas previamente.
2. **Preparação dos dados:** imputação pela mediana dentro de `Pipeline`; padronização quando necessária para os modelos. Identificadores não entram como preditores.
3. **Particionamento por estrela:** separação inicial entre desenvolvimento e teste, sem compartilhamento de `kepid` entre conjuntos.
4. **Validação cruzada aninhada:** cinco folds externos para avaliar o procedimento e três folds internos para otimização e seleção, com `StratifiedGroupKFold`.
5. **Otimização:** buscas com **Optuna (TPE)**, com orçamento de **100 tentativas por família em cada busca**, e seleção da configuração pelo F1 macro interno.
6. **Política de classificação:** escolha do limiar a partir de previsões *out-of-fold* (OOF), antes da avaliação no teste reservado.
7. **Avaliação e interpretação:** matriz de confusão, precisão, recall, F1 macro, ROC AUC, *average precision*, bootstrap por estrela, curvas de aprendizado e importância por permutação.

### Algoritmos comparados

| Modelo | Papel no experimento |
|---|---|
| `DummyClassifier` | Baseline baseado na classe mais frequente |
| Regressão logística | Referência linear com regularização |
| k-NN sem PCA | Classificação por proximidade nos atributos padronizados |
| k-NN com PCA | Análise da transformação em componentes principais |
| Árvore de decisão | Modelo baseado em regras de divisão |
| Random Forest | Conjunto de árvores para classificação |
| HistGradientBoosting | Aprendizado por boosting em árvores |
