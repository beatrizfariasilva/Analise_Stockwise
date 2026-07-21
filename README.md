# Análise de Descontinuidade de Produtos - Stockwise

Projeto de insights sobre o inventário de uma empresa fictícia do setor de mercearia (Stockwise), com o objetivo de investigar possíveis causas para a descontinuidade e o atraso (*backorder*) de produtos no estoque.

## Objetivo

Identificar se características do produto, volume de vendas, giro de estoque, capital imobilizado ou categoria, ajudam a explicar por que um produto acaba com status `Discontinued` ou `Backordered`, com base em dados coletados entre fevereiro de 2024 e fevereiro de 2025.

## Hipóteses iniciais

1. **H1**: Produto é descontinuado porque vende pouco em quantidade comparado aos demais.
2. **H2**: Produto é descontinuado porque vence rápido demais e corre risco de expirar em estoque.
3. **H3**: Produto é descontinuado porque vende pouco a ponto de manter capital parado em estoque (capital imobilizado alto).
4. **H4**: Produto entra em *backorder* porque o fornecedor tem histórico de atraso na entrega.

## Dataset

- 990 registros, 16 colunas originais (`Product_ID`, `Product_Name`, `Category`, `Supplier_ID`, `Supplier_Name`, `Stock_Quantity`, `Reorder_Level`, `Reorder_Quantity`, `Unit_Price`, `Date_Received`, `Last_Order_Date`, `Expiration_Date`, `Warehouse_Location`, `Sales_Volume`, `Inventory_Turnover_Rate`, `Status`).
- Variável-alvo: `Status` (`Active`, `Backordered`, `Discontinued`).

## Ferramentas

Python · pandas · matplotlib · seaborn · scipy (teste qui-quadrado)

## Processo

### 1. Limpeza de dados
- Correção de 1 valor nulo em `Category`, classificado manualmente com base no produto (`Cabbage` → `Fruits & Vegetables`).
- Identificação e correção de inconsistência entre `Supplier_Name` e `Supplier_ID`: o mesmo fornecedor aparecia com múltiplos IDs distintos. Corrigido via padronização pelo primeiro ID associado a cada nome.
- Conversão de tipos: `Unit_Price` (string → float) e colunas de data (string → datetime).
- Nenhuma duplicata encontrada.

### 2. Análise univariada
- A variável-alvo `Status` está quase perfeitamente balanceada entre as três classes (~33% cada), o que é incomum para dado real de negócio, o que já sinaliza cautela quanto à origem dos dados.
- `Inventory_Turnover_Rate` mostrou distribuição praticamente uniforme (média ≈ mediana ≈ 50, quartis igualmente espaçados), padrão atípico para giro de estoque real e que sugere ausência de sinal informativo.
- `capital_imobilizado` (criada como `Stock_Quantity × Unit_Price`) apresentou forte assimetria à direita, causada por outliers de preço unitário inconsistentes com a realidade (ex: item "Banana" custando $98,43 a unidade).
- **Problema crítico identificado:** as colunas de data (`Date_Received`, `Last_Order_Date`, `Expiration_Date`) não possuem relação cronológica confiável entre si, cerca de 50% dos registros mostravam datas de validade anteriores ao recebimento, ou pedidos "recebidos" antes de serem feitos. Por esse motivo, **H2 e H4 foram descartadas** já nesta etapa, por falta de sustentação nos dados.

### 3. Análise bivariada
- `Inventory_Turnover_Rate`, `Sales_Volume` e `capital_imobilizado` (mediana) foram comparados entre os grupos de `Status` via gráficos de barra e boxplot: em nenhum dos três casos houve diferença relevante entre os grupos `Active`, `Backordered` e `Discontinued`.
- `Category` foi cruzada com `Status` via tabela de contingência e teste qui-quadrado de independência: **p-valor ≈ 0,9999**, não havendo associação estatisticamente significativa.

### 4. Análise multivariada
- Matriz de correlação entre as variáveis numéricas (`Inventory_Turnover_Rate`, `Sales_Volume`, `capital_imobilizado`) mostrou correlações próximas de zero entre todas as combinações.

## Principais conclusões

- **Nenhuma das hipóteses testáveis (H1 e H3) foi sustentada pelos dados.** As variáveis analisadas não diferenciam estatisticamente produtos ativos, descontinuados ou em atraso.
- **H2 e H4 não puderam ser testadas** devido a inconsistências estruturais nas colunas de data.
- O achado mais relevante do projeto não é uma correlação, e sim um **problema de qualidade de dado**: sem datas confiáveis, é impossível investigar causas ligadas a tempo (validade, atraso de fornecedor), que eram, a princípio, as hipóteses de negócio mais plausíveis.

## Recomendações

1. Implementar validação de integridade no sistema de cadastro (ERP), impedindo o registro de datas de entrega anteriores à data do pedido, e de validade anterior ao recebimento.
2. Após um período de coleta com dados temporais confiáveis, revisitar H2 e H4, que continuam sendo as hipóteses de negócio mais plausíveis para explicar descontinuidade.
3. Com uma base de dados corrigida, aplicar modelos de classificação (ex: Random Forest) para prever risco de descontinuidade de forma preditiva, não apenas descritiva.

## Estrutura do repositório

```
├── stockwise.csv
├── analise_stockwise.ipynb
└── README.md
```

## Como reproduzir

```bash
pip install pandas matplotlib seaborn scipy
jupyter notebook analise_stockwise.ipynb
```
