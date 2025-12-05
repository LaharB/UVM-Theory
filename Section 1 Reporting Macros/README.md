## Section 1 : Working with Reporting Macro
### This is a basic example of UVM_INFO reporting macros 

### Code

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

### 2.Printing Values of variables without automation

### Code

```
`include "uvm_macros.svh"
import uvm_pkg::*;

module tb;
  
  int data = 101;
  
  initial begin
    `uvm_info("TB_TOP", $sformatf("Value of data : %0d", data) , UVM_LOW); 
  end

endmodule

```
### Simualtion 

![alt_text](2.Printing%20Values%20of%20variables%20without%20automation.png)

### Will update more...... 



