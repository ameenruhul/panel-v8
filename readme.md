# Code Explanation

This code is for a microcontroller project controlling a process (likely a dyeing or similar process) with temperature and time control, displayed on a TFT screen with touch input and keypad interaction.  Here's a breakdown of each line of code:

## Includes

*   `#include <FS.h>`: Includes the File System library, allowing the program to access the SPIFFS file system for storing calibration data.
*   `#include <SPI.h>`: Includes the SPI (Serial Peripheral Interface) library, necessary for communication with the TFT screen and potentially the SD card (if used for SPIFFS).
*   `#include <TFT_eSPI.h>`: Includes the TFT_eSPI library, providing functions for controlling the TFT screen.
*   `#include <OneWire.h>`: Includes the OneWire library, used for communication with the DS18B20 temperature sensors.
*   `#include <DallasTemperature.h>`: Includes the DallasTemperature library, providing functions for reading temperature data from the DS18B20 sensors.
*   `#include <Wire.h>`: Includes the Wire library, used for I2C communication (needed for EEPROM access).
*   `#include "Free_Fonts.h"`: Includes a custom header file containing definitions for free fonts used for text rendering on the TFT screen.
*   `#include <Keypad.h>`: Includes the Keypad library, used for reading input from a 4x4 keypad.
*   `#include <Arduino.h>`: Includes the core Arduino library.

## Enumeration and State Variables

*   `enum TouchState { ... }`: Defines an enumeration called `TouchState` with possible values representing different states of the touch interaction.
    *   `STATE_IDLE`: The initial state, where the system is waiting for input.
    *   `STATE_CONFIRM_WAIT`:  Waiting for confirmation after an action is initiated (e.g., Auto mode).
    *   `STATE_AUTO_PROCESS`:  Executing the auto mode process.
    *   `STATE_RESET_PROCESS`: Executing the reset process.
    *   `STATE_MANUAL_SAVE`: Saving manual settings.
    *   `STATE_PHASE_START`: Starting a phase in phase mode.
*   `TouchState currentState = STATE_IDLE;`: Declares a variable `currentState` of type `TouchState` and initializes it to `STATE_IDLE`. This variable tracks the current state of the application.
*   `unsigned long stateStartTime = 0;`: Declares a variable `stateStartTime` to store the timestamp (in milliseconds) when the current state was entered. Used for timeouts.

## TFT Initialization

*   `TFT_eSPI tft = TFT_eSPI();`: Creates an instance of the `TFT_eSPI` class named `tft`. This object will be used to control the TFT screen.

## Touch Calibration

*   `#define CALIBRATION_FILE "/TouchCalData1"`: Defines a constant string `CALIBRATION_FILE` containing the filename for storing touch calibration data in the SPIFFS file system.
*   `#define REPEAT_CAL false`: Defines a boolean constant `REPEAT_CAL`. If `true`, the touch calibration process will run every time the program starts. If `false`, it will only run if the calibration file doesn't exist.
*   `uint16_t x = 0, y = 0;`: Declares two unsigned 16-bit integer variables `x` and `y` to store the coordinates of the touch event.
*   `int textWidth = 60;`: Defines the width of text areas.
*   `int textHeight = 28;`: Defines the height of text areas.
*   `#define offset_row 20`: Defines a vertical offset for drawing elements on the screen.
*   `#define offset_col 0.5`: Defines a horizontal offset for drawing elements on the screen.
*   `#define LINE tft.setCursor(offset_col, offset_row + tft.getCursorY())`: Defines a macro `LINE` to set the cursor position for text output, adding an offset to the current cursor position.

## Home Key Definitions

*   `char homeKeyLabel[4][7] = {"MANUAL", "AUTO", "PHASE", "RESET"};`: Defines a 2D character array `homeKeyLabel` containing the labels for the four home keys (Manual, Auto, Phase, Reset).
*   `#define CUSTOM_PURPLE tft.color565(255 - 244, 255 - 92, 255 - 250)`: Defines a macro `CUSTOM_PURPLE` to represent a custom purple color using the `tft.color565()` function.
*   `#define CUSTOM_LIGHT_PINK tft.color565(255 - 92, 255 - 250, 255 - 122)`: Defines a macro `CUSTOM_LIGHT_PINK` for a light pink color.
*   `#define CUSTOM_LIGHT_RED tft.color565(255 - 79, 255 - 87, 255 - 249)`: Defines a macro `CUSTOM_LIGHT_RED` for a light red color.
*   `#define CUSTOM_LIGHT_BLUE tft.color565(255 - 248, 255 - 79, 255 - 79)`: Defines a macro `CUSTOM_LIGHT_BLUE` for a light blue color.

## Relay and Home Key Instances

*   `#define relay1 15`: Defines a constant `relay1` representing the pin number connected to the first relay.
*   `#define relay2 4`: Defines a constant `relay2` representing the pin number connected to the second relay.
*   `uint16_t homeKeyColor[4] = {CUSTOM_PURPLE, CUSTOM_LIGHT_PINK, CUSTOM_LIGHT_RED, CUSTOM_LIGHT_BLUE};`: Defines an array `homeKeyColor` containing the colors for the four home keys.
*   `TFT_eSPI_Button homeKey[4];`: Declares an array `homeKey` of four `TFT_eSPI_Button` objects. These objects represent the four home keys on the TFT screen.

## Manual Keypad Definitions

*   `TFT_eSPI_Button manualKeypadKey[16];`: Declares an array of 16 `TFT_eSPI_Button` objects for the keypad buttons.
*   `char* manualKeypadKeyLabel[16] = { ... };`: Defines an array of character pointers containing the labels for each keypad button.
*   `TFT_eSPI_Button manualKey[4];`: Declares an array of four `TFT_eSPI_Button` objects for the manual control keys (HOME, SAVE, START, RESET).
*   `char* manualKeyLabel[4] = { ... };`: Defines an array of character pointers containing the labels for the manual control keys.
*   `TFT_eSPI_Button manualPhaseKey[4];`: Declares an array of four `TFT_eSPI_Button` objects for the manual phase keys (YELLOW, LAMINA, COLOR F., STEAM).
*   `char* manualPhaseKeyLabel[4] = { ... };`: Defines an array of character pointers containing the labels for the manual phase keys.

## Phase Key Definitions

*   `TFT_eSPI_Button phaseKey[4];`: Declares an array of four `TFT_eSPI_Button` objects for the phase keys.
*   `char* phaseKeyLabel[6] = { ... };`: Defines an array of character pointers containing the labels for the phase keys.
*   `uint16_t phaseKeyColor[6] = { ... };`: Defines an array of colors for the phase keys.

## Confirm Key and Touch Variables

*   `TFT_eSPI_Button confirmKey[2];`: Declares an array of two `TFT_eSPI_Button` objects for the confirmation keys (YES, NO).
*   `uint16_t t_x = 0, t_y = 0;`: Declares variables to store the x and y coordinates of the touch event.

## Field Color Controller

*   `byte selected_manual_field = 1;`: Declares a byte variable `selected_manual_field` to keep track of the currently selected field for manual input (1 for dry temp, 2 for wet temp, 3 for time).

## Manual Field Strings

*   `String manual_dry_temp_str = "";`: Declares a string variable `manual_dry_temp_str` to store the dry temperature input.
*   `String manual_wet_temp_str = "";`: Declares a string variable `manual_wet_temp_str` to store the wet temperature input.
*   `String manual_time_str = "";`: Declares a string variable `manual_time_str` to store the time input.

## Phase Time Value String

*   `String phase_time_value_str = "";`: Declares a string variable `phase_time_value_str` to store the phase time value.

## Home Page Time Strings

*   `String home_cur_time_str = "00:00:00";`: Declares a string variable `home_cur_time_str` to store the current time.
*   `String home_limit_time_str = "00:00:00";`: Declares a string variable `home_limit_time_str` to store the limit time.

## Phase and Mode Labels

*   `char* phase_label[5] = { ... };`: Defines an array of character pointers containing the labels for the different phases.
*   `char* mode_label[4] = { ... };`: Defines an array of character pointers containing the labels for the different modes.

## Phase Parameters

*   `const byte phase_temp[4][14][2] = { ... };`: Defines a 3D array `phase_temp` containing temperature values for each phase and hour.
*   `const byte phase_duration_hour[5] = { ... };`: Defines an array `phase_duration_hour` containing the duration in hours for each phase.
*   `const byte phase_duration_min[5] = { ... };`: Defines an array `phase_duration_min` containing the duration in minutes for each phase.
*   `const byte phase_duration_sec[5] = { ... };`: Defines an array `phase_duration_sec` containing the duration in seconds for each phase.

## Keypad Variables

*   `const byte ROWS = 4;`: Defines the number of rows in the keypad.
*   `const byte COLS = 4;`: Defines the number of columns in the keypad.
*   `char hexaKeys[ROWS][COLS] = { ... };`: Defines a 2D character array `hexaKeys` representing the layout of the keypad.
*   `byte rowPins[ROWS] = { ... };`: Defines an array `rowPins` containing the pin numbers connected to the rows of the keypad.
*   `byte colPins[COLS] = { ... };`: Defines an array `colPins` containing the pin numbers connected to the columns of the keypad.
*   `Keypad keypad = Keypad(makeKeymap(hexaKeys), rowPins, colPins, ROWS, COLS);`: Creates an instance of the `Keypad` class named `keypad`, initializing it with the keypad layout and pin connections.

## Temperature Sensor Variables

*   `#define temp1 12`: Defines the pin number connected to the first temperature sensor.
*   `#define temp2 13`: Defines the pin number connected to the second temperature sensor.
*   `OneWire oneWire1(temp1);`: Creates an instance of the `OneWire` class named `oneWire1` for the first temperature sensor.
*   `OneWire oneWire2(temp2);`: Creates an instance of the `OneWire` class named `oneWire2` for the second temperature sensor.
*   `DallasTemperature sensor1(&oneWire1);`: Creates an instance of the `DallasTemperature` class named `sensor1` for the first temperature sensor.
*   `DallasTemperature sensor2(&oneWire2);`: Creates an instance of the `DallasTemperature` class named `sensor2` for the second temperature sensor.
*   `float tempF[2] = {0, 0};`: Declares an array `tempF` to store the temperature readings in Fahrenheit.
*   `String dry_temp_str = " --";`: Declares a string variable `dry_temp_str` to store the dry temperature reading as a string.
*   `String wet_temp_str = " --";`: Declares a string variable `wet_temp_str` to store the wet temperature reading as a string.
*   `String set_dry_temp_str = " --";`: Declares a string variable `set_dry_temp_str` to store the set dry temperature reading as a string.
*   `String set_wet_temp_str = " --";`: Declares a string variable `set_wet_temp_str` to store the set wet temperature reading as a string.

## Controller Variables

*   `uint8_t pre_interface = 0;`: Stores the previous interface.
*   `uint8_t selected_interface = 1;`: Stores the currently selected interface (1: Home, 2: Manual, 3: Phase).
*   `uint8_t selected_phase_phase = 0;`: Stores the selected phase.
*   `uint64_t cur_time = 0;`: Stores the current time in milliseconds.
*   `uint64_t pre_time = 0;`: Stores the previous time in milliseconds.
*   `uint64_t pre_eeprom_time = 0;`: Stores the previous time when EEPROM was updated.
*   `uint8_t second = 0;`: Stores the current second.
*   `uint8_t minute = 0;`: Stores the current minute.
*   `uint8_t hour = 0;`: Stores the current hour.
*   `uint8_t limit_sec = 0;`: Stores the limit second.
*   `uint8_t limit_min = 0;`: Stores the limit minute.
*   `uint8_t limit_hour = 0;`: Stores the limit hour.
*   `int limit_db = 0;`: Stores the dry bulb temperature limit.
*   `int limit_wb = 0;`: Stores the wet bulb temperature limit.
*   `byte selected_mode = 0;`: Stores the selected mode (0: None, 1: Auto, 2: Phase, 3: Manual).
*   `byte selected_phase = 0;`: Stores the selected phase.
*   `byte selected_pre_phase = -1;`: Stores the previously selected phase.
*   `bool cur_time_flag = false;`: Flag to indicate if the current time is valid.
*   `byte temp_phase_select = 0;`: Stores the selected phase for temperature control.

## EEPROM Address Definitions

*   Defines integer variables to represent EEPROM addresses for storing various settings.

## EEPROM Functions

*   `void writeIntToEEPROM(byte deviceAddress, byte memAddress, int data) { ... }`: Writes an integer value to the EEPROM at the specified address.
*   `int readIntFromEEPROM(byte deviceAddress, byte memAddress) { ... }`: Reads an integer value from the EEPROM at the specified address.
*   `void check_eeprom() { ... }`: Reads settings from EEPROM and initializes variables.

## Helper Functions

*   `bool flip = true;`: A boolean variable used to alternate between two EEPROM storage locations.
*   `void touch_calibrate() { ... }`: Calibrates the touch screen.
*   `String time_to_str(uint8_t h, uint8_t m, uint8_t s) { ... }`: Converts time values (hour, minute, second) to a string in the format "HH:MM:SS".
*   `void home_touch() { ... }`: Handles touch events on the home screen.
*   *   `void manual_touch() { ... }`: Handles touch events on the manual control screen.
*   `void update_field(char ch) { ... }`: Updates the manual input fields based on the character entered.
*   `void field_change_update() { ... }`: Updates the display of manual input fields.
*   `void phase_touch() { ... }`: Handles touch events on the phase control screen.
*   `void update_phase_field() { ... }`: Updates the display of the selected phase.
*   `void update_phase_time_field() { ... }`: Updates the display of the phase time.
*   `void phase_screen() { ... }`: Draws the phase control screen.
*   `void print_phase_select_label() { ... }`: Prints the label for the phase selection.
*   `void print_phase_select_value() { ... }`: Prints the selected phase value.
*   `void print_phase_manual_time() { ... }`: Prints the label for the phase time.
*   `void print_phase_manual_time_value() { ... }`: Prints the phase time value.
*   `void draw_phase_key() { ... }`: Draws the phase control buttons.
*   `void manual_screen() { ... }`: Draws the manual control screen.
*   `void print_manual_dry_label() { ... }`: Prints the label for the dry temperature input.
*   `void print_manual_dry_value() { ... }`: Prints the dry temperature value.
*   `void print_manual_wet_label() { ... }`: Prints the label for the wet temperature input.
*   `void print_manual_wet_value() { ... }`: Prints the wet temperature value.
*   `void print_manual_time_label() { ... }`: Prints the label for the time input.
*   `void print_manual_time_value() { ... }`: Prints the time value.
*   `void draw_manual_keypad() { ... }`: Draws the keypad buttons.
*   `void draw_manual_key() { ... }`: Draws the manual control buttons.
*   `void draw_manual_phase_key() { ... }`: Draws the manual phase control buttons.
*   `void confirmation_screen() { ... }`: Draws the confirmation screen.
*   `void confirmation_label() { ... }`: Prints the confirmation label.
*   `void draw_confirm_key() { ... }`: Draws the confirmation buttons.
*   `bool return_confirm() { ... }`: Waits for confirmation input.
*   `void home_screen() { ... }`: Draws the home screen.
*   `void print_db_label() { ... }`: Prints the dry bulb temperature label.
*   `void print_db_temp() { ... }`: Prints the dry bulb temperature value.
*   `void print_wb_label() { ... }`: Prints the wet bulb temperature label.
*   `void print_wb_temp() { ... }`: Prints the wet bulb temperature value.
*   `void print_dbset_label() { ... }`: Prints the set dry bulb temperature label.
*   `void print_dbset_temp() { ... }`: Prints the set dry bulb temperature value.
*   `void print_wbset_label() { ... }`: Prints the set wet bulb temperature label.
*   `void print_wbset_temp() { ... }`: Prints the set wet bulb temperature value.
*   `void print_curtime_label() { ... }`: Prints the current time label.
*   `void print_curtime() { ... }`: Prints the current time.
*   `void print_limittime_label() { ... }`: Prints the limit time label.
*   `void print_limittime() { ... }`: Prints the limit time.
*   `void print_phase_label() { ... }`: Prints the phase label.
*   `void print_phase() { ... }`: Prints the phase.
*   `void print_mode_label() { ... }`: Prints the mode label.
*   `void print_mode() { ... }`: Prints the mode.
*   `void draw_home_keypad() { ... }`: Draws the home screen keypad.
*   `void interface_control() { ... }`: Switches between different interfaces (Home, Manual, Phase).
*   `void touch_control() { ... }`: Handles touch input based on the current interface.
*   `void handleStateMachine() { ... }`: Implements a state machine to manage different application states.
*   `void handleEEPROMUpdate() { ... }`: Writes settings to EEPROM.
*   `void handleTemperatureSensors() { ... }`: Reads and processes temperature sensor data.
*   `void handleTimerUpdate() { ... }`: Updates the time.
*   `void warning_section(int ct) { ... }`: Plays a warning sound.
*   `void temp_control() { ... }`: Controls the relays based on temperature readings.
*   `void phase_control() { ... }`: Controls the phase based on time and settings.
*   `void resetPhaseControl() { ... }`: Resets the phase control settings.
*   `void initializePhaseTime() { ... }`: Initializes the phase time.
*   `void sensor_warning() { ... }`: Displays a warning if temperature sensors are not responding.
*   `void temp_measure() { ... }`: Reads temperature from sensors.

## Setup Function

    *   `void setup() { ... }`: Initializes the hardware and software.
    *   `limit_db = 0; limit_wb = 0;`: Initializes temperature limits to 0.
    *   `delay(1000);`: Waits for 1 second.
    *   `Serial.begin(115200);`: Initializes serial communication at 115200 baud.
    *   `Wire.begin();`: Initializes the I2C communication.
    *   `tft.init();`: Initializes the TFT screen.
    *   `tft.setRotation(0);`: Sets the screen rotation.
    *   `tft.fillScreen(TFT_BLACK);`: Clears the screen with black color.
    *   `check_eeprom();`: Reads settings from EEPROM.
    *   `keypad.addEventListener(keypadEvent);`: Attaches the `keypadEvent` function to the keypad event.
    *   `touch_calibrate();`: Calibrates the touch screen.
    *   `pinMode(relay1, OUTPUT);`: Sets relay1 pin as an output.
    *   `pinMode(relay2, OUTPUT);`: Sets relay2 pin as an output.
    *   `pinMode(buzzer, OUTPUT);`: Sets the buzzer pin as an output.
    *   `digitalWrite(buzzer, LOW);`: Turns off the buzzer.

## Loop Function

    *   `void loop() { ... }`: The main loop of the program.
    *   `cur_time = millis();`: Gets the current time in milliseconds.
    *   `if(cur_time_flag && selected_interface == 1 && (selected_mode == 1 || selected_mode == 2) && alarmable){ ... }`: Updates temperature limits based on the selected phase if in auto or phase mode.
    *   `interface_control();`: Switches between interfaces.
    *   `touch_control();`: Handles touch input.
    *   `handleStateMachine();`: Executes the state machine.
    *   `handleEEPROMUpdate();`: Updates EEPROM with settings.
    *   `keypad.getKey();`: Reads the keypad input.
    *   `phase_control();`: Controls the phase.
    *   `temp_measure();`: Measures temperature.
    *   `handleTemperatureSensors();`: Handles temperature sensor readings.
    *   `temp_control();`: Controls the relays based on temperature.
    *   `if(cur_time - pre_time >= 900 && selected_interface == 1 && cur_time_flag){ ... }`: Updates the time every 900 milliseconds.

This code provides a comprehensive control system for a process, with a user-friendly interface on a TFT screen, touch and keypad input, and temperature control.  The state machine adds robustness and structure to the application logic.  The EEPROM storage allows for persistent settings.