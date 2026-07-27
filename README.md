# Embedded Electrochemical Design Review

`embedded-electrochemical-design-review` is a reusable Codex Skill for reviewing embedded electrochemical instruments. It combines electrochemical topology constraints, component datasheet compliance, analog signal-chain calculations, UART/BLE data-integrity analysis, bench-test planning, PCB layout review, and automated Word report generation.

The Skill is hardware-independent. It does not assume a particular MCU, ADC, DAC, operational amplifier, electrochemical AFE, wireless module, EDA package, or mobile application.

The default final report is generated on the user's Desktop as:

```text
Recommendations for embedded electrochemical design.docx
```

## Contents

1. Purpose
2. Main capabilities
3. Supported inputs
4. Review workflow
5. Installation
6. Usage examples
7. Included scripts
8. Input and output models
9. Evidence and priority levels
10. Word report contents
11. Safety and technical limitations
12. Repository structure
13. Troubleshooting

## Purpose

Embedded electrochemical systems cross several engineering domains:

- electrochemical potential control;
- three-electrode interfaces;
- weak-current measurement;
- analog amplification and filtering;
- ADC/DAC conversion;
- embedded firmware;
- UART, SPI, I2C, and BLE communication;
- PCB leakage, grounding, and electromagnetic interference;
- oscilloscope and multimeter verification.

A circuit may appear reasonable at one level and still fail at another. For example:

- the electrode topology may be correct while the control-loop polarity is reversed;
- the analog circuit may work while the ADC input common-mode range is violated;
- the ADC code may be correct while firmware interprets it as an unsigned integer;
- the plotted curve may be smooth while BLE frames are missing or duplicated;
- a datasheet reference circuit may be copied into an incompatible bandwidth or gain mode;
- a schematic may be correct while PCB contamination creates unacceptable leakage at the reference-electrode or TIA input.

This Skill reviews those layers as one traceable system. It separates observed facts, calculations, assumptions, inferences, and recommendations so that the result can be checked by another engineer.

## Main Capabilities

### 1. Electrochemical schematic logic review

The Skill reconstructs the intended signal and control paths:

```text
Potential command -> DAC/excitation -> control amplifier -> CE
                                            ^
                                            |
                                            RE

WE current -> TIA -> gain/filter -> ADC -> MCU -> communication
```

It checks:

- whether the working electrode, reference electrode, and counter electrode have the correct roles;
- whether RE is used as a high-impedance sensing node rather than a current-return path;
- whether CE is driven by the potentiostat control loop;
- whether WE current is measured through the intended TIA or current-sense path;
- whether the feedback polarity produces the requested `E_WE - E_RE`;
- whether any electrode has been incorrectly shorted, grounded, or AC-coupled;
- whether the CE driver has sufficient voltage and current compliance;
- whether compensation networks belong across nodes or were accidentally placed in series with an electrode;
- whether LSV, CV, CA, IT, SWV, and DPV waveforms remain inside the hardware range at every instantaneous point.

The Skill requires an explicit potential convention. By default, reviews use:

```text
Applied potential = E_WE - E_RE
```

Current polarity is not assumed. It is derived from the submitted TIA topology and firmware conversion.

### 2. Voltage, current, resistance, impedance, bandwidth, and noise analysis

The Skill calculates a generic analog transfer chain from sourced component values.

For a parallel TIA feedback resistor and capacitor:

```text
Zf(f) = Rf / (1 + j*2*pi*f*Rf*Cf)
Vout = Vbias - Iwe*Zf(f)
fp = 1 / (2*pi*Rf*Cf)
```

It can estimate:

- DC operating points;
- positive and negative current range;
- frequency-dependent feedback impedance;
- transimpedance and downstream voltage gain;
- ADC input voltage and ideal code;
- analog and converter saturation margins;
- first-order bandwidth and settling time;
- resistor thermal noise;
- first-order input-referred op-amp noise;
- ADC quantization and input-noise contribution;
- nominal, typical, worst-case, and calibrated uncertainty when sufficient data is available.

The Skill distinguishes resistance from impedance. A resistor value alone does not establish AC impedance; frequency and reactive components are required.

The noise model is deliberately conservative about its limits. A first-order estimate is not presented as a complete noise simulation unless noise gain, sensor capacitance, 1/f corners, reference noise, filter transfer functions, and equivalent noise bandwidth are known.

### 3. Gain, accuracy, and firmware scaling audit

The Skill reconstructs:

```text
Electrode current
  -> TIA transimpedance
  -> instrumentation/differential gain
  -> active filter gain
  -> ADC PGA
  -> ADC code
  -> firmware voltage/current unit
```

It checks:

- resistor and instrumentation-amplifier gain equations;
- gain-setting resistor selection;
- bias and level-shift terms;
- ADC reference and full-scale span;
- input common-mode and differential range;
- PGA restrictions;
- two's-complement, offset-binary, or straight-binary coding;
- endianness and sign extension;
- integer overflow and unsigned subtraction around a bias code;
- calibration coefficients and their hardware-version association;
- ideal resolution, ENOB-limited resolution, accuracy, precision, and uncertainty as separate concepts.

The Skill does not report a single unsupported "accuracy percentage." Every accuracy statement must identify the included terms and calculation method.

### 4. UART and BLE data analysis

The Skill accepts CSV, JSONL, hexadecimal captures, raw binary streams, serial logs, and documented BLE application frames.

It checks:

- frame boundaries and lengths;
- BLE fragmentation and concatenated application frames;
- endianness and signedness;
- fixed-point scale and offset;
- CRC or checksum;
- sample and run identifiers;
- duplicate, missing, and out-of-order sequence numbers;
- timestamp monotonicity and sample-interval jitter;
- effective sampling frequency;
- saturation ratio;
- mean, standard deviation, RMS, peak-to-peak variation, percentiles, and linear drift.

Raw captures must remain unchanged. Decoded and filtered data are stored separately. The Skill does not silently interpolate missing formal measurement data.

### 5. Oscilloscope and multimeter test planning

The Skill converts schematic calculations into a practical measurement table. Each critical node can include:

- power or experiment state;
- instrument and probe mode;
- reference node;
- nominal, minimum, and maximum value;
- expected waveform;
- tolerance source;
- pass/fail criterion;
- interpretation of high, low, noisy, clipped, oscillating, or floating readings;
- the next measurement that distinguishes competing fault hypotheses.

The normal test sequence is:

1. Power-off short and resistance inspection.
2. Power rails, regulator outputs, references, reset, and clocks.
3. Digital buses at both source and target pins.
4. Static DAC and excitation levels.
5. Passive dummy-cell verification.
6. RE tracking, CE compliance, TIA polarity, and ADC scaling.
7. Ramp, triangle, step, and pulse waveforms.
8. Real electrochemical cell only after critical checks pass.

For high-impedance analog nodes, the default scope recommendation is a compensated 10x probe with a 1 Mohm input. The Skill must not recommend 50 ohm termination unless the source and transmission line were designed to drive 50 ohm.

### 6. Component datasheet compliance review

The Skill reviews each submitted component against the exact official datasheet and package.

It records:

- manufacturer and complete orderable part number;
- package code and pin-1 orientation;
- datasheet revision and publication date;
- schematic symbol and footprint;
- official reference design and evaluation-board differences;
- relevant page, table, figure, and section references.

For every pin it can create a compliance matrix containing:

- pin number and mnemonic;
- electrical type and power domain;
- actual schematic connection;
- required or permitted connection;
- voltage, current, timing, or loading limit;
- startup and inactive state;
- compliance status;
- evidence citation.

Specific review categories include:

#### Operational and instrumentation amplifiers

- supply range;
- input common-mode and differential limits;
- load-dependent output swing;
- input/output phase reversal;
- offset, bias current, and drift;
- voltage and current noise;
- gain bandwidth and slew rate;
- capacitive-load stability;
- feedback and compensation.

#### ADCs

- analog and digital supplies;
- reference source and decoupling;
- differential and common-mode input range;
- PGA restrictions;
- source impedance and acquisition settling;
- anti-alias filtering;
- clock, reset, DRDY, SPI mode, status, and CRC;
- calibration and unused-input handling.

#### DACs

- reference input/output role;
- code transfer equation;
- output buffer and load;
- settling and power-on code;
- update timing and bus format;
- monotonic range and output filtering.

#### Electrochemical AFEs

- electrode pin roles;
- internal switch matrix;
- bias and zero DAC range;
- TIA mode and RTIA/CTIA;
- compensation and reconstruction nodes;
- sequencer timing and wake-up delay;
- calibration resistor and electrode protection.

#### Power, reference, protection, and passive components

- input/output range and dropout;
- current and thermal limits;
- required capacitance and ESR;
- startup, enable, reverse current, and sequencing;
- tolerance, TCR, voltage coefficient, leakage, and power rating;
- ESD leakage and capacitance at sensitive analog nodes.

The Skill distinguishes `NC` from `DNC`. It also treats regulator, reference, and capacitor pins as possible internal outputs or decoupling nodes rather than automatically connecting them to an external supply.

### 7. PCB layout review

The Skill reviews placement and routing using the exact component guidance and physical signal sensitivity.

It checks:

- supply-decoupling loop area;
- regulator and reference capacitor placement;
- TIA feedback resistor/capacitor placement;
- WE, RE, and summing-node trace length and leakage exposure;
- guard geometry and guard-driving potential;
- clock, SPI, PWM, switching-regulator, and radio coupling;
- ADC differential routing and input-filter placement;
- continuous return paths;
- AGND, DGND, exposed-pad, shield, chassis, and isolated-ground treatment;
- CE high-current paths;
- antenna keepout;
- footprint, pin-1, polarity, thermal vias, and test access;
- flux residue, humidity, connector leakage, and solder-mask openings around high-impedance nodes.

The Skill does not mechanically demand a split or star ground. It analyzes actual return-current paths and follows the exact converter or manufacturer reference design.

### 8. Automatic Word report

After the review model has been completed and validated, the Skill generates:

```text
Recommendations for embedded electrochemical design.docx
```

The default location is the operating system's actual Desktop directory. An alternative output path is used only when explicitly requested.

The report uses real Word heading styles, document control, a table-of-contents field, page numbering, structured compliance tables, evidence grades, priority blocks, formulas, test limits, and references.

## Supported Inputs

The review can use any useful subset of:

| Input | Preferred form | Purpose |
|---|---|---|
| Schematic | Vector PDF and netlist | Recover topology and exact connections |
| PCB | Copper plots, stackup, coordinates, layout PDF/images | Review placement, return paths, leakage, and coupling |
| BOM | CSV/XLSX with complete MPNs | Identify exact parts and packages |
| Datasheets | Official manufacturer PDF | Pin, range, timing, peripheral, and layout compliance |
| Firmware | MCU source, configuration, register dumps | Check pin mapping, timing, scaling, and converter setup |
| UART/BLE data | Raw bytes, hex, CSV, JSONL, logs | Decode framing and measure integrity |
| Bench measurements | Raw scope captures, screenshots, DMM logs | Compare actual values with calculated limits |
| Experiment parameters | Method, potential, timing, gain, sample rate | Check instantaneous range and acquisition timing |

A low-resolution screenshot may be adequate for orientation, but it is not enough for a definitive pin, junction, hidden-layer, or numerical-noise conclusion.

## Review Workflow

The Skill follows this sequence:

```text
1. Inventory evidence
2. Identify exact devices, packages, and datasheet revisions
3. Reconstruct electrochemical and analog functional blocks
4. Audit the three-electrode control loop
5. Build per-component and per-pin compliance matrices
6. Calculate operating points, gain, impedance, bandwidth, and noise
7. Audit ADC/DAC transfer and firmware scaling
8. Decode and analyze UART/BLE data
9. Generate oscilloscope and DMM test points
10. Review PCB layout and leakage risks
11. Assign P0/P1/P2/P3 findings
12. Validate the normalized review JSON
13. Generate and visually inspect the Word report
```

The review finishes as one of:

- `PASS`: all applicable requirements verified and no unresolved P0/P1 findings;
- `CONDITIONAL`: no known P0 issue, but important assumptions or P1 verification remain;
- `FAIL`: a P0 issue or proven violation invalidates the intended operation;
- `INSUFFICIENT_EVIDENCE`: the submitted artifacts do not support a defensible conclusion.

## Installation

### Install from a downloaded ZIP

1. Extract the archive.
2. Locate the folder containing `SKILL.md`.
3. Copy `embedded-electrochemical-design-review` into the Codex skills directory.

Windows example:

```text
C:\Users\<username>\.codex\skills\embedded-electrochemical-design-review\
```

macOS or Linux example:

```text
~/.codex/skills/embedded-electrochemical-design-review/
```

4. Start a new Codex task or refresh the available Skills.

### Install from GitHub

The repository or selected GitHub subdirectory must preserve this structure:

```text
embedded-electrochemical-design-review/
  SKILL.md
  agents/
  references/
  scripts/
  assets/
```

Use the Codex skill installer with the GitHub repository or subdirectory URL, or copy the cloned folder into the local skills directory.

### Python dependency

The calculation and data-analysis scripts use the Python standard library. Word generation additionally requires `python-docx`:

```bash
python -m pip install -r requirements.txt
```

Do not install dependencies into a production embedded firmware environment. Run the review tools in a separate analysis environment.

## Usage Examples

Explicitly invoke the Skill with `$embedded-electrochemical-design-review`.

### Full design review

```text
Use $embedded-electrochemical-design-review to review the attached schematic,
PCB, BOM, datasheets, firmware, and UART capture. Check the three-electrode
logic, every component connection, gain, noise, converter range, data integrity,
and layout. Generate the final Word report on the Desktop.
```

### Schematic and datasheet review

```text
Use $embedded-electrochemical-design-review to compare this schematic against
the submitted component datasheets. Produce a per-pin compliance matrix and
identify all required decoupling, protection, timing, and layout rules.
```

### Analog performance review

```text
Use $embedded-electrochemical-design-review to calculate the TIA impedance,
current range, total gain, pole frequencies, ADC utilization, first-order noise,
and uncertainty. Clearly separate typical estimates from guaranteed limits.
```

### UART or BLE analysis

```text
Use $embedded-electrochemical-design-review to decode this BLE hex capture
using the attached protocol. Check CRC, signed ADC conversion, sequence gaps,
sample-rate jitter, saturation, noise, and drift.
```

### Bench-test plan

```text
Use $embedded-electrochemical-design-review to create a safe oscilloscope and
multimeter test plan for every critical rail and analog node. Calculate expected
values and explain what each abnormal waveform would mean.
```

## Included Scripts

### `calculate_signal_chain.py`

Calculates a generic parallel-RF/CF TIA, downstream gain, range, first-order noise, ADC utilization, and RSS uncertainty terms.

```bash
python scripts/calculate_signal_chain.py \
  assets/signal-chain-example.json \
  --output signal-chain-result.json
```

Important limitations:

- assumes RF and CF are parallel;
- assumes supplied spot-noise densities are white over the stated bandwidth;
- uses a first-order op-amp voltage-noise conversion;
- does not replace SPICE or a complete loop/noise-gain analysis.

### `decode_frames.py`

Decodes fixed-length raw or hexadecimal UART/BLE frames from a JSON protocol definition.

Supported numeric fields include:

```text
uint8, int8, uint16, int16, uint32, int32,
uint64, int64, float32, float64
```

Supported integrity checks include:

```text
sum8, xor8, CRC-16/MODBUS
```

Example:

```bash
python scripts/decode_frames.py \
  capture.hex \
  assets/frame-protocol-example.json \
  --output-json decoded.json \
  --output-csv decoded.csv
```

The protocol definition must match the real application protocol. Do not choose endianness, offsets, field types, or CRC coverage by visual trial and error.

### `analyze_stream.py`

Analyzes CSV or JSONL samples for sequence continuity, timing, saturation, distribution, and linear drift.

```bash
python scripts/analyze_stream.py data.csv \
  --format csv \
  --seq seq \
  --time time_s \
  --value current_a \
  --sat-min -0.00001 \
  --sat-max 0.00001 \
  --output stream-analysis.json
```

Standard deviation over a full voltammogram is not automatically baseline noise. Select a physically valid baseline or steady-state region before interpreting the statistic as noise.

### `validate_review.py`

Checks the normalized review model before report generation.

```bash
python scripts/validate_review.py review.json
```

It checks finding IDs, priorities, evidence grades, required fields, component identity, and missing citations for critical findings.

### `generate_report.py`

Generates the final Word document:

```bash
python scripts/generate_report.py review.json
```

To override the default Desktop location:

```bash
python scripts/generate_report.py review.json --output output/review.docx
```

After generation, render the DOCX to page images with an available Word/LibreOffice document renderer. Inspect every page for clipped text, broken tables, missing glyphs, overlaps, unexpected blank pages, and incorrect page breaks.

## Input and Output Models

Start a normalized review from:

```text
assets/review-template.json
```

The top-level model contains:

```json
{
  "metadata": {},
  "executive_summary": [],
  "inputs": [],
  "assumptions": [],
  "electrochemical_topology": {},
  "component_reviews": [],
  "operating_points": [],
  "calculations": [],
  "data_quality": {},
  "measurements": [],
  "layout_findings": [],
  "recommendations": [],
  "unresolved": [],
  "references": []
}
```

Detailed examples and required report fields are documented in `references/report-model.md`.

## Evidence and Priority Levels

### Evidence grades

| Grade | Meaning |
|---|---|
| A | Confirmed by multiple independent primary artifacts |
| B | Directly derived from a primary source and transparent calculation |
| C | Estimate based on nominal or typical values with declared assumptions |
| D | Unresolved interpretation, missing revision, unclear image, or weak source |

A likely conclusion is not automatically Grade A. Evidence grade describes traceability, not confidence or severity.

### Finding priorities

| Priority | Meaning |
|---|---|
| P0 | Damage risk, unsafe measurement, invalid potentiostat topology, electrode short, or guaranteed invalid data |
| P1 | Likely saturation, instability, operating-range violation, severe timing fault, or missing mandatory circuit |
| P2 | Material noise, accuracy, calibration, layout, robustness, or maintainability issue |
| P3 | Optional optimization or documentation improvement |

Every finding should include:

```text
ID
priority
location
observation
datasheet/electrochemical requirement
consequence
recommended correction
verification step
evidence grade
page/table/figure citation
```

## Word Report Contents

The generated report contains all applicable sections:

1. Executive summary and review status
2. Submitted evidence and assumptions
3. Electrochemical topology and sign convention
4. Component and datasheet compliance matrices
5. Power rails and DC operating points
6. Voltage, current, impedance, gain, bandwidth, and saturation
7. Noise, resolution, accuracy, and uncertainty
8. ADC/DAC and firmware conversion audit
9. UART/BLE integrity analysis
10. Oscilloscope and multimeter measurement plan
11. PCB layout review
12. P0/P1/P2/P3 recommendations and acceptance criteria
13. Unresolved items and residual risks
14. Datasheet and technical references

The report preserves the language used in `review.json`, allowing English, Chinese, or mixed technical content.

## Safety and Technical Limitations

### Electrical measurement safety

- A bench oscilloscope ground clip is commonly bonded to protective earth.
- Connecting it to a floating bridge node, negative rail, electrode, or isolated ground can create a short circuit.
- Confirm grounding before probing; use a suitable differential or isolated probe when necessary.
- Do not measure resistance or continuity on a powered circuit.
- Do not connect a real electrochemical cell until supply, bias, control-loop, dummy-cell, and ADC checks pass.

### Datasheet limitations

- Use the exact MPN, package, and revision.
- Do not use absolute maximum ratings as recommended operating conditions.
- Do not treat typical graphs as guaranteed production limits.
- Do not assume an internal regulator output is an external supply input.
- Do not treat `NC` and `DNC` as equivalent.
- Do not copy an evaluation-board component value without checking its operating mode and bandwidth.

### Calculation limitations

- Calculations are only as reliable as the supplied topology and parameter sources.
- First-order noise estimates do not replace SPICE, impedance measurement, or spectral analysis.
- A theoretical circuit-noise floor is not an experimental LOD or LOQ.
- Chemical LOD/LOQ require blank replicates, calibration data, a stated response model, and statistical validation.
- A visible CV, SWV, or DPV peak does not by itself identify an analyte.
- A stable DAC output does not prove the actual electrode potential unless `E_WE - E_RE` is verified.

### Data limitations

- A screenshot is not a substitute for raw data in quantitative noise or packet-loss analysis.
- Missing frames must not be silently reconstructed for formal analysis.
- Filtering must not overwrite raw samples.
- Incomplete, saturated, unit-ambiguous, or sequence-discontinuous experiments must be clearly marked.

### Scope

This Skill is an engineering review aid. It does not provide regulatory certification, electrical-safety certification, clinical validation, or assurance that a biological or chemical assay is fit for diagnosis.

## Repository Structure

```text
embedded-electrochemical-design-review/
|-- README.md
|-- SKILL.md
|-- requirements.txt
|-- agents/
|   `-- openai.yaml
|-- assets/
|   |-- frame-protocol-example.json
|   |-- review-template.json
|   `-- signal-chain-example.json
|-- references/
|   |-- analog-analysis.md
|   |-- bench-measurement.md
|   |-- component-datasheet-review.md
|   |-- converter-accuracy.md
|   |-- pcb-layout.md
|   |-- potentiostat-logic.md
|   |-- report-model.md
|   |-- serial-ble-analysis.md
|   `-- workflow-and-evidence.md
`-- scripts/
    |-- analyze_stream.py
    |-- calculate_signal_chain.py
    |-- decode_frames.py
    |-- generate_report.py
    `-- validate_review.py
```

`SKILL.md` contains the compact execution workflow. Detailed rules remain in `references/` and are loaded only when relevant. Deterministic arithmetic, byte decoding, validation, and document generation remain in `scripts/`.

## Troubleshooting

### The Skill does not trigger automatically

Invoke it explicitly:

```text
$embedded-electrochemical-design-review
```

Confirm that the folder containing `SKILL.md` is directly inside the configured Codex skills directory, then start a new task or refresh the application.

### The report generator cannot import `docx`

Install the declared dependency:

```bash
python -m pip install -r requirements.txt
```

### The Word table of contents is empty

The document contains a real Word TOC field. Open the document in Word and update fields, or use `Ctrl+A` followed by `F9`.

### The report is generated outside the expected Desktop

The script queries the operating system's Desktop location. Use an explicit output path when the Desktop is redirected or unavailable:

```bash
python scripts/generate_report.py review.json --output "C:/desired/path/review.docx"
```

### A schematic connection is marked `UNRESOLVED`

Provide one or more of:

- vector schematic PDF;
- EDA netlist;
- clearer crop with junction dots visible;
- hierarchical net-label listing;
- BOM with exact MPN and package;
- PCB copper-layer plots.

The Skill intentionally refuses to invent a connection from ambiguous pixels.

### Noise results disagree with the bench

Check whether the calculation includes:

- actual equivalent noise bandwidth;
- op-amp 1/f noise;
- noise gain and sensor capacitance;
- reference and bias noise;
- ADC input noise at the selected data rate and filter;
- power-supply and digital coupling;
- grounding and probe noise;
- PCB surface leakage and environmental interference.

Treat the difference as diagnostic evidence rather than forcing the measured result to match a simplified model.
