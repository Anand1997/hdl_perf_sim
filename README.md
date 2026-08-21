# DSL for the SICP circuit simulator

Interpreted Lisp-like language for digital circuits and testbenches (SICP Chapter 4).
Primitives map onto `circuit_sim` (core). The **reader** turns source into lists;
`eval` is not written yet.

Requires **lark**. Comments use `;`. Hyphens and underscores are equivalent.

```text
python -m pytest -v
```

## Core mapping

| DSL | `circuit_sim` |
|-----|----------------|
| `(wire a)` / `(bus data 8)` | `make_wire` / `make_bus` |
| `(bit data 3)` `(slice data 7 0)` | index into a bus |
| `(set-bus data 42)` | `set_bus_values` |
| `(assert-bus data 69)` / `(print-bus data)` | `get_bus_values` |
| `(and …)` `(or …)` `(inv …)` `(nand …)` `(nor …)` `(xor …)` `(compound-or …)` | matching gates |
| `(half-adder (a b) (s c))` | `half_adder` |
| `(full-adder (a b cin) (sum cout))` | `full_adder` |
| `(ripple-carry (a b) (sum cout))` | `ripple_carry_adder` (optional `cin` in inputs) |
| `(mux (a b sel) (y))` | `multiplexer` |
| `(demux (in sel) (oa ob))` | `demultiplexer` |
| `(sr-latch …)` `(d-latch …)` `(d-ff …)` `(t-ff …)` | matching sequential |
| `(binary-counter (clk) (count))` | `binary_counter` (`count` is a bus) |
| `(probe s c)` | `probe` |
| `(agenda sicp)` `(agenda simpy)` | `make_agenda` / `make_simpy_agenda` |
| `(agenda realtime (factor 0.01) (strict))` | `make_realtime_agenda` |
| `(clock clk (high 10) (low 10) (cycles 5) (initial 0))` | `clock_generator` |
| `(pulse d (15 0) (20 1) (repeat))` | `pulse_generator` (`repeat` optional) |
| `(schedule d (0 0) (15 1) (35 0))` | `signal_schedule` (absolute times) |
| `(propagate)` `(propagate (steps N))` `(run N)` | `propagate` / `run_until` |
| `(delays fast (and 1) (or 2) (inv 1) (nand 1) (nor 2) (xor 3))` | `Delays` |

Engine internals (`Queue`, `after_delay`, OOP gate classes, `logical_*`, `ProbeRecorder`) stay in Python.

## Syntax

### Delays

```lisp
(delays fast
  (and 1) (or 2) (inv 1) (nand 1) (nor 2) (xor 3))
```

### Wires and buses

```lisp
(wire a b clk)
(bus data 8)          ; make_bus(8, "data")
(wire (also 4))       ; same idea: name + width
```

Bits: `(bit data 3)`, `(slice data 7 0)`.

### Gates and catalog circuits

```lisp
(and (delay 3) (a b) (out))
(half-adder (a b) (s c))
(full-adder (a b cin) (sum cout))
(ripple-carry (a-bus b-bus) (sum-bus cout))
(ripple-carry (a-bus b-bus cin) (sum-bus cout))
(mux (a b sel) (y))
(demux (in sel) (out-a out-b))
(binary-counter (clk) (count))
```

Optional `(delay N)` after a primitive gate name only. Catalog circuits use the active delay profile.

### Sequential and assign

```lisp
(d-ff (d clk) (q q-bar))
(assign y (| (& a (~ sel)) (& b sel)))   ; also nand / nor / xor
```

### Simulation

```lisp
(sim
  (agenda simpy)
  (delays fast)
  (init (in1 0) (in2 0))
  (set-bus a 42)
  (at 0 (in1 1))
  (clock clk (high 10) (low 10) (cycles 5) (initial 0))
  (pulse d (15 0) (20 1) (25 0))
  (schedule d (0 0) (15 1) (35 0))
  (run 80)
  (propagate (steps 100))
  (assert (s 1) (c 0) (at 16) (message "ha"))
  (assert-bus sum 69)
  (print "state:" s (bus-val sum))
  (print-bus sum))
```

Agenda engines: `sicp`, `simpy`, `realtime`.

See `examples/half_adder.sim`, `examples/ripple_carry.sim`, `examples/clocked_counter.sim`.
