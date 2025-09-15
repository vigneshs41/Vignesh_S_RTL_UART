# Verilog UART Readme


## Deprecation Notice

About Me:
My name is Vignesh, from India, and I am currently working at CDAC as an RTL Design Engineer. I have 3.8+ years of experience in RTL design, specializing in microarchitecture, cache design, and verification.

Project: Advanced Supercomputing Research (ACR):
Project Duration: Ongoing
Project Title: Design of a 64-bit RISC-V Superscalar Out-of-Order CPU

Roles & Responsibilities:
Primary Tasks
Microarchitecture Definition:
1. Designed I-Cache and D-Cache as configurable cache memories (dual-port RAM with basic cache structure including LRU)
Developed tag comparator, victim cache, branch predecode unit, MSHR, TLB, prefetcher, and way predictor.
Block Specification & Component Design:
1. Designed a pipelined L1 I-Cache and L1 D-Cache (cache size, ways, and cache line length are configurable) using Verilog
2. Implemented MESI Protocol supporting both L1 caches.
3. Integrated cache architecture with prefetch unit, way predictor, victim cache, and MSHR to enhance cache performance.
4. Developed an AXI interface for integrating the cache with main memory.
Verification:
1. Verified each module using Verilog testbenches and performed coverage checks.
Conducted linting checks for quality assurance before sign-off.
Synthesis:
1. Performed synthesis using Cadence Genus (28 nm, 2 GHz) and analyzed synthesis results.
Created detailed documentation for both I-Cache and D-Cache.
Automation:
1. Developed automation scripts using TCL and Shell on Linux to streamline synthesis processes.

Tools Used:
Simulation: Questa Sim-64 2019.2, Vivado 2019.1
Synthesis: Cadence Genus
Linting: Spyglass Lint

## IDEL Introduction

This is a basic UART to AXI Stream IP core, written in Verilog with cocotb
testbenches.

## IDEL Documentation

The main code for the core exists in the rtl subdirectory.  The uart_rx.v and
uart_tx.v files are the actual implementation, uart.v simply instantiates both
modules and makes a couple of internal connections.

The UART transmitter and receiver both use a single transmit or receive pin.
The modules take one parameter, DATA_WIDTH, that specifies the width of both
the data bus and the length of the actual data words communicated.  The
default value is 8 for an 8 bit interface.  The prescale input determines the
data rate - it should be set to Fclk / (baud * 8).  This is an input instead
of a parameter so it can be changed at run time, though it is not buffered
internally so care should be used to avoid corrupt data.  The main interface
to the user design is an AXI4-Stream interface that consists of the tdata,
tvalid, and tready signals.  tready flows in the opposite direction.  tdata
is considered valid when tvalid is high.  The destination will accept data
only when tready is high.  Data is transferred from the source to the
destination only when both tvalid and tready are high, otherwise the bus is
stalled.

Both interfaces also present a 'busy' signal that is high when an operation is
taking place.  The receiver also presents overrun error and frame error strobe
outputs.  If the data word currently in the tdata output register is not read
before another word is received, then a single cycle pulse will be emitted
from overrun_error and the word is discarded.  If the receiver does not get a
stop bit of the right level, then a single pulse will be emitted from the
frame_error output and the received word will be discarded.

### IDEL Source Files

    rtl/uart.v     : Wrapper for complete UART
    rtl/uart_rx.v  : UART receiver implementation
    rtl/uart_tx.v  : UART transmitter implementation

### IDEL: AXI Stream Interface Example

two byte transfer with sink pause after each byte

              __    __    __    __    __    __    __    __    __
    clk    __/  \__/  \__/  \__/  \__/  \__/  \__/  \__/  \__/  \__
                    _____ _________________
    tdata  XXXXXXXXX_D0__X_D1______________XXXXXXXXXXXXXXXXXXXXXXXX
                    _______________________
    tvalid ________/                       \_______________________
           ______________             _____             ___________
    tready               \___________/     \___________/


Work I have done:
1. I cloned the repository https://github.com/alexforencich/verilog-uart and committed it without modifications, then pushed it into my repository on the main (master) branch.


2. I created a branch named “1-correct-error-in-uart-transmitter-uart_tx”, linked to Issue #1 – Correct error in UART Transmitter (uart_tx). In this branch, I specifically modified (added a bug) in the TX module. Along with the issue, I provided a specification for the TX module. Based on the specification and the buggy code, I expected the LLM to generate the corrected version of the code.


3. I created another branch named “3-correct-error-in-uart-receiver-uart_rx”, linked to Issue #2 – Correct error in UART Receiver (uart_rx). In this branch, I specifically modified (added a bug) in the RX module. Along with the issue, I provided a specification for the RX module. Based on the specification and the buggy code, I expected the LLM to generate the corrected version of the code.


4. I then created a branch named “5-complete-the-design-uart-top-module”, linked to Issue #5 – Complete the design of UART Top Module. In this branch, I specifically modified (added a bug) in the Top module. Along with the issue, I provided a specification for the Top module. Based on the specification and the buggy code, I expected the LLM to generate the corrected version of the code.

  
Description of the changes I made to break the UART tx, UART rx, uart modules.
In uart_tx - error i made: 

I made critical errors in the following parts:
1. bit_cnt – the overall bit count does not match the expected value.
2. data_reg – the data is not stored in proper UART format.
3. Shift logic – I introduced a bug in the left-shift logic that was supposed to ensure UART formatting. Because of this, the overall output becomes completely zero.
4. Prescale computation logic – requires modification.
5. bit_cnt calculation logic – requires correction as well.


Regarding the specification, I have provided a clear explanation in Issue #1:
 https://github.com/vigneshs41/Vignesh_S_RTL_UART/issues/1 

For a more detailed view of the code differences, please check this pull request:

“https://github.com/vigneshs41/Vignesh_S_RTL_UART/pull/2/files” 



In uart_rx - error i made: 


I made the following errors:
1. In rxd, instead of taking data from the register, I mistakenly took it directly from the input port, which causes a synchronous issue.
2. In data_reg assignment, which results in improper data storage.
3. In bit_cnt and prescale_reg computation, which leads to incorrect bit count calculation and baud rate computation.
4. In overrun_error_reg, which results in incorrect error handling.


Regarding the specification, I have provided a clear explanation in the issue:
https://github.com/vigneshs41/Vignesh_S_RTL_UART/issues/3 

For more detailed regrading code difference:  https://github.com/vigneshs41/Vignesh_S_RTL_UART/pull/4/files  


In top module uart - error i made: 

1. From the top module, I made bugs in the rst signal, DATA_WIDTH assignment, as well as in the input and output port connections between the top module and TX/RX modules, which are completely misaligned.

Regarding the specification, I have provided a clear explanation in the issue: https://github.com/vigneshs41/Vignesh_S_RTL_UART/issues/5  
For more detailed regrading code difference:  https://github.com/vigneshs41/Vignesh_S_RTL_UART/pull/6/files   


