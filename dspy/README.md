# dspy

**Source:** https://github.com/stanfordnlp/dspy  
**Author:** Stanford NLP Group  
**Language:** Python

## Description

DSPy (Declarative Self-improving Python) is a framework for algorithmically optimizing LLM prompts and weights. Rather than hand-crafting prompts, you define the behavior you want using composable modules and DSPy compiles your program, automatically tuning prompts or fine-tuning weights to maximize a metric. Designed for building reliable, modular LLM pipelines.

### Key Features

- Declarative LM programming — define what you want, not how to prompt it
- Automatic prompt optimization via teleprompters (BootstrapFewShot, MIPRO, etc.)
- Modular signatures and typed I/O
- Supports fine-tuning as an alternative to prompting
- Works with OpenAI, Anthropic, local models (Ollama, vLLM), and more
- Built-in assertions and suggestions for self-refinement
- Composable pipeline primitives: ChainOfThought, ReAct, ProgramOfThought, RAG

## Installation

```bash
pip install dspy

# With extras
pip install "dspy[anthropic]"
pip install "dspy[all]"
```

## Core Concepts

### Signatures
Define input/output behavior declaratively:
```python
import dspy

class BasicQA(dspy.Signature):
    """Answer questions with short factual answers."""
    question = dspy.InputField()
    answer = dspy.OutputField(desc="often between 1 and 5 words")
```

### Modules
Chain behavior together:
```python
class CoTQA(dspy.Module):
    def __init__(self):
        self.prog = dspy.ChainOfThought(BasicQA)
    
    def forward(self, question):
        return self.prog(question=question)
```

### Teleprompters (Optimizers)
Automatically tune prompts:
```python
from dspy.teleprompt import BootstrapFewShot

teleprompter = BootstrapFewShot(metric=my_metric)
optimized_program = teleprompter.compile(CoTQA(), trainset=trainset)
```

## Common Usage Examples

```python
import dspy

# Configure LM
lm = dspy.LM("anthropic/claude-3-5-sonnet-20241022")
dspy.configure(lm=lm)

# Simple chain of thought
cot = dspy.ChainOfThought("question -> answer")
result = cot(question="What is the capital of France?")
print(result.answer)

# ReAct agent with tools
react = dspy.ReAct("question -> answer", tools=[search, calculator])
result = react(question="What is 2+2?")

# RAG pipeline
class RAG(dspy.Module):
    def __init__(self, num_passages=3):
        self.retrieve = dspy.Retrieve(k=num_passages)
        self.generate = dspy.ChainOfThought("context, question -> answer")
    
    def forward(self, question):
        context = self.retrieve(question).passages
        return self.generate(context=context, question=question)

# Optimize with training data
from dspy.teleprompt import BootstrapFewShot, MIPRO

# BootstrapFewShot — fast, good for small datasets
optimizer = BootstrapFewShot(metric=validate_answer, max_bootstrapped_demos=4)
optimized = optimizer.compile(RAG(), trainset=trainset)

# MIPRO — better results, more expensive
optimizer = MIPRO(metric=validate_answer, auto="medium")
optimized = optimizer.compile(RAG(), trainset=trainset, requires_permission_to_run=False)

# Save/load optimized program
optimized.save("optimized_rag.json")
loaded = RAG()
loaded.load("optimized_rag.json")
```

## Built-in Modules

| Module | Description |
|---|---|
|  | Basic LM call |
|  | Reasoning before answering |
|  | CoT with a hint field |
|  | Generates and executes code |
|  | Reasoning + tool use agent |
|  | Compares multiple CoT chains |
|  | Vector retrieval |

## Supported LM Providers

```python
# OpenAI
dspy.configure(lm=dspy.LM("openai/gpt-4o"))

# Anthropic
dspy.configure(lm=dspy.LM("anthropic/claude-sonnet-4-5"))

# Local via Ollama
dspy.configure(lm=dspy.LM("ollama_chat/llama3", api_base="http://localhost:11434"))

# Local via vLLM
dspy.configure(lm=dspy.LM("openai/mistral-7b", api_base="http://localhost:8000/v1"))
```

## Potential Security / Offensive Use Cases

- Automated prompt injection payload generation and optimization
- Building self-improving recon agents with dynamic tool use (ReAct)
- Optimizing LLM-assisted vulnerability classification pipelines
- Generating adversarial inputs against RAG systems
- Fine-tuning smaller models for domain-specific offensive tasks

## Notes

- Optimization requires training examples + a metric function
- BootstrapFewShot is the fastest optimizer; MIPRO produces better prompts but costs more
- Programs are serializable — compile once, deploy anywhere
- Assertions (, ) allow self-correction at inference time
