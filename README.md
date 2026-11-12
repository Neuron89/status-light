# status-light

A desk-side status indicator that mirrors a Home Assistant controlled "do not disturb" lamp, built from an ESP32 board, an RGB LED, and the smart button that already drives the lamp.

<p align="center">
  <img src="assets/Button1.jpeg" width="260" alt="Finished build: XIAO ESP32-C6 and RGB LED mounted on the smart button">
</p>

## Problem

An RGB lamp outside a home office shows whether the occupant can be disturbed. The lamp is set from a wireless smart button through Home Assistant automations (single press, double press, long press). The person pressing the button cannot see the lamp from the desk, so the status is easily left on the wrong colour.

## Solution

A small microcontroller is wired to the smart button's own switch contact. It decodes the same press gestures that Home Assistant acts on and lights a local RGB LED in the corresponding colour, so the desk indicator and the lamp outside the room stay in step.

The firmware does not talk to Home Assistant over the network. Synchronisation comes from both systems reading the same physical button: Home Assistant receives the press from the smart button and sets the lamp, while the ESP32 reads the shared contact and sets the LED.

## Hardware

| Part | Notes |
|---|---|
| Seeed Studio XIAO ESP32-C6 | Any CircuitPython-capable board works; adjust the pin names in the code |
| RGB LED (common cathode) | One GPIO per colour channel, driven active-high; a current-limiting resistor per channel is recommended |
| Wireless smart button | The button already paired with Home Assistant; its switch contact is tapped as the input |
| Hook-up wire, heat shrink | |

### Wiring

| Board pin | Connects to |
|---|---|
| `D0` | Smart button switch contact (input, internal pull-up, active low) |
| `D2` | LED red channel |
| `D3` | LED green channel |
| `D4` | LED blue channel |
| `GND` | LED common cathode and smart button ground |
| `3V3` | Smart button 3 V rail |

<p align="center">
  <img src="assets/pinout.png" width="500" alt="XIAO ESP32-C6 pinout">
</p>

The ESP32 and the smart button share a ground so that the button's contact pulls `D0` low when pressed. Verify the polarity of the button's contacts with a multimeter before soldering.

## Firmware

The firmware is a single CircuitPython script, [`Src/Code.py`](Src/Code.py), using only the built-in `board`, `digitalio`, and `time` modules.

| Gesture | Detection | LED colour |
|---|---|---|
| Single press | Released in under 0.8 s, no second press within 0.25 s | Green |
| Double press | Second press within 0.25 s of the first release | Red |
| Long press | Held for 0.8 s or longer | Blue |

The LED is off at power-up and holds the last selected colour until the next gesture. To change the colour mapping so that it matches a different set of Home Assistant automations, edit the `set_color(r, g, b)` calls in `detect_button_press()` and `check_double_tap()`.

### Installation

1. Install CircuitPython on the board, following Adafruit's [CircuitPython on ESP32 quick start](https://learn.adafruit.com/circuitpython-with-esp32-quick-start/web-serial-esptool).
2. Copy `Src/Code.py` to the board as `code.py`.
3. If a different board or pin assignment is used, update the four pin constants at the top of the script.
4. Reset the board. The script runs automatically.

## Home Assistant integration

Home Assistant needs no extra configuration for the indicator itself. The existing setup is:

- the smart button paired with Home Assistant, and
- one automation per gesture (single, double, long press) that sets the lamp to a colour.

Keep the firmware's gesture-to-colour table aligned with those automations. Because the indicator tracks button presses rather than lamp state, a colour change made from the Home Assistant UI or another automation is not reflected on the desk LED.

## Build notes

The first prototype was breadboarded with a separate push button. The final version removes that button and solders three leads (3 V, GND, and the switch signal on `D0`) to the smart button's circuit board instead, so one press drives both systems.

<p align="center">
  <img src="assets/CircutBoard.jpeg" width="420" alt="Leads soldered to the smart button circuit board">
</p>

<p align="center"><em>Early wiring attempt; the positive and negative leads are reversed in this photo.</em></p>

Running the whole assembly from the smart button's 3 V coin cell was also tested. The ESP32 booted, but the cell could not supply enough current to drive the LED at the intended brightness, so the coin cell is not a suitable sole power source; power the board over USB-C for full brightness.

<p align="center">
  <img src="assets/Button2.jpeg" width="200" alt="Assembled unit">
  <img src="assets/Button3.jpeg" width="200" alt="Assembled unit, alternate view">
</p>

<p align="center">
  <img src="assets/Blue.jpeg" width="200" alt="LED showing blue">
  <img src="assets/Red.jpeg" width="200" alt="LED showing red">
  <img src="assets/Green.jpeg" width="200" alt="LED showing green">
</p>

## Limitations and possible improvements

- The indicator follows button gestures, not the lamp's actual state. Subscribing to the lamp entity over Wi-Fi (for example with ESPHome or MQTT) would make it authoritative.
- The XIAO ESP32-C6 supports Zigbee and Thread, so the board could replace the smart button entirely.
- The LED is bare; an enclosure with a diffuser would improve visibility and durability.

## Project structure

```
Src/Code.py    CircuitPython firmware
assets/        Pinout diagram and build photos
```
