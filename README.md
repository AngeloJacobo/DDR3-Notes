# DDR3-Notes

A DRAM chip contains multiple banks. A bank contains multiple DRAM rows and 1 row of sense amplifiers. A DRAM cell is composed of a capacitor and an access transistor. Only 1 row can be open in an open bank. 
![image](https://user-images.githubusercontent.com/87559347/210326311-5a2c8eee-7705-4c6f-a9f6-e7b116c0fca2.png)

## Steps:
1. Initialize
2. Set 2 registers: Mode Register (latency, burst length...) and Extended Mode Register (DLL settings) 
3. Activate = Transferring the charge from a DRAM row to sense amp. This is destructive so the sense amp will then restore the value of the DRAM cells. *Input: Bank and Row address*
4. Read/Write = Read/Write to a specified column. *Input: Bank and Column address*
3. Precharge = Close current row and prepare the sense amp for next activation. The sense amp is restored to VDD/2. The DRAM cell must already be restored to its original value before precharging. *Input: Bank address*  

Precharging charges the bit line and sense amp to VDD/2 so that reading/writing a DRAM cell is faster (the bit line needs only small charge to latch either direction). Sense amp are just SRAM cells (a D-latch with 6 CMOS so access is fast). There are other commands to refresh other rows and bank that is not used. Retention time is usually 64ms above so 64ms is used usually as the refresh time.

## Refresh:
A refresh period is usually 64ms (32ms for higher temp since leakage is higher). If there are 8192 rows (13-bit row address), then 64ms/8192rows=7.8us/row. So a row must be refreshed EVERY 7.8us to cover all 8192 rows. Now this is a bottleneck since refresh takes time, what we can do is to **access other bank while another bank is being refreshed.** Refresh is essentially activate + precharge a row every 64ms. Distributed refresh is what used mostly in today's controllers than burst refresh (all rows are refreshed consecutively immediately which has long pause time). DRAM Bank is unavailable during refresh. 
- Auto-refresh = The controller gives external clock
- Self-refresh = No need for external clock  

In the power-down state (precharge power-down or active power-down), the DRAM still requires refresh commands to be sent to maintain the memory contents. Since no refresh operations are performed in this mode, the device may not remain in the power-down state longer than the refresh period (64ms) 

## Latency:
Latency or delay is due to the fact that DRAM uses capacitor which induces delay.
 - Ready-to-access Latency = Activate->Read/Write. Time required to move charge from DRAM cell to sense amp before data can be read. At this time, the sense amp might not have yet restored the DRAM cell to its original value as long as the sense amp is already charged.

 - Activation Latency/tRAS= Activate->Precharge. Time required to ensure the sense amp have already restored the DRAM cell to its original value before closing the row via Precharge. 

 - Precharge Latency = Precharge->Activate. Time required to restore the sense amp to "normal state" (half-VDD where it is neither 0 nor 1 but middle) before the row can be activated again

![image](https://user-images.githubusercontent.com/87559347/210330740-06b1eb74-d0ab-462c-b947-80c6e3db7fe4.png)
![image](https://user-images.githubusercontent.com/87559347/211511172-65bc9fa4-0899-4841-82d6-a447ae69d9c5.png)


### Note: The timing parameters are set for worst case, so we can reduce the timing delay slightly to optimise it further                 

## DDR Controller Structural Hierarchy:
1. CPU system (@800MHZ to GHz clock frequency)
2. DDR Controller 
3. DDR PHY = (@ ~200MHz clock frequency)
4. DDR Memory inside IO_wrapper (@200MHz clock frequency)

In CPU, memory becomes bottleneck. Intel i7 is at 3GHz with 64 bit data path (192Gbit/sec). However, the DDR memory core can only work at 200MHz since higher frequency means tighter timing constraint thus lower manufacturing yield. To compensate for this low frequency, prefetching is used on the DDR memory. 2n-prefetching fetches two words at same time to be stored to internal buffer. This will then be released by the DDR memory on the positive and negative edge of clock. DDR also needs DLL (Delay-Locked Looped to align DQ to DQS) and PLL (Phase-Locked Loop used by PHY to generate input clock for DDR memory).


## Pins:
- {we, cas, ras} = command control
- A0-A13 = Row/Column Address Bus
- BA0-BA1 = Bank Address Bus
- DQ0-DQN = Data (Q for "output of FF, bidirectional for read/write) 
- DQM = Data mask (unidirectional for write only)
- DQS = Data strobe, clock used by data (bidirectional for read/write)

## DRAM Subsystem Organization:
![image](https://user-images.githubusercontent.com/87559347/211479497-71289465-53c8-47f0-86a7-6cf455fc31de.png)


## SDR (Single Data Rate) vs DDR (Double Data Rate):  
![image](https://user-images.githubusercontent.com/87559347/210721957-f9eb339d-7710-4a35-8453-a6dce700c819.png)

- SDR don't have prefetch architecture nor DQS (only DQM for masking). 

## On-Die Termination (ODT):
- ODT, or On-Die Termination, is a feature used in DDR3 memory to reduce signal reflections caused by mismatched impedances on the memory bus. When ODT is activated, a resistor is placed in parallel with the memory bus to match the impedance of the bus. The ODT signal is typically driven by the memory controller and is used to control the state of the ODT resistor. A low signal in ODT corresponds to the ODT resistor being turned off (high impedance since not connected), while a high signal corresponds to the ODT resistor being turned on.

- When the memory controller sends a write command to the memory device, it also asserts the ODT signal. This causes the ODT resistor to be placed in parallel with the memory bus, effectively matching the impedance of the bus and reducing signal reflections. By having the ODT resistor turned on during a write operation, it helps to ensure that the write data is properly received by the memory device.

- ODT is usually turned off during read operations, but this is not because of the absence of signal reflections during read operations, but rather the ODT is turned off to minimize the noise that may be present on the memory bus and improve the signal integrity.

- When you enable dynamic ODT, and there is no write operation, the DDR3 SDRAM terminates to a termination setting of RTT_NOM; when there is a write operation, the DDR3 SDRAM terminates to a setting of RTT_WR. You can preset the values of RTT_NOM and RTT_WR by programming the mode registers, MR1 and MR2.

- ZQ calibration is a process used in DDR3 memory systems to adjust precisely the internal impedance of the DDR for its ODT impedances and internal buffers. The ZQ pin is linked to a highly precise external resistor, which is used for high definition adjustments of the “On” impedance of output drivers and ODT impedances.

## DDR Read and Write Operation:
- In write operation, the DDR memory is sampling DQ every edge of DQS (the DQS is aligned to the input clock). This means the DDR controller is tasked to shift the data so that data is stable at every clock edge. 
- In read operation, the DDR memory changes the data every edge of DQS (the DQS is aligned to the input clock). This means data is stable except on the edges of the DDR memory input clock. The usual way for the DDR controller to read/sample the data is to shift the DDR controller's internal clock by 90° so it can sample the data on the middle of data eye.
- On writes, the host sends DQS phase shifted by 90 degrees and the DRAM catches DQ in the middle of its validity window (using DQS as the clock edge). On reads, the DRAM sends DQS aligned with DQ, and the host (the controller) phase-shifts DQS internally using a Delay Locked Loop (DLL) so that it can capture DQ on the middle of its data eye.  
![image](https://user-images.githubusercontent.com/87559347/212459471-fb62f163-1973-4428-a3ac-8bf4c93bc2df.png)

## Extra Notes:  
![image](https://user-images.githubusercontent.com/87559347/211517717-3ed998da-2816-47d8-b65e-24ce6662a99a.png)

- X8 means 8 bit data bus. So 4 instance of X8 will be needed to have a total of 32 bit word. X4 (8 instance), x8(4 instance), x16(2 instance)
- In DDR3, there is no page burst unlike in SDR. Just burst lengths of 4 and 8. 
- DQS strobe is like a secondary clock used for data transfers



- High throughput sequencing necessary in Genome analysis is limited by data movement of DRAM (memory bottleneck)
- Core count doubling every 2 years. DRAM DIMM capacity doubling (via putting more cells in a die) every 3 years. DRAM bandwidth (xGbit/sec) trend increase slower (via increasing clock rate and pin count). Latency did not changed much in past 20 years since this is limited by capacitor physics itself.
- LPDDR (Low Power DDR) are low power but higher latency.
- Refresh is a big downside of DRAM. In processors, 50% of the time is wasted waiting for memory. About 40% power is wasted on DRAM due to refresh cycle.
- Heterogenous system = A CPU core has multiple memory sources: DRAM, NVM, GPU, and more. Memory sources are different from each other and is thus "heterogenous".
- Activation moves DRAM row charge to row buffer of Sense Amplifier. This row buffer is essentially a cache which increases hit rate if access is consecutive.
- Rank = Consist of multiple DRAM chips to form a wide data interface (share address and command busses but provide different data, so an X8 DRAM chip will need 4 DRAM chips in 1 rank to form a 32-bit interface).
- Channel = Different channel means a separate/different 64-bit data interface and memory controller.
- The banks shares the address, data, and command buses.
- With more banks, pipeline-access increases (accessing banks consecutively with least delay due to precharging+activate). With more channel, bandwidth increases since there are more data interface.
- Address mapping tells how address bits are separated to row-bank-column address. Cache block interleaving is the most common one.
![image](https://user-images.githubusercontent.com/87559347/211482538-9d9a3554-b9e9-4b4b-a004-26b69b5ca230.png)
- Only small percentage of data retention file is actually 64ms (<1%). More than 75% is 256ms. 
![image](https://user-images.githubusercontent.com/87559347/211484745-a459bb27-41f7-4fec-974a-e9c324130204.png)
- Row Buffer Management Policy determines if row buffer will be closed or remain opened after read/write access:
![image](https://user-images.githubusercontent.com/87559347/211484662-71810b34-f0aa-4d19-9af3-4f21cc2d5bc9.png)
- There are now a lot of DRAM types:
![image](https://user-images.githubusercontent.com/87559347/211484917-fd0d628a-e889-4c6f-8a31-0d962976c67e.png)
- DRAM scheduling policies is too complicated:
![image](https://user-images.githubusercontent.com/87559347/211502130-ca59e179-4ee8-4466-b02a-d8cd8a475637.png)
- Activate opens the row and recharge the cell, we can read as long as open phase of activate is finished and then precharged only when recharge phase is over (precharge only when activate is completely finished)
- DRAM simulator is very useful to provide abstraction when trying to add new features. Ramulator is one such simulator:
![image](https://user-images.githubusercontent.com/87559347/211517301-3c90f11c-83d6-43c5-ab21-5eed697164a4.png)

- The DDR3 memory operates at high speeds, and the internal DLL helps to ensure that the data is read and written at the correct time. The DLL takes in the external clock signal and creates a new clock signal that is phase-aligned with the incoming data. 

- The the DRAM core frequency is low (100-200 MHz), but with prefetching high data throughput can be achieved:
![image](https://user-images.githubusercontent.com/87559347/212458190-c17437d8-1f8f-4922-aca7-22e82a578488.png)

## February Notes
You can use Micron DDR3 Simulation module to test the controller even without physical ddr3

ODDR2 is just used so that the internal clock can be directed to the output pin, this is the more likely use than doing DDR using ODDR2

Low Speed: Use the clk (50MHz) and divide the frequency by 4 to generate a slow clock than can be shifted 180 degrees or 90 degrees. LOW SPEED IS USED TO TEST CONTROLLER BEFORE RUNNING IN HIGHER FREQUENCY 

High Speed: Use PLL to generate 333.333 MHz from 50MHZ.(with 0,90,180,270 phase shift) . Then use DPLL to dynamically change phase shift for read operation

OBUF is used to connect internal pins to FPGA pin fabric
 


# Reference (YouTube):  
- [Memory and DRAM Basics Series (Onur Mutlu Lectures)](https://www.youtube.com/playlist?list=PL5Q2soXY2Zi-IymxXpH_9vlZCOeA7Yfn9)  
- [DRAM Basics in 7-part Series (Computer Science)](https://www.youtube.com/playlist?list=PLTd6ceoshpreE_xQfQ-akUMU1sEtthFdB)  
- [DDR DRAM Basics (VLSIGuru Training Institute)](https://www.youtube.com/watch?v=dtE3RGmUxDw&list=PLqaZ9TGIKESdSTNiqb3M0FTcI0iNqShG0&index=4&t=5096s)  
- [SoftMC and RowHammer Series (Onur Mutlu Lectures)](https://www.youtube.com/playlist?list=PL5Q2soXY2Zi_1trfCckr6PTN8WR72icUO)  
- [Memory Controllers (Onur Mutlu Lectures)](https://www.youtube.com/watch?v=TeG773OgiMQ&list=PLqaZ9TGIKESdSTNiqb3M0FTcI0iNqShG0&index=6&t=1232s)
- [DDR4 Timings Explained Manually Series (Actually Hardcore Overclocking)](https://www.youtube.com/playlist?list=PLpS0n7xxSadUJE1fEuWfEMGvmMsVYGAbA)
- [Transprecision-aware DDR Controller with PHY (oprecomp)](https://www.youtube.com/watch?v=Oet4I5u7HOE&list=PLqaZ9TGIKESdSTNiqb3M0FTcI0iNqShG0&index=6&t=1303s)

# Reference (Sites):
- [DRAM Basic Commands](https://www.allaboutcircuits.com/technical-articles/executing-commands-memory-dram-commands/) 
- [SDRAM as simple state machine (SERIES)](https://www.anandtech.com/show/3851/everything-you-always-wanted-to-know-about-sdram-memory-but-were-afraid-to-ask/3)  
- [DDR4 SDRAM - Initialization, Training and Calibration](https://www.systemverilog.io/ddr4-initialization-and-calibration)  
- [DDR operation and timing parameters](https://course.ccs.neu.edu/com3200/parent/NOTES/DDR.html)  
- [DDR Read and Write Operation](https://www.design-reuse.com/articles/4916/external-memory-interfaces-delivering-bandwidth-to-silicon.html)
- [DDR3 Write Leveling and Read Leveling](https://daffy1108.wordpress.com/2010/09/02/understanding-ddr3-write-leveling-and-read-leveling/)  
- [Why need write and read leveling? (SERIES)](https://bit-tech.net/reviews/tech/memory/the_secrets_of_pc_memory_part_4/4/)
- [More on write and read leveling](https://www.intel.com/content/www/us/en/docs/programmable/683385/17-0/read-and-write-leveling.html)
- [Why use DQS for read/write?](https://electronics.stackexchange.com/questions/484746/why-cant-clock-be-directly-used-instead-of-dqs-in-ddr-during-read-and-write) 
- [ODT and ZQ Calibration (SERIES)](https://bit-tech.net/reviews/tech/memory/the_secrets_of_pc_memory_part_4/5/)  
 






















