---
title: "While Big Labs Race to AGI, Here I Am With My Qwen3.8 27B:"
subtitle: "Benchmarks are one thing. The real test is pointing a local LLM at a working project and seeing what it actually ships."
date: 2026-09-11T00:00:00+07:00
lastmod: 2026-09-11T00:00:00+07:00
draft: false
author: "Dodi Prasetyo"
description: "Qwen 3.8 27B runs on a single 24GB GPU and reports frontier-level coding scores. I tested it the hard way: building and shipping a full multiplayer game, test by test."
tags: ["AI", "LLMs", "Qwen", "Open Source", "Rust", "Intermediate"]
categories: ["Artificial Intelligence", "LLMs"]
resources:
- name: "featured-image"
  src: "featured-image.png"
---

The race to AGI has become the loudest arms race in tech. Every quarter, the big labs roll out another frontier model, each one smarter and more autonomous than the last, each announcement framed as one step closer to the endgame everyone keeps name-dropping. OpenAI, Anthropic, Google. The budgets are absurd, the compute clusters are absurd, and the goal is a machine that can do any intellectual work a human can do.

Here's the part that feels a little scary to me: capability this large is being gatekept by a handful of companies. The most powerful models live behind API keys, and their pricing, terms, and availability are business decisions, not technical facts. A policy change you didn't vote for can lock you out of the best tool you have. We should be honest about how concentrated that is.

Open weights is the counter-move. You download the model, you host it on hardware you control, and you harness it however you want. No vendor can deprecate, reprice, or gate it. You own the thing you run.

I'll be fair about the gap, though: I'm still just a user. Almost nobody I know, me included, actually understands how these models work under the hood. You can run a model, tune it, and ship real work with it while sitting one layer above the magic. But independence is independence. Renting a model and owning a model feel very different the day something goes wrong.

So the question this post answers is a practical one. Can you hand a 27B model you run at home a real, multi-file project and get working software out the other side? Not a passing benchmark. A program that compiles, passes its tests, and survives a restart. I picked **Qwen 3.8 27B**, which Alibaba released on August 14, 2026 under Apache 2.0, and I did the thing that actually settles it: I made it my default agent model and pointed it at a project I was building from scratch.

## What Actually Shipped

The checkpoint is **27B parameters, dense**. Every parameter is active on every token, there is no MoE routing, and a native vision encoder reads images and video alongside text. Context is **262,144 tokens natively** and extends to 1M with YaRN. Max output is 131,072 tokens. Thinking is on by default, but you can toggle it per request and tune depth with a `reasoning_effort` knob (`xhigh`, `medium`, or `low`).

The distribution story is what makes it worth caring about:

| Channel | What you get |
|---------|--------------|
| Hugging Face `Qwen/Qwen3.8-27B` | Official BF16 weights, Apache 2.0 |
| Hugging Face `Qwen/Qwen3.8-27B-FP8` | Official FP8 for vLLM / SGLang serving |
| unsloth `Qwen3.8-27B-GGUF` | Community Q4_K_XL quant, ~18GB, for llama.cpp and LM Studio |
| bartowski `Qwen3.8-27B-GGUF` | Community quant alternative for llama.cpp |

Apache 2.0 is the part I check first. No user-count clause, no regional carve-out, no "Qwen License" strings attached. Commercial use, fine-tuning, and redistribution are all clean. That puts it in the safe bucket for team use and a tier above Meta's community license.

One fun wrinkle on the distribution front: on September 3, 2026, **NVIDIA announced it's acquiring Hugging Face for about $12.9 billion**. The company that sells you the chips is now buying the hub where you download the open weights. The stated goal is to "expand access to AI," which sounds nice and will probably be fine. But I'm skeptical on principle: the open-weights commons now answers to the largest hardware vendor on the planet, and that's a concentration of power worth watching, not celebrating. My practical response is simple. I mirror what I want and keep a local copy. The license is Apache 2.0, so the weights stay yours no matter who owns the building. That's rather the point of this whole post.

> 💡 **The dense detail matters for hardware.** Dense 27B means the KV cache grows fast as context grows. It fits in 24GB with room to spare for short-to-medium context, but if you're feeding it whole repositories at 256K, plan the memory headroom. That's the honest trade-off for a dense model over an MoE.

---

## The Benchmark Evidence

Let's look at the numbers before we trust them or doubt them. These are Qwen-reported, from the official model card, so read them with the usual vendor discount in mind:

| Benchmark | Qwen3.6-27B | Qwen3.8-27B | Claude Opus 4.6 Max |
|-----------|-------------|-------------|---------------------|
| SWE-bench Pro (agentic coding) | 53.5 | **61.7** | 53.4 |
| Terminal-Bench 2.1 (agentic terminal) | 63.4 | **73.0** | 78.2 |
| DeepSWE 1.1 (agentic coding) | 13.3 | **42.2** | n/a |
| LiveCodeBench v6 (competitive coding) | 83.9 | **90.3** | 88.8 |
| OSWorld-Verified (computer use) | 63.9 | **84.3** | 72.7 |
| GPQA Diamond (science reasoning) | 87.8 | 89.2 | **91.3** |

The jump from the previous generation is not incremental. On **SWE-bench Pro**, the score that measures solving real multi-file coding tasks, a 27B dense model lands at 61.7, which is **above the 53.4 that Opus 4.6 Max reports** in Qwen's own table. That is the single most interesting cell in the whole grid. On **OSWorld-Verified** (driving a real OS with a GUI) it leads Opus by eleven points.

But the honest reads are just as important:

- **Terminal-Bench 2.1** is where the gap shows. At 73.0 it's still a solid 7 to 8 points behind Opus (78.2). For the very longest autonomous install-test-fix loops that run dozens of steps, the frontier models still fail less.
- **GPQA Diamond** still favors Opus, 91.3 to 89.2.
- **DeepSWE** jumping from 13.3 to 42.2 is the signal that the agentic training is real and not a tuning exercise on one suite. A three-fold leap like that does not come from benchmark gaming.

The shape of it: strong on agentic coding and computer use, close to or above the cloud frontier on the tasks that actually matter for building software, and still a step behind on marathon terminal autonomy.

---

## Running It Locally (My Actual Setup)

I'm not running this through a cloud API. I serve it with **llama.cpp** on a separate box on my home network, exposed as an OpenAI-compatible endpoint, and it's the default model behind my agent. That's the setup this whole review is based on, so the "experience" numbers below are real usage, not a demo.

For the rest of you, the path is short. A GGUF quant plus llama.cpp is the setup I'd recommend:

```bash
# Download the quant plus the vision projector
huggingface-cli download unsloth/Qwen3.8-27B-GGUF \
  --include "Qwen3.8-27B-UD-Q4_K_XL.gguf" "mmproj-F16.gguf"

# Serve it as an OpenAI-compatible endpoint
llama-server \
  --model Qwen3.8-27B-UD-Q4_K_XL.gguf \
  --mmproj mmproj-F16.gguf \
  --ctx-size 128000 \
  --flash-attn on \
  --cache-type-k q8_0 \
  --cache-type-v q8_0 \
  --host 0.0.0.0 \
  --port 8033
```

Once it's up, point any OpenAI-compatible client at `http://<host>:8033/v1`. That's the whole deployment. If you'd rather serve the official **FP8** weights with vLLM or SGLang instead, both projects ship a day-one recipe for it. The model card recommends `temperature=1.0, top_p=0.95` for thinking mode and `temperature=0.7, top_p=0.80, presence_penalty=1.5` when you turn thinking off.

> 💡 **When to use thinking mode.** Keep it on for planning and debugging, where the reasoning trace earns its tokens. Turn it off for routine implement-and-fix loops, where you want speed and you're paying for every token in your own compute time. The `reasoning_effort` setting is a good knob: `xhigh` for the gnarly work, `low` when you just need a fast, competent edit.

---

## The Real Test: Building Poker Banting

Benchmarks tell you what a model can do. They don't tell you what happens over hours of back-and-forth on a project that's actually going to be used. So I gave it a real one.

**Poker Banting** is a four-player multiplayer card game, and I've been playing the actual cards since junior high. Not on a screen: in the classroom, deck of real cards and a handful of friends who were far worse at lying than they thought. The teacher caught us mid-trick, misread the whole situation, and told my parents I was a gambler. I still don't know what I was supposed to say to that.

It carried into senior high with the same energy. We'd deal in the corner of the classroom, and when a teacher walked in, the deck went under the sajadah, the prayer mat rolled against the wall. Nobody was in any danger. Everybody was dying of laughter.

The game itself: 13 cards each, a three-discard phase, then tricks where you must play a valid combo (single, pair, triple, straight, full house, or four of a kind) or pass. The first three players to empty their hand end the game with ranked scores. The fourth player is the one who left everyone else behind, and they pay for it.

That's why I chose it as the test project. It has no business value yet and it won't impress anyone. But I know its rules so well they're burned in. The discard phase, the combo edge cases, the penalty for finishing last. If a model butchers a detail of a game I've played for fifteen years, I'll notice immediately, and I'll know the SWE-bench number is lying. And if it survives, I get something I can actually play, for free, in a browser. Nostalgia is a surprisingly good test oracle.

And it's playable, by the way. [poker-banting.dodistyo.com](https://poker-banting.dodistyo.com) is mobile-first, free, and no signup. Up to four players per table. Bring your friends, or I'll just fill the seats with bots.

The stack is the interesting part, because it spans the exact kinds of work a coding model is supposed to handle:

| Layer | What it is |
|-------|-----------|
| Server | Rust (Axum + tokio) WebSocket game engine, **6,731 lines** |
| Client | Vanilla JS, no framework, mobile-first PWA, **3,061 lines** |
| Server tests | **214** unit + integration tests |
| Client tests | **115** unit tests + **21** Playwright e2e specs |
| History | **133** commits across two git repos |
| Packaging | Multi-stage Docker build ending in `FROM scratch` |
| Hosting | poker-banting.dodistyo.com (Cloud Run behind a Cloudflare tunnel) |

No framework on the client, an in-memory state model that I chose on purpose for now, and a container image so small it's basically a static binary. That last bit is not nothing. The build cross-compiles the Rust server to a static musl binary and drops it into a scratch image, because the app has no runtime and no dependencies to ship.

How it actually got built matters for the review, because the "one-shot prompt" story is not this one. There was no magic single prompt. I built the game in my spare time, one feature at a time. A task in my agent's Telegram chat, a review of what it shipped, repeat, over weeks. And the project has a two-generation history: the initial build was driven by **Qwen 3.6 27B** through the opencode harness. The BOMB and POKER rules I'll describe next were built by its older sibling, this Qwen 3.8 27B, driven through Hermes Agent from my phone. Same family, a generation apart, different harnesses. The newer one handled the trickier rules cleanly.

### A Feature That Wasn't a Hello-World

The game itself is straightforward: 13 cards each, and every trick you either outplay the table (same combo type, more cards, higher rank) or pass. The legal combos are single, pair, triple, straight (3 to 5 cards, same suit, 3-10 or exactly J-Q-K), full house, and four of a kind. Keep playing until your hand runs out. The first three players empty win their points; the one left holding cards pays for it.

The **BOMB**: four cards of the same rank (anything except 2), playable only as a response to a single 2. It beats everything and ends the trick. The trap a shallow pass would hit: four 2s is impossible by design, because that's exactly what triggers the **4×2 "poker" rule** at deal time. So the model had to add a **guarded deal** that silently re-deals the round whenever anyone is dealt four 2s, without desyncing the table. One change, four modules at once.

The bomb feature itself was worked test-first: failing tests up front, watched to fail, then driven green over the next few sessions. The suite landed at 214 server tests passing, with the client mirroring the same rules so a human and a bot can't disagree about what a legal move is.

That's the kind of task the SWE-bench Pro number is supposed to predict. A 27B model, on my hardware, coordinating a change across four modules in a language I don't write daily, held together. On the surface this is a simple card game. Under the hood it has a surprising amount of logic, and Qwen 3.8 27B delivered on it cleanly.

---

## Where It Holds Up, and Where I Still Steer

The fair review has a "but." Here's where it lived up to the benchmarks, and where it didn't.

**Where it was genuinely good:**
- Multi-file Rust it never had to ask me how to write. The idioms, the error handling, the test structure, all came out clean and it compiled on the first or second pass.
- Test-first discipline stuck when I kept it there. Write the red test, make it green, re-run. It followed that loop without being re-prompted every step.
- It caught edge cases I'd have to flag by hand, like the four-2s interaction with the deal guard.

**Where I had to stay on it:**
- **It thinks too much.** With thinking on by default, it will spend a lot of reasoning tokens on a change that could have been done in one pass. I think that's on purpose, and honestly it's the point: the slowness is a trade-off for accuracy and quality. On the gnarly rules it paid for itself, but for routine edits I turned thinking off and let it move.
- **The long-loop gap is real.** On the most involved multi-step builds, the kind that run dozens of tool calls in a row, I could feel the Terminal-Bench difference. It's a step behind the frontier models on marathon autonomy, exactly as the 7 to 8 point gap suggests.
- **Dense is slower than MoE.** Next to Qwen3.6-35B-A3B on the same box, the 27B is the slower one. For agentic work where I care about quality over raw tokens-per-second, that's a fine trade. For latency-sensitive stuff, I'd pick the MoE.

> 💡 **The working style that made it useful.** I treat the local model like a careful junior engineer who overthinks the easy things. Give it a clear spec, let it think where it counts, make it prove the work with tests, and verify every claim against real output. Do that and a 27B model on your own card does work that was cloud-only a year ago.

---

## The Takeaway

Qwen 3.8 27B is the strongest local coding backend I've run on a consumer GPU. It tops the Opus 4.6 Max number on the agentic coding benchmark, leads it on computer use, and it actually shipped a working multi-language game on my hardware, tests included. It is not the frontier model for the longest unattended loops, and it is dense, so it's not the fastest thing you can serve in that VRAM budget.

But the question this post started with has an answer now. Can you hand a 27B local model a real project and get software out the other side? On a 24GB card, with the model doing the drafting and me doing the verifying, yes. And it costs you nothing beyond the electricity.

### What's Next

Once you've got a local model driving real builds, the natural next steps:

- Try the same TDD loop on a second language or framework and see where the confidence holds
- Compare it head-to-head with the MoE Qwen3.6-35B-A3B on your actual tasks, not the leaderboard
- Point its vision at a UI screenshot and have it fix the layout it can see. That's the native-vision feature most people never test

The full build, server and client, is the proof. Run your own test the same way, and see what a model on your own hardware will actually ship.
