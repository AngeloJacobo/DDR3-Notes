# DDR3-Notes

A DRAM chip contains multiple banks. A bank contains multiple DRAM rows and 1 row of sense amplifiers. A DRAM row contains a row of DRAM cells. A DRAM cell is composed of a capacitor and an access transistor. Only 1 row can be open in an open bank. 
![image](https://user-images.githubusercontent.com/87559347/210326311-5a2c8eee-7705-4c6f-a9f6-e7b116c0fca2.png)

## Steps:
1) Activate = Transferring the charge from DRAM cells of a single DRAM row to sense amp. This is destructive so the sense amp will then restore the DRAM cells with the same value. Input: Bank and Row address
2) Read/Write = Read/Write to a specified column. Input: Bank and Column address
3) Precharge = Close current row and prepare the sense amp for next activation. The sense amp is restored to half-VDD (neither 0 nor 1). The DRAM cell must already be restored to its original value(via Activate) before precharging. Input: Bank address

There are other commands to refresh other rows and bank that is not used. Retention time is usually 64ms above so 64ms is used usually as the refresh time.

## Latency:
 - Ready-to-access Latency = Activate->Read/Write. Time required to move charge from DRAM cell to sense amp before data can be read. At this time, the sense amp might not have yet restored the DRAM cell to its original value as long as the sense amp is already charged.

 - Activation Latency/tRAS= Activate->Precharge. Time required to ensure the sense amp have already restored the DRAM cell to its original value before closing the row via Precharge. Activate is now complete. 

 - Precharge Latency = Precharge->Activate. Time required to restore the sense amp to "normal state" (half-VDD where it is neither 0 nor 1 but middle) before the row can be activated again

![image](https://user-images.githubusercontent.com/87559347/210330740-06b1eb74-d0ab-462c-b947-80c6e3db7fe4.png)


### Note: The timing parameters are set for worst case, so we can reduce the timing delay slightly to optimise it further 


SDR -> DDR ->DDR2 -> DDR3 -> DDR4 -> LPDDR (Low-power DDR)                      

## Refresh
A refresh period is usually 64ms. If there are 8192 rows (13 bit row address), then 64ms/8192rows=7.8us/row. So a row must be refreshed EVERY 7.8us to cover all 8192 rows. Now this is a bottleneck since refresh takes time, what we can do is to **access other bank while another bank is being refreshed.**   
- Auto-refresh = The controller gives clock
- Self-refresh = No need for external clock

## Structure Hierarchy:
1. CPU system (@800MHZ to GHz clock frequency)
2. DDR Controller 
3. DDR PHY = (@ ~200MHz clock frequency)
4. DDR Memory inside IO_wrapper (@200MHz clock frequency)

In CPU, memory becomes bottleneck. Intel i7 is at 3GHz with 64 bit data path (192Gbit/sec). However, the DDR memory core can only work at 200MHz since higher frequency means tighter timing constraint thus lower manufacturing yield. To compensate for this low frequency, prefetching is used on the DDR memory. 2n-prefetching fetches two words at same time to be stored to internal buffer. This will then be released by the DDR memory on the positive and negative edge of clock. DDR also needs DLL (Delay-Locked Looped to align DQ to DQS) and PLL (Phase-Locked Loop used by PHY to generate input clock for DDR memory).


## Pins 
{we, cas, ras} = command control
A0-A13 = Row/Column Address Bus
BA0-BA1 = Bank Address Bus
DQ0-DQN = Data (Q for "output of FF, bidirectional for read/write) 
DQM = Data mask (unidirectional for write only)
DQS = Data strobe, clock used by data (bidirectional for read/write)


Bit line is Precharge to VDD/2 so that reading/writing a DRAM cell is faster (bit line needs only small charged to latch). Sense amp are just SRAM cell (a D-latch with 6 CMOS) so access is faster.


SDR don't have prefetch architecture and DQS (only DQM for masking). 


# Steps
1. Initialize
2. Set 2 registers: mode register (latency, burst length...) and extended mode register (DLL settings) 
 3.  Activate row
 4. Read/Write (use 2n prefetch) 
 5. Precharge (close row by precharging bitline to VDD/2)

X4 (8 instance), x8(4 instance), x16(2 instance), = 32 bit word
X8 means 8 bit data bus. So 4 instance will be needed to have a total of 32 bit word. 

In DDR3, there is no page burst just 4 and 8 burst. 




# Reference:
https://safari.ethz.ch/projects_and_seminars/spring2022/doku.php?id=softmc
