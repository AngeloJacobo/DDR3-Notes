# DDR3-Notes

A DRAM chip contains multiple banks. A bank contains multiple DRAM rows and 1 row of sense amplifiers. A DRAM row contains a row of DRAM cells. A DRAM cell is composed of a capacitor and an access transistor. Only 1 row can be open in an open bank. 
![image](https://user-images.githubusercontent.com/87559347/210326311-5a2c8eee-7705-4c6f-a9f6-e7b116c0fca2.png)

# Steps:
1) Activate a Row = Transferring the charge from DRAM cells of a single DRAM row to sense amp. This is destructive so the sense amp will then restore the cells with the same value. Input: Bank and Row address
2) Read/Write = Read/Write to a specified column. Input: Bank and Column address
3) Precharge = Close current row and prepare the sense amp for next activation. The sense amp is restored to half-VDD (neither 0 nor 1). The cell must already be restored to its original value(via Activate) before precharging. Input: Bank address

There are other commands to refresh other rows and bank that is not used. Retention time is usually 64ms above so 64ms is used usually as the refresh time.

# Latency:
Ready-to-access Latency = Activate->Read/Write. Time required to move charge from cell to sense amp  (restore phase might not be finished at this, at least the transfer of charge from cell to sense amp is finished) 

Activation Latency/= Activate->Precharge. Time required to ensure the sense amp have already restored fully the cell (restore phase of Activate) BEFORE CLOSING IT VIA PRECHARGE.
Also called t_RAS = Time required before cell is restored back to original value. 

Precharge Latency = Precharge->Activate. Time required to restore the sense amp to "normal state" or half-VDD where it is neither 0 nor 1 but middle. 

## The timing parameters are set for worst case, so we can reduce the timing delay slightly to optimise it further 

# Reference:
https://safari.ethz.ch/projects_and_seminars/spring2022/doku.php?id=softmc
