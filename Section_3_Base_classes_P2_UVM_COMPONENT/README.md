# Section 3 : UVM Base Classes P2 : UVM_COMPONENT

<details>
 <summary><b>26.Creating UVM Component</b></summary><br>

This is a basic example of how to create an uvm_component class

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

class comp extends uvm_component;
  `uvm_component_utils(comp)
  
  function new(string path = "comp", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  //build_phase function
  virtual function void build_phase(uvm_phase phase);   //this line 
    super.build_phase(phase);  //and this line is standard for writing build_phase 
    `uvm_info("COMP", "Build Phase of comp executed", UVM_NONE);
  endfunction

endclass

module tb;
 
  /*
  //System verilog style
  comp c;
  
  initial begin
    c = comp::type_id::create("c", null);  //always use instance name as path name 
    //here parent arguement is null because comp is child of uvm_top 
    c.build_phase(null);  //here null is used only for demonstration purpose  
    
  end
  */

  //UVM style
  initial begin 
    run_test("comp");   //give name of the class (here "comp") which we want to run
  end
  
endmodule

``` 
### Simulation Result 

![alt text](<Simulation Results/26.Creating UVM Component part 1.png>)

If we use system verilog method , we get a warning 

![alt text](<Simulation Results/26.Creating UVM Component part 2.png>)

</details>

__________________________________________________________

<details>
 <summary><b>27.UVM_TREE</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

//creating the child classes first 

class a extends uvm_component;
  `uvm_component_utils(a)
  
  function new(string path = "a", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    `uvm_info("a", "Class a executed", UVM_NONE);
  endfunction
  
endclass

class b extends uvm_component;
  `uvm_component_utils(b)
  
  function new(string path = "b", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    `uvm_info("b", "Class b executed", UVM_NONE);
  endfunction
  
endclass

//creating the parent class c 

class c extends uvm_component;
  `uvm_component_utils(c)
  
  //instances of child a and b
  a a_inst; 
  b b_inst;
  
  function new(string path = "c", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    //create method for creating objects for the instnaces
    a_inst = a::type_id::create("a_inst", this);  // using "this" as parent arguement because classes a and b are children of class c 
    b_inst = b::type_id::create("b_inst", this);    
  endfunction
  
  //to see heirarchy
  virtual function void end_of_elaboration_phase(uvm_phase phase);
    super.end_of_elaboration_phase(phase);
    uvm_top.print_topology();
  endfunction
  
endclass

module tb;
  
  initial begin
    run_test("c");
  end 
  
endmodule 

``` 
### Simulation Result 

![alt text](<Simulation Results/27.UVM_TREE.png>)

</details>

__________________________________________________________

<details>
 <summary><b>28.Understanding typical format of config_db</b></summary><br>

### Code

```systemverilog 
class env extends uvm_env;
  `uvm_component_utils(env)  //using uvm_component_utils as env is static compoenent build from uvm_component
  int data;
  
  function new(string path = "env", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    
    //step 2
    if(uvm_config_db#(int)::get(null, "uvm_test_top", "data", data))
      `uvm_info("ENV", $sformatf("Value of data : %0d", data), UVM_NONE)
     else 
       `uvm_error("ENV", "Unable to access the value");   
  endfunction
  
endclass


class test extends uvm_test;
  `uvm_component_utils(test)  //using uvm_component_utils as env is static compoenent build from uvm_component
  
  env e;
  
  function new(string path = "test", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    e = env::type_id::create("e", this);  //this as test is parent inside which instance of env class is build
    
    //step 1
    //using config_db to get access of resources inside env class from test class
    uvm_config_db#(int)::set(null, "uvm_test_top", "data", 15); //4 arguements(context + instance name + key + value) 
   //using null so that every class can get access to env class  
    // use the data member name as key  
      
  endfunction
  
endclass
    
module tb;
  
  initial begin
    run_test("test");
  end
endmodule 

``` 
### Simulation Result 

![alt text](<Simulation Results/28.Understanding typical format of config_db.png>)

</details>

__________________________________________________________

<details>
 <summary><b>29.Use case of config_db</b></summary><br>

### Code

### Design code

```systemverilog
module adder(
  input [3:0] a, b,
  output [4:0] y
);
  
  assign y = a + b;
  
endmodule 

interface adder_if;
  logic [3:0]a;
  logic [3:0]b;
  logic [4:0]y;
  
endinterface 
```

### Testbench code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

class drv extends uvm_driver;
  `uvm_component_utils(drv)
  
  virtual adder_if aif;  //interface instance , using virtual as interface is static component 
  
  function new(input string path = "drv", uvm_component parent = null);
    super.new(path, parent);  
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    //step2
    //use of config_db to give access of driver resources 
    if(!uvm_config_db#(virtual adder_if)::get(this, "", "aif", aif)) //this keyword - gives the full path - uvm_test_top.env.agent.drv so have to use the same path name as arguement in set method in module tb
      `uvm_error("drv", "Unable to access interface");
  endfunction
      
  //step3 - run_phase -task is used for run_phase
  virtual task run_phase(uvm_phase phase);
    phase.raise_objection(this);
    for(int i = 0; i < 10; i++)
      begin
        aif.a <= $urandom;
        aif.b <= $urandom;
        #10;
      end
    phase.drop_objection(this);
  endtask 
  
endclass
///////////////////////////////////////////////////////////////////////////
class agent extends uvm_agent;
  `uvm_component_utils(agent)
  
  function new(input string inst = "agent", uvm_component c);
    super.new(inst, c);
  endfunction
  
  drv d;
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    d = drv::type_id::create("drv", this);    
  endfunction
  
endclass
//////////////////////////////////////////////////////////////////////////////
class env extends uvm_env;
  `uvm_component_utils(env)
  
  function new(input string inst = "env", uvm_component c);
    super.new(inst, c); 
  endfunction
  
  agent a;
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    a = agent::type_id::create("agent", this);  //use class name as path when outside testbench
  endfunction

endclass
//////////////////////////////////////////////////////////////////////////////
class test extends uvm_test;
  `uvm_component_utils(test)
  
  function new(input string inst = "test", uvm_component c);
    super.new(inst, c); 
  endfunction
  
  env e;
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    e = env::type_id::create("env", this); ////use class name as path when outside testbench
  endfunction

endclass
/////////////////////////////////////////////////////////////////////////////
module tb;
  
  adder_if aif();
  
  adder dut(.a(aif.a), .b(aif.b), .y(aif.y));
  
  initial begin
    //step 1 
    //4 arguemnts
    uvm_config_db#(virtual adder_if)::set(null, "uvm_test_top.env.agent.drv", "aif", aif);
    run_test("test");
  end
  
  initial begin
    $dumpfile("dump.vcd");
    $dumpvars;
  end
  
endmodule
``` 
### Simulation Result 

![alt text](<Simulation Results/29.Use case of config_db.png>)

</details>

__________________________________________________________

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

![alt text](<Simulation Results/31.Execution of build_phase in multiple components.png>)

</details>

__________________________________________________________
