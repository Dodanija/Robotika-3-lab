
# Electronic Safe System

An Arduino-based electronic safe system with a four-digit PIN code, servo-controlled locking mechanism, LED indicators, and EEPROM storage.

The project includes two implementations: a physical Arduino Uno version with an 8x8 LED matrix and a Tinkercad simulation version using the Serial Monitor.

Both versions use the same PIN verification, locking, timing, and EEPROM functionality.

## Project Description

The system allows the user to enter a four-digit PIN using a potentiometer and a push button.

The potentiometer is used to select a digit from 0 to 9, while pressing the button confirms the selected digit. Four blue LEDs indicate how many digits have been entered.

After entering all four digits, the system compares the entered PIN with the code stored in EEPROM.

- **Correct PIN:** The servo rotates to 90 degrees, unlocking the safe.
- **Incorrect PIN:** An error indicator is activated for 1.5 seconds.
- **Three incorrect attempts:** The system is blocked for 30 seconds.
- **Automatic locking:** The safe locks again after 10 seconds.
- **PIN change:** Holding the button for 3 seconds while the safe is unlocked activates PIN change mode.

The new PIN is saved in EEPROM and remains stored after the Arduino is powered off.

The default PIN is `4826`.

## Project Versions

The repository contains two separate Arduino sketches.

### 1. Physical Arduino Version

This version is designed for a physical Arduino Uno circuit.

It uses a MAX7219 8x8 LED matrix to display:

- Selected digits from 0 to 9
- A check mark when the PIN is correct
- A cross when the PIN is incorrect
- A lock symbol when the system is blocked
- The letter C when PIN change mode is activated

Seven additional LEDs connected through an SN74HC595N shift register indicate PIN entry progress and system status.

### 2. Tinkercad Simulation Version

This version is designed for simulation in Tinkercad.

Since the MAX7219 module is not used in the simulation, the display functionality is implemented using the Serial Monitor.

The Serial Monitor displays:

- The currently selected digit
- Each confirmed PIN digit
- The number of entered digits
- LOCKED
- UNLOCKED
- WRONG PIN
- BLOCKED
- CHANGE PIN

All other functionality, including PIN verification, EEPROM storage, servo control, interrupts, and timing, remains the same as in the physical version.

## Components

- Arduino Uno
- Potentiometer
- Push button
- Servo motor
- SN74HC595N shift register
- 7 LEDs (4 blue, 1 green, 1 red, 1 yellow)
- 7 current-limiting resistors
- MAX7219 8x8 LED matrix (physical version only)
- Breadboard
- Jumper wires

## Wiring

### Arduino Connections

| Component | Arduino Pin |
|-----------|-------------|
| Potentiometer | A0 |
| Push button | D2 |
| Servo motor | D9 |
| 74HC595 DATA | D4 |
| 74HC595 CLOCK | D7 |
| 74HC595 LATCH | D8 |
| MAX7219 DIN | D3 |
| MAX7219 CLK | D6 |
| MAX7219 CS | D5 |

The MAX7219 connections apply only to the physical Arduino version.

### LED Indicators

The LEDs are controlled using the SN74HC595N shift register.

| Shift Register Output | Function |
|-----------------------|----------|
| Q0 | Not used |
| Q1 | First PIN digit entered |
| Q2 | Second PIN digit entered |
| Q3 | Third PIN digit entered |
| Q4 | Fourth PIN digit entered |
| Q5 | Safe unlocked |
| Q6 | Incorrect PIN |
| Q7 | System blocked |

## System Design

The system uses a non-blocking, state-based architecture without `delay()` calls.

Five system states are implemented:

- `ENTERING` - Waiting for PIN input
- `UNLOCKED` - Safe is unlocked
- `WRONG_PIN` - Incorrect PIN entered
- `BLOCKED` - System temporarily blocked
- `CHANGE_PIN` - Entering a new PIN

### Timer Interrupt

Timer2 is configured in CTC mode with a prescaler of 64.

It generates an interrupt every 1 ms and increments a counter used for timing operations.

The system uses the following timing intervals:

| Operation | Duration |
|-----------|----------|
| Potentiometer sampling | 100 ms |
| Button debounce | 25 ms |
| Long button press | 3 seconds |
| Incorrect PIN indication | 1.5 seconds |
| Automatic locking | 10 seconds |
| PIN change timeout | 10 seconds |
| System blocking | 30 seconds |

### External Interrupt

An external interrupt on Arduino pin D2 detects button presses.

The interrupt sets a flag, while the main loop processes button input and debounce logic.

A short press confirms the selected digit, while holding the button for 3 seconds when the safe is unlocked activates PIN change mode.

### Memory-Mapped I/O (MMIO)

The SN74HC595N shift register is controlled directly through the Arduino Uno registers:

- `PORTD` - DATA and CLOCK signals
- `PORTB` - LATCH signal
- `PIND` - Reading the push button state

This allows GPIO operations without using `digitalRead()` or `digitalWrite()`.

### EEPROM

EEPROM stores the four-digit PIN and a validity marker.

The `EEPROM.update()` function is used to avoid unnecessary writes when the stored value has not changed.

The saved PIN remains available after disconnecting the power supply.

## How to Use

### Physical Arduino Version

1. Connect all components according to the wiring diagram.
2. Open the physical Arduino sketch in Arduino IDE.
3. Install the `LedControl` library.
4. Select the Arduino Uno board.
5. Upload the code.
6. Use the potentiometer to select a digit displayed on the 8x8 LED matrix.
7. Press the button to confirm each digit.
8. Enter the correct four-digit PIN to unlock the safe.

To change the PIN, unlock the safe and hold the button for 3 seconds. Then enter four new digits.

### Tinkercad Version

1. Open the Tinkercad circuit.
2. Open the Code editor.
3. Insert the Tinkercad version of the Arduino sketch.
4. Start the simulation.
5. Open the Serial Monitor.
6. Select digits using the potentiometer.
7. Press the button to confirm each digit.
8. Observe the selected digits, confirmed digits, and system status in the Serial Monitor.

The Serial Monitor uses a baud rate of 9600.

## Future Improvements

Possible improvements include:

- Replacing the potentiometer with a numeric keypad
- Adding a physical enclosure for the safe
- Implementing a mechanical locking mechanism connected to the servo motor
- Adding an audible alarm for incorrect PIN attempts

## Technologies Used

- Arduino Uno
- C/C++
- Arduino IDE
- Tinkercad
- Servo library
- EEPROM library
- LedControl library
- AVR hardware timers and interrupts
- Memory-Mapped I/O (MMIO)
