---
layout: post
title: "Handle Interrupt for Chip 8"
date:   2016-09-27
categories: emulator
---

## Introduction ([Reference](http://devernay.free.fr/hacks/chip8/C8TECH10.HTM#keyboard))

Interrupt in chip 8 is very simple: signal from its keyboard. Chip 8 has a
16-key hexadecimal with the following layout:

|---|---|---|---|
| 1 | 2 | 3 | C |
| 4 | 5 | 6 | D |
| 7 | 8 | 9 | E |
| A | 0 | B | F |

To be able to emulate this, we just need to map this keyboard to our current
keyboard (in this example, I will use normal QWERTY keyboard)

## Implementation

In Chip8 instruction codes, there are two codes which relate to keyboard:

- 0xE?9E: ignore an instruction if a key is pressed
- 0xE?A1: ignore an instruction if a key is not pressed

Because Chip8 supports multiple keys press, we have to store an array to check
if a key is pressed or not:

```ruby
keys = Array.new(0xF)
```

In my [chip 8](https://github.com/hieuk09/chip_8), I use [gosu](https://www.libgosu.org/) for catching keyboard events. Gosu
`#button_up` and `#button_down` will return a variable represent the key which
is pressed or released. We need a way to convert this variable to the index of
the keys array:

```ruby
def button_up(key)
  key_char = Gosu.button_id_to_char(key)
  return unless key_char # we only process non-control keys

  button_name = KEY_MAP[key_char] # KEY_MAP is a map which map the keyboard keys to chip_8 keys, for ex: 'a' => 0xA
  return unless button_name # we only process valid keys

  keys[button_name] = 1
end
```

We can implement the instruction now:

```ruby
when 0xE000
  key = chip.registers[(opcode & 0x0F00) >> 8]

  case opcode & 0x00FF
  when 0x009E
    if chip.keys[key] != 0
      increase_program_counter
    end

  when 0x00A1
    if chip.keys[key] == 0
      increase_program_counter
    end
  end
# ...
```

The chip 8 program will response to your keyboard now :)

## Result

You can check out the code in [chip_8](https://github.com/hieuk09/chip_8/commit/86f5403154e9b53ff680cc79d89cac4ac2497033) and take a look.
