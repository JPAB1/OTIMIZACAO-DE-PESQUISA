# 🚀 Otimização com PySpark

Este projeto demonstra como utilizar **PySpark** para leitura, manipulação e otimização de dados em formato **Parquet**, explorando conceitos como **joins**, **repartition**, **coalesce** e **explain** para análise de performance.

---

## 📦 Instalação

Antes de executar o notebook, instale o PySpark:

```bash
pip install pyspark
```
## 📚 Importações

O projeto utiliza os seguintes pacotes:

- **pyspark** → Biblioteca principal para processamento distribuído  
- **pyspark.sql.SparkSession** → Criação da sessão Spark  
- **pyspark.sql.functions** → Funções SQL para manipulação de dados  
- **py4j.java_gateway** → Comunicação entre Python e JVM  

## ⚙️ Etapas do Projeto

### 1. Criação da sessão Spark
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()
```

### 2. Leitura dos arquivos Parquet

Os arquivos de entrada são lidos e armazenados em DataFrames:

- **videos-preparados.snappy.parquet** → DataFrame `df_video`  
- **video-comments-tratados.snappy.parquet** → DataFrame `df_comments`  

```python
df_video = spark.read.option("header", "true").parquet(
    'drive/MyDrive/Colab Notebooks/spark/data/material_de_apoio_m27/videos-preparados.snappy.parquet'
)

df_comments = spark.read.option("header", "true").parquet(
    'drive/MyDrive/Colab Notebooks/spark/data/material_de_apoio_m27/videos-comments-tratados.snappy.parquet'
)
```
### 3. Criação de tabelas temporárias

As tabelas temporárias são criadas a partir dos DataFrames para permitir consultas SQL:

```python
df_video.createOrReplaceTempView("df_video")
df_comments.createOrReplaceTempView("df_comments")
```
### 4. Join entre vídeos e comentários

Realizando o join entre os vídeos e seus respectivos comentários:

```sql
SELECT v.Title, v.`Video ID`, c.Comments, c.Likes
FROM df_video v
JOIN df_comments c ON v.`Video ID` = c.`Video ID`
```
### 5. Otimização com repartition e coalesce

- **repartition(n)** → Redistribui os dados em **n** partições  
- **coalesce(n)** → Reduz o número de partições sem redistribuição completa  

#### Exemplo em Python:
```python
df_video_rep = df_video.repartition(3)
df_video_rdd = df_video.coalesce(2)
```
### 6. Análise de plano de execução com explain

Permite entender como o Spark está processando os dados:

```python
df_video.explain()
df_video_rdd.explain()
```
### 7. Join otimizado com filtro

Filtrando apenas vídeos com número de likes acima da média:

```python
media_likes = df_video.select(avg("Likes").alias('media_likes'))

join_video_comments_otimizado = df_video.join(
    media_likes,
    df_video['Likes'] > media_likes['media_likes']
).select(df_video['*'])
```
### 8. Salvando resultado em Parquet

Após realizar o join otimizado, o resultado é salvo em formato **Parquet** para armazenamento eficiente:

```python
join_video_comments_otimizado.write.mode("overwrite").parquet(
    'drive/MyDrive/Colab Notebooks/spark/data/join-videos-comments-otimizado_parquet'
)
```
## 📊 Conceitos Importantes

- **Repartition**: aumenta ou redistribui partições → útil para balancear carga.  
- **Coalesce**: reduz partições sem shuffle → mais eficiente para diminuir partições.  
- **Explain**: mostra o plano lógico e físico de execução → essencial para otimização.  
- **Join + Filter**: aplicar filtros antes ou durante o join reduz custo computacional.  

---

## 🛠️ Requisitos

- **Python 3.7+**  
- **PySpark**  
- **Google Colab** ou ambiente com suporte a Spark  
- Arquivos Parquet de entrada:  
  - `videos-preparados.snappy.parquet`  
  - `video-comments-tratados.snappy.parquet`  

## 📂 Estrutura de Saída

Após execução, será gerado o arquivo:

```bash
join-videos-comments-otimizado_parquet/
```
Contendo os dados otimizados do join entre vídeos e comentários.

## ✅ Conclusão

Este notebook mostra como:

- **Ler e manipular dados com PySpark**  
- **Criar joins entre DataFrames**  
- **Otimizar consultas com repartition, coalesce e filtros**  
- **Analisar planos de execução com explain**  
- **Exportar resultados em formato Parquet**  
