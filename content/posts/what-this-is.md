---
title: "What this is"
date: 2026-04-06
draft: false
tags: ["ovms", "claude-code", "egolf"]
---

I've been running a 2019 VW e-Golf since late last year. It came with Car-Net — VW's remote access app. Nothing fancy, but useful: you could start the climate pre-conditioning, check the charge level, see where the car was. It worked well enough that you missed it when it was gone.

VW pulled the plug on Car-Net when mobile operators started decommissioning their 2G networks. The app ran over 2G. In Sweden, Telenor went down first of December last year — VW had built on Telenor's infrastructure, so that was that. Telia still has 2G running until 2027, but it didn't matter. The back end was gone.

I'd come across OVMS before — Open Vehicle Monitoring System — in the context of home automation research. The idea is an open hardware device that plugs into the OBD port and gives you real remote access to your car. Not a cloud subscription, not dependent on the manufacturer keeping the lights on. But there was no e-Golf support, so I never went further with it.

Losing Car-Net changed that calculus. I decided to try and contribute e-Golf support myself. The problem: I haven't written C++ in years. My day job is enterprise systems deployment and process development — I haven't been close to low-level code in a long time.

I'd used Claude Code on small personal projects before. I decided to try it here, on something genuinely hard — contributing to an established open source project in a language I'm rusty in, on hardware I was reverse engineering as I went.

This blog is about that experience. Specifically: how to use Claude Code in a way that produces results worth something. Not generated slop — the OVMS maintainer has seen plenty of that and is rightly skeptical. But actual useful contributions, made faster than I could have done alone.

What works, what doesn't, and what you actually need to bring yourself.
