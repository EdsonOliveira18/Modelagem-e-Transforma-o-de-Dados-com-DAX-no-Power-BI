# Modelagem e Transformação de Dados com DAX no Power BI

## Descrição do Projeto

Este projeto tem como objetivo criar um modelo de dados otimizado no Power BI utilizando o **Esquema Estrela (Star Schema)**. O desafio proposto envolve a transformação de uma única tabela, chamada `Financial Sample`, em um conjunto de tabelas fato e dimensão para facilitar a análise de dados e o desempenho nas consultas. O projeto inclui a criação de medidas e colunas calculadas usando fórmulas **DAX** (Data Analysis Expressions) para gerar insights relevantes.

## Estrutura do Esquema Estrela

O **Esquema Estrela** é um modelo de dados simples que organiza os dados em:

- **Tabela Fato**: Contém dados transacionais, como valores de vendas, lucros, etc.
- **Tabelas Dimensão**: Contêm dados descritivos que ajudam a classificar e filtrar as transações da tabela fato, como informações sobre produtos, datas e regiões.

### Tabelas Criadas

1. **F_Vendas (Tabela Fato)**
   - **Descrição**: A tabela fato contém os dados transacionais das vendas. Cada linha representa uma transação de venda, incluindo informações sobre produto, preço, quantidade vendida, desconto e lucro.
   - **Campos**:
     - SK_ID (chave surrogada)
     - ID_Produto
     - Produto
     - Units Sold
     - Sales Price
     - Discount
     - Discount Band
     - Segment
     - Country
     - Sellers
     - Profit
     - Date

2. **D_Produtos (Tabela Dimensão)**
   - **Descrição**: Esta tabela armazena informações agregadas sobre os produtos. É utilizada para fornecer detalhes descritivos que facilitam o agrupamento e a análise de vendas.
   - **Campos**:
     - ID_produto
     - Produto
     - Média de Unidades Vendidas
     - Média do Valor de Vendas
     - Mediana do Valor de Vendas
     - Valor Máximo de Venda
     - Valor Mínimo de Venda

3. **D_Produtos_Detalhes (Tabela Dimensão)**
   - **Descrição**: Armazena informações adicionais sobre os produtos, como preço de venda e preço de fabricação, além da faixa de desconto.
   - **Campos**:
     - ID_produto
     - Discount Band
     - Sale Price
     - Units Sold
     - Manufacturing Price

4. **D_Descontos (Tabela Dimensão)**
   - **Descrição**: Fornece informações sobre os descontos aplicados a cada produto.
   - **Campos**:
     - ID_produto
     - Discount
     - Discount Band

5. **D_Calendário (Tabela Dimensão)**
   - **Descrição**: Esta tabela é gerada automaticamente com a função DAX `CALENDAR()` e contém as datas que permitem realizar análises temporais.
   - **Campos**:
     - Date (dia)
     - Year
     - Month
     - Quarter

6. **D_Detalhes** (Pendente)
   - **Descrição**: Esta tabela será criada para incluir informações adicionais de vendas que não foram contempladas nas outras tabelas dimensão.

---

## Fórmulas DAX Utilizadas

Para transformar os dados, foram utilizadas diversas fórmulas DAX, tanto para a criação das tabelas quanto para o cálculo de medidas importantes. A seguir estão as principais fórmulas DAX utilizadas em cada tabela.

### 1. **Tabela F_Vendas**
Esta tabela foi criada para armazenar as transações de vendas. Utilizamos a função `SELECTCOLUMNS` para selecionar e renomear as colunas da tabela `Financial Sample`.

```DAX
F_Vendas = 
SELECTCOLUMNS(
    financials,
    "SK_ID", financials[ID_Produto] & "-" & financials[Date],
    "ID_Produto", financials[ID_Produto],
    "Produto", financials[Product],
    "Units Sold", financials[Units Sold],
    "Sales Price", financials[Sales Price],
    "Discount", financials[Discount],
    "Discount Band", financials[Discount Band],
    "Segment", financials[Segment],
    "Country", financials[Country],
    "Sellers", financials[Sellers],
    "Profit", financials[Profit],
    "Date", financials[Date]
)
```
### 2. **Tabela D_Produtos**
A tabela `D_Produtos` foi criada utilizando a função `SUMMARIZE` para agregar as métricas de vendas por produto, incluindo medidas de média, mediana, máximo e mínimo.

```DAX
Copiar código
D_Produtos = 
SUMMARIZE(
    financials,
    financials[ID_Produto],
    financials[Product],
    "Média de Unidades Vendidas", AVERAGE(financials[Units Sold]),
    "Média do Valor de Vendas", AVERAGE(financials[Sales Price]),
    "Mediana do Valor de Vendas", MEDIAN(financials[Sales Price]),
    "Valor Máximo de Venda", MAX(financials[Sales Price]),
    "Valor Mínimo de Venda", MIN(financials[Sales Price])
)
```

### 3. **Tabela D_Produtos_Detalhes**
A função `SELECTCOLUMNS` foi utilizada novamente para selecionar as colunas relevantes que detalham os produtos.

```DAX
Copiar código
D_Produtos_Detalhes = 
SELECTCOLUMNS(
    financials,
    "ID_produto", financials[ID_Produto],
    "Discount Band", financials[Discount Band],
    "Sale Price", financials[Sales Price],
    "Units Sold", financials[Units Sold],
    "Manufacturing Price", financials[Manufactoring Price]
)
```

### 4. **Tabela D_Descontos**
Esta tabela foi criada para armazenar as informações sobre descontos, utilizando a função `SUMMARIZE` para agrupar os produtos e calcular o desconto médio.

```DAX
Copiar código
D_Descontos = 
SUMMARIZE(
    financials,
    financials[ID_Produto],
    "Discount", AVERAGE(financials[Discount]),
    "Discount Band", FIRSTNONBLANK(financials[Discount Band], "")
)
```

### 5. **Tabela D_Calendário**
A tabela de calendário foi gerada utilizando a função `DAX CALENDAR`, que cria um intervalo de datas baseado nas transações.

```DAX
Copiar código
D_Calendário = 
CALENDAR(
    MIN(financials[Date]),
    MAX(financials[Date])
)
```

### Relacionamento com o Esquema Estrela
**Tabelas Fato e Dimensão:** O esquema estrela neste projeto é construído separando os dados transacionais (vendas) na tabela fato F_Vendas, enquanto as informações descritivas dos produtos, descontos e datas são armazenadas nas tabelas dimensão (D_Produtos, D_Descontos, D_Calendário, etc.).

**Desempenho e Facilidade de Análise:** A separação dos dados em tabelas de dimensão e fato otimiza o desempenho do Power BI, facilitando análises rápidas e a criação de relatórios dinâmicos. Ao usar um modelo estrela, as consultas se tornam mais eficientes, uma vez que as informações descritivas estão normalizadas nas tabelas dimensão.

**Fórmulas DAX:** As fórmulas DAX foram usadas tanto para transformar os dados em colunas calculadas quanto para criar medidas personalizadas que auxiliam na análise dos dados. As funções SUMMARIZE, AVERAGE, MAX, MIN, MEDIAN e SELECTCOLUMNS foram fundamentais para agregar e reorganizar os dados.

### Conclusão
Este projeto demonstra como transformar uma tabela transacional em um modelo de dados otimizado utilizando o esquema estrela no Power BI. O uso de fórmulas DAX permitiu criar tabelas personalizadas e colunas calculadas para facilitar a análise de dados e melhorar o desempenho das consultas. O resultado final é um modelo eficiente, escalável e fácil de manter."
