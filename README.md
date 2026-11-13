<p align="center">
  <img src="IMAGENS/Brasão p Tiamat.png" alt="Brasão dos Observadores de Lumi — Ad Astra Ad Infinitum" width="250">
</p>

# Tiamat — Classificação de candidatos a exoplanetas

**Autor:** Gustavo Placido Aniceto da Silva  
**Instituição:** Ilum Escola de Ciência — CNPEM

O notebook investiga a reprodução dos rótulos `CANDIDATE` e `FALSE POSITIVE` de `koi_pdisposition` no catálogo Kepler DR25. A análise utiliza 25 atributos físicos e observacionais para comparar regressão logística, Random Forest e HistGradientBoosting, com um `DummyClassifier` como referência.

O objetivo é avaliar a classificação de registros do catálogo. Um objeto classificado como candidato ainda não equivale a um planeta confirmado.

## Arquivos de leitura

| Arquivo | Conteúdo |
|---|---|
| [Tiamat_Exoplanetas.ipynb](Tiamat_Exoplanetas.ipynb) | Código, textos explicativos e saídas já presentes no notebook |
| `README.md` | Objetivo, organização, execução e resumo dos resultados |

**Nota de consistência:** a primeira célula foi preservada integralmente. Ela menciona k-NN, PCA e árvore de decisão isolada, mas esses procedimentos não estão implementados nas células de código deste arquivo. A descrição deste README acompanha a implementação efetiva.

## Dados e preparação

A consulta ao NASA Exoplanet Archive utiliza a tabela `q1_q17_dr25_koi` e o filtro `koi_count = 1`. São coletadas 29 colunas: 25 entradas dos modelos, uma variável-alvo e três campos de identificação e auditoria (`kepid`, `kepoi_name` e `koi_count`).

Os atributos abrangem magnitudes fotométricas, estatísticas de detecção, parâmetros do trânsito e propriedades estelares. O dicionário completo está na seção 3 do notebook. Os identificadores não são fornecidos aos classificadores.

Na preparação, são mantidas as duas classes de interesse, removidas duplicatas exatas e excluídos registros com valores ausentes nos atributos. São verificadas a unicidade dos identificadores, a ausência de valores não finitos e a inexistência de entradas idênticas. As classes recebem a codificação:

| Código | Rótulo do catálogo | Significado na modelagem |
|---|---|---|
| 0 | `FALSE POSITIVE` | Classe negativa |
| 1 | `CANDIDATE` | Classe positiva |

## Procedimento implementado

1. Importação das bibliotecas e configuração de pastas, semente e paralelismo.
2. Aquisição ou leitura do CSV e conferência de sua integridade por SHA-256.
3. Dicionário dos atributos, limpeza e auditoria das exclusões por classe.
4. Divisão estratificada em aproximadamente 60% para treino, 20% para validação e 20% para teste.
5. Análise exploratória apenas do treino: estatísticas, distribuição de classes, histogramas e correlações de Spearman.
6. Avaliação do baseline que sempre prevê a classe mais frequente.
7. Otimização de cada algoritmo com 100 tentativas do Optuna, usando o F1 macro da validação.
8. Seleção do algoritmo pela validação e reajuste das melhores configurações em treino + validação.
9. Avaliação no teste, gráficos, análise de erros e exportação dos resultados.

| Modelo | Preparação e ajuste |
|---|---|
| Baseline | `DummyClassifier(strategy='most_frequent')`, sem Optuna |
| Regressão logística | Pipeline com `StandardScaler`; busca de `C` e ponderação das classes |
| Random Forest | Busca de parâmetros das árvores, amostragem e ponderação das classes |
| HistGradientBoosting | Busca de taxa de aprendizado, iterações, complexidade das árvores e regularização |

O F1 macro atribui o mesmo peso ao F1 de cada classe. Também são calculados acurácia, acurácia balanceada, precisão, recall e F1 de candidatos, além de ROC-AUC. As previsões de classe utilizam `predict`, sem otimização de limiar.

As 100 tentativas são executadas sequencialmente em cada estudo. `N_JOBS = 2` limita o paralelismo interno em operações compatíveis. O código verifica se houve pelo menos 100 tentativas concluídas e 100 configurações distintas; essa verificação interrompe a execução se a condição não for atendida, sem gerar tentativas extras automaticamente.

## Como executar

É necessário um ambiente Python com acesso a notebooks Jupyter. As dependências podem ser instaladas com:

```bash
python -m pip install numpy pandas matplotlib scikit-learn optuna threadpoolctl ipython jupyterlab
```

A célula inicial de instalação está comentada e menciona `requirements.txt`. Esse arquivo não acompanha esta entrega; o comando acima lista as dependências diretamente. A instalação sem versões fixadas pode produzir um ambiente diferente daquele das saídas salvas.

Abra um terminal na pasta do projeto e execute:

```bash
jupyter lab
```

Abra `Tiamat_Exoplanetas.ipynb` e execute as células em ordem. As pastas `dados/` e `resultados/` são criadas em `Path.cwd()`, isto é, no diretório de trabalho do kernel.

As saídas preservadas registram Python 3.12.14, NumPy 2.3.5, pandas 2.2.3, scikit-learn 1.8.0 e Optuna 4.5.0. São registros da execução existente, não uma declaração de compatibilidade com todas as versões atuais. O metadado do kernel informa Python 3.13.7; a tabela produzida durante a execução é o registro utilizado aqui para descrever o ambiente das saídas.

## Resultados já registrados

Os números abaixo foram transcritos das saídas presentes no arquivo. O notebook não foi reexecutado nesta revisão textual.

- Registros originais: **6.148**.
- Registros após a limpeza: **6.041**, com 25 atributos.
- Exclusões por valores ausentes: **107**, sendo 99 falsos positivos e 8 candidatos.
- Duplicatas exatas removidas: **0**.

| Conjunto | Registros | Candidatos | Falsos positivos |
|---|---:|---:|---:|
| Treino | 3.624 | 1.407 | 2.217 |
| Validação | 1.208 | 469 | 739 |
| Teste | 1.209 | 469 | 740 |

| Modelo | F1 macro na validação | F1 macro no teste | Acurácia no teste | ROC-AUC no teste |
|---|---:|---:|---:|---:|
| Baseline | 0,3796 | 0,3797 | 0,6121 | 0,5000 |
| Regressão logística | 0,8307 | 0,7942 | 0,8015 | 0,8777 |
| Random Forest | 0,8714 | 0,8387 | 0,8470 | 0,9103 |
| HistGradientBoosting | **0,8734** | **0,8462** | **0,8536** | **0,9209** |

O **HistGradientBoosting** foi selecionado na validação. No teste, recuperou 383 dos 469 candidatos (recall de 0,8166), deixou de recuperar 86 e classificou 91 falsos positivos do catálogo como candidatos. Outros 649 falsos positivos foram corretamente classificados na classe 0.

## Arquivos gerados pela execução

Os arquivos abaixo são criados pelo código; não estão incluídos como arquivos separados nesta entrega.

| Local | Arquivos ou conteúdo |
|---|---|
| `dados/` | `kepler_dr25_original.csv`, `proveniencia.json`, `consulta.sql`, `dicionario_atributos.csv` |
| `resultados/` | `auditoria_limpeza.csv` e identificadores em `ids_treino.csv`, `ids_validacao.csv`, `ids_teste.csv` |
| `resultados/` | `optuna_modelo_1.csv`, `optuna_modelo_2.csv`, `optuna_modelo_3.csv`, respectivamente para logística, Random Forest e HistGradientBoosting |
| `resultados/` | `comparacao_validacao.csv`, `melhores_parametros.json`, `metricas_teste.csv`, `resumo_execucao.json`, `predicoes_teste.csv` |
| `resultados/` | Seis gráficos PNG: classes, distribuições, correlações, comparação de modelos, matrizes de confusão e curvas ROC |

O notebook mantém os modelos ajustados em memória, mas não os exporta como arquivos de modelos treinados.

## Alcance da análise

A comparação utiliza um único holdout e uma validação reutilizada durante a otimização. O teste foi separado nesta execução, mas a base já havia sido utilizada no desenvolvimento anterior. Os resultados não representam uma avaliação externa ou historicamente inédita.

A exclusão de registros incompletos e de sistemas com múltiplos objetos restringe a população estudada. As métricas medem concordância com o catálogo, sem confirmar planetas, estabelecer causalidade ou demonstrar transferência para outras missões.
