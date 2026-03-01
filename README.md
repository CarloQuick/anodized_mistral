# Anodized Mistral

# Anodized Mistral — Fine-Tuning Mistral 7B to Generate Formal Specifications for Rust

> **Mistral Worldwide Hackathon 2026 — W&B Fine-Tuning Track**

## Overview

[Anodized](https://github.com/mkovaxx/anodized) is a new Rust crate that adds formal specifications to Rust functions via `#[spec]` annotations — preconditions (`requires`), postconditions (`ensures`), and invariants (`maintains`). Rust is already the gold standard for memory safety, but anodized goes further by enforcing logical correctness: "this divisor can never be zero," "this index must be in bounds."

Máté Kovács (Tokyo Rust president) showed that these specs work as machine-readable documentation on stable Rust. **This project completes the circle** — training an LLM to write the specs themselves.

Because anodized is brand new (~70 stars, nominated for crate of the month), no base LLM has seen its syntax. Fine-tuning is the only viable path.

## Results

After fine-tuning on just **12 examples** from the anodized repo (with the crate author's blessing), the model learned correct `#[spec]` annotation syntax.

### Before (Vanilla Mistral 7B)

Hallucinates fake crates, puts conditions inside function bodies, uses `result` instead of `*output`:

```rust
#![feature(specs)]
#[spec]
fn safe_subtract(a: u32, b: u32) -> u32 {
    requires a > b;
    ensures result == a - b;
    a - b
}
```

### After (Fine-tuned Mistral 7B)

Correct annotation format, proper clause syntax, uses `*output`:

```rust
#[spec(
    requires: a >= b,
    ensures: *output <= a && *output >= b,
    maintains: *output == a - b,
)]
```

## Training

- **Base model:** Mistral 7B Instruct v0.3 (4-bit quantized via Unsloth)
- **Method:** LoRA (r=16, alpha=16) on QKV and O projections
- **Data:** 12 training pairs, 4 held-out eval pairs
- **Source:** anodized repo test cases + hand-written examples
- **Training:** 3 epochs, learning rate 2e-4, batch size 1
- **Training loss:** 2.16 → 0.66
- **Eval loss:** 2.20 → 1.58

### Note on Model Selection

The original plan was to fine-tune Codestral, Mistral's code-specialized model. However, Codestral was unavailable for chat-style fine-tuning through Mistral's API during the hackathon. Mistral 7B Instruct was selected as an alternative. Despite being a general-purpose model with no code specialization, fine-tuning still produced correct anodized syntax — arguably a stronger demonstration of fine-tuning's impact.

## Links

| Resource | Link |
|---|---|
| **W&B Report** | [Fine-Tuning Mistral 7B to Generate Anodized Specifications for Rust](https://wandb.ai/mistral-hackathon-2026/anodized-mistral/reports/Fine-Tuning-Mistral-7B-to-Generate-Anodized-Specifications-for-Rust--VmlldzoxNjA2NzExOQ) |
| **HuggingFace Model** | [cQuick/anodized-mistral-lora](https://huggingface.co/cQuick/anodized-mistral-lora) |
| **Colab Notebook** | [Training Notebook](https://colab.research.google.com/drive/1vSU8VW6kTar_EUFCigD0ZlnBw7oQdYHh?usp=sharing) |
| **Demo Video** | [YouTube](https://youtu.be/6GxY4UlQDzU) |
| **Anodized Crate** | [mkovaxx/anodized](https://github.com/mkovaxx/anodized) |

## Next Steps

- Expand training data to 50+ examples covering more anodized patterns
- Fine-tune Codestral when API fine-tuning becomes available
- Add compilation-based evaluation (does the generated spec compile?)
- Propose integration to anodized crate author

## Philosophy

> "LLMs don't write your code — they harden it."

## License

MIT
