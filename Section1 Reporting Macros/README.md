## 1.Working with Reporting Macro
### This is a basic example of UVM_INFO reporting macros 

``` 
`include "uvm_macros.svh"
 import uvm_pkg::*;

    module tb;
  
    int data = 101;
  
    initial begin
    `uvm_info("TB_TOP", $sformatf("Value of data : %d", data) , UVM_LOW);  
    end
  
    endmodule
``` 
### Simulation Result 
![alt_text](1.Working%20with%20Reporting%20Macros.png) 

