# Section 5 : UVM_PHASES

<details>
 <summary><b>30.How to override phases</b></summary><br>

- Only Build phase executes in **top to bottom fashion.**
- Build Phase and Connect phase do not consume time so we declare them using **function + super method.**
- Also , Cleanup phase does not consume any time.
- Run phase consumes so we declare it using **task** 

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

class test extends uvm_test;
  `uvm_component_utils(test)  //uvm_test belongs to uvm_component (which is dynamic)
  
  function new(string path = "test", uvm_component parent = null);
   super.new(path, parent);             
  endfunction
  
  //////////////////////constructio phases/////////////////////
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

- build phase executes in top down fashion because until we call test class first ,the build phase of test class won't get executed and the object of env class won't get created and thus build_phase of env class won't be executed as well.
- likewise if we don't call the env class, the drv and mon object won't get created and thus their respective build phases won't be executed

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

<details>
 <summary><b>33.Execution of multiple instance phases</b></summary><br>

- When there are multiple objects to be created in a class, then whose build_phase will get executed first ?
- For example , inside env, we have drv and mon objects to be created so whose build_phase - drv or mon - will get executed first ?
- Similarly whose connect_phase will get executed first - drv or mon ?

### Code 1

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

//whose build_phase and connect_phase - driver's or monitor's will get executed first ?
// it follows Lexicographic order i.e. lower ASCII values gets first priority
// BUT while using small letters for instance and path name , priority according alphabetical order 

  function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    
    //driver build_phase gets executed first then monitor build_phase
    d = driver::type_id::create("d", this);  //path name should be same as instance name
    m = monitor::type_id::create("m", this); //path name should be same as instance name
  endfunction
 
  
  /*
  //name driver instance's path name as m and monitor's as d 
  //monitor connect_phase gets executed first then driver connect_phase
  d = driver::type_id::create("m", this);  //path name should be same as instance name
  m = monitor::type_id::create("d", this); //path name should be same as instance name
  endfunction
  */
  
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
### Code 2

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

//whose build_phase and connect_phase - driver's or monitor's will get executed first ?
// it follows Lexicographic order i.e. lower ASCII values gets first priority
// BUT while using small letters for instance and path name , priority according alphabetical order 

  function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    /*
    //driver build_phase gets executed first then monitor build_phase
    d = driver::type_id::create("d", this);  //path name should be same as instance name
    m = monitor::type_id::create("m", this); //path name should be same as instance name
  endfunction
 */
  
  
  //name driver instance's path name as m and monitor's as d 
  //monitor connect_phase gets executed first then driver connect_phase
  d = driver::type_id::create("m", this);  //path name should be same as instance name
  m = monitor::type_id::create("d", this); //path name should be same as instance name
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

Using d for driver and m for monitor so driver connect phase gets executed first then monitor's.

![alt text](<Simulation Results/33.Execution of multiple instance phases P1.png>)

Using d for monitor and m for driver so monitor connect phase gets executed first then driver's.

![alt text](<Simulation Results/33.Execution of multiple instance phases P2.png>)

</details>

__________________________________________________________

<details>
 <summary><b>34.Raising Objection</b></summary><br>

- UVM won't automatically wait for the time till our time consuming phases like run phase finish executing 
- So to hold the simulator till this task is completed, we use raise and drop objection.

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

class comp extends uvm_component;
  `uvm_component_utils(comp)
  
  function new(string path = "comp", uvm_component parent);
    super.new(path, parent);
  endfunction
  
//use a run_phase - eg lets use reset_phase
  task reset_phase(uvm_phase phase);
//to hold the simulator till this task is completed, we use raise and drop objection
    phase.raise_objection(this); // "this" -> run_phase has raised objection
    `uvm_info("comp", "Reset Started", UVM_NONE);
    #10;
    `uvm_info("comp", "Reset Completed", UVM_NONE);
    phase.drop_objection(this); 
    
  endtask
  
endclass

module tb;
  
  initial begin
    run_test("comp");
  end 
  
endmodule 
``` 
### Simulation Result 

![alt text](<Simulation Results/34.Raising Objection.png>)

</details>

__________________________________________________________

<details>
 <summary><b>35.How time consuming phases work in a single component ?</b></summary><br>

- Run phases also work sequentially just like other phases i.e until Reset phase gets completed , configure phase wont start executing.
- Once Configure phase gets executed only then main phase will execute 
- Once Main phase gets executed only then Shutdown phase will execute 

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
 
 
class comp extends uvm_component;
  `uvm_component_utils(comp)
  
 
  function new(string path = "comp", uvm_component parent = null);
    super.new(path, parent);
  endfunction
// we have 2 types of run_phases in a single component - "comp"  
  task reset_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("comp","Reset Started", UVM_NONE);
     #10;
    `uvm_info("comp","Reset Completed", UVM_NONE);
    phase.drop_objection(this);
  endtask
  
  task main_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("mon", " Main Phase Started", UVM_NONE);
    #100;
    `uvm_info("mon", " Main Phase Ended", UVM_NONE);
    phase.drop_objection(this);
  endtask
  
endclass
 
///////////////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("comp");
  end

endmodule 
``` 
### Simulation Result 

![alt text](<Simulation Results/35.How time consuming phases work in a single component.png>)

</details>

__________________________________________________________

<details>
 <summary><b>36.Time consuming phases in multiple components</b></summary><br>

- Until and unless the **reset phases** of all components ( here , both drv and mon) gets executed, their **main phases** will not start.
- Similarly for their respective Configure phase and Shutdown phase.

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
 
///////////////////////////////////////////////////////////////
 
class driver extends uvm_driver;
  `uvm_component_utils(driver) 
  
  
  function new(string path = "test", uvm_component parent = null);
    super.new(path, parent);
  endfunction
//until the run_phases of both driver and monitor are over , their main_phases wont start  
  task reset_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("drv", "Driver Reset Started", UVM_NONE);
    #100;
    `uvm_info("drv", "Driver Reset Ended", UVM_NONE);
    phase.drop_objection(this);
  endtask
  
  
  task main_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("drv", "Driver Main Phase Started", UVM_NONE);
    #100;
    `uvm_info("drv", "Driver Main Phase Ended", UVM_NONE);
    phase.drop_objection(this);
  endtask
  
  
 
  
endclass
 
///////////////////////////////////////////////////////////////
 
class monitor extends uvm_monitor;
  `uvm_component_utils(monitor) 
  
  
  function new(string path = "monitor", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  task reset_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("mon", "Monitor Reset Started", UVM_NONE);
     #300;
    `uvm_info("mon", "Monitor Reset Ended", UVM_NONE);
    phase.drop_objection(this);
  endtask
  
  
  task main_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("mon", "Monitor Main Phase Started", UVM_NONE);
     #400;
    `uvm_info("mon", "Monitor Main Phase Ended", UVM_NONE);
    phase.drop_objection(this);
  endtask
  
endclass
 
////////////////////////////////////////////////////////////////////////////////////
 
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
  
 
  
endclass
 
 
 
////////////////////////////////////////////////////////////////////////////////////////
 
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
  
  
endclass
 
///////////////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("test");
  end
   
endmodule 
``` 
### Simulation Result 

![alt text](<Simulation Results/36.Time consuming phases in multiple components.png>)

</details>

__________________________________________________________

<details>
 <summary><b>37.Timeout</b></summary><br>

- In this example , our run phase is taking 110 ns to complete and we have set a sufficient time of 200 ns as limit for timeout so our phases will executed before UVM_FATAL gets called.
- However, if we set the timeout value to say 100 ns , then our phases will not executed completely before UVM_FATAL gets called.

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
///default timeout value in UVM : 9200sec
 
 
class comp extends uvm_component;
  `uvm_component_utils(comp)
  
 
  function new(string path = "comp", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  task reset_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("comp","Reset Started", UVM_NONE);
     #10;
    `uvm_info("comp","Reset Completed", UVM_NONE);
    phase.drop_objection(this);
  endtask
  
  task main_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("mon", " Main Phase Started", UVM_NONE);
    #100;
    `uvm_info("mon", " Main Phase Ended", UVM_NONE);
    phase.drop_objection(this);
  endtask
  
  
  function void build_phase(uvm_phase phase);
    super.build_phase(phase); 
  endfunction
  
  
endclass
//////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    //uvm_top is used to get access to uvm_root 
    //set_timeout has 2 arguements - time value and overridable bit (0/1) by other components
    uvm_top.set_timeout(200ns, 0);  //0 so not overridable
    run_test("comp");   
  end
endmodule  
``` 
### Simulation Result 

![alt text](<Simulation Results/37.Timeout.png>)

</details>

_________________________________________________________

<details>
 <summary><b>38.Drain Time: Individual component</b></summary><br>

- Drain time is used to give some buffer to the DUT after providing it the stimulus so that the DUT can perform the functionality and process the signals and send it back.
- Here, the main phase starts at 10ns and runs for 100ns and after it is completed, it stays in the main phase for another 200ns as we set the drain time to 200ns.

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

class comp extends uvm_component;
  `uvm_component_utils(comp)
  
  function new(string path = "comp", uvm_component parent);
    super.new(path, parent);
  endfunction
  
  task reset_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("comp", "Reset Started", UVM_NONE);
    #10;
    `uvm_info("comp", "Reset Completed", UVM_NONE);
	phase.drop_objection(this);   
  endtask
  
 //using drain time for an individual component          
  task main_phase(uvm_phase phase);
    phase.phase_done.set_drain_time(this, 200); //first main phase starts at 10ns and runs for 100ns and after it is completed, it stays in the main phase for another 200ns as we set the drain time to 200ns.
    phase.raise_objection(this);
    `uvm_info("comp", "Main Phase Started", UVM_NONE);
    #100;
    `uvm_info("comp", "Main Phase Completed", UVM_NONE);
	phase.drop_objection(this);   
  endtask
  
  //After the drain time is over, this post main phase will start at 310ns
  task post_main_phase(uvm_phase phase);
    `uvm_info("comp", "Post-Main Phase Started", UVM_NONE);   
  endtask
  
endclass
/////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("comp");
  end 
  
endmodule  
``` 
### Simulation Result 

![alt text](<Simulation Results/38.Drain Time - Individual component.png>)

</details>

__________________________________________________________

<details>
 <summary><b>39.Drain Time: Multiple component</b></summary><br>

- When working with multiple components, drain time is always specified from end_of_elaboration_phase
- Here, the drain time will start only after all the main phases of all the components have completed at 350ns 

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

class driver extends uvm_driver;
  `uvm_component_utils(driver)
  
  function new(string path = "driver", uvm_component parent);
    super.new(path, parent);
  endfunction
  
  task reset_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("driver", "Driver Reset Started", UVM_NONE);
    #100;
    `uvm_info("driver", "Driver Reset Completed", UVM_NONE);
	phase.drop_objection(this);   
  endtask
           
  task main_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("driver", "Driver Main Phase Started", UVM_NONE);
    #100;
    `uvm_info("driver", "Driver Main Phase Completed", UVM_NONE);
	phase.drop_objection(this);   
  endtask
  
  task post_main_phase(uvm_phase phase);
    `uvm_info("driver", "Driver Post-Main Phase Started", UVM_NONE); 
  endtask
  
endclass
/////////////////////////////////////////////////////////////////
class monitor extends uvm_monitor;
  `uvm_component_utils(monitor)
  
  function new(string path = "monitor", uvm_component parent);
    super.new(path, parent);
  endfunction
  
  task reset_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("monitor", "Monitor Reset Started", UVM_NONE);
    #150;
    `uvm_info("monitor", "Monitor Reset Completed", UVM_NONE);
	phase.drop_objection(this);   
  endtask
           
  task main_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("monitor", "Monitor Main Phase Started", UVM_NONE);
    #200;
    `uvm_info("monitor", "Monitor Main Phase Completed", UVM_NONE);
	phase.drop_objection(this);   
  endtask
  
  task post_main_phase(uvm_phase phase);
    `uvm_info("monitor", "Monitor Post-Main Phase Started", UVM_NONE); 
  endtask
  
endclass
/////////////////////////////////////////////////////////////////
class env extends uvm_env;
  `uvm_component_utils(env)
  
  function new(string path = "monitor", uvm_component parent);
    super.new(path, parent);
  endfunction
  
  driver d;
  monitor m;
  
  function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    d = driver::type_id::create("d", this);
    m = monitor::type_id::create("m", this);
  endfunction
  
endclass
////////////////////////////////////////////////////////////////
class test extends uvm_test;
  `uvm_component_utils(test)
  
  function new(string path = "test", uvm_component parent);
    super.new(path, parent);
  endfunction
  
  env e;
  
  function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    e = env::type_id::create("e", this);  
  endfunction
  
  //drain_time should always be set from end_of_elaboration_phase
  function void end_of_elaboration_phase(uvm_phase phase);
    uvm_phase main_phase;   //variable "main_phase" of type "uvm_phase" to get access of main_phase
    super.end_of_elaboration_phase(phase);
    main_phase = phase.find_by_name("main",0); //to get access to main_phases of all the components
    main_phase.phase_done.set_drain_time(this, 100); //all the main phase will get a drain time of 100ns
  endfunction
 
endclass

////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("test");
  end 
  
endmodule 
``` 
### Simulation Result 

![alt text](<Simulation Results/39.Drain Time - Multiple component.png>)

</details>

__________________________________________________________

<details>
 <summary><b>40.Phase Debug Switch</b></summary><br>

- We have to use the command in run options to see the phase trace report:
+UVM_PHASE_TRACE
- With this command , we can see in the console when the various phases started and completed one after another.

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
 
class comp extends uvm_component;
  `uvm_component_utils(comp)

  function new(string path = "comp", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  task reset_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("comp","Reset Applied", UVM_NONE);
     #100;
    `uvm_info("comp","Reset Removed", UVM_NONE);
    phase.drop_objection(this);
  endtask
  
  task main_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("comp", "Random Stimulus Applied", UVM_NONE);
    #500;
    `uvm_info("comp", "Random Stimulus Removed", UVM_NONE);
    phase.drop_objection(this);
  endtask
  
endclass
 
///////////////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("comp");
  end
  
endmodule 
``` 
### Simulation Result 

### +UVM_PHASE_TRACE

![alt text](<Simulation Results/40.Phase Debug Switch P1 - +UVM_PHASE_TRACE 1.png>)
![alt text](<Simulation Results/40.Phase Debug Switch P1 - +UVM_PHASE_TRACE 2.png>)
![alt text](<Simulation Results/40.Phase Debug Switch P1 - +UVM_PHASE_TRACE 3.png>)
![alt text](<Simulation Results/40.Phase Debug Switch P1 - +UVM_PHASE_TRACE 4.png>)

</details>

__________________________________________________________

<details>
 <summary><b>41.Objection Debug Switch</b></summary><br>

- We have to use the command in run options to see the objection trace report:
+UVM_OBJECTION_TRACE
- With this command, we can see in the console, the various components in the run_phase when they raise and drop an objection
- Everytime an objection is raised, the count goes up by 1 and when it is dropped, the count goes down by 1.

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
 
class comp extends uvm_component;
  `uvm_component_utils(comp)

  function new(string path = "comp", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  task reset_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("comp","Reset Applied", UVM_NONE);
     #100;
    `uvm_info("comp","Reset Removed", UVM_NONE);
    phase.drop_objection(this);
  endtask
  
  task main_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("comp", "Random Stimulus Applied", UVM_NONE);
    #500;
    `uvm_info("comp", "Random Stimulus Removed", UVM_NONE);
    phase.drop_objection(this);
  endtask
  
endclass
 
///////////////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("comp");
  end
  
 
endmodule 
``` 
### Simulation Result 

### +UVM_OBJECTION_TRACE

![alt text](<Simulation Results/41.Objection Debug Switch Part 1.png>)
![alt text](<Simulation Results/41.Objection Debug Switch Part 2.png>)

</details>

__________________________________________________________