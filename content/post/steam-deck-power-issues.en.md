---
title: Steam Deck (OLED) Power issues
date: 2025-08-11
---

Yesterday, I finally carved out some time for *Kingdom Come: Deliverance 2*.

Unfortunately… my Steam Deck OLED **was dead**. I plugged it into a 200W
USB-C charger, expecting that friendly white LED blink, but instead—nothing.
LED off. Silent. Lifeless.

Here’s the journey I went through to bring it back to life.

<!--more-->


## First, the Usual Tricks

The go-to advice online is to enter the BIOS: hold the volume-up button and
the power button for about 12 seconds. Result? Nothing. No light, no sound,
no signs of life.

Okay—plan B:  
- Try a different charger (I tested two, both over 100W)  
- Swap to a known-good cable (rated for at least 45W) 
- Leave it charging for a while  

Still nothing.


## A Trip to MacPodpora

I stopped by [MacPodpora] in Prague Smíchov, since they handle both warranty
and post-warranty repairs. If your Deck is under warranty, they simply send
it to Valve — which I could do myself — but they did provide me a clue:

> My Deck was actually **drawing a stable charge** the whole time.  
> It just wouldn’t turn on, and the LED stayed dark.

So I messaged Valve.

*Note to myself:* Buy USB-C charger with display for debugging.


## Valve’s Secret Firmware Reflash

Three hours later, Valve replied with a fix. Here’s their procedure to
reflash the Deck’s firmware and BIOS:

1. **Power off completely**: Hold the power button for 10 seconds.
2. **Button combo**: Hold **volume-down** and the **• • •** button, then press
the power button once. Release all buttons.  
3. **Watch for the signs**: After a chime, the white LED will start blinking
(can take 5+ seconds). This means the process is working.

**Note:** The Deck may take 1–2 minutes to start up, and the display will remain black until it’s done.


## Back From the Dead

After running those steps, my Steam Deck powered on like nothing had happened.
Firmware reflashed, system restored.

Big thanks to Despoina from Valve support for the clear instructions!


## A Word of Caution

If you’re leaving your Deck idle for a while, **actually power it off** instead of just hibernating it. Letting the battery drain completely seems to have triggered my issue.

[MacPodpora]: https://macpodpora.cz/apple-servis-oprava/herni-konzole/steam-deck/nenabiji-nejde-obraz-usb-c/
