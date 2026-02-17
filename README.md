# AI Learning Planner Agent

Assignment submission for Cyfuture GenAI & Agentic AI — Campus 2026

---

## What this is about

The idea here is to build an AI agent that helps someone learn a topic in a structured way. For example, if a user wants to learn Data Science in 30 days, the agent should break that down into a plan, fetch relevant resources, build a day-by-day schedule, and update it if the user says something is too fast or too slow. It also needs to remember things across sessions so the user does not have to repeat themselves every time.

---

## Architecture Flow

```
USER INPUT
  Goal | Daily Availability | Feedback
          |
          v
    Input Parser
          |
          v
    AGENT REASONING CORE  (LangChain / LangGraph)
      |           |           |           |
      v           v           v           v
  Goal          Knowledge   Effort      Feedback
  Decomp        Retrieval   Estimation  Handler
  Module        Module      Module
      |           |           |           |
      v           v           v           |
  Learning     Reference   Time per      |
  Units &      Materials   Topic Unit    |
  Milestones   & Topics                  |
      |           |           |           |
      +-----------+-----------+           |
                  |                       |
                  v                       |
          Schedule Builder                |
                  |                       |
                  v                       |
          Feasibility Evaluator           |
            |               |             |
        Not Feasible     Feasible         |
            |               |             |
            v               |             |
        Plan Adjuster       |             |
            |               |             |
            +---------------+             |
                  |                       |
                  v                       v
              MEMORY STORE
          SQLite / Redis / LangChain
                  |
                  v
          Response Generator
                  |
                  v
            OUTPUT TO USER
      Day-wise Plan | Milestones | Resources
                  |
        User gives feedback
                  |
                  v
            Input Parser
          (loop begins again)
```

---

## What each part does

**Input Parser**
Takes whatever the user types — their goal, how many hours they have per day, and any feedback on previous plans — and converts it into a structured format the agent can work with.

**Agent Reasoning Core**
This is the main part that runs everything. It looks at the input and decides which modules to call, in what order, and how to put their results together. Built using LangChain or LangGraph. It does not just run things once — it loops back when feedback comes in.

**Goal Decomposition Module**
Takes the main goal and splits it into smaller learning units. For example, "Learn Data Science in 30 days" becomes Week 1: Python basics, Week 2: Statistics, Week 3: Pandas and NumPy, and so on. It also figures out the right order — you learn Python before Pandas.

**Knowledge Retrieval Module**
Pulls up relevant study materials and topic structures from a knowledge base. Uses a vector database like FAISS or ChromaDB so it finds things by meaning, not just keyword matching.

**Effort Estimation Module**
Looks at each learning unit and figures out roughly how many hours it will take. It then checks this against how many hours the user has per day and adjusts accordingly.

**Feedback Handler**
When the user says something like "Week 1 was too fast" or "I already know SQL", this module figures out exactly what needs to change in the plan and flags those parts for revision.

**Schedule Builder**
Puts it all together — the topics, the materials, and the time estimates — into an actual day-by-day schedule. Each day gets a topic, a task, and a resource link.

**Feasibility Evaluator**
Checks if the schedule actually fits in the time the user has. If it is too packed, it sends the plan to the adjuster instead of the user.

**Plan Adjuster**
Takes the overloaded plan and spreads it out — maybe moves some topics, removes lower priority ones, or merges shorter tasks into the same day.

**Memory Store**
Remembers everything — the current plan, past feedback, and user preferences. So the next time the user comes back, the agent knows where they left off. Uses SQLite or Redis with LangChain Memory.

**Response Generator**
Takes the final plan and formats it into something clean and readable. Milestones, daily tasks, and resources all laid out clearly.

---

## Tools and Libraries used

| Component | Tool | Why we picked it |
|---|---|---|
| Agent Reasoning Core | LangChain / LangGraph | Good for multi-step reasoning and routing between modules |
| Goal Decomposition | LangChain Chains | Makes it easy to chain prompts for structured breakdown |
| Knowledge Retrieval | FAISS + OpenAI Embeddings | Fast and works well for semantic search over study content |
| Memory Store | LangChain Memory + SQLite | Simple way to persist user history without overcomplicating things |
| Effort Estimation | Custom LLM Prompt Chain | Flexible enough to adapt estimates to different user profiles |
| Schedule Builder | LangChain Output Parsers | Keeps the output format consistent and structured |
| Feedback Routing | LangGraph Conditional Edges | Cleanly routes feedback to the right step without messy if-else logic |
| LLM Backend | OpenAI GPT-4 / Mistral | Strong reasoning and instruction-following for all the prompts |

---

## How it all works together

When a user first comes in with a goal, the agent breaks it down, grabs relevant materials, estimates effort, builds a schedule, checks if it is feasible, and then sends back the final plan. If the plan gets flagged as too heavy, the adjuster fixes it before it reaches the user.

When the user comes back with feedback, the Feedback Handler looks at what needs to change, the Memory Store provides the context of what was planned before, and the agent revises only the parts that need updating. It does not rebuild the whole plan from zero — just the parts that are affected. This is what makes it adaptive rather than just a one-time planner.

