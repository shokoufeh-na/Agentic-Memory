ExperimentOps Agent
AI Agent with Persistent Memory using CockroachDB × AWS

Overview
ExperimentOps Agent is an AI agent designed for the CockroachDB × AWS Hackathon. It demonstrates how agentic applications can use CockroachDB as a durable, production‑grade memory layer and AWS for scalable execution. The agent stores experiment history, retrieves relevant past runs, analyzes failures, and recommends next steps — showing how persistent memory makes agents actually useful in real workflows.

What It Does
Tracks machine learning experiments and metadata

Stores long‑term agentic memory in CockroachDB

Uses distributed vector indexing for semantic search

Retrieves similar experiments and past failures

Uses Amazon Bedrock for reasoning and recommendations

Runs serverlessly on AWS Lambda
