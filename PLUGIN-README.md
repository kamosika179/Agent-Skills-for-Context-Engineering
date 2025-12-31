# Context Engineering Skills Plugin

A comprehensive collection of skills for building production-grade AI agent systems. Covers context engineering fundamentals, optimization strategies, multi-agent patterns, memory systems, tool design, and evaluation frameworks.

## Structure

```
context-engineering-skills/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── skills/
│   ├── context-fundamentals/
│   │   └── SKILL.md         # Context basics and mechanics
│   ├── context-degradation/
│   │   └── SKILL.md         # Degradation patterns and mitigation
│   ├── context-compression/
│   │   └── SKILL.md         # Compression strategies
│   ├── context-optimization/
│   │   └── SKILL.md         # Optimization techniques
│   ├── multi-agent-patterns/
│   │   └── SKILL.md         # Multi-agent architectures
│   ├── memory-systems/
│   │   └── SKILL.md         # Memory architecture design
│   ├── tool-design/
│   │   └── SKILL.md         # Tool design for agents
│   ├── evaluation/
│   │   └── SKILL.md         # Evaluation frameworks
│   ├── advanced-evaluation/
│   │   └── SKILL.md         # LLM-as-judge techniques
│   └── project-development/
│       └── SKILL.md         # Project development methodology
└── README.md                 # This file
```

## Skills Overview

### Context Fundamentals
Foundational understanding of context windows, attention mechanics, and progressive disclosure. Start here when designing agent architectures or debugging context-related issues.

### Context Degradation
Patterns of context failure including lost-in-middle, context poisoning, and context clash. Essential for diagnosing unexpected agent behavior.

### Context Compression
Strategies for compressing context in long-running sessions. Covers anchored iterative summarization, structured summaries, and tokens-per-task optimization.

### Context Optimization
Techniques for extending effective context capacity: compaction, observation masking, KV-cache optimization, and context partitioning.

### Multi-Agent Patterns
Architectural patterns for distributing work across agents: supervisor/orchestrator, peer-to-peer/swarm, and hierarchical approaches.

### Memory Systems
Memory architecture from simple file-system patterns to temporal knowledge graphs. Covers entity tracking, cross-session persistence, and memory consolidation.

### Tool Design
Principles for designing tools that agents can use effectively. Includes the consolidation principle, architectural reduction, and MCP tool naming.

### Evaluation
Building evaluation frameworks for agent systems. Covers multi-dimensional rubrics, LLM-as-judge, and continuous evaluation pipelines.

### Advanced Evaluation
Production-grade LLM-as-judge techniques: direct scoring, pairwise comparison, rubric generation, and bias mitigation.

### Project Development
End-to-end project methodology from task-model fit evaluation through deployment. Covers pipeline architecture and agent-assisted development.

## Installation

Install directly from the Claude Code plugin system:

```
/plugin install context-engineering-skills@claude-plugin-directory
```

Or browse in `/plugin > Discover`

## Usage

Skills activate automatically based on your task context. Each skill's description contains trigger phrases that Claude uses to determine relevance.

Examples of activating queries:
- "Help me design a multi-agent architecture"
- "How do I compress context in long-running sessions?"
- "What causes lost-in-middle problems?"
- "Build an evaluation framework for my agent"

## Author

**Muratcan Koylan**  
muratcan.koylan@outlook.com

## License

MIT

