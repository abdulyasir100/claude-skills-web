---
name: hugging-face-datasets
description: Create and manage datasets on Hugging Face Hub. Supports initializing repos, defining configs/system prompts, streaming row updates, and SQL-based dataset querying/transformation. Designed to work alongside HF MCP server for comprehensive dataset workflows.
---

# Overview
This skill provides tools to manage datasets on the Hugging Face Hub with a focus on creation, configuration, content management, and SQL-based data manipulation. It is designed to complement the existing Hugging Face MCP server by providing dataset editing and querying capabilities.

## Integration with HF MCP Server
- **Use HF MCP Server for**: Dataset discovery, search, and metadata retrieval
- **Use This Skill for**: Dataset creation, content editing, SQL queries, data transformation, and structured data formatting

# Version
2.1.0

# Dependencies
# This skill uses PEP 723 scripts with inline dependency management
# Scripts auto-install requirements when run with: uv run scripts/script_name.py

- uv (Python package manager)
- Getting Started: See "Usage Instructions" below for PEP 723 usage

# Core Capabilities

## 1. Dataset Lifecycle Management
- **Initialize**: Create new dataset repositories with proper structure
- **Configure**: Store detailed configuration including system prompts and metadata
- **Stream Updates**: Add rows efficiently without downloading entire datasets

## 2. SQL-Based Dataset Querying
Query any Hugging Face dataset using DuckDB SQL via `scripts/sql_manager.py`:
- **Direct Queries**: Run SQL on datasets using the `hf://` protocol
- **Schema Discovery**: Describe dataset structure and column types
- **Data Sampling**: Get random samples for exploration
- **Aggregations**: Count, histogram, unique values analysis
- **Transformations**: Filter, join, reshape data with SQL
- **Export & Push**: Save results locally or push to new Hub repos

## 3. Multi-Format Dataset Support
Supports diverse dataset types through template system:
- **Chat/Conversational**: Chat templating, multi-turn dialogues, tool usage examples
- **Text Classification**: Sentiment analysis, intent detection, topic classification
- **Question-Answering**: Reading comprehension, factual QA, knowledge bases
- **Text Completion**: Language modeling, code completion, creative writing
- **Tabular Data**: Structured data for regression/classification tasks
- **Custom Formats**: Flexible schema definition for specialized needs

## 4. Quality Assurance Features
- **JSON Validation**: Ensures data integrity during uploads
- **Batch Processing**: Efficient handling of large datasets
- **Error Recovery**: Graceful handling of upload failures and conflicts

# Usage Instructions

> **All paths are relative to the directory containing this SKILL.md file.**
> Scripts are run with: `uv run scripts/script_name.py [arguments]`

- `scripts/dataset_manager.py` - Dataset creation and management
- `scripts/sql_manager.py` - SQL-based dataset querying and transformation

### Prerequisites
- `uv` package manager installed
- `HF_TOKEN` environment variable must be set with a Write-access token

---

# SQL Dataset Querying (sql_manager.py)

Query, transform, and push Hugging Face datasets using DuckDB SQL.

## Quick Start

```bash
# Query a dataset
uv run scripts/sql_manager.py query \
  --dataset "cais/mmlu" \
  --sql "SELECT * FROM data WHERE subject='nutrition' LIMIT 10"

# Get dataset schema
uv run scripts/sql_manager.py describe --dataset "cais/mmlu"

# Sample random rows
uv run scripts/sql_manager.py sample --dataset "cais/mmlu" --n 5

# Count rows with filter
uv run scripts/sql_manager.py count --dataset "cais/mmlu" --where "subject='nutrition'"
```

## SQL Query Syntax

Use `data` as the table name in your SQL:

```sql
SELECT * FROM data LIMIT 10
SELECT * FROM data WHERE subject='nutrition'
SELECT subject, COUNT(*) as cnt FROM data GROUP BY subject ORDER BY cnt DESC
SELECT question, choices[answer] AS correct_answer FROM data
```

## Common Operations

```bash
# Explore structure
uv run scripts/sql_manager.py describe --dataset "cais/mmlu"
uv run scripts/sql_manager.py unique --dataset "cais/mmlu" --column "subject"
uv run scripts/sql_manager.py histogram --dataset "cais/mmlu" --column "subject" --bins 20

# Query and push to new dataset
uv run scripts/sql_manager.py query \
  --dataset "cais/mmlu" \
  --sql "SELECT * FROM data WHERE subject='nutrition'" \
  --push-to "username/mmlu-nutrition-subset" --private

# Export to local files
uv run scripts/sql_manager.py export \
  --dataset "cais/mmlu" \
  --sql "SELECT * FROM data LIMIT 100" \
  --output "sample.jsonl" --format jsonl

# Specify config/split
uv run scripts/sql_manager.py query \
  --dataset "ibm/duorc" --config "ParaphraseRC" \
  --sql "SELECT * FROM data LIMIT 5"

# Raw SQL with full hf:// paths (for joins etc.)
uv run scripts/sql_manager.py raw --sql "
  SELECT a.*, b.*
  FROM 'hf://datasets/dataset1@~parquet/default/train/*.parquet' a
  JOIN 'hf://datasets/dataset2@~parquet/default/train/*.parquet' b
  ON a.id = b.id LIMIT 100
"
```

---

# Dataset Creation (dataset_manager.py)

### Workflow

```bash
# Initialize new dataset
uv run scripts/dataset_manager.py init --repo_id "your-username/dataset-name" [--private]

# Configure with system prompt
uv run scripts/dataset_manager.py config --repo_id "your-username/dataset-name" --system_prompt "$(cat system_prompt.txt)"

# Quick setup with template
uv run scripts/dataset_manager.py quick_setup \
  --repo_id "your-username/dataset-name" --template classification

# Add data with template validation
uv run scripts/dataset_manager.py add_rows \
  --repo_id "your-username/dataset-name" --template qa \
  --rows_json "$(cat your_qa_data.json)"

# View stats
uv run scripts/dataset_manager.py stats --repo_id "your-username/dataset-name"

# List templates
uv run scripts/dataset_manager.py list_templates
```

### Templates

- **chat**: Multi-turn dialogues with tool usage
- **classification**: Text classification with labels and confidence
- **qa**: Question-answering with context and difficulty
- **completion**: Text completion with domain and style
- **tabular**: Structured data with column definitions
