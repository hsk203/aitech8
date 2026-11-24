---
title: "Pika Enact: Direct AI Video Live, While It Plays"
description: "Pika Enact turns AI video into live directing: drag actors, speak notes, steer the camera while the scene plays. A research preview you can't use yet."
date: 2026-11-22
lang: en
category: tutorials
tags: ["Pika", "Pika Enact", "AI video", "Pika H3 Flex", "real-time video"]
cover: tool
coverImage: "/images/tools/pika-enact/home.svg"
featured: false
draft: false
---

**On November 18, 2026, Pika published a research preview called Pika Enact — and ordinary users cannot use it yet.** Instead of writing a prompt, waiting for a clip, and starting over, you direct the scene while it plays: drag an actor across the frame, speak a note to the cast, swing the camera around the room, and the video redraws itself under your hands.

Pika's official blog calls Enact an experimental live-streaming environment built on Pika H3 Flex, generating video and sound in sub-second chunks and folding each new instruction into the next one. Pika says plainly that it is a research preview, unavailable for consumer use, with no public waitlist or price. Currently the only published route in is an email to the company's enterprise team.

It is a research layer on top of an existing model, not a new model and not a product. Even so, it attacks the exact part of the workflow that wastes creators' time: the prompt-and-wait loop.

## The problem Enact is trying to solve

Every [AI video tool](/posts/best-ai-video-generator-for-youtube-shorts/) in common use today works the same way. You describe a shot, the model renders a finished clip, you watch it, you change a word, and you render again. The waiting has shrunk to seconds, but the rhythm has not changed: sample, judge, resample. The first ten seconds of a thirty-second ad usually come easily; then the client asks for the product turned toward the camera and warmer light, and each note means a new render that changes everything slightly, so the version you liked is gone.

A film set works the other way around: the scene stays in front of you while you adjust one thing at a time: move the chair two feet left, run the line again with more anger, push in for a close-up. The note and the result happen in the same moment, and everything you are not changing stays put. A prompt describes a result; directing describes a correction. Enact is built for the second job, where most professional time actually goes.

## What Pika Enact is

Pika describes Enact as a system designed to mimic the experience of directing on set: a research layer on top of Pika H3 Flex, the video-and-audio model Pika released in late October 2026 (Pika's post-trained version of MiniMax's H3, per launch coverage), which generates in six denoising steps instead of fifty and ships with open weights. For comparison, see how [today's leading video models stack up](/posts/higgsfield-vs-runway-vs-kling/).

Where H3 Flex normally produces finished clips, Enact turns it into a stream: the model generates video and sound in short chunks, each one carrying the scene's context forward, so new direction can be folded into the ongoing generation instead of restarting it.

When the technical sections below mention a "real-time student model," that is the distilled, few-step version of H3 Flex itself, not a separate new model.

That streaming design makes the three interaction modes possible, one for each job a director names on set: staging, performance, camera.

## Mode 1: Live Staging — drag the scene with your mouse

Live Staging lets you place and reposition actors or objects inside a running video with your mouse. The scene keeps playing while you drag. Pika's blog shows people being lifted, moved, and set down, and a person brought into a city-square scene that originally had nobody there. The training data even covers inserting new subjects from photos into live scenes.

One thing Pika's post does not say: whether Live Staging generates or preserves audio. The mode is described in video terms only ("mouse in, video out"), while sound is discussed for the system as a whole and for Modes 2 and 3.

Three things make this different from dragging a layer in an editor. Placement is precise: where a prompt can only suggest "a bit to the left," a cursor puts the subject on the exact spot. The model regenerates the moved subject together with the rest of the scene, so pose, lighting, and shadows adapt, and the place it came from is redrawn instead of left as a hole.

There are no cut-out edges and no compositing pass afterwards. The subject also reacts to being moved, shifting weight and adjusting posture mid-air before settling where you let go.

The speed figures here are Pika's own claims from its own tests, and they should be read that way. Pika says the model generates the video 1.8 times faster than it plays on a single server. In a low-latency setting, the picture responds in under 0.4 seconds on H100 hardware, decoding included. Pika does not disclose how many GPUs that test used.

To keep up with the cursor, Pika distilled the model to a few sampling steps and streams mouse input and decoding in small pieces. Generation runs in 0.7-second chunks by default, or 0.17 seconds in low-latency mode.

The training data behind this is synthetic by necessity. Real footage cannot provide consistent before-and-after video pairs synchronized with frame-level cursor movements, so Pika built a physics-driven synthetic data engine. It animates a 3D stand-in along a drag path to simulate weight and momentum, then renders photorealistic target clips while logging every mouse position.

## Mode 2: Live Actor Direction — talk to the cast

Once a scene is staged, the next step on a real set is directing the cast. Enact turns the people in the video into actors you can talk to. You type or speak a note (Pika's examples are "move closer to her," "tell him why you're upset," and "make it a kitchen at night") and the actors play it out while the video keeps running, answering in character, in their own [AI-generated voices](/posts/best-ai-voice-generators/).

Pika says picture and sound generate 1.75 times faster than they play, so the conversation happens at a real-time pace. That rate is Pika's own lab figure, not an independent measurement.

Each actor, according to Pika's blog, is an AI agent with a distinct personality, its own logic, and a memory of the scene. They remember your notes, what you liked, what you did not like, and everything that has happened so far. You never repeat a note or start from scratch.

Behind the curtain, the job splits in two. A script model turns each note into structured beats (action, dialogue, emotion, gaze, lighting) while a real-time video model performs them. When no instruction is queued, the scene holds naturally, with subtle breathing and blinking.

The workflow mirrors a rehearsal: you work through a performance line by line, and every line is logged verbatim. When it feels right, you call "Action," and the script model rewrites the whole exchange into one seamless performance, shot fresh from the first frame. The rehearsal is the draft; the take is clean.

## Mode 3: Live Camera Control — steer the lens while it rolls

The third mode is the camera. Enact lets you steer a virtual camera with your mouse while a scene is continuously generating (panning, dollying in for a close-up, or orbiting around the subjects) instead of waiting for an offline render to see whether a move works. You shape upcoming frames as the scene unfolds.

Unlike post-hoc editing, this mode needs no existing video. The system generates audio and video from scratch. An appearance image anchors the cast and the set, while a lightweight 3D motion proxy (Pika's example is an untextured Blender viewport) guides the camera path.

Training data came from an automated Blender-to-Video pipeline that records camera moves, motion references, video frames, and spatial audio together.

The performance figures Pika cites for this mode are the most specific, and the most revealing about the hardware. On a node with eight Nvidia B300 accelerators, at 640 by 480 pixels and 24 frames per second, Pika says the model renders the first frames in 0.51 seconds and generates subsequent chunks at 1.50 times real-time speed.

That works out to 0.473 seconds of compute per 0.708-second chunk, by Pika's own arithmetic. Audio is generated alongside the video, so as the camera moves away from a sound source, the scene is supposed to sound different too.

## The numbers, with their caveats

Every figure in this section comes from Pika's own tests. None has been independently verified, because the product is not public.

The three headline rates look comparable. They are not, and Pika's own post makes that clear if you read the details. Staging is measured as response latency on H100 hardware, with Pika not disclosing how many GPUs were used. Camera control is measured as time to first frames on an eight-chip B300 node at 640 by 480. Actor direction is a combined video-and-audio rate with no hardware disclosed.

Independent coverage makes the same point: the figures use different measures and hardware, so "1.8×" and "1.5×" do not describe the same workload, and neither tells you what a session costs.

| Mode | Pika's claimed rate | Hardware and measure Pika cites |
| --- | --- | --- |
| Live Staging | 1.8× real time | Under 0.4 s response on H100 hardware (GPU count not disclosed), decoding included |
| Live Actor Direction | 1.75× real time, video and audio | No latency figure or hardware disclosed |
| Live Camera Control | 1.50× real time at 640×480, 24 fps | 0.51 s to first frames on 8× Nvidia B300 |

Two caveats matter more than the rates. First, the resolution: 640 by 480 is the only resolution Pika discloses for the real-time path, and Pika does not disclose the staging or actor-direction resolutions at all. That is a long way from the 1080p and 4K outputs creators actually deliver.

Second, the quality: in [an interview with The Neuron](https://www.theneuron.ai/newsletter/watch-pika-lets-you-direct-ai-actors-live/) on November 18, 2026 ("The Neuron: AI Explained," episode 82), Pika CEO Demi Guo said the real-time model produces somewhat lower image quality than Pika's non-real-time version. Independent coverage of the announcement reports the same tradeoff. That frames what Enact is for right now: a directing environment, not a finishing environment. A director who approves a take live and re-renders it offline at full quality has still gained something, but the live output is not the deliverable.

The hardware is the other half of the story. H100 hardware is datacenter class, not a laptop; eight B300 accelerators is a rack. Whatever Enact becomes commercially, it will not run on your machine. Any future product would be a cloud service, with the compute cost reflected in a price Pika has not announced, because there is no product yet.

## How the streaming works

A normal video model generates a whole clip at once. Enact turns H3 Flex into a stream by generating short causal chunks, keeping per-second compute and memory cost bounded during extended sessions. The streaming design differs by mode, because the modes start from different inputs.

Live Staging streams chunks through a sliding key-value cache that retains the prompt, the source video, the first chunk, and the four most recent chunks. The source video stays in the cache because staging starts from an existing clip that the creator is rearranging.

Live Actor Direction sits between the two. Its cache attends to the opening anchor frame and the recent seconds of context, keeping characters, sets, and native voices consistent across long takes.

Live Camera Control has no source video to retain. It attends to the opening anchor frame and a sliding history window, and it caches static appearance prefixes across chunks. That is what lets it generate from scratch while keeping characters and room geometry consistent across long orbits.

The second design choice is distillation. The real-time student model (the distilled, few-step version of H3 Flex, not a separate model) uses only a few sampling steps per chunk. It is trained with self-forcing distribution matching distillation, generating chunks from its own past outputs under the guidance of the full-clip teacher model. For staging, that cuts sampling from twenty steps to a few while keeping cursor tracking sharp.

## What a creator can and cannot do with it today

Today, the honest answer is: you cannot do anything with Enact directly. There is no public build, no waitlist, and no price. Pika's blog says it is a research preview, unavailable for consumer use, and invites anyone interested to email enterprise@pika.art. That tells you who it is for right now: studios, production companies, and platform builders, not individual creators.

What you can do today is narrower, but real:

- **Study the demos.** Pika's blog post includes demo videos for all three modes; watching a drag redraw a background, or a rehearsal become a clean take, teaches you more than any spec sheet.
- **Try the model underneath.** H3 Flex, the model Enact is built on, shipped with open weights in late October 2026, so it is the part of this story you can actually run today. Launch coverage notes it produces clips up to 15 seconds with synchronized stereo audio at 480p or 768p natively. That is Enact's ceiling until the real-time path improves. A useful first test is a short two-line dialogue scene: if the voices and lip sync hold together across the full 15 seconds, the foundation Enact depends on is solid. If they drift, you have found the current limit before betting a project on it.
- **Prepare for the workflow, not the tool.** The skill Enact rewards is directing: blocking a scene, giving a playable note, choosing a camera move. Those are the same skills that improve your results with today's prompt-based tools. If you make product videos or short ads, practice describing a shot as a sequence of corrections rather than a single description. That is the habit Enact assumes.
- **Do not plan a project around it.** No release date, no pricing, and no public access means Enact cannot be part of a production schedule. Treat it as a research signal, not a tool in your stack.

## Who it is for, and what to watch next

In the near term, Enact's natural audience is advertising and previsualization. A thirty-second product spot is exactly the kind of work where a client gives twenty small notes, and where adjusting one layer of a shot without re-rendering everything else saves real money. Previz is another fit: a director could block and rehearse a scene with AI actors before a single day of shooting is booked. Longer-form storytelling is further away: nothing in the preview shows a scene holding together across minutes.

Three things will tell you whether Enact becomes a product or stays a demo. First, whether a session can be saved, resumed, and shared: a tool nobody can hand off is a toy, however impressive the live interaction. Second, whether audio direction and camera moves stay in sync when both change at once, which is the failure mode live systems tend to hide in the transitions. Third, whether the quality gap closes: if the approved take still has to be re-rendered offline, Enact is only a convenience layer, and its value is the time it saves.

Others are moving the same way, and the pattern is worth naming. Google's Flow is built around shot-by-shot AI filmmaking, assembling a scene from generated clips instead of writing one prompt per shot. Adobe's Project Live Set explores live scene editing, and Luma's Modify Video and Ray3 Modify let creators swap an actor or element inside a finished shot instead of re-rendering it from scratch.

Each of these automates one step of the old prompt-and-wait loop. Enact tries to replace the loop itself. None of them will replace a film set this year, but the direction is unmistakable, and Enact is its clearest version so far: not a better prompt, but a viewfinder you can talk to.

## The bottom line

Pika Enact does not give creators a new tool today. It gives them a credible picture of what the tool after next looks like: video generation you steer while it plays, in three modes (staging, performance, camera), each running faster than real time on datacenter hardware, at modest resolution, with somewhat lower quality than offline generation. The performance figures are Pika's own lab numbers, measured three different ways, and should be treated as a research claim, not a benchmark. The access situation is equally plain: enterprise contact only, no public release in sight.

If you make video for a living, the useful response is not to wait for Enact. It is to notice what it assumes about you: that you think in shots, notes, and camera moves, not in paragraphs of prompt text. Then start working that way with the tools you already have. When a live directing environment does reach creators, the people who already direct will know what to do with it. For the announcement itself, with demos for each mode, [Pika's official blog post](https://pika.art/blog/pika-enact) is the source to read.
