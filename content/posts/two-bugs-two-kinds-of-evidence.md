---
title: "Two bugs, two kinds of evidence"
date: 2026-04-06
draft: false
tags: ["ovms", "claude-code", "egolf", "can-bus", "gps", "bap"]
---

The cabin temperature post was about recognising a pattern. These two are about what happens after — writing code that's actually correct, and knowing when it is.

## The GPS frame

Frame `0x486` decodes latitude and longitude from four bytes each. The original code had a `TODO` where the hemisphere sign bits should be, and wrote the metrics unconditionally — no check on whether the values made sense first.

The DBC showed 55 of 64 bits in use. The remaining bits were the sign flags. Bit 55 negates latitude for the southern hemisphere. Bit 56 negates longitude for the western hemisphere. The original code never read either of them.

There was also the sentinel problem. When the GPS module has no fix it broadcasts all `0xFF` bytes, which decode to 134°N and 268°E. Both are geometrically impossible. The old code wrote those to the position metrics before clearing the GPS lock flag — so a fresh boot with no fix would silently overwrite whatever valid position had been stored.

I'd already noticed something was wrong from the car side. The module was firing tow alerts and the position was jumping around. I hadn't had time to dig into it. Claude found the same bug independently, reading the code — no hardware, no alerts, just the DBC and the decoder.

We wrote four tests: a real capture frame producing the correct coordinates, a sentinel frame that should clear the lock without touching the metrics, and one each for southern and western hemisphere captures with the sign bits set.

The sentinel test failed on the first run. The range check was in the right place but the metric writes were still inside the old conditional block — gpslock cleared, coordinates still written. The failure output made the mistake obvious. One line, all 22 tests green.

That's the clean loop. Laptop, test harness, five minutes.

## The clima TX drop

Climate control over BAP required three CAN frames on the extended-ID channel: a multi-frame header, a continuation, and a single-frame trigger. There is no unit test for this. The test is whether the car starts pre-conditioning.

It took two or three button presses before it did.

`WriteExtended` returns an `esp_err_t`. We were discarding it. The first diagnostic step was adding logging to capture it — not a fix, just evidence collection. That was itself a commit, because understanding the failure mode was the first deliverable.

With verbose logging on and the car connected, the first BAP frame came back `ok1=ESP_ERR_TIMEOUT`. The TWAI TX queue fills briefly during the 500ms wake settle window — OCU heartbeat frames firing while the bus is coming up. The default `maxqueuewait=0` means any frame that can't be enqueued immediately is silently dropped. No error event. No indication anything went wrong. Just a button press that didn't work.

The fix: `pdMS_TO_TICKS(20)` on all three `WriteExtended` calls. Block briefly instead of dropping.

Same session turned up a second bug. `ms_v_env_hvac` was checking `remote_mode == 3`, but a full 15-minute capture showed the ECU only emits mode 3 for the first few seconds — steady state is mode 2. The condition was wrong. Changed it to `remote_mode != 0`.

## What the two look like together

The GPS case is tidy. Write the test, watch it fail, read the output, fix the line. The whole loop runs on the laptop.

The clima case is slower. The "test" is a log line from a device in a car park. Adding return-value logging was a commit because it was the work — you can't fix what you can't see. The fix followed once the evidence was in.

Both follow the same thing underneath: one change per commit, the reason in the message, no fix without a clear cause. The GPS case just had the luck of a test harness. The clima case had to go find its own evidence first.
