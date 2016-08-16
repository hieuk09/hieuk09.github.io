---
layout: post
title: "NES specification - reading note - part 1"
date:   2016-08-16
categories: [emulator]
---

# Note on NES specification

## Introduction

NES is Nintendo Entertainment System, a gaming console which is popular in 1980s

## Hardware Overview

Since I'm reading the specification while implementing the emulator for Chip8,
      there will be some comparison between them.

### Processing Unit

- NES has 2 processing units, which are CPU & PPU (Picture Processing Unit).
  Chip8 has only one processing unit.
- Each chip has its own memory (RAM) and registers
- CPU memory load operation code while PPU load sprites and rendering
  information

### Memory

- NES memory bus is 16-bit number, so it can control 16KB memory. However,
  memory size of a Processing unit in NES is actually 32KB (from 0x00000 to
  0x10000).
- While Chip8 only has 2 sections in memory (0x000-0x200 for default
  sprites, the rest for loading data), memory in NES CPU is a lot more
  complicated. It has 11 sections.
- There are 3 mirrors section which mirrors a part of the memory for
  quicker execution (Memory locations 0x0000-0x07FF are mirrored three times
  at 0x0800-0x1FFF).
- ROM data is loaded by page into RAM instead of loading it one time to
  overcome the limitation of memory. This allows the NES ROM to be much more
  complicated than what the 16KB memory allows to do. Compared to this,
  Chip8 loads all ROM data into memory at once, so it's much simpler.

### Registers

- It has less general purpose registers than Chip8, with only 2 of them
  (compared to 16).
- Other specific one (like program counter, stack counter, ...) is pretty
  much the same.
- The status register is 8-bit, 7 bits are used for storing status
  (compared to Chip8, it only has 1-bit register I for status)
