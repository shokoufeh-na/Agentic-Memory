ExperimentOps Agent
AI Agent with Persistent Memory using CockroachDB × AWS
ExperimentOps Agent — CockroachDB × AWS Hackathon Submission
An agentic application that uses CockroachDB as a persistent, production‑grade memory layer and AWS for scalable execution.
The ExperimentOps Agent helps developers track ML experiments, analyze failures, retrieve semantically similar runs, and recommend next steps — powered by distributed vector indexing and Bedrock‑based reasoning.
Today
help analyize and write this code CockroachDB × AWS Hackathon:
CockroachDB and AWS invite developers, engineers, and AI builders to create the next generation of agentic applications. Harness CockroachDB's distributed AI capabilities, fully managed MCP Server, agent-ready ccloud CLI, open-source Agent Skills Repo, LangChain integrations and Claude/Cursor plugins - all on AWS - to build AI agents with production-grade, persistent memory.

Why Agentic Memory? Why Now?
AI agents are rapidly moving from experiments into real production workflows, like writing code, running pipelines, diagnosing incidents, and driving more application traffic than any human could. But here's the problem: agents need memory that never goes down.

An agent whose memory goes offline doesn't degrade gracefully, it stops. Traditional databases were optimized for human-scale reads and writes. Agentic systems are different: they spawn autonomously, write constantly, and require memory that persists across regions, failures, and scale  (with zero data loss and no maintenance windows).

CockroachDB was built for exactly this. It is the system of record for agentic memory: globally distributed, always-on, PostgreSQL-compatible, and now natively integrated into the agent toolchain through MCP, cloud, and an open-source skills ecosystem.

This hackathon is your invitation to build on that foundation.

Requirements
The Challenge
Build an agentic application that uses CockroachDB as its persistent memory layer, deployed on AWS.

Your agent should store, retrieve, and act on memory whether that's conversation history, user context, task state, embeddings, or structured transactional data. The best submissions will demonstrate that memory is not an afterthought, it is the thing that makes an agent useful in production.

All submissions must use at least two of the following CockroachDB tools:

CockroachDB Cloud Managed MCP Server — Connect AI agents directly to CockroachDB clusters with a single config snippet from the Cloud Console. Works natively with Claude Code, Cursor, and VS Code. Safe by default: read-only mode, full audit logging, zero custom proxy required. Endpoint: https://cockroachlabs.cloud/mcp
CockroachDB Distributed Vector Indexing — Store and query embeddings at scale using CockroachDB's vector support with distributed indexing. Semantic search and retrieval stay fast as your data grows — no separate vector store to maintain, no reindexing pain, and no consistency gaps between your vector data and your operational database. Ideal for RAG pipelines, long-term agent memory, and semantic search applications.
ccloud CLI (Agent-Ready) — Give your agent direct, secure access to the full CockroachDB Cloud control plane. Provision clusters, manage backups, configure networking, monitor audit logs — all from the terminal. Designed for AI with consistent noun-verb patterns, JSON output on every command, and granular service-account-based RBAC.
CockroachDB Agent Skills Repo (Open Source) — A curated, open-source collection of machine-executable Agent Skills encoding CockroachDB expertise. Skills span onboarding, query/schema design, operations, performance, security, and observability. Portable across Claude, Cursor, LangChain, and any MCP-compatible client.
All submissions must also use at least one AWS service:

Amazon Bedrock (foundation models, knowledge bases, or agents)
AWS Lambda (serverless agent execution)
Amazon ECS / EKS (containerized agent workloads)
Amazon S3 (artifact or document storage)
Amazon SageMaker (model training or inference)
Amazon Bedrock Agents (multi-step agentic workflows)
Any other AWS service that powers your agent's environment
See my thinking
You picked a fun one—this hackathon is basically “build a real agent, not a toy.” Let’s turn this into something concrete and code-able.

1. Boiling down the requirements
Core idea:

Build an AI agentic app (not just a chatbot).

CockroachDB = persistent memory layer (must read/write real state).

Use ≥ 2 CockroachDB tools from:

Managed MCP Server

Distributed Vector Indexing

ccloud CLI

Agent Skills Repo

Use ≥ 1 AWS service (Bedrock, Lambda, ECS/EKS, S3, SageMaker, etc.).

So your submission needs:

A real workflow (tasks, state, history).

Memory-first design (agent is useless without its CockroachDB memory).

Clear use of CockroachDB tools + AWS in the architecture.

2. A concrete project idea (you can actually build this)
Let’s design something that fits your ML/AI vibe:

Project: “ExperimentOps Agent”
An AI agent that:

Tracks ML experiments (configs, metrics, artifacts).

Suggests next experiments based on past runs.

Uses semantic search over experiment descriptions.

Can provision/scale CockroachDB clusters via ccloud CLI when load grows.

Tools used:

CockroachDB Distributed Vector Indexing

Store embeddings of experiment descriptions, error messages, notes.

Managed MCP Server

Let the agent query CockroachDB via natural language tools.

AWS Lambda

Run the agent backend serverlessly.

(Optional) Amazon Bedrock

Use Claude/other FM for reasoning + embedding generation.

This hits: agentic memory, vector search, MCP, AWS.

3. High-level architecture
Flow:

User / Dev sends a request:

“Show me similar experiments to the one that failed with CUDA OOM.”

Lambda handler:

Calls Bedrock to embed the query.

Queries CockroachDB vector index for nearest experiments.

Fetches structured data (hyperparams, metrics, timestamps).

Agent logic:

Uses MCP tools + Agent Skills to:

Analyze patterns (e.g., batch size vs GPU memory).

Optionally suggest schema changes or new tables.

Response:

Returns explanation + recommended next experiment.

Logs this interaction as new memory (query + result + suggestion).

4. CockroachDB schema & vector memory
Example SQL schema
sql
CREATE TABLE experiments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    name STRING NOT NULL,
    description STRING,
    config JSONB,
    metrics JSONB,
    created_at TIMESTAMPTZ DEFAULT now(),
    embedding VECTOR(1536)  -- match your model's dimension
);

CREATE INDEX ON experiments USING GIN (embedding);
You’ll then:

Store structured data in config/metrics.

Store semantic memory in embedding.

5. Minimal Python backend (Lambda-style)
Lambda handler skeleton
python
import os
import json
import psycopg2
from psycopg2.extras import Json
# from your_bedrock_client import embed_text, generate_answer

DB_URL = os.getenv("COCKROACH_DB_URL")

def get_db_conn():
    return psycopg2.connect(DB_URL)

def lambda_handler(event, context):
    body = json.loads(event.get("body", "{}"))
    query_text = body.get("query", "suggest next experiment")

    # 1. Get embedding from Bedrock (or other FM)
    query_embedding = embed_text(query_text)  # returns list[float]

    # 2. Query CockroachDB vector index
    conn = get_db_conn()
    cur = conn.cursor()
    cur.execute(
        """
        SELECT id, name, description, config, metrics
        FROM experiments
        ORDER BY embedding <-> %s
        LIMIT 5;
        """,
        (query_embedding,)
    )
    rows = cur.fetchall()
    conn.close()

    # 3. Build context for the agent
    experiments = [
        {
            "id": r[0],
            "name": r[1],
            "description": r[2],
            "config": r[3],
            "metrics": r[4],
        }
        for r in rows
    ]

    # 4. Ask Bedrock/Claude to reason over these
    answer = generate_answer(query_text, experiments)

    # 5. (Optional) store this interaction as memory
    # insert into a `interactions` table with query, answer, related_experiment_ids

    return {
        "statusCode": 200,
        "body": json.dumps({"answer": answer, "experiments": experiments})
    }
You’d implement:

embed_text() using Bedrock’s embedding model.

generate_answer() using Bedrock’s Claude or other FM.

6. Using MCP + Agent Skills (to show “agentic” behavior)
You don’t have to write MCP server from scratch—the Managed MCP Server gives you:

A config snippet from CockroachDB Cloud.

Tools like:

Cluster monitoring

Query engine

Vector search

Backup & restore

You can:

Configure Claude Code / Cursor to connect via MCP.

Use the Agent Skills Repo to:

Let the agent automatically pick skills like “analyze slow queries” or “recommend index”.

In your write-up, show:

How the agent uses skills to:

Inspect performance of experiment queries.

Suggest schema changes (e.g., new index on created_at or user_id).

This proves: memory is operational, not just “chat history.”

7. How to frame this in your submission
When you write the project description / code comments:

Emphasize:

Agentic memory = CockroachDB (structured + vector).

Resilience: globally distributed, always-on, no maintenance windows.

Agent tools: MCP + Agent Skills + ccloud CLI (if you add infra automation).

AWS: Lambda for execution, Bedrock for reasoning/embeddings.

Show:

Schema + vector index.

Lambda handler + Bedrock calls.

Example queries and how memory changes the answer.

If you want, next step I can:

Help you write the README + architecture diagram text.

Or turn this into a more specific domain (incident response, data pipelines, etc.) that matches what you enjoy building.

write readme for github
Got you, Parmidaa.  
Here’s a clean, production‑ready README.md for your GitHub repo, tailored to the CockroachDB × AWS Hackathon and the agentic‑memory project we designed. It’s structured, polished, and ready to drop directly into your repo.

ExperimentOps Agent — CockroachDB × AWS Hackathon Submission
An agentic application that uses CockroachDB as a persistent, production‑grade memory layer and AWS for scalable execution.
The ExperimentOps Agent helps developers track ML experiments, analyze failures, retrieve semantically similar runs, and recommend next steps — powered by distributed vector indexing and Bedrock‑based reasoning.

Today
help analyize and write this code CockroachDB × AWS Hackathon:
CockroachDB and AWS invite developers, engineers, and AI builders to create the next generation of agentic applications. Harness CockroachDB's distributed AI capabilities, fully managed MCP Server, agent-ready ccloud CLI, open-source Agent Skills Repo, LangChain integrations and Claude/Cursor plugins - all on AWS - to build AI agents with production-grade, persistent memory.

Why Agentic Memory? Why Now?
AI agents are rapidly moving from experiments into real production workflows, like writing code, running pipelines, diagnosing incidents, and driving more application traffic than any human could. But here's the problem: agents need memory that never goes down.

An agent whose memory goes offline doesn't degrade gracefully, it stops. Traditional databases were optimized for human-scale reads and writes. Agentic systems are different: they spawn autonomously, write constantly, and require memory that persists across regions, failures, and scale  (with zero data loss and no maintenance windows).

CockroachDB was built for exactly this. It is the system of record for agentic memory: globally distributed, always-on, PostgreSQL-compatible, and now natively integrated into the agent toolchain through MCP, cloud, and an open-source skills ecosystem.

This hackathon is your invitation to build on that foundation.

Requirements
The Challenge
Build an agentic application that uses CockroachDB as its persistent memory layer, deployed on AWS.

Your agent should store, retrieve, and act on memory whether that's conversation history, user context, task state, embeddings, or structured transactional data. The best submissions will demonstrate that memory is not an afterthought, it is the thing that makes an agent useful in production.

All submissions must use at least two of the following CockroachDB tools:

CockroachDB Cloud Managed MCP Server — Connect AI agents directly to CockroachDB clusters with a single config snippet from the Cloud Console. Works natively with Claude Code, Cursor, and VS Code. Safe by default: read-only mode, full audit logging, zero custom proxy required. Endpoint: https://cockroachlabs.cloud/mcp
CockroachDB Distributed Vector Indexing — Store and query embeddings at scale using CockroachDB's vector support with distributed indexing. Semantic search and retrieval stay fast as your data grows — no separate vector store to maintain, no reindexing pain, and no consistency gaps between your vector data and your operational database. Ideal for RAG pipelines, long-term agent memory, and semantic search applications.
ccloud CLI (Agent-Ready) — Give your agent direct, secure access to the full CockroachDB Cloud control plane. Provision clusters, manage backups, configure networking, monitor audit logs — all from the terminal. Designed for AI with consistent noun-verb patterns, JSON output on every command, and granular service-account-based RBAC.
CockroachDB Agent Skills Repo (Open Source) — A curated, open-source collection of machine-executable Agent Skills encoding CockroachDB expertise. Skills span onboarding, query/schema design, operations, performance, security, and observability. Portable across Claude, Cursor, LangChain, and any MCP-compatible client.
All submissions must also use at least one AWS service:

Amazon Bedrock (foundation models, knowledge bases, or agents)
AWS Lambda (serverless agent execution)
Amazon ECS / EKS (containerized agent workloads)
Amazon S3 (artifact or document storage)
Amazon SageMaker (model training or inference)
Amazon Bedrock Agents (multi-step agentic workflows)
Any other AWS service that powers your agent's environment
See my thinking
You picked a fun one—this hackathon is basically “build a real agent, not a toy.” Let’s turn this into something concrete and code-able.

1. Boiling down the requirements
Core idea:

Build an AI agentic app (not just a chatbot).

CockroachDB = persistent memory layer (must read/write real state).

Use ≥ 2 CockroachDB tools from:

Managed MCP Server

Distributed Vector Indexing

ccloud CLI

Agent Skills Repo

Use ≥ 1 AWS service (Bedrock, Lambda, ECS/EKS, S3, SageMaker, etc.).

So your submission needs:

A real workflow (tasks, state, history).

Memory-first design (agent is useless without its CockroachDB memory).

Clear use of CockroachDB tools + AWS in the architecture.

2. A concrete project idea (you can actually build this)
Let’s design something that fits your ML/AI vibe:

Project: “ExperimentOps Agent”
An AI agent that:

Tracks ML experiments (configs, metrics, artifacts).

Suggests next experiments based on past runs.

Uses semantic search over experiment descriptions.

Can provision/scale CockroachDB clusters via ccloud CLI when load grows.

Tools used:

CockroachDB Distributed Vector Indexing

Store embeddings of experiment descriptions, error messages, notes.

Managed MCP Server

Let the agent query CockroachDB via natural language tools.

AWS Lambda

Run the agent backend serverlessly.

(Optional) Amazon Bedrock

Use Claude/other FM for reasoning + embedding generation.

This hits: agentic memory, vector search, MCP, AWS.

3. High-level architecture
Flow:

User / Dev sends a request:

“Show me similar experiments to the one that failed with CUDA OOM.”

Lambda handler:

Calls Bedrock to embed the query.

Queries CockroachDB vector index for nearest experiments.

Fetches structured data (hyperparams, metrics, timestamps).

Agent logic:

Uses MCP tools + Agent Skills to:

Analyze patterns (e.g., batch size vs GPU memory).

Optionally suggest schema changes or new tables.

Response:

Returns explanation + recommended next experiment.

Logs this interaction as new memory (query + result + suggestion).

4. CockroachDB schema & vector memory
Example SQL schema
sql
CREATE TABLE experiments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    name STRING NOT NULL,
    description STRING,
    config JSONB,
    metrics JSONB,
    created_at TIMESTAMPTZ DEFAULT now(),
    embedding VECTOR(1536)  -- match your model's dimension
);

CREATE INDEX ON experiments USING GIN (embedding);
You’ll then:

Store structured data in config/metrics.

Store semantic memory in embedding.

5. Minimal Python backend (Lambda-style)
Lambda handler skeleton
python
import os
import json
import psycopg2
from psycopg2.extras import Json
# from your_bedrock_client import embed_text, generate_answer

DB_URL = os.getenv("COCKROACH_DB_URL")

def get_db_conn():
    return psycopg2.connect(DB_URL)

def lambda_handler(event, context):
    body = json.loads(event.get("body", "{}"))
    query_text = body.get("query", "suggest next experiment")

    # 1. Get embedding from Bedrock (or other FM)
    query_embedding = embed_text(query_text)  # returns list[float]

    # 2. Query CockroachDB vector index
    conn = get_db_conn()
    cur = conn.cursor()
    cur.execute(
        """
        SELECT id, name, description, config, metrics
        FROM experiments
        ORDER BY embedding <-> %s
        LIMIT 5;
        """,
        (query_embedding,)
    )
    rows = cur.fetchall()
    conn.close()

    # 3. Build context for the agent
    experiments = [
        {
            "id": r[0],
            "name": r[1],
            "description": r[2],
            "config": r[3],
            "metrics": r[4],
        }
        for r in rows
    ]

    # 4. Ask Bedrock/Claude to reason over these
    answer = generate_answer(query_text, experiments)

    # 5. (Optional) store this interaction as memory
    # insert into a `interactions` table with query, answer, related_experiment_ids

    return {
        "statusCode": 200,
        "body": json.dumps({"answer": answer, "experiments": experiments})
    }
You’d implement:

embed_text() using Bedrock’s embedding model.

generate_answer() using Bedrock’s Claude or other FM.

6. Using MCP + Agent Skills (to show “agentic” behavior)
You don’t have to write MCP server from scratch—the Managed MCP Server gives you:

A config snippet from CockroachDB Cloud.

Tools like:

Cluster monitoring

Query engine

Vector search

Backup & restore

You can:

Configure Claude Code / Cursor to connect via MCP.

Use the Agent Skills Repo to:

Let the agent automatically pick skills like “analyze slow queries” or “recommend index”.

In your write-up, show:

How the agent uses skills to:

Inspect performance of experiment queries.

Suggest schema changes (e.g., new index on created_at or user_id).

This proves: memory is operational, not just “chat history.”

7. How to frame this in your submission
When you write the project description / code comments:

Emphasize:

Agentic memory = CockroachDB (structured + vector).

Resilience: globally distributed, always-on, no maintenance windows.

Agent tools: MCP + Agent Skills + ccloud CLI (if you add infra automation).

AWS: Lambda for execution, Bedrock for reasoning/embeddings.

Show:

Schema + vector index.

Lambda handler + Bedrock calls.

Example queries and how memory changes the answer.

If you want, next step I can:

Help you write the README + architecture diagram text.

Or turn this into a more specific domain (incident response, data pipelines, etc.) that matches what you enjoy building.

write readme for github
Got you, Parmidaa.  
Here’s a clean, production‑ready README.md for your GitHub repo, tailored to the CockroachDB × AWS Hackathon and the agentic‑memory project we designed. It’s structured, polished, and ready to drop directly into your repo.

ExperimentOps Agent — CockroachDB × AWS Hackathon Submission
An agentic application that uses CockroachDB as a persistent, production‑grade memory layer and AWS for scalable execution.
The ExperimentOps Agent helps developers track ML experiments, analyze failures, retrieve semantically similar runs, and recommend next steps — powered by distributed vector indexing and Bedrock‑based reasoning.





🚀 Overview
Modern AI agents need memory that never goes down. This project demonstrates how to build a real production agent using:

CockroachDB for globally distributed, durable agentic memory

Distributed Vector Indexing for semantic search over experiment logs

Managed MCP Server for agent‑native database access

AWS Lambda for serverless agent execution

Amazon Bedrock for embeddings + reasoning

The result: an agent that stores, retrieves, and acts on memory — not a toy chatbot, but a workflow‑driving system.

🧠 What the Agent Does
Stores ML experiment metadata, configs, metrics, and notes

Generates embeddings for semantic search

Retrieves similar experiments using CockroachDB’s vector index

Analyzes patterns (e.g., hyperparameters causing failures)

Suggests next experiments using Bedrock reasoning

Logs each interaction as persistent memory

This showcases agentic memory as the core feature, not an afterthought.
Architecture
Core Components
CockroachDB Cloud

Persistent memory layer

Vector indexing for semantic retrieval

MCP Server for agent‑native access

AWS Lambda

Stateless execution environment

Runs the agent logic

Amazon Bedrock

Embedding generation

Claude reasoning for experiment recommendations
Architecture
Core Components
CockroachDB Cloud

Persistent memory layer

Vector indexing for semantic retrieval

MCP Server for agent‑native access

AWS Lambda

Stateless execution environment

Runs the agent logic

Amazon Bedrock

Embedding generation

Claude reasoning for experiment recommendations
🚀 Overview
Modern AI agents need memory that never goes down. This project demonstrates how to build a real production agent using:

CockroachDB for globally distributed, durable agentic memory

Distributed Vector Indexing for semantic search over experiment logs

Managed MCP Server for agent‑native database access

AWS Lambda for serverless agent execution

Amazon Bedrock for embeddings + reasoning

The result: an agent that stores, retrieves, and acts on memory — not a toy chatbot, but a workflow‑driving system.
