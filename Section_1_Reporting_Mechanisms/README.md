# Section 1 : UVM Reporting Mechanisms

<details>
 <summary><b>1. Working with Reporting Macros</b></summary><br>

This is a basic example of UVM_INFO reporting macros vs $display 

### Code

```systemverilog 
`include "uvm_macros.svh"
 import uvm_pkg::*;

module tb;
  
    int data = 101;
  
    initial begin
    `uvm_info("TB_TOP", $sformatf("Value of data : %d", data) , UVM_LOW);
    $display("Hello World with Display");  
    end
  
endmodule
``` 
### Simulation Result 

![alt text](<Simulation Results/1.WorkingWithReportingMacros.png>)

</details>

_____________________________________________________________________

<details><summary><b>2.Printing Values of variables without automation</b></summary>

### Code

```systemverilog
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

![alt text](<Simulation Results/2.Printing Values of variables without automation.png>)

</details>

_____________________________________________________________________

<details><summary><b>3.Working with Verbosity level</b></summary>

### Code

```systemverilog  
`include "uvm_macros.svh"
import uvm_pkg::*;

module tb;
  initial begin
    uvm_top.set_report_verbosity_level(UVM_HIGH);  // HIGH means HIGH and lower than HIGH i.e. LOW, MEDIUM will print
    $display("Default Verbosity Level:%0d", uvm_top.get_report_verbosity_level);
    `uvm_info("TB_TOP", "String", UVM_HIGH);
  end
  
endmodule
```

### Simulation Result

![alt text](<Simulation Results/3.Working with Verbosity level.png>)

</details>

_____________________________________________________________________

<details><summary><b>4.Working with Verbosity level and ID</b></summary><br>

### Code

```systemverilog
`include "uvm_macros.svh"
import uvm_pkg::*;
`include "uvm_macros.svh"
import uvm_pkg::*;
//////////////////////////////////////////////////
class driver extends uvm_driver;
  `uvm_component_utils(driver)
  
  function new(string path, uvm_component parent);
    super.new(path, parent);
  endfunction
  
  task run();
    `uvm_info("DRV1", "Executed Driver1 Code", UVM_HIGH);
    `uvm_info("DRV2", "Executed Driver2 Code", UVM_HIGH);
  endtask 
endclass

module tb;
  
  driver drv;
  
  initial begin
    drv = new("DRV", null);
    drv.set_report_id_verbosity("DRV1",UVM_HIGH);
    //drv.set_report_verbosity_level(UVM_HIGH);
    drv.run();
  end
```
### Simulation

![alt text](<Simulation Results/4.Working with Verbosity level and ID.png>)

</details>

_____________________________________________________________________

<details><summary><b>5.Working with Individual Component</b></summary><br>

### Code

```systemverilog
`include "uvm_macros.svh"
import uvm_pkg::*;
 
//////////////////////////////////////////////////
class driver extends uvm_driver;
  `uvm_component_utils(driver)
  
  function new(string path , uvm_component parent);
    super.new(path, parent);
  endfunction
   
  task run();
    `uvm_info("DRV1", "Executed Driver1 Code", UVM_HIGH);
    `uvm_info("DRV2", "Executed Driver2 Code", UVM_HIGH);
  endtask
  
endclass
 
//////////////////////////////////////////////////
 
class env extends uvm_env;
  `uvm_component_utils(env)
  
  function new(string path , uvm_component parent);
    super.new(path, parent);
  endfunction
  
  
  task run();
    `uvm_info("ENV1", "Executed ENV1 Code", UVM_HIGH);
    `uvm_info("ENV2", "Executed ENV2 Code", UVM_HIGH);
  endtask
  
endclass
////////////////////
 
 
module tb;
  
 driver drv;
 env e;
  
  
 initial begin
   drv = new("DRV", null);
   e   = new("ENV", null);

//by deafult : verbosity level is UVM_MEDIUM , so none of the uvm info messages will be displayed without setting them to UVM_HIGH 

//    e.set_report_verbosity_level(UVM_HIGH);
//    drv.set_report_verbosity_level(UVM_HIGH);
   drv.run();
   e.run();
    
  end
 
endmodule
```
### Simulation

![alt text](<Simulation Results/5.Working with Individual Component.png>)



</details>

_____________________________________________________________________

<details>
 <summary><b>To be updated<b></summary><br>




</details>

_____________________________________________________________________

