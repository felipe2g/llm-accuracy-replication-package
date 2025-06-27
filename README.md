# Stack Overflow Data Preparation and Prompt Generation for LLM Analysis

## Overview

This project is designed to prepare a dataset of Stack Overflow questions and answers, specifically focusing on Java-related content, for subsequent analysis using Large Language Models (LLMs). The core process involves extracting data from an XML dump, storing it in a PostgreSQL database, filtering and sampling relevant questions, and then generating structured prompts for LLM evaluation. The output includes `.jsonl` files ready for batch LLM API calls and a detailed `.csv` file for tracking and later analysis of LLM performance.

## Features

* **Data Ingestion**: Parses Stack Overflow XML dumps (`Posts.xml`) and loads relevant question and answer data into a PostgreSQL database.
* **Question Filtering**: Filters questions based on specific criteria such as `PostTypeId` (only questions, ID 1), minimum answer count (at least 5), the presence of a required tag (`java`), and specific keywords within the question body (e.g., `<code>` present, `<img>` forbidden).
* **Answer Association**: Extracts and inserts answer data, linking them to their respective parent questions.
* **Data Cleaning**: Includes a utility function (`clean_html_except_code`) to strip HTML tags from post bodies while preserving `<code>` blocks and converting `<a>` tags to their `href` values.
* **Strategic Answer Selection**: For each filtered question, five distinct answers are strategically selected: the accepted answer, the worst-scoring answer, an intermediate-scoring answer, and two random answers. These are then shuffled.
* **Prompt Generation**: Constructs structured JSON prompts (in the OpenAI Chat Completion format) for each question, including the question title, tags, body, and the five selected answers. These prompts are designed for LLM evaluation, asking the model to identify the most accurate answer.
* **Metadata Export**: Generates `.jsonl` files containing the LLM prompts and a comprehensive wide-format `.csv` file (`answers_wide.csv`) that includes all question details, selected answer IDs, bodies, types, scores, and the correct answer's position, crucial for post-LLM response analysis.

## Installation

Ensure you have Python installed. All necessary dependencies can be installed via `pip`. It's highly recommended to use a virtual environment for dependency management.

1.  **Clone the repository (or download the files):**
    ```bash
    git clone <your-repo-url>
    cd <your-repo-name>
    ```

2.  **Create and activate a virtual environment:**
    ```bash
    python -m venv .venv
    source .venv/bin/activate  # On Windows: .venv\Scripts\activate
    ```

3.  **Install dependencies:**
    The following command installs all required libraries as listed in the Jupyter Notebook:
    ```bash
    pip install lxml psycopg2-binary langchain-google-genai langchain-core langchain-openai python-dotenv pandas numpy sqlalchemy scipy matplotlib seaborn jsonlines bs4 scikit-learn openpyxl
    ```

## Configuration

Before running the Jupyter Notebook, you'll need to set up your database connection details and file paths. API keys are mentioned in the notebook's imports, but the provided code primarily focuses on data preparation for *later* LLM calls, not making them directly.

1.  **Database Configuration (within `stack-overflow-dump-xml-to-postgres.ipynb`):**
    Locate the "CONFIGURAÇÃO DO BANCO DE DADOS" section and update the variables with your PostgreSQL credentials:
    ```python
    DB_NAME="dumpstack"
    DB_USER="postgres"
    DB_PASSWORD="postgres"
    DB_HOST="localhost"
    DB_PORT="5432"
    ```

2.  **XML File Path (within `stack-overflow-dump-xml-to-postgres.ipynb`):**
    Set the path to your Stack Overflow Posts XML dump file. This file (`stackoverflow.com-Posts.7z`) can be obtained from [Stack Exchange - Archive.org](https://archive.org/details/stackexchange).
    ```python
    XML_FILE_PATH="data/Posts.xml"
    ```
    *Ensure you create a `data` directory and place your `Posts.xml` file inside it, or update the path accordingly.*

3.  **Processing Rules (within `stack-overflow-dump-xml-to-postgres.ipynb`):**
    You can customize the filtering criteria for questions and the required tag:
    ```python
    POST_TYPE_ID_TO_PROCESS=1
    MINIMUM_ANSWER_COUNT=5
    REQUIRED_TAG_IN_POST="java"
    REQUIRED_KEYWORDS_IN_BODY=['<code>']
    FORBIDDEN_KEYWORDS_IN_BODY=['<img>']
    ```

## Database Schema

Ensure you have a PostgreSQL database set up and the necessary tables created before running the notebook. Use the following SQL commands:

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
    answer_count INTEGER
);

CREATE TABLE answers (
   post_id INTEGER PRIMARY KEY,
   post_type_id INTEGER,
   parent_id INTEGER,
   creation_date TIMESTAMP,
   score INTEGER,
   body TEXT
);