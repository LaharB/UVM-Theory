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
 <summary><b>38.Drain Time: Individual component</b></summary><br>

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
    phase.phase_done.set_drain_time(this, 200);
    phase.raise_objection(this);
    `uvm_info("comp", "Main Phase Started", UVM_NONE);
    #100;
    `uvm_info("comp", "Main Phase Completed", UVM_NONE);
	phase.drop_objection(this);   
  endtask
  
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
    uvm_phase main_phase;   //variable "main_phase" of type "uvm_phase"
    super.end_of_elaboration_phase(phase);
    main_phase = phase.find_by_name("main",0); //to get access to main_phases of all the components
    main_phase.phase_done.set_drain_time(this, 100);
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

We have to use the command in run options to see the phase trace report:
+UVM_PHASE_TRACE

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

### +UVM_OBJECTION_TRACE

![alt text](<Simulation Results/40.Phase Debug Switch P2 - UVM_OBJECTION_TRACE.png>)

</details>

__________________________________________________________

<details>
 <summary><b>41.Objection Debug Switch</b></summary><br>

We have to use the command in run options to see the objection trace report:
+UVM_OBJECTION_TRACE

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

![alt text](<Simulation Results/41.Objection Debug Switch.png>)

</details>

__________________________________________________________
