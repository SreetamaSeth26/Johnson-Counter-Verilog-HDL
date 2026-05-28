# Johnson Counter — Verilog HDL

A 4-bit **Johnson Counter** (twisted-ring counter) implemented in Verilog, simulated on **EDA Playground** and synthesized using the **Yosys** open-source synthesis suite on macOS.

---

## Overview

A Johnson Counter shifts bits through a register and feeds the inverted last bit back to the front, cycling through 8 unique states for a 4-bit counter.

---

## Project Structure

```
johnson-counter/
├── johnson.v        # Johnson Counter RTL module
├── tb.v             # Testbench
├── johnson.json     # Yosys netlist export
├── johnson.svg      # Synthesized schematic (netlistsvg output)
└── dump.vcd         # Waveform output (auto-generated on simulation)
```

---

## Module Description

### `johnson.v` — Design Under Test

```verilog
module johnson(input clk, reset, input [3:0] load, output reg [3:0] q);
  always @(posedge clk) begin
    if (reset)
      q <= load;
    else
      q <= {~q[0], q[3:1]};
  end
endmodule
```

| Port    | Direction | Width | Description                               |
|---------|-----------|-------|-------------------------------------------|
| `clk`   | Input     | 1-bit | Clock signal (rising-edge triggered)      |
| `reset` | Input     | 1-bit | Synchronous reset — loads `load` into `q` |
| `load`  | Input     | 4-bit | Parallel load value on reset              |
| `q`     | Output    | 4-bit | Counter output                            |

**Shift logic:** `q <= {~q[0], q[3:1]}`  
The LSB is inverted and fed into the MSB position; all other bits shift right by one.

---

### `tb.v` — Testbench

- Clock period: **10 ns** (`#5` half-period toggle)
- Loads `4'b0000` on reset
- De-asserts reset after 10 ns, then runs for 100 ns
- Dumps VCD for waveform analysis via `$dumpfile` / `$dumpvars`
- Monitors `q` continuously with `$monitor`

---

## Johnson Sequence (starting from `0000`)

| Cycle | Q      |
|-------|--------|
| 0     | `0000` |
| 1     | `1000` |
| 2     | `1100` |
| 3     | `1110` |
| 4     | `1111` |
| 5     | `0111` |
| 6     | `0011` |
| 7     | `0001` |
| 8     | `0000` | ← repeats

---

## Tools Used

| Tool | Purpose | Platform |
|------|---------|----------|
| [EDA Playground](https://www.edaplayground.com/) | Online Verilog simulation (Icarus Verilog / VCS) | Web |
| [Yosys](https://yosyshq.net/yosys/) | Open-source RTL synthesis | macOS (Terminal) |
| [netlistsvg](https://github.com/nturley/netlistsvg) | Netlist schematic visualization | macOS (Terminal) |

### Running on EDA Playground

1. Paste `johnson.v` into the **Design** pane
2. Paste `tb.v` into the **Testbench** pane
3. Select **Icarus Verilog 12.0** (or similar) as the simulator
4. Enable **"Open EPWave after run"** to view the VCD waveform
5. Click **Run**

### Synthesis with Yosys (macOS)

Install Yosys via Homebrew:

```bash
brew install yosys
```

Run synthesis:

```bash
yosys -p "read_verilog johnson.v; synth -top johnson; show"
```

### Netlist Visualization with netlistsvg

Export a visual gate-level schematic of the synthesized netlist as an SVG.

**Step 1 — Install netlistsvg** (requires Node.js):

```bash
npm install -g netlistsvg
```

**Step 2 — Generate the JSON netlist inside Yosys:**

```bash
cd ~/Desktop
yosys
```

Then inside the Yosys prompt:

```
read_verilog johnson.v
prep -top johnson
write_json johnson.json
exit
```

**Step 3 — Render and open the SVG:**

```bash
netlistsvg johnson.json -o johnson.svg
open johnson.svg
```

This opens the synthesized gate-level schematic of the Johnson Counter directly in Preview/browser on macOS.

---

## 📊 Simulation Output (Expected)

```
Time = 0  || Q = 0000
Time = 10 || Q = 1000
Time = 20 || Q = 1100
Time = 30 || Q = 1110
Time = 40 || Q = 1111
Time = 50 || Q = 0111
Time = 60 || Q = 0011
Time = 70 || Q = 0001
Time = 80 || Q = 0000
...
```

---



---
