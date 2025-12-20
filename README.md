This repository consists of all the basic concepts and practices required to learn UVM coding 

- Section 1 : UVM Reporting Macros
- Section 2 : Base Classes Part 1 : UVM_OBJECT
- Section 3 : Base Classes Part 2 : UVM_COMPONENT
- Section 4 : config_db
- Section 5 : UVM_PHASES
- Section 6 : TLM
- Section 7 : Sequence

<details><summary><b>Section 1 : UVM Reporting Mechanisms</b></summary><br>

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

![alt text](<Section_1_Reporting_Mechanisms/Simulation Results/1.WorkingWithReportingMacros.png>)

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

![alt text](<Section_1_Reporting_Mechanisms/Simulation Results/2.Printing Values of variables without automation.png>)

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

![alt text](<Section_1_Reporting_Mechanisms/Simulation Results/3.Working with Verbosity level.png>)

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

![alt text](<Section_1_Reporting_Mechanisms/Simulation Results/4.Working with Verbosity level and ID.png>)

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

![alt text](<Section_1_Reporting_Mechanisms/Simulation Results/5.Working with Individual Component.png>)

</details>

___________________________________________________________________

<details>
 <summary><b>6.Working with Hierarchy</b></summary><br>

From tb_top , we can set verbosity level of reporting macros for an entire heirarchy of classes.

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
    `uvm_info("DRV", "Executed Driver Code", UVM_HIGH);
  endtask
  
endclass
///////////////////////////////////////////////////
 
class monitor extends uvm_monitor;
  `uvm_component_utils(monitor)     
  
  function new(string path , uvm_component parent);
    super.new(path, parent);
  endfunction
 
  
  task run();
    `uvm_info("MON", "Executed Monitor Code", UVM_MEDIUM);
  endtask
  
endclass
 
//////////////////////////////////////////////////
 
class env extends uvm_env;
  `uvm_component_utils(env)
  
  driver drv;
  monitor mon;
  
  function new(string path , uvm_component parent);
    super.new(path, parent);
  endfunction
  
  
  
  task run();
    drv = new("DRV", this);
    mon = new("MON", this);
    drv.run();
    mon.run();
  endtask
  
endclass
////////////////////
 
 
module tb;
  
 
 env e;
  
  
 initial begin
   e  = new("ENV", null);
   //e.set_report_verbosity_level_hier(UVM_HIGH);
   e.set_report_verbosity_level_hier(UVM_MEDIUM); //only uvm-info of MON will show message on console 
   e.run(); 
  end
  
  
endmodule
```
### Simulation

![alt text](<Section_1_Reporting_Mechanisms/Simulation Results/6.Working with Hierarchy.png>)

</details>

______________________________________________________________________

<details><summary><b>7.Other Working Macros</b></summary><br>

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
    `uvm_info("DRV", "Informational Message", UVM_NONE);
    `uvm_warning("DRV", "Potential Error");
    `uvm_error("DRV", "Real Error");
     #10;
    `uvm_fatal("DRV", "Simulation cannot continue");
  endtask
  
endclass
 
/////////////////////////////////////////////
  
module tb;
  driver d;
  
  initial begin
    d = new("DRV", null);
    d.run();
  end  
endmodule
```
### Simulation

![alt text](<Section_1_Reporting_Mechanisms/Simulation Results/7.Other Working Macros.png>)

</details>

_____________________________________________________________________

<details><summary><b>8.Changing severity of Macros</b></summary><br>

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
    `uvm_info("DRV", "Informational Message", UVM_NONE);
    `uvm_warning("DRV", "Potential Error");
    `uvm_error("DRV", "Real Error");
     #10;
    `uvm_fatal("DRV", "Simulation cannot continue");
    #10
    `uvm_fatal("DRV1", "Simulation Cannot Continue");
  endtask
  
 
  
endclass
 
/////////////////////////////////////////////
 
 
module tb;
  driver d;
  
  initial begin
    d = new("DRV", null);
    d.set_report_severity_id_override(UVM_FATAL,"DRV", UVM_ERROR);
    d.run();
  end  
endmodule
```
### Simulation

![alt text](<Section_1_Reporting_Mechanisms/Simulation Results/8.Changing severity of Macros.png>)

</details>

_________________________________________________________________________

<details><summary><b>9.Changing Associated Action of Macros</b></summary><br>

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
    `uvm_info("DRV", "Informational Message", UVM_NONE);
    `uvm_warning("DRV", "Potential Error");
    `uvm_error("DRV", "Real Error"); ///uvm_count
    `uvm_error("DRV", "Second Real Error");
    
     #10;
    `uvm_fatal("DRV", "Simulation cannot continue DRV1"); /// uvm_exit
    #10;
    `uvm_fatal("DRV1", "Simulation Cannot Continue DRV1");
   
  endtask
  
 
  
endclass
 
/////////////////////////////////////////////
 
 
module tb;
  driver d;
  
  initial begin
    d = new("DRV", null);
    d.set_report_max_quit_count(3);
 
    d.run();
  end
    
endmodule
```
### Simulation

![alt text](<Section_1_Reporting_Mechanisms/Simulation Results/9.Changing Associated Action of Macros.png>)

</details>

_____________________________________________________________________

<details><summary><b>10.Working with log file</b></summary><br>

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
    `uvm_info("DRV", "Informational Message", UVM_NONE);
    `uvm_warning("DRV", "Potential Error");
    `uvm_error("DRV", "Real Error"); 
    `uvm_error("DRV", "Second Real Error");
  endtask
  
 
  
endclass
 
/////////////////////////////////////////////
 
 
module tb;
  driver d;
  int file;
  
  initial begin
    file = $fopen("log.txt", "w");
    d = new("DRV", null);
    d.set_report_default_file(file);
   
    //we want UVM_ERROR, UVM_INFO , UVM_WARNING to behave like UVM_LOG and UVM_DISPLAY actions
    d.set_report_severity_action(UVM_ERROR, file); //to save only UVM_ERROR into the file 
    d.set_report_severity_action(UVM_ERROR, UVM_DISPLAY | UVM_LOG);
    
    /*
    d.set_report_severity_action(UVM_INFO, UVM_DISPLAY | UVM_LOG);
    d.set_report_severity_action(UVM_WARNING, UVM_DISPLAY | UVM_LOG);
    
 	*/
    
    d.run();
    
    #10;
    $fclose(file);
  end
  
endmodule
```
### Simulation

![alt text](<Section_1_Reporting_Mechanisms/Simulation Results/10.Working with log file Part1.png>)
![alt text](<Section_1_Reporting_Mechanisms/Simulation Results/10.Working with log file Part2.png>)

</details>

</details>

__________________________________________________________

<details><summary><b>Section 2 : UVM Base Classes P1 : UVM_OBJECT</b></summary><br>

<details>
 <summary><b>11.Using field macros P1 INT</b></summary><br>

This is a basic example of how to register data members to uvm_field_macros for using automation.

### Code

```systemverilog 
`include "uvm_macros.svh";
import uvm_pkg::*;

class obj extends uvm_object;
  //`uvm_object_utils(obj)
  
  function new(string path = "obj");
    super.new(path);
  endfunction
  
  rand bit [3:0] a;
  
  `uvm_object_utils_begin(obj)
  `uvm_field_int(a, UVM_DEFAULT | UVM_BIN);
  `uvm_object_utils_end
  
endclass

module tb;
  
  obj o;
  
  initial begin
    o = new("obj");
    o.randomize();
    o.print(uvm_default_table_printer);
  end
``` 
### Simulation Result 

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/11.Using field macros P1 INT.png>)

</details>

</details>

