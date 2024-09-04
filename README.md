# README.md

## Overview

Este script em Python realiza a análise de perguntas e respostas técnicas, utilizando modelos de linguagem (LLMs) do OpenAI e do Google, para identificar perguntas técnicas em um banco de dados PostgreSQL e determinar a resposta correta entre várias opções. O script também realiza análises estatísticas sobre a precisão do modelo. 

## Instalação

Certifique-se de ter o Python instalado. As dependências necessárias podem ser instaladas utilizando o `pip`. Execute os seguintes comandos em seu terminal:

```bash
pip install lxml psycopg2-binary langchain-google-genai langchain-core langchain-openai python-dotenv pandas numpy sqlalchemy scipy matplotlib seaborn
```

## Configuração

Antes de executar o script, defina as chaves de API e as configurações de banco de dados:

1. **Chaves de API**:
   - `GEMINI_KEY`: Chave da API Google.
   - `OPEN_AI_KEY`: Chave da API OpenAI.

2. **Banco de Dados**:
   - `DB_NAME`: Nome do banco de dados PostgreSQL.
   - `DB_USER`: Usuário do banco de dados.
   - `DB_PASSWORD`: Senha do banco de dados.
   - `DB_HOST`: Host do banco de dados.
   - `DB_PORT`: Porta do banco de dados.

3. **Modelo de Linguagem**:
   - `LLM_PROVIDER`: Escolha o modelo de linguagem entre `"google"` ou `"openai"`.

4. **Arquivo XML**:
   - `XML_FILE`: Caminho para o arquivo XML contendo os dados das perguntas e respostas. Pode ser obtido em [Stack Exchange - Archive.org](https://archive.org/details/stackexchange), no arquivo stackoverflow.com-Posts.7z.

## Estrutura do Banco de Dados

Certifique-se de que as tabelas necessárias estejam criadas no banco de dados PostgreSQL. Use os seguintes comandos SQL para criar as tabelas:

```sql
CREATE TABLE posts (
    post_id INTEGER PRIMARY KEY,
    creation_date TIMESTAMP,
    parent_id INTEGER,
    post_type_id INTEGER,
    accepted_answer_id INTEGER,
    score INTEGER,
    body TEXT,
    title TEXT,
    tags TEXT[],
    answer_count INTEGER,
    is_technical_question BOOL DEFAULT FALSE
);

CREATE TABLE api_calls (
    post_id INTEGER PRIMARY KEY,
    answer_id_1 INTEGER NOT NULL,
    answer_id_2 INTEGER NOT NULL,
    answer_id_3 INTEGER NOT NULL,
    answer_id_4 INTEGER NOT NULL,
    answer_id_5 INTEGER NOT NULL,
    chatgpt_response TEXT NOT NULL,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Uso

### 1. Processamento de Perguntas
O script processa um arquivo XML contendo perguntas e respostas. 
Ele filtra perguntas técnicas sobre "Java" que têm uma resposta aceita e pelo menos 5 respostas totais.
 As perguntas são então inseridas no banco de dados.

### 2. Amostragem de Perguntas
As perguntas são divididas em quatro partes, e 5% de cada parte é selecionado aleatoriamente para análise.

### 3. Avaliação por LLM
O script usa o modelo de linguagem configurado para avaliar se as perguntas selecionadas são de fato técnicas. 
Essa avaliação é armazenada no banco de dados.

### 4. Análise de Respostas
Para cada pergunta técnica identificada, o script busca as cinco respostas com mais votos e solicita ao modelo de linguagem que identifique a melhor resposta. 
A resposta do modelo é comparada com a resposta aceita para análise da precisão.

### 5. Análise Estatística
Os resultados são analisados estatisticamente para avaliar a precisão do modelo, utilizando testes binomiais para determinar a taxa de acerto.

Para análise, é gerado um arquivo `.csv` contendo os seguintes campos:

- **post_id**: Identificador da pergunta
- **creation_date**: Data de criação da pergunta
- **score**: Pontuação da pergunta
- **accepted_answer_id**: Resposta escolhida como correta pelo autor
- **answer_id_x**: Identificador da resposta "x"
- **score_answer_x**: Pontuação da resposta "x"
- **chatgpt_response**: Resposta do ChatGPT sem tratamento
- **timestamp**: Data e hora da consulta
- **chatgpt_response_integers_only**: Resposta do LLM considerando somente números inteiros (ex: "id: 4445232" -> "4445232")
- **check_llm_answer_exists_in_answers**: Checa se a resposta do LLM é uma das perguntas informadas no prompt
- **chatgpt_response_numeric**: Coluna convertendo resposta do LLM para o tipo numérico
- **rq1_is_llm_answer_correct**: Verificação se a resposta escolhida pelo LLM é a resposta aceita (desconsiderando respostas posicionais ex: "4")
- **check_max_score_answer**: Coluna da resposta que possui maior quantidade de votos
- **max_score_answer_id**: Identificador da resposta que possui maior quantidade de votos
- **is_max_score_answer_equal_accepted_answer**: Verificação se a resposta com maior quantidade de votos é a opção aceita
- **chatgpt_response_numeric_with_integer_responses**: Resposta do LLM considerando retornos numéricos (ex: "4" -> "answer_id_4")
- **is_llm_answer_correct_with_integer_responses**: Verificação se a resposta escolhida pelo LLM é a resposta aceita (considerando respostas posicionais ex: "4")
- **selected_max_score_and_not_selected_accepted_answer**: Verifica se foi escolhida a resposta com maior pontuação mesmo que não seja a resposta aceita
- **selected_max_score**: Verifica se a resposta do LLM é a que possui maior quantidade de upvotes


## Execução

Para executar o script, utilize o Jupyter Notebook.

## Visualização e Análise

Os resultados podem ser visualizados e analisados utilizando bibliotecas como `matplotlib` e `seaborn`, que já estão incluídas nas dependências do projeto.