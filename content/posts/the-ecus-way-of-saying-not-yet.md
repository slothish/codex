---
title: "The ECU's way of saying \"not yet\""
date: 2026-04-06
draft: false
tags: ["ovms", "claude-code", "egolf", "can-bus"]
---

The first thing I did after setting up the repo was ask Claude to have a look at the e-Golf decoder code and the metrics I'd pulled from my own module. No specific question — just a first pass to see if anything looked off.

It flagged the cabin temperature. Pinned at 77°C.

That's not a plausible cabin temperature. It's hot enough to kill. My first thought was a unit mix-up — 77°F is about 25°C, which is a completely normal interior reading. Reasonable suspicion.

Reading the decoder killed that theory. The formula for frame `0x066E` is `d[4] * 0.5 - 50.0`. That's already in Celsius — there's no Fahrenheit conversion anywhere near it. If you get 77 out, you put 254 in. `0xFE * 0.5 - 50.0 = 77.0`. The math is right. The input is the problem.

`0xFE` is what the ECU broadcasts on that byte before the interior sensor is ready. It's not a decode error. It's the car saying "I don't have a value yet" — in the only language a CAN frame knows, which is a number.

The fix is two lines: discard the frame if `d[4] == 0xFE`, and clear the metric at startup so a previously persisted 77°C doesn't survive a reboot while waiting for real data. We wrote it, confirmed it, parked it. Climate control requires reverse engineering the BAP protocol first — no point shipping a cabin temp fix without the thing that actually uses cabin temp.

Then we kept finding them.

SoC reporting 127%. Battery current at 2047 amps. GPS coordinates at 134°N, 268°E — both impossible, both produced by the GPS module broadcasting `0xFF` bytes before it has a fix. Battery temperature at 87°C. Climate ECU values decoding to ~62°C. Every one of them `0xFE` or `0xFF` in the raw byte, every one of them the ECU's way of saying the sensor isn't ready yet.

It's a consistent pattern once you see it. The ECU doesn't have a null type. It has a reserved value at the top of each field's range — something no real measurement would ever produce — and it broadcasts that until the real data arrives. If your decoder doesn't know to discard it, you get a cabin temperature that would melt the dashboard and a state of charge that exceeds physics.

Claude spotted the first one. The rest were the same question asked of each new frame we decoded.
