# rp2040-ctcss
Create CTCSS tones in Micropython or C++ with a PIO machine on the RP2040. The output signal must be filtered with a low-pass filter.

# How to use in Micropython
See example in `ctcss.py`.

# How to use in C++
Copy `ctcss.pio` into your project folder.

## in CMakeListe.txt
```
pico_generate_pio_header(<your project name> ${CMAKE_CURRENT_LIST_DIR}/ctcss.pio)
```

## somewhere
```C++
#include "hardware/pio.h"
#include "hardware/clocks.h"
#include "ctcss.pio.h"

inline const auto CTCSS_PIO = pio1;   // pio0 or pio1
inline const uint CTCSS_PIN = 10;     // any GPIO pin
inline const uint CTCSS_CYCLES = 180; // do not change
```
...
```C++
// setup ctcss pio
uint ctcssOffset = pio_add_program(CTCSS_PIO, &ctcss_program);
uint ctcssSm = pio_claim_unused_sm(CTCSS_PIO, true);
pio_sm_config ctcssConfig = ctcss_program_get_default_config(ctcssOffset);
sm_config_set_set_pins(&ctcssConfig, CTCSS_PIN, 1);
pio_gpio_init(CTCSS_PIO, CTCSS_PIN);
pio_sm_init(CTCSS_PIO, ctcssSm, ctcssOffset, &ctcssConfig);
pio_sm_set_enabled(CTCSS_PIO, ctcssSm, false);
```
...
```C++
// run ctcss pio
const double fCtcss = 151.4; // ctcss frequency
const auto sysClock = clock_get_hz(clk_sys);
const double clkDiv = sysClock / (fCtcss * CTCSS_CYCLES);
pio_sm_set_clkdiv(CTCSS_PIO, ctcssSm, clkDiv);
pio_sm_set_enabled(CTCSS_PIO, ctcssSm, true);
