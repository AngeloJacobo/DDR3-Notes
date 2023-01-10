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

## Latency:
Latency or delay is due to the fact that DRAM uses capacitor which induces delay.
 - Ready-to-access Latency = Activate->Read/Write. Time required to move charge from DRAM cell to sense amp before data can be read. At this time, the sense amp might not have yet restored the DRAM cell to its original value as long as the sense amp is already charged.

 - Activation Latency/tRAS= Activate->Precharge. Time required to ensure the sense amp have already restored the DRAM cell to its original value before closing the row via Precharge. 

 - Precharge Latency = Precharge->Activate. Time required to restore the sense amp to "normal state" (half-VDD where it is neither 0 nor 1 but middle) before the row can be activated again

![image](https://user-images.githubusercontent.com/87559347/210330740-06b1eb74-d0ab-462c-b947-80c6e3db7fe4.png)
![image](https://user-images.githubusercontent.com/87559347/211511172-65bc9fa4-0899-4841-82d6-a447ae69d9c5.png)


### Note: The timing parameters are set for worst case, so we can reduce the timing delay slightly to optimise it further                 

## Structure Hierarchy:
1. CPU system (@800MHZ to GHz clock frequency)
2. DDR Controller 
3. DDR PHY = (@ ~200MHz clock frequency)
4. DDR Memory inside IO_wrapper (@200MHz clock frequency)

In CPU, memory becomes bottleneck. Intel i7 is at 3GHz with 64 bit data path (192Gbit/sec). However, the DDR memory core can only work at 200MHz since higher frequency means tighter timing constraint thus lower manufacturing yield. To compensate for this low frequency, prefetching is used on the DDR memory. 2n-prefetching fetches two words at same time to be stored to internal buffer. This will then be released by the DDR memory on the positive and negative edge of clock. DDR also needs DLL (Delay-Locked Looped to align DQ to DQS) and PLL (Phase-Locked Loop used by PHY to generate input clock for DDR memory).


## Pins 
- {we, cas, ras} = command control
- A0-A13 = Row/Column Address Bus
- BA0-BA1 = Bank Address Bus
- DQ0-DQN = Data (Q for "output of FF, bidirectional for read/write) 
- DQM = Data mask (unidirectional for write only)
- DQS = Data strobe, clock used by data (bidirectional for read/write)

## DRAM Subsystem Organization
![image](https://user-images.githubusercontent.com/87559347/211479497-71289465-53c8-47f0-86a7-6cf455fc31de.png)


## SDR vs DDR  
![image](https://user-images.githubusercontent.com/87559347/210721957-f9eb339d-7710-4a35-8453-a6dce700c819.png)

- SDR don't have prefetch architecture nor DQS (only DQM for masking). 

## Extra Notes  
![image](https://user-images.githubusercontent.com/87559347/210725173-83f4db35-af40-4493-a682-37cc0f00fc87.png)

- X8 means 8 bit data bus. So 4 instance of X8 will be needed to have a total of 32 bit word. X4 (8 instance), x8(4 instance), x16(2 instance)
- In DDR3, there is no page burst unlike in SDR. Just burst lengths of 4 and 8. 
- DQS strobe is like a secondary clock used for data transfers
- In write operation, the DDR memory is sampling DQ every clock edge of input clock. This means the DDR controller is tasked to shift the data so that data is stable at every clock edge. 
- In read operation, the DDR memory changes the data every clock edge of input clock. This means data is stable except on the edges of the DDR memory input clock. The usual way for the DDR controller to read/sample the data is to shift the DDR controller's internal clock by 90° so it can sample the data away from this edge (middle of ages) 

# Onur Lectures

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


Add DRAM simulator

# Reference:
https://safari.ethz.ch/projects_and_seminars/spring2022/doku.php?id=softmc  
https://www.youtube.com/watch?v=dtE3RGmUxDw&t=4491s
