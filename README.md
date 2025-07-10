# Wumpus World Logic Agent

This repository contains a Python implementation of a logical agent designed to solve the classic Wumpus World problem. The agent uses a knowledge base and propositional logic with resolution-based inference to navigate a dangerous cave, avoid pits and the Wumpus, and find the gold.

This project is an implementation based on the concepts from "Artificial Intelligence: A Modern Approach" by Stuart Russell and Peter Norvig.

## Table of Contents
- [About the Wumpus World](#about-the-wumpus-world)
- [How the Agent Works](#how-the-agent-works)
  - [Knowledge Base Setup](#knowledge-base-setup)
  - [Inference and Decision Making](#inference-and-decision-making)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [How to Run](#how-to-run)
- [Code Breakdown (`wumpusworld.py`)](#code-breakdown-wumpusworldpy)

## About the Wumpus World

The Wumpus World is a classic grid-based environment for testing intelligent agents. The rules are as follows:

- **Goal:** The agent must find the Gold and return to the starting square `[1,1]` without getting killed. (Note: This implementation's goal is simplified to just finding the gold).
- **The Agent:** Starts at square `[1,1]`. It can move North, South, East, or West.
- **The Wumpus:** A deadly monster that resides in a single, unknown square. If the agent enters the Wumpus's square, it is killed.
- **Pits:** Several squares may contain bottomless pits. If the agent enters a pit, it is killed.
- **Percepts:** The agent receives information about its current square:
  - A **Stench** is perceived in squares adjacent to the Wumpus.
  - A **Breeze** is perceived in squares adjacent to a Pit.
  - A **Glitter** is perceived in the square containing the Gold.

The agent must use these percepts to deduce the locations of dangers and safely navigate the world.

## How the Agent Works

The `WumpusWorldAgent` is a `KnowledgeBasedAgent` that maintains a Knowledge Base (KB) of logical sentences to reason about the world.

### Knowledge Base Setup

Upon initialization, the agent populates its KB with the fundamental rules of the Wumpus World. It iterates through every square `(i, j)` on the board and adds biconditional sentences that link percepts to their causes:

1.  **Breeze Rule:** "A square `B(i,j)` has a breeze if and only if at least one of its neighbors has a pit `P(x,y)`."
    - This is translated into a logical sentence like: `B1_1 <=> (P1_2 | P2_1)`

2.  **Stench Rule:** "A square `S(i,j)` has a stench if and only if at least one of its neighbors contains the Wumpus `W(x,y)`."
    - This is translated into a logical sentence like: `S1_1 <=> (W1_2 | W2_1)`

These rules provide the agent with the background knowledge required to make inferences.

### Inference and Decision Making

The agent operates in a `perceive-think-act` cycle managed by the `play` function:

1.  **Perceive:** The agent enters a new room `(x,y)`. The `World.perceive()` method provides sensory information (stench, breeze).
2.  **Tell (Update KB):** This new information is added to the agent's KB as facts. For example:
    - If the agent is at `(1,1)` and feels no breeze, it `TELL`s the KB `~B1_1`.
    - Because the agent survived entering `(1,1)`, it also `TELL`s the KB `~P1_1` and `~W1_1`.
3.  **Ask (Infer and Act):** The agent uses the `inference.resolution` engine to query its KB and determine which moves are safe.
    - The `safe()` method identifies squares `(i,j)` for which the KB can prove both `~P(i,j)` (no pit) and `~W(i,j)` (no Wumpus).
    - The agent then uses this information (via the parent `KnowledgeBasedAgent.choose_location` method) to select a safe, unvisited square to explore next.
4.  **Loop:** This cycle continues until the agent finds the gold (WIN) or enters a dangerous square (DEATH).

## Project Structure

The repository is organized into the following key files:

- `wumpusworld.py`: The main file. Defines the `WumpusWorldAgent`, the `World` class, and the simulation loop (`play`).
- `logic.py`: Provides the core data structures for propositional logic, including `PropKB` (the Knowledge Base) and the `expr` function for creating logical expressions.
- `inference.py`: Contains the resolution-based inference engine (`resolution`) and the base `KnowledgeBasedAgent` class.
- `utils.py`: Contains supporting utility functions used by the other modules.

## Prerequisites

- Python 3.x

No external libraries are required.

## How to Run

1.  Ensure all four Python files (`wumpusworld.py`, `logic.py`, `inference.py`, `utils.py`) are in the same directory.
2.  Open a terminal or command prompt in that directory.
3.  Run the script with the following command:
    ```bash
    python wumpusworld.py
    ```
The script will automatically run two simulations defined in the `main()` function: one without a Wumpus and one with a Wumpus. The output will show the agent's perceptions and chosen moves turn-by-turn for each simulation, followed by the final result (WIN or DEATH).

Example Output Snippet:
```
You enter room [1,1]
You feel a breeze
the next location chosen by the agent:(2, 1)
You enter room [2,1]
the next location chosen by the agent:(2, 2)
...
You enter room [2,3]
You found the gold!
RESULT_WIN: You have won!, RESULT_DEATH: You have died :(, RESULT_GIVE_UP: You have left the cave without finding the gold :(
RESULT_WIN
```

## Code Breakdown (`wumpusworld.py`)

- **`WumpusWorldAgent(inference.KnowledgeBasedAgent)`**:
  - `__init__(cave_size)`: Initializes the KB with the universal rules of the Wumpus World.
  - `safe()`: Returns a set of squares that can be proven to be free of both Pits and the Wumpus.
  - `not_unsafe()`: A functionally similar method to `safe()`.
  - `unvisited()`: Returns a set of squares the agent has not yet entered.

- **`World`**:
  - `__init__(size, gold, pits, wumpus)`: Represents the "ground truth" of a specific cave layout.
  - `perceive(myTuple, KB)`: Simulates the agent's senses in a given room and updates the agent's KB with new facts. It also handles game-over conditions (death or victory).

- **`play(world)`**:
  - The main game loop that orchestrates the interaction between the `agent` and the `world`.

- **`main()`**:
  - The entry point of the script. It sets up and runs two predefined Wumpus World scenarios. You can easily add more worlds here to test the agent's performance.
