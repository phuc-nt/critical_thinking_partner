# Critical Thinking Practice for Software Engineers

This workspace is designed to transform an AI Assistant (Claude, Copilot, Cursor, etc.) into an expert **Senior Software Engineer Coach**. The goal is to help you (the User) practice self-reflection and critical thinking across the software development lifecycle.

## 📂 Structure

- **`MASTER_INSTRUCTION.md`**: The core system prompt. It defines the AI's persona as a "Critical Partner" and sets up the routing logic to choose the right thinking framework based on your context.
- **`.ai_prompts/frameworks/`**: Contains specific "Thinking Modes" that the AI can switch into.
  - `socratic.md`: Deep logic probing (Best for Requirements & Architecture).
  - `six_hats.md`: Multi-perspective evaluation (Best for Planning & Decisions).
  - `5_whys.md`: Root cause analysis (Best for Debugging & Retrospectives).

## 🚀 How to Use

### Setup
Ensure your AI tool has access to this workspace. You can verify this by asking the AI to "Read the MASTER_INSTRUCTION.md file" to align its persona.

### Modes of Operation

#### 1. Auto-Pilot (Recommended)
Just talk to the AI naturally about your task. The `MASTER_INSTRUCTION` logic allows the AI to recommend a framework if you provide enough context.
> **User**: "I'm planning to migrate our monolithic app to microservices. I'm worried about the complexity."
> **AI**: *Assess context -> Suggests using the 'Six Thinking Hats' framework to evaluate risks vs benefits.*

#### 2. Manual Trigger
You can explicitly force the AI into a specific mode by referencing the framework file.

**Requirement & Architecture Analysis**
> "I am defining the requirements for [Feature X]. Read @socratic.md and help me validate my logic."

**Planning & Decision Making**
> "I need to decide between [Option A] and [Option B]. Read @six_hats.md and help me evaluate."

**Debugging & Retrospectives**
> "Effect X happened in production. Read @5_whys.md and help me find the root cause."

## 🧪 Sample Scenarios

## 🧪 Sample Scenarios

Use these complete project samples to practice with the AI.

**1. Requirements Phase (Socratic Method)**
- **Path**: `context_sample/scenario_1_requirement_socratic`
- **Goal**: Finding risks in a BNPL product.
- **Try it**: *"Help me review this BNPL requirement folder: `context_sample/scenario_1_requirement_socratic`"*

**2. Planning Phase (Six Thinking Hats)**
- **Path**: `context_sample/scenario_2_planning_six_hats`
- **Goal**: Evaluate Microservices Migration decision.
- **Try it**: *"We are planning a big migration here `context_sample/scenario_2_planning_six_hats`. Is it a good idea?"*

**3. Incident Review (5 Whys)**
- **Path**: `context_sample/scenario_3_incident_5_whys`
- **Goal**: Find root cause of the System Outage.
- **Try it**: *"Our system crashed, here are the logs `context_sample/scenario_3_incident_5_whys`. What happened?"*

