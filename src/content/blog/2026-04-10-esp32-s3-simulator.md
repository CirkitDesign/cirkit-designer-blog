---
title: "We Built an ESP32-S3 Simulator That Runs Real Firmware in the Browser"
description: "Cirkit Designer now includes an instruction-accurate ESP32-S3 simulator written in Rust/WASM. It boots real firmware locally in your browser with support for GPIO, SPI, I2C, NeoPixels, Wi-Fi, and more."
pubDate: 2026-04-10
heroImage: "/images/posts/esp32_crypto_demo.gif"
author: "Austin Small"
tags: ["announcement", "esp32", "simulation"]
---

Today we're opening up the first beta of our ESP32-S3 simulator in Cirkit Designer.

Try the demos:

- [Crypto Price Tracker](https://app.cirkitdesigner.com/project/e6abd77f-c55f-40f5-bdf9-b503e3135672/6): Wi-Fi + HTTPS
- [Hotel Safe](https://app.cirkitdesigner.com/project/b6be54fa-7669-46eb-be47-9345e9f43241): keypad, peripherals, and control logic
- [DHT11 Temperature on LCD Display](https://app.cirkitdesigner.com/project/a4a2a875-89e6-4fac-97e4-6af6418be9a4): sensor + display, classic IoT starter project
- [Joystick-Controlled LED Matrix](https://app.cirkitdesigner.com/project/5032dceb-3bc6-4fce-8ce7-d40207aaac7b): bit-banged SPI with MAX7219
- [All ESP32-S3 examples](https://learn.cirkitdesigner.com/platforms/esp32-s3#example-circuits)

> **Note:** The editor UI is currently optimized for desktop. Mobile support is coming.

## How It Works

This is not a mocked "ESP32-like" environment. It's an instruction-accurate ESP32-S3 emulator written in Rust and compiled to WebAssembly. It executes real Xtensa instructions locally in your browser, booting through the ESP32-S3 ROM and bootloader just like real hardware. Arduino sketches compile on our server, and the compiled firmware runs unmodified in the simulator. No hardware required.

We validate the CPU core with golden tests against QEMU and integration tests that load and run real Xtensa binaries. That's how we catch the kind of problems that only show up in the ugly parts of the architecture, like windowed registers and RETW edge cases.

## What's Supported

The current beta supports:

- **GPIO, UART, SPI, I2C, PWM, ADC, hardware timers**
- **NeoPixel / WS2812** output through RMT
- **Wi-Fi**: 802.11 layer emulation with working HTTP, HTTPS, MQTT, WebSocket, and UDP support

See the [full list of supported peripherals](https://learn.cirkitdesigner.com/platforms/esp32-s3/#simulation-features).

## Why We Didn't Use Backend QEMU

A big part of this project was deciding what *not* to fake. We considered a backend QEMU approach. It looked like the fastest path and would have let us piggyback on existing MCU support.

But we didn't want the simulator living on the backend and streaming state into the UI over WebSockets. We wanted the simulation itself to live next to the UI, so circuits stay interactive, latency stays low, and the cost of running a project doesn't scale with server-side compute.

That decision made the implementation much harder. But the result is a simulation that runs entirely in your browser with nothing to spin up, nothing to wait for, and no per-session server costs.

## What Was Hardest

- **Booting through Espressif's proprietary ROM**: undocumented and tricky to get right
- **Xtensa windowed registers**: handling the register window model and RETW edge cases correctly
- **Browser performance**: keeping the emulator interactive inside a circuit editor
- **Wi-Fi**: the useful abstraction wasn't "pretend HTTP works." It was emulating the chip's 802.11 layer closely enough that the firmware's own network stack could do the rest, with very limited documentation to work from

Wi-Fi ended up being one of the most interesting parts of the project. A lot of the work came down to reverse engineering undocumented parts of Espressif's Wi-Fi stack, then matching those expectations closely enough that real firmware would stay happy.

## Built by a Tiny Team

This was built by a very small team: just me, Austin Small, and my dad, Vince Small. It took about 8 months to get to this beta. There are definitely rough edges. We'd love to hear what works, what breaks, and what peripherals or workflows you'd want to see supported next.

You can try the simulator in [Cirkit Designer](https://app.cirkitdesigner.com), browse the [ESP32-S3 docs](https://learn.cirkitdesigner.com/docs/platforms/esp32-s3), and share feedback in [Discord](https://discord.gg/2R2DY37VpE).
