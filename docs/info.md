## How it works

Mocarabe is a coarse-grained reconfigurable array: a
2x2 torus of processing elements
(adders, multipliers and IO PEs) joined by a time-multiplexed network with
2 channels per link. Every 8-bit
value moves through scheduled switches and waits in a 16-deep FIFO at
its consumer. A kernel's dataflow graph is placed and scheduled offline by the
Mocarabe toolchain into a modulo schedule of II cycles (up to II_MAX =
4), which is loaded at run time as a serial bitstream. The
chip starts a new iteration of the kernel every II cycles.

The bundled demo is `int_level1_linear` at II = 4: 4 inputs and
1 output share the input and output pins, each in its own slot.

## How to test

1. Hold `rst_n` low and `cfg_en` (`uio[0]`) low for at least one clock.
2. Raise `cfg_en` and present one word of `test/bitstream.hex` on `ui_in` per clock
   (63 words, 8 bits each), then drop `cfg_en`.
3. Keep `rst_n` low with `ui_in` at 0 for 48 clocks, then release it.
   The first rising edge with `rst_n` high is run cycle 0.
4. Iteration k of each input is sampled from `ui_in` at run cycle `cycle + k*4`:
   `x1` at cycle 2, `x2` at cycle 3, `c1` at cycle 12, `c2` at cycle 21.
5. Iteration k of each output is on `uo_out` at run cycle `cycle + k*4`, with
   `out_valid` (`uio[1]`) high: `ret` at cycle 55.

`test/test.py` does exactly this for 16 iterations of random inputs and checks
every output against the dataflow graph in `test/mocarabe_program.json`.

## External hardware

None: the RP2040 on the demo board can drive the bitstream and inputs.
