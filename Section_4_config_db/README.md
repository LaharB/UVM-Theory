# Section 4 : config_db

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

![alt text](<Simulation results/28.Understanding typical format of config_db.png>)

</details>

__________________________________________________________

<details>
 <summary><b>29.Use case of config_db</b></summary>

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

![alt text](<Section_4_config_db/Simulation results/29.Use case of config_db.png>)

</details>

___________________________________________________________