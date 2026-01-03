# Adjusting Alloy Controller Parameters

### Firmware Parameter Adjustment Guide (Internal)

**Audience:** Assembly / firmware technicians

**Purpose:** Adjust visual behavior of a prebuilt lighting effect by changing parameters only

**Scope:** This guide explains **what values may be changed**, **what they do**, and **how to translate settings from the Attitude website into firmware code**.

---

## 1. What This Code Is

This firmware runs a **predefined lighting effect engine** (`AttitudeEngine3C`) on an Arduino-based controller.

* The **visual effect** is designed in the **Attitude website**
* For clients who do not use the website, we:

  1. Recreate the show in code
  2. Upload it to a simple hardware controller
* The technician’s job is **not to write new code**, but to:

  * Change parameters
  * Enter RGB color values
  * Adjust timing if needed

> **Rule of thumb:**
> If you don’t know what a line does, **don’t touch it unless this guide explains it**.

---

## 2. How This File Is Organized

At the top of this code is a section of comments explaining some info about the file. 

```
// Alloy Rotating Poodle Brush
// firmware for the alloy rotating poodle brush for mister, Jan 3rd 2026
// copyright 2025 Drew Shipps, J2 Systems
```

In code, a comment is any line that begins with two slashes, `//`. This line will be ignored by the computer and is only useful for humans, hence, it's a "comment" about the code.

The firmware has two main parts:

### `setup()`

Runs once when the controller powers on.

* Initializes DMX output
* Sets everything to black (off)

⚠️ **Do not modify anything in `setup()`**


### `loop()`

Runs continuously and contains:

* All effect parameters
* Color definitions
* Output mapping
* Timing control

✅ **All allowed changes happen here**

Within the loop, there is code to configure the attitude engine with the correct parameters for the show, as well as some code to output the show data to the fixtures through DMX. 

This is an example of the full loop. 

```
// loop
void loop() {
  // Set parameters
  Engine.setShowType(AE3C_CHASE);
  Engine.setSpeed(180);
  Engine.setSize(1);
  Engine.setDirection(AE3C_LR);
  Engine.setSplits(10);
  Engine.setTransition(AE3C_BOTH);
  Engine.setTransitionWidth(0);

  // Set color steps
  Engine.setColorStep(1, 0, 0, 255);
  Engine.setColorStep(2, 255, 255, 255);
  Engine.setColorStep(3, 0, 0, 255);
  Engine.setColorStep(4, 255, 255, 255);
  Engine.setColorCount(4);

  // run FX engine
  Engine.run();

  // using color mode RGBW
  int TOTAL_PIXEL_COUNT = 70;
  byte red, green, blue;

  // iterate over each RGBW pixel
  for (int i = 0; i < TOTAL_PIXEL_COUNT; i++) {
    // get pixel colors
    Engine.getFixtureColor(i+1, TOTAL_PIXEL_COUNT, red, green, blue);

    // set DMX values
    DMXSerial.write(i*3+1, red);
    DMXSerial.write(i*3+2, green);
    DMXSerial.write(i*3+3, blue);
  }

  // delay for timing of FX engine
  delay(0);
}
```

Notice the comments explaining the lines of code. Empty lines are ignored by the computer, so we can use them as spacers so that sections of code are easy to read. For example, the section to set parameters is easy to read and includes lines like `Engine.setSpeed(180);` and `Engine.setTransitionWidth(0);`.

Each line of code must end with a semicolon `;`. Forgetting a semicolon is one of the easiest mistakes to make in programming, and Arduino will give you an error if you make that mistake. 

---

## 3. Effect Parameters

### Translating Website Controls → Code

This section shows how settings from the Attitude website map directly to code.

All parameters are set using lines that start with:

```cpp
Engine.set...
```

In this section, you'll learn how to modify these lines so that you can create shows using code.

---

### 3.1 Show Type

| Website Label | Code Enum     |
| ------------- | ------------- |
| Static        | `AE3C_STATIC` |
| All           | `AE3C_ALL`    |
| Chase         | `AE3C_CHASE`  |
| Pulse         | `AE3C_PULSE`  |

**Code example:**

```cpp
Engine.setShowType(AE3C_CHASE);
```

**Example Use:**

To modify this to use the static show type, change the line of code above to be 

```cpp
Engine.setShowType(AE3C_STATIC);
```

Note how everything else is the same, except for the text within parenthesis. To change other parameters like speed, you'll also just change the value within parenthesis. Keep reading to see those additional parameters.

---

### 3.2 Speed (BPM)

* Controls how fast the effect moves
* Matches the **Speed / Tempo** control on the website

**Allowed range:** `10 – 180`

```cpp
Engine.setSpeed(180);
```

**Examples:**

* `60` → slow
* `120` → medium
* `180` → fast (maximum)

---

### 3.3 Size

* Controls how wide each color block appears
* Larger value = larger chunks of color

**Allowed range:** `1 – 200`

```cpp
Engine.setSize(50);
```

---

### 3.4 Direction

| Website Icon / Meaning | Code Enum      |
| ---------------------- | -------------- |
| Left → Right           | `AE3C_LR`      |
| Right → Left           | `AE3C_RL`      |
| Middle → Ends          | `AE3C_MID_END` |
| Ends → Middle          | `AE3C_END_MID` |
| Random                 | `AE3C_RANDOM`  |

```cpp
Engine.setDirection(AE3C_LR);
```

---

### 3.5 Splits

* Divides the strip into repeating sections
* Matches the **Splits** slider on the website

**Allowed range:** `1 – 10`

```cpp
Engine.setSplits(7);
```

---

### 3.6 Transition Type

| Website Icon | Code Enum       |
| ------------ | --------------- |
| Both         | `AE3C_BOTH`     |
| Leading      | `AE3C_LEADING`  |
| Trailing     | `AE3C_TRAILING` |

```cpp
Engine.setTransition(AE3C_TRAILING);
```

---

### 3.7 Transition Width (Fade)

* Controls how soft the transition is between colors
* Uses a decimal (floating-point) value

**Allowed range:** `0.0 – 1.0`

```cpp
Engine.setTransitionWidth(0.85);
```

**Examples:**

* `0.0` → hard edge
* `0.25` → light fade
* `0.5` → medium fade
* `1.0` → very soft fade

---

## 4. Color Steps

Color steps define **what colors appear and in what order**.

Colors come from the **Attitude website color picker**:

* Click a color and take note of the RGB color values listed
* Enter them into the code

---

### 4.1 Setting a Color Step

Each color step is a line of code, like so:

```cpp
Engine.setColorStep(stepNumber, red, green, blue);
```

On the Attitude website, each color step is a colored square you can click on. Here, it's a line of code specifying the step number and RGB values.

**Example:**

```cpp
Engine.setColorStep(1, 11, 60, 208);    // Alloy Blue
Engine.setColorStep(2, 255, 255, 255); // White
```

* `stepNumber` starts at **1**
* RGB values must be between **0 – 255**

---

### 4.2 Color Count (Required)

You **must** tell the engine how many color steps exist.

```cpp
Engine.setColorCount(2);
```

⚠️ If this number is wrong:

* Colors may not show correctly
* Effect may appear broken

This needs to match the number of color steps that you've put in. 

---

### 4.3 Looping Colors

If a customer specifies looping colors by putting in the same colors looping on the website, simply duplicate that behavior with each line of code. 

**Example loop:**

  * Blue → White → Blue → White

```cpp
Engine.setColorStep(1, 11, 60, 208);
Engine.setColorStep(2, 255, 255, 255);
Engine.setColorStep(3, 11, 60, 208);
Engine.setColorStep(4, 255, 255, 255);
Engine.setColorCount(4);
```

This would be valid code for the set color steps section.

---

## 5. Total Pixel Count

This must match the **physical number of fixtures or pixels**.

```cpp
int TOTAL_PIXEL_COUNT = 70;
```

Change this if the install has more or fewer fixtures. *As a general rule, better to have this number be too high than too low.* I'm totally fine with this number being roughly accurate and higher than the number of pixels on the strip. It doesn't have to be exact.


---

## 6. Timing / Speed Control (Frame Delay)

At the bottom of `loop()`:

```cpp
delay(20);
```

### What This Does

* Controls how often the effect updates
* Standard value: **20 ms**

### Allowed Values

| Delay | Result                      |
| ----- | --------------------------- |
| `20`  | Normal operation            |
| `10`  | Faster                      |
| `5`   | Very fast                   |
| `0`   | Maximum speed (intentional) |

⚠️ **Never exceed 20 ms**

---

## 7. Safe Editing Workflow

1. Create a copy of the firmware file provided by Drew. Make sure to copy the whole folder, then rename the folder and the file within it so that the names match. Make sure to include the date in the filename.
2. Update the comments on the first few lines of code to include any changes you've made or notes about this code
3. Change:

   * Parameters
   * Color steps
   * Pixel count (if needed)
4. Upload to controller using programmer
5. Verify visual output
6. Flash remaining units

---

**Document owner:** Drew Shipps, J Squared Systems

**Internal use only**

**Last updated:** January 3, 2026
