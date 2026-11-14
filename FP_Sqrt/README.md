# FP Square Root

One module, `fp_sqrt`: IEEE-754 **binary32** (single precision) square root. CI checks it with a trusted harness; the expected results
come from an exact model, so "correct" means *bit-exact*.

### Module Specification

```SystemVerilog
module fp_sqrt (
  input  logic        clk,
  input  logic        reset,      // active high
  input  logic        in_valid,   // high for ONE clock edge; a is valid only then
  input  logic [31:0] a,
  output logic        out_valid,  // high while result is valid (same cycle .. 64 cycles after in_valid)
  output logic [31:0] result
);
```

### Ports (names, widths and directions are fixed)

| Port | Dir | Width | Meaning |
|---|---|---|---|
| `clk` | in | 1 | clock, rising edge |
| `reset` | in | 1 | active high; synchronous or asynchronous, you choose. Held for several cycles before the first operation |
| `in_valid` | in | 1 | high for **one** clock edge when an operation is presented |
| `a` | in | 32 | binary32 operand, valid **only** in the cycle `in_valid` is high |
| `out_valid` | out | 1 | high while `result` is valid |
| `result` | out | 32 | binary32 result, `sqrt(a)` |

### Handshake (latency-insensitive)

* The testbench drives the operand(s) with `in_valid = 1` across exactly one rising clock edge. In the next cycle those
  inputs are changed to garbage, so **capture them on that edge** (register them, or finish the computation combinationally in that
  cycle).
* The result is the value of `result` in the **first cycle `out_valid` is high**: the same cycle as `in_valid` (a purely
  combinational design with `out_valid = in_valid`), the next cycle, or up to **64 cycles** later (iterative dividers / square-root
  units). Fixed or variable latency are both fine.
* `out_valid` must be low whenever no result is being presented. A one-cycle pulse is the simplest; do not hold it high.
* One operation at a time: the testbench waits for `out_valid` and then idles two cycles before the next operation.
* No result within 64 cycles counts as a wrong result.

### Arithmetic

* **Format.** `[31]` sign, `[30:23]` biased exponent (bias 127), `[22:0]` fraction. Exponent `0` with fraction `0` is a zero, exponent `0`
  with a non-zero fraction is a **subnormal** (value `0.fraction * 2^-126`, no hidden bit). Exponent `255` is infinity (fraction `0`) or
  NaN (fraction `!= 0`).
* **Rounding.** Round to nearest, ties to even, on every result (including subnormal results). **Subnormals** are supported on inputs
  and outputs: no flush-to-zero. Overflow rounds to infinity with the correct sign.
* **NaN.** Any NaN operand (quiet or signalling, any sign or payload) and every invalid operation give the **canonical quiet NaN
  `32'h7FC0_0000`**. Exception flags are not part of the interface.
* **Special values.** `sqrt(+0) = +0`, `sqrt(-0) = -0`, `sqrt(+inf) = +inf`. Any other negative operand (including `-inf` and
  negative subnormals) is invalid and gives the canonical NaN. The result of a square root is never subnormal, but a subnormal
  **operand** must be handled (normalise it first).

### What is checked

About 8,000 operations: 24 corner values, then constrained-random operands (random bit patterns with a few negatives, subnormals,
short mantissas, exponents around 1.0 and at the ends of the range, and exact squares such as `k*k` scaled by even powers of two).
Every result must match bit for bit.

### Hints

* Make the exponent even (shift the significand by one when the unbiased exponent is odd), then take the integer square root of the
  significand scaled up by a large even power of two (a bit-by-bit / digit-recurrence root takes about one cycle per result bit).
* Keep at least 26 result bits and use "remainder is non-zero" as a sticky bit; an exact square gives an exact result.
* Yosys 0.33 (the CI synthesizer) cannot parse assignment patterns (`'{default: 0}`), and `small` is a reserved word.

### Interface

> [!IMPORTANT]
> Your module, port and parameter names are up to you. Your design must have the same number of ports, with the same directions and widths, as shown above, and must behave as specified. Map your names to this spec in `rnr-stage.yaml` (see the assignment handout).
