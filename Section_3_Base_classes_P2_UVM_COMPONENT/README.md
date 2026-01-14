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

![alt text](<26.Creating UVM Component.png>)

</details>

__________________________________________________________
