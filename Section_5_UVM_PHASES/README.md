# Section 5 : UVM_PHASES

<details>
 <summary><b>30.How to override phases</b></summary><br>

Only Build phase executes in top to bottom fashion.

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

class test extends uvm_test;
  `uvm_component_utils(test)  //uvm_test belongs to uvm_component (which is dynamic)
  
  function new(string path = "test", uvm_component parent = null);
   super.new(path, parent);             
  endfunction
  
  //////////////////////construction phases/////////////////////
  //1.build_phase
  function void build_phase(uvm_phase phase);//good to use virtual before function
   super.build_phase(phase);
   `uvm_info("test", "Build phase executed", UVM_NONE);
  endfunction
               
  //2.connect_phase
  function void connect_phase(uvm_phase phase);//good to use virtual before function
   super.connect_phase(phase);
   `uvm_info("test", "Connect phase executed", UVM_NONE);
  endfunction
  
  //3.end_of_elaboration_phase
  function void end_of_elaboration_phase(uvm_phase phase);//good to use virtual before function
   super.end_of_elaboration_phase(phase);
   `uvm_info("test", "End of Elaboration phase executed", UVM_NONE);
  endfunction
               
  //4.start_of_simulation_phase
  function void start_of_simulation_phase(uvm_phase phase);//good to use virtual before function
   super.start_of_simulation_phase(phase);
   `uvm_info("test", "Start of Simulation phase executed", UVM_NONE);
  endfunction
               
  /////////////////////////Main Phases////////////////////////////////////////
               
  //run_phase
  task run_phase(uvm_phase phase);
    `uvm_info("test", "Run Phase executed", UVM_NONE);               
  endtask
                
  /////////////////////////Cleanup Phases///////////////////////////////////
  //1.extract_phase
  function void extract_phase(uvm_phase phase);
    super.extract_phase(phase);
    `uvm_info("test", "Extract phase executed", UVM_NONE);
  endfunction
  
  //2.check_phase
  function void check_phase(uvm_phase phase);
    super.check_phase(phase);
    `uvm_info("test", "Check phase executed", UVM_NONE);
  endfunction
  
  //3.report_phase
  function void report_phase(uvm_phase phase);
    super.report_phase(phase);
    `uvm_info("test", "Report phase executed", UVM_NONE);
  endfunction
  
  //4.final_phase
  function void final_phase(uvm_phase phase);
    super.final_phase(phase);
    `uvm_info("test", "Final phase executed", UVM_NONE);
  endfunction
               
endclass
  
module tb;
  
  initial begin
    run_test("test");
  end
  
endmodule 

``` 
### Simulation Result 

![alt text](<Simulation Results/30.How to override phases.png>)

</details>

__________________________________________________________

<details>
 <summary><b>31.Execution of build_phase in multiple components</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

//only build_phase executes in top_down fashion rest all in botom_top fashion

class driver extends uvm_driver;
  `uvm_component_utils(driver)
  
  function new(string path = "driver", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    `uvm_info("driver", "Driver Build Phase Executed", UVM_NONE);
  endfunction
  
endclass
//////////////////////////////////////////////////////////////////////////
class monitor extends uvm_monitor;
  `uvm_component_utils(monitor)
  
  function new(string path = "monitor", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    `uvm_info("monitor", "Monitor Build Phase Executed", UVM_NONE);
  endfunction
  
endclass
//////////////////////////////////////////////////////////////////////////
class env extends uvm_env;
  `uvm_component_utils(env)
  
  driver drv;  
  monitor mon;
  
  function new(string path = "env", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    `uvm_info("env", "Env Build Phase Executed", UVM_NONE);
    drv = driver::type_id::create("drv", this);
    mon = monitor::type_id::create("mon", this);
  endfunction
  
endclass
//////////////////////////////////////////////////////////////////////////
class test extends uvm_test;
  `uvm_component_utils(test)
  
  env e;  
  
  function new(string path = "test", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    `uvm_info("test", "Test Build Phase Executed", UVM_NONE);
    e = env::type_id::create("e", this);   
  endfunction
  
endclass
///////////////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("test");
  end
  
endmodule 

``` 
### Simulation Result 

![alt text](<Simulation Results/31.Execution of build_phase in multiple components.png>)

</details>

__________________________________________________________

<details>
 <summary><b>32.Understanding execution of connect phase</b></summary><br>

Connect phase and the other phases execute in bottom to top fashion.

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

////connect_phase executes in bottom_top fashion
class driver extends uvm_driver;
  `uvm_component_utils(driver)  
  
  function new(string path = "driver", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    `uvm_info("driver", "Driver Connect Phase Executed", UVM_NONE);   
  endfunction
  
endclass
//////////////////////////////////////////////////////////////////////
class monitor extends uvm_monitor;
  `uvm_component_utils(monitor)  
  
  function new(string path = "monitor", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    `uvm_info("monitor", "Monitor Connect Phase Executed", UVM_NONE);   
  endfunction
  
endclass
//////////////////////////////////////////////////////////////////////
class env extends uvm_env;
  `uvm_component_utils(env)
  
  driver d;
  monitor m;
  
  function new(string path = "env", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    d = driver::type_id::create("d", this);
    m = monitor::type_id::create("m", this);
  endfunction
  
  function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    `uvm_info("env", "Env Connect Phase Executed", UVM_NONE);   
  endfunction
  
endclass
//////////////////////////////////////////////////////////////////////
class test extends uvm_test;
  `uvm_component_utils(test)
  
  env e;
  
  function new(string path = "test", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    e = env::type_id::create("e", this);
  endfunction
  
  function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    `uvm_info("test", "Test Connect Phase Executed", UVM_NONE);   
  endfunction
  
endclass
////////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("test");
  end
endmodule 

``` 
### Simulation Result 

![alt text](<Simulation Results/32.Understanding exec of connect phase.png>)

</details>

__________________________________________________________