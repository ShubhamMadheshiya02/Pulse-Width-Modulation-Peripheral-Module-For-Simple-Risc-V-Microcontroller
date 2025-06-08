# Pulse-Width-Modulation-Peripheral-Module-For-Simple-Risc-V-Microcontroller
## Introduction

A parameterizable Verilog PWM generator designed as a peripheral module for a simple RISC-V microcontroller.  
You can adjust clock prescaling, period, and duty cycle at runtime.

Pulse Width Modulation (PWM) is a technique by which the width of a pulse is varied while keeping the frequency of the wave constant , or if required we can vary frequency also here.

Through the PWM technique, we can control the power delivered to the load by using the ON-OFF signal and also we can vary the frequency of the output signal. The PWM signals can be used to control the speed of DC motors and to change the intensity of the LED. Moreover, it can also be used to generate sine signals.

This PWM peripheral module is designed in Vivado using Verilog, later on implemented on FPGA.

---

## ➤ Repository Layout
├─ src/ # Verilog RTL sources
│ ├─ prescalar2.v
│ ├─ period_reg.v
│ ├─ tmr2_cnt.v
│ ├─ mag_cmpa.v
│ ├─ mag_cmpb.v
│ ├─ duty_reg_master.v
│ ├─ duty_reg2.v
│ └─ sr_latch.v
├─ tb/ # Testbench files
│ ├─ prescalar_tb.v
│ └─ tb_pwm1.v

## Details for each module :
1. **Prescaler**:
            *Programmable clock prescaler*
            <br/>Gates `clk` with `en_prescalar`, feeds a 32-stage T-FF divider (Qt[i] = clk/2^(i+1)), then uses the 5-bit `prescale_value` to pick which divided clock drives `clk_out`. Supports division ratios from 2^1 up to 2^32; bypasses when disabled. 
 
2. **Period_Reg**:
             <br/>| **period_reg.v**  | **Terminal-count register**<br/>On reset, sets `PR` to 0xFFFF (max period); otherwise, loads the 16-bit `pr_in` value into `PR`, which will come into effect at next rising `clk`, defining the PWM’s Period. |

3. **TIMER**:
            <br/>| **tmr_cnt.v**     | **16-bit timer/counter**<br/>Resets `TMR` to 0 on `rst` or when `EN_TMR`=0; otherwise increments on each rising `clk` and rolls over to 0 when reaching the programmed period `PR`. |
4. **Comparator for TMR and PR:**
            <br/>| **comparator_b.v** | **Equality comparator (Comparator B)**<br/>Continuously compares the 16-bit `TMR` and `PR` values and asserts `S` high when `TMR == PR`; used as the Set input for the SR latch to implement PWM rollover. |
5. **Duty_cycle_registers:**
            <br/>**Master/Slave pair for glitch-free duty updates**<br/>• **Master (duty_reg_master.v):** combinatorially mirrors the 16-bit `duty_cycle_in` input (or clears to 0 on `rst`), providing a writeable value.<br/>• **Slave  (duty_slave.v):** on each rising `clk` when the `rollover` strobe is asserted (timer == period), latches the master’s `duty_cycle_stored` into `duty_cycle_out` (or clears to 0 on `rst`), ensuring a clean, synchronized update in the timer domain. 

6.  **Comparator for TMR and Duty_cycle** 
            <br/>| **comparator_a.v** | Compares `TMR` and `duty_cycle_out`. When equal, sets `R = 1` to reset the SR latch and make PWM output low. 

7.  **SR_Latch** 
            <br/>|**sr_latch.v**| SR latch that drives `pwm_signal`.  
                     - On `rst`, forces low.  
                     - When `R=1` (timer == duty), resets low.  
                     - When `S=1` (timer == period), sets high.  
                     - Otherwise holds its state.
