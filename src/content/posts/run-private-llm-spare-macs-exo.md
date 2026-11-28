---
title: "Run a Private LLM on Spare Macs with EXO"
description: "Six retired MacBook Airs and the free EXO project can run a 30B model no single laptop holds — privately, with zero API fees. Setup, speed, and traps."
date: 2026-11-25
lang: en
category: tutorials
tags: ["EXO", "Private LLM", "Local AI", "Apple Silicon", "MacBook", "Open Source"]
cover: tool
coverImage: "/images/tools/private-llm-spare-macs/home.svg"
featured: false
draft: false
---

**If you have two or more spare Apple silicon Macs, you can pool them into one private AI cluster that runs a model too big for any single machine — for free, with nothing leaving your office.** Quickchat AI, a small software team, ran a 30-billion-parameter model across six retired MacBook Airs at about 20 tokens per second, and published every step including what went wrong. The catch is speed: clustering makes models bigger, not faster, so it is only worth doing when the model does not fit on one Mac.

That trade-off is the whole story. This guide explains how the clustering works, walks through the setup, lists the five traps that waste the most time, and gives you honest numbers so you can decide before you start digging cables out of a drawer.

## What EXO does

EXO is an open-source project from exo labs that connects your devices into a single AI cluster. It splits a large language model across several machines, so each one holds part of the model and they answer together. The project has over 48,000 stars on GitHub and is licensed under Apache 2.0, which means you can use it commercially without paying anything.

The idea is simple: a transformer model is a stack of layers, and layers can be split across machines. Machine A holds the first layers, machine B holds the next ones, and a token's activations flow through one machine, hop across the network, and continue through the next. Your separate Macs behave — logically — like one machine with their memory added together.

Three features do the heavy lifting:

- **Automatic device discovery.** Macs running EXO find each other on the same local network with no manual configuration. Start the app on each machine and they join the same cluster.
- **Topology-aware splitting.** EXO looks at each device's memory and the speed of each link, then assigns each machine a share of the model proportional to what it can hold. A 64 GB Mac Studio carries more layers than a 16 GB MacBook Air, automatically.
- **One compatible API.** The cluster exposes a dashboard and an OpenAI-compatible API at `http://localhost:52415`, and it also speaks Anthropic's Claude Messages format and the Ollama API. Most tools that work with the OpenAI API can point at your cluster instead of a cloud provider — including the [AI coding assistants](/posts/best-ai-coding-assistant-vscode/) many developers already use, and the [meeting notetakers](/posts/best-ai-meeting-notetakers/) that turn transcripts into notes.

Under the hood on a Mac, EXO runs on Apple's MLX framework, which uses the Metal GPU. That detail matters: on macOS, EXO uses the GPU; on Linux, EXO currently runs on CPU only, with GPU support still in development, so Apple silicon is where the project is genuinely good. The maintainers' own showcase runs pool four M3 Ultra Mac Studios to serve models like DeepSeek v3.1 at 671 billion parameters — hardware most people will never own, but the same software runs on laptops you already have.

## The six-MacBook test, as Quickchat AI ran it

The clearest demonstration of the spare-hardware case comes from Quickchat AI, a small software team that moved its development work to a remote product and ended up with six idle MacBook Airs, each with an M2 chip and 16 GB of memory. Sixteen gigabytes is not much for a language model, but six of them add up to 96 GB — enough for a fairly large one, if they can share it.

They connected the six Airs in a line with five Thunderbolt cables (each Air has two Thunderbolt ports), installed EXO on all of them, and loaded Qwen3-30B-A3B, a 30-billion-parameter mixture-of-experts model that takes about 16 GB in the team's setup. That model is too big for any one of those laptops on its own. That was the point.

The timeline is refreshingly ordinary. The first two Macs answered together at 13:12, after the setup was handed to Claude Code over SSH. All six were answering as one cluster by 13:54 — before lunch. The cluster wrote about 20 tokens per second, faster than a person can read, and correctly listed the first 40 prime numbers. Short questions got quick answers; long documents took a minute or more before the first word appeared, because the model reads the whole prompt before it starts writing.

The same test also produced the most useful part: a list of everything that went wrong in the team's own write-up. The cables took most of the time, not the software. That is normal for this kind of project, and the traps are predictable once you know them.

## How to set it up

You need at least two Apple silicon Macs, a Thunderbolt cable for each link between them, and about an evening. The macOS app is the easiest path; installing from source is the alternative for people who want the latest code.

**Step 1: Install EXO on every Mac.** Download the app from the project's release page, or install it with Homebrew using `brew install --cask exo`. The app needs a Mac with Apple silicon and macOS Sequoia 15.1 or later. Run the same version on every machine: devices running different versions of EXO do not join each other, and the dashboard will tell you a device is on another version.

**Step 2: Connect the Macs.** Plug them together with Thunderbolt cables for the fastest link. EXO also works over Wi-Fi or Ethernet, but Thunderbolt gives much lower latency between machines, and latency is the bottleneck for this whole approach. A MacBook Air has two Thunderbolt ports, so machines can be daisy-chained in a line; a Mac Studio has more ports and more options.

**Step 3: Allow local network access.** When you first open EXO, macOS asks whether it may find devices on your local network. Say yes. This is the step the Quickchat team's setup tripped over: the permission was off, the two Macs silently ignored each other, and nothing explained why. If discovery fails, check System Settings → Privacy & Security → Local Network on every Mac and make sure EXO is switched on. Running EXO as a root system service also works around it on machines you only reach over SSH.

**Step 4: Confirm the cluster in the dashboard.** Open `http://localhost:52415` on any Mac. Every device should appear in the cluster view within a few seconds. If a Mac does not show up, check the version match first, then the Local Network permission, then whether the network itself blocks devices from seeing each other — guest Wi-Fi and some office networks do, and a Thunderbolt cable bypasses that.

**Step 5: Download a model and try it.** Pick a model from the dashboard and start with something modest to verify the cluster works. The model downloads to every Mac — EXO has each node download the whole model even though each one uses only its share. On a fast connection that is a minor annoyance; on office Wi-Fi it can take over an hour per machine, so download once and copy the files to the other Macs over Thunderbolt instead. In the Quickchat team's write-up, copying the model to the other five machines over the cables took about 30 seconds, while the slowest machine still had 80–90 minutes of Wi-Fi downloading left.

**Step 6: Point your tools at the cluster.** EXO serves an OpenAI-compatible API on every Mac, so any app that accepts a custom base URL can use it: set the base URL to `http://localhost:52415/v1` and pick the model you loaded. Chat clients, coding assistants, and scripts that already talk to the OpenAI API generally work unchanged. If you would rather skip local setup entirely, a cloud workspace such as [Google AI Studio](/posts/google-ai-studio-build-android-app/) runs the same kind of model without any hardware of your own.

## Five traps that waste the most time

| Trap | What happens | Fix |
| --- | --- | --- |
| A charging cable, not a Thunderbolt cable | The cable fits the same port and looks identical, but carries little or no data. One link in the Quickchat team's test showed a cable plugged in and no data at all. | Test each cable before you build the chain. A real Thunderbolt cable is usually marked with a lightning symbol; if a link shows no data, swap the cable first. |
| The "Allow accessory to connect?" prompt | A Mac asks this when something new is plugged in. Nobody had clicked Allow on the retired laptops, so two of them saw each other only as plain USB devices. | Walk to each Mac and click Allow when you plug the cable in. Headless machines need this done in person once. |
| Local Network privacy is off | macOS blocks EXO from talking to other devices without showing any error. Each Mac decides it is a cluster of one. | Turn EXO on in System Settings → Privacy & Security → Local Network on every Mac. Running EXO as a root system service also works around it on machines you only reach over SSH. |
| Different EXO versions | Devices on different versions do not join each other. EXO flags the mismatch in the dashboard and logs it in `~/.exo/exo_log/exo.log`. | Update every Mac together. The app and a source checkout number their versions differently, so mixing install methods in one cluster needs a shared cluster name — and still the same version — to join at all. |
| Downloading the model six times | EXO downloads the full model onto every node over your internet connection, even though each node uses only its share. | Download once, then copy the model files to the other Macs over Thunderbolt. In the team's run, copying took about 30 seconds instead of the 80–90 minutes of Wi-Fi downloading the slowest machine still had left. |

None of these is hard. Most of them fail quietly — nothing crashes, the machines just sit disconnected or stall, with no error pointing at the cause — which is why they eat evenings.

## The honest speed math

This is where you should decide whether to bother. Clustering Macs pools memory, not speed, and the link between machines is far slower than the memory inside one machine. The blog Think Different spent two weeks testing exactly this setup, and its published measurements put numbers on the gap: inside one Mac Studio, the M2 Ultra's unified memory delivers around 800 GB/s to the GPU, while a Thunderbolt 4 bridge between two Macs delivered roughly 2.5–3 GB/s of usable throughput — about 250 times slower.

The same publication's measured generation speeds for the same 70-billion-parameter model at 4-bit quantization tell the same story:

| Setup | Model | Measured speed |
| --- | --- | --- |
| One M2 Ultra Mac Studio, 128 GB, MLX | Llama 3.3 70B, 4-bit | ~11.5 tokens/sec |
| Two Macs (M2 Ultra + M3 Max), EXO over Thunderbolt bridge | Llama 3.3 70B, 4-bit | ~8.2 tokens/sec |
| Two Macs (M2 Ultra + M3 Max), EXO over 10 GbE | Llama 3.3 70B, 4-bit | ~6.9 tokens/sec |
| Six M2 MacBook Airs, 16 GB each, EXO | Qwen3-30B-A3B (~16 GB) | ~20 tokens/sec |

Two notes on reading this table. First, the six-Air result used a smaller model, so it is not directly comparable to the 70B rows — it shows what a six-node cluster of weak machines can do, not that six machines beat one strong one. (Speeds as measured and published by Think Different for the 70B rows, and by Quickchat AI for the 30B row.) Second, the project's own benchmarks claim up to 1.8x speedup on two devices and 3.2x on four, but those numbers come from tensor-parallel runs with RDMA over Thunderbolt 5 on Mac Studios, a much faster interconnect than the Thunderbolt 4 bridge in the independent test. On ordinary cables, expect the cluster to be slower per token than the best single machine you own.

The rule that falls out of their measurements is simple: **the math only flips when the model does not fit on any single machine you own.** If a model fits on one Mac, run it there — it will be faster. If it does not fit anywhere, and the alternative is not running it at all, the cluster turns hardware you already own into capability for free. Buying new Macs to build a cluster makes no sense: two Mac Studio M2 Ultra units with 128 GB each cost close to five figures combined, and a single machine with that memory runs the same model faster.

## EXO vs the alternatives

EXO is not the only way to run a model locally. Which tool fits depends on how many machines you have and how much setup you want to do.

| Tool | Best for | How it works | Trade-off |
| --- | --- | --- | --- |
| **EXO** | Two or more spare Macs; a model too big for one machine | Splits one model across machines automatically; OpenAI-compatible API | Slower per token than one strong Mac; Apple silicon is the mature path, Linux currently runs on CPU |
| **Ollama** | One machine; the simplest local setup | Runs open models locally on one Mac, PC, or Linux box with a single command | Limited to what fits on that one machine; no clustering |
| **llama.cpp RPC mode** | Mixed hardware; people comfortable with manual setup | A main node offloads layers to worker machines over the network | More configuration than EXO, but works across Mac, Linux, and Windows hardware |
| **Apple MLX directly** | One strong Mac; maximum speed on Apple silicon | Runs models straight on the Metal GPU, with a built-in OpenAI-compatible server (`mlx_lm.server`) and distributed inference primitives | A multi-Mac cluster needs manual configuration — no auto-discovery or GUI like EXO |
| **vLLM** | Linux servers with NVIDIA GPUs | Production-grade serving for high-throughput inference | Needs real server hardware; overkill for a home or small-office cluster |

If you have one good Mac, start with Ollama or MLX. If you have several idle Macs and a model that will not fit on any one of them, EXO is the fastest way to use them together. If your hardware is NVIDIA cards on Linux, use vLLM or llama.cpp RPC instead — EXO is not there yet.

## What it is good for — and what it is not

A private cluster earns its keep in three situations. First, **privacy**: the model runs entirely on your machines, so nothing you ask it leaves the office. That matters for internal documents, client work, and anything you would not paste into a cloud chatbot. Second, **cost**: after the hardware you already own, there are no per-token fees, which makes it cheap for heavy batch work. Third, **batch jobs**: summarizing documents, extracting structured data from archives, or processing the day's meeting transcripts do not care about per-token latency, so a slow cluster doing overnight work is genuinely useful.

It is not good for interactive chat where every token of delay is felt, and it is not a reason to buy hardware. Long prompts are the worst case: the model reads the whole prompt before writing anything, so a long document can take a minute or more before the first word appears. If your main use is chatting with a model, a cloud API or a single strong Mac will feel better.

There is also a maintenance cost that the glossy version skips: every Mac in the cluster needs the same EXO version, the same cluster name, and the same model files, and an update on one machine drops it out of the cluster until the rest catch up. For a home lab that is a minor chore. For a team depending on it daily, it is a small operations job.

## The bottom line

Spare Macs plus EXO is a real, working way to run a private model that no single machine you own can hold — and the six-MacBook test Quickchat AI published proves it can be done before lunch, cables included. Go in with the right expectation: you are buying memory, not speed. If the model fits on one Mac, run it there. If it does not, and the Macs are already sitting in a drawer, EXO turns sunk cost into a private AI service for the price of a few Thunderbolt cables and an evening of setup.

Start with two Macs and a small model. Verify they see each other, watch the dashboard, and benchmark honestly against one machine before you commit to a bigger cluster. Budget an evening for your first try — the published six-Mac run came together in under an hour once the cables were sorted, but the cable drawer is where most of a first-timer's time goes.

*EXO is under active development, and setup steps, system requirements, and performance claims may change. Check the project's GitHub page for the current version and requirements before installing.*
