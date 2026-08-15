# Benchmark de Desempenho e Modelação Preditiva em Larga Escala
### **Estudo Comparativo de Frameworks de Big Data (CPU/GPU) e Machine Learning**

Este repositório contém o projeto prático desenvolvido para a unidade curricular de **Ciência de Dados em Larga Escala (CDLE)** na **Faculdade de Ciências da Universidade do Porto (FCUP)** durante o ano letivo de 2025/2026. 

O objetivo do estudo é avaliar empiricamente a performance, consumo de recursos e limites de escalabilidade de cinco ecossistemas de processamento em Python (**Pandas**, **Dask**, **PySpark/Koalas**, **Modin** e **Joblib**) em CPU, contrastando-os com a aceleração por hardware em GPU (**RapidsAI - cuDF/cuML**), além de desenvolver um pipeline preditivo robusto.

---

## Autores
*   **João Levandeira**
*   **Nuno Antunes** 

---

## Estrutura do Repositório

O projeto está organizado nas seguintes pastas para facilitar a avaliação:

* **`Final/`**: Contém os notebooks Jupyter originais limpos (templates sem outputs), configurados para execução do início ao fim.
    *   `1_benchmarks.ipynb`
    *   `2_profiling.ipynb`
    *   `3_rapidai_benchmark.ipynb`
    *   `4_ml_pipeline.ipynb`
* **`Corridos/`**: Contém as versões executadas com todos os outputs das execuções oficiais, logs de depuração, métricas de ML e tabelas de tempos geradas diretamente na Cloud.
    *   `1_benchmarks_res.ipynb` (Executado na Escala Grande de ~9M linhas)
    *   `2_profiling_res3.ipynb` (Análise detalhada do cProfile)
    *   `3_rapidai_benchmark_res.ipynb` (Execução de GPU/Tesla T4)
    *   `4_ml_pipeline_res.ipynb` (Métricas e curvas do pipeline preditivo)

---

## Descrição dos Notebooks e Fases do Projeto

### **Fase 1: Estudo Comparativo em CPU** (`1_benchmarks.ipynb`)
Execução de 6 operações relacionais (equivalentes a base de dados) sob 3 volumetrias do dataset *NYC Yellow Taxi* (Janeiro a Março de 2022):
1.  **Read Data**: Leitura de ficheiros Parquet.
2.  **Count**: Contagem total de linhas.
3.  **Value Counts**: Distribuição de frequências da coluna `VendorID`.
4.  **GroupBy**: Agrupamento por `payment_type` com média de `fare_amount`.
5.  **Add Column**: Criação de coluna calculada (`Total_Calculated = fare_amount + tip_amount + tolls_amount`).
6.  **Filter**: Filtragem de viagens com tarifa superior a 10 dólares.

*Volumetrias de Teste:*
*   **Pequena Escala:** ~123.000 linhas.
*   **Média Escala:** ~2.460.000 linhas.
*   **Grande Escala:** ~9.071.244 linhas.

### **Fase 2: CPU Profiling e Diagnóstico** (`2_profiling.ipynb`)
Utilização do profiler determinístico `cProfile` do Python para auditar e expor os gargalos físicos de processamento (*bottlenecks*) a nível interno das frameworks CPU. Permite identificar problemas como a contenção de threads pelo GIL no Dask, custos de sockets IPC no PySpark, e custos de cópia de arrays em memória no Modin.

### **Fase 3: Aceleração em GPU** (`3_rapidai_benchmark.ipynb`)
Mapeamento de operações da CPU diretamente para a memória gráfica (VRAM) através do ecossistema **RapidsAI (cuDF e Dask-cuDF)**. Avalia a transferência de dados via barramento PCIe contra o ganho paralelo massivo dos núcleos CUDA e compara tempos com o cluster CPU.

### **Fase 4: Pipeline de Machine Learning** (`4_ml_pipeline.ipynb`)
Desenvolvimento de uma arquitetura preditiva robusta sem vazamento de dados (*data leakage*) para estimar o valor final da tarifa (`fare_amount`).
*   **Regressão (fare_amount contínuo):** O modelo *RandomForestRegressor* obteve um $R^2 = 0.9884$ e $MAE = 0.3640\$$. O *XGBRegressor* obteve performance semelhante mas treinou **10 vezes mais rápido**.
*   **Classificação (fare_amount discretizado em 3 classes):** A *LogisticRegression* linear atingiu a melhor exatidão global (**96.83% de Accuracy** e F1-score), demonstrando a alta eficácia da normalização e codificação efetuadas no *ColumnTransformer*.

---

## Ambientes de Execução

As experiências e medições oficiais foram efetuadas em diferentes cenários de infraestrutura:

1.  **Ambiente Local (macOS/ARM):** CPU-only (Darwin/arm64, 10 CPU cores, 16 GB RAM), servindo como baseline para datasets pequenos e médios.
2.  **Ambiente Cloud (GCP Dataproc Cluster):**
    *   **Single-Node (Desenvolvimento):** 1 Master Node da família `n4-highmem-4` (4 vCPUs, 32 GB RAM, 50 GB SSD).
    *   **Multi-Node (Distributed Spark):** 1 Master Node + 2 Worker Nodes `n4-highmem-4` (total de 8 vCPUs e 64 GB RAM em cluster).
3.  **Ambiente de GPU Acelerada:** VM Linux dotada de GPU **NVIDIA Tesla T4 (16 GB VRAM GDDR6)** para execução de cuDF, Dask-cuDF e cuML.

---

## Requisitos e Execução

Para correr os notebooks limpos da pasta `Final/` no seu ambiente local ou Cloud, certifique-se de que instala as dependências adequadas:

```bash
# Instalar bibliotecas de escalamento local e visualização
pip install "modin[ray]" matplotlib joblib pandas dask

# O PySpark requer uma instalação ativa do Java (JRE/JDK 8 ou 11) no sistema
pip install pyspark
```

*Nota para a GPU:* Para correr o notebook `3_rapidai_benchmark.ipynb`, é necessária uma máquina equipada com GPU NVIDIA e CUDA Toolkit configurado, ou utilizar um ambiente Google Colab com acelerador T4 ativo.

---

## Fontes e Referências

1.  **Estudo de Referência (Baseline)**:
    *   *Databricks Blog*: "Benchmark: Koalas, PySpark, and Dask" (April 7, 2021). [Link do Artigo](https://www.databricks.com/blog/2021/04/07/benchmark-koalas-pyspark-and-dask.html).
2.  **Origem dos Dados**:
    *   *New York City Taxi and Limousine Commission (TLC)*: [TLC Trip Record Data Portal](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page).
