# Section 2 : UVM Base Classes P1 : UVM_OBJECT

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

![alt text](<Simulation Results/11.Using field macros P1 INT.png>)

</details>

__________________________________________________________

<details>
 <summary><b>12.Using field macros P1 INT continued</b></summary><br>

UVM_NOPRINT usage demonstratation.We can observe that the value of b is not printed in the console.

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
  rand bit [3:0] b;
  
  `uvm_object_utils_begin(obj)
  `uvm_field_int(a, UVM_DEFAULT | UVM_DEC);
  `uvm_field_int(b, UVM_NOPRINT | UVM_BIN);
  `uvm_object_utils_end
  
endclass

module tb;
  
  obj o;
  
  initial begin
    o = new("obj");
    o.randomize();
    o.print(uvm_default_table_printer);
  end
endmodule
``` 
### Simulation Result 

![alt text](<Simulation Results/12.Using field macros P1 INT continued.png>)

</details>

__________________________________________________________________

<details>
 <summary><b>13.Using field macros P2 ENUM, REAL</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh";
import uvm_pkg::*;

class obj extends uvm_object;
  //`uvm_object_utils(obj)
  
  function new(string path = "obj");
    super.new(path);
  endfunction
  
  typedef enum bit [1:0]{s0, s1, s2, s3}state_type;
  rand state_type state;
  
  real temp = 12.34;
  string str = "UVM Good";
  
  `uvm_object_utils_begin(obj)
   `uvm_field_enum(state_type, state, UVM_DEFAULT); //3 arguments for uvm_field_enum
   `uvm_field_real(temp, UVM_DEFAULT);
   `uvm_field_string(str, UVM_DEFAULT);
  `uvm_object_utils_end
  
endclass

module tb;
  
  obj o;
  
  initial begin
    o = new("obj");
    o.randomize();
    o.print(uvm_default_table_printer);
  end
endmodule 
``` 
### Simulation Result 

![alt text](<Simulation Results/13.Using field macros P2 ENUM, REAL.png>)

</details>

__________________________________________________________________

<details>
 <summary><b>14.Using field macros P3 OBJECT</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

class parent extends uvm_object;
  
  function new(string path = "parent");
    super.new(path);
  endfunction
  
  rand bit [3:0] data;
  
  `uvm_object_utils_begin(parent)
  `uvm_field_int(data, UVM_DEFAULT);
  `uvm_object_utils_end
  
endclass

class child extends uvm_object;
  
  parent p;
  
  function new(string path = "child");
    super.new(path);
    p = new("parent");
  endfunction
  
  `uvm_object_utils_begin(child)
  `uvm_field_object(p, UVM_DEFAULT);
  `uvm_object_utils_end
  
endclass

module tb;
  child c;
  
  initial begin
    c = new("child");
    c.p.randomize();
    c.print();
  end

endmodule
``` 
### Simulation Result 

![alt text](<Simulation Results/14.Using field macros P3 OBJECT.png>)

</details>

__________________________________________________________________

