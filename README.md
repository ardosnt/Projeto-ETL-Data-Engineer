# ETL E-commerce (Data Engineer Local Practice)

Pipeline de ETL local em Python, aplicado sobre uma base fictícia de transações de e-commerce (250.000 registros), com extração de CSV, tratamento de qualidade de dados, transformação de colunas e carga em um banco PostgreSQL local.

## Estrutura do pipeline

O notebook `notebooks/etl_full.ipynb` está organizado em 4 etapas:

### 1. Extract
Leitura do arquivo `Ecommerce_DBS.csv` com pandas, contendo dados de clientes, produtos, valores de compra e localização geográfica.

### 2. Data Quality
- Conversão da coluna `Purchase Date` para o tipo `datetime`
- Verificação de valores nulos
- Correção de nomes de colunas (espaços extras, erros de digitação)
- Detecção de anomalias:
  - Valores negativos ou zerados em preço, quantidade e valor total
  - Idades fora de uma faixa plausível (0–100)
  - Coordenadas geográficas inválidas (fora do range de latitude/longitude)
  - Linhas duplicadas
- Remoção das linhas identificadas como anômalas

### 3. Transform
Criação de colunas derivadas para enriquecer a análise:
- `Total Purchase Unique`: valor real da transação (`Product Price × Quantity`), diferente de `Total Purchase Amount`, que representa um valor cumulativo do cliente
- `Regular Costume`: flag indicando se o `Customer ID` aparece mais de uma vez na base (cliente recorrente)
- `Week Day`: dia da semana extraído de `Purchase Date`

### 4. Load
Carga do DataFrame tratado em uma tabela PostgreSQL local (`ecommerce`), via SQLAlchemy + psycopg2.

## Tecnologias utilizadas

- Python 3.14
- pandas
- SQLAlchemy
- psycopg2-binary
- python-dotenv
- PostgreSQL (local)

## Como rodar o projeto

### 1. Instalar as dependências
```bash
pip install pandas sqlalchemy psycopg2-binary python-dotenv
```

### 2. Configurar variáveis de ambiente
Criar um arquivo `.env` na raiz do projeto (não versionado) com:
```
DB_USER=postgres
DB_PASSWORD=sua_senha
DB_HOST=localhost
DB_PORT=5432
DB_NAME=nome_do_banco
```

### 3. Criar o banco no PostgreSQL
```sql
CREATE DATABASE nome_do_banco;
```

### 4. Rodar o notebook
Executar `notebooks/etl_full.ipynb` célula a célula. Ao final, a tabela `ecommerce` estará disponível no banco configurado, pronta para consultas SQL (joins, CTEs, window functions, etc.).

## Estrutura de pastas
```
ETL-Ecommerce/
├── notebooks/
│   └── etl_full.ipynb
├── Ecommerce_DBS.csv
├── .env          (não versionado)
└── README.md
```

## Observações

- `Total Purchase Amount` é um valor cumulativo por cliente, não o valor da transação individual — por isso não bate com `Product Price × Quantity`. Para o valor real de cada compra, usar a coluna derivada `Total Purchase Unique`.
- O `.env` contém credenciais e está listado no `.gitignore` — nunca deve ser commitado.
