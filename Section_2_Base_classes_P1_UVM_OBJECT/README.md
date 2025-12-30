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

<details>
 <summary><b>15.Using field macros P4 ARRAYS</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

class array extends uvm_object;
  
  ///static array 
  int arr1[3] = {1,2,3};
  //dynamic array
  int arr2[];
  //queue
  int arr3[$];
  //associative array
  int arr4[int];
  
  function new(string path ="array");
    super.new(path);   
  endfunction
  
  `uvm_object_utils_begin(array)
  `uvm_field_sarray_int(arr1, UVM_DEFAULT);
  `uvm_field_array_int(arr2, UVM_DEFAULT);
  `uvm_field_queue_int(arr3, UVM_DEFAULT);
  `uvm_field_aa_int_int(arr4, UVM_DEFAULT);
  `uvm_object_utils_end
  
  task run();
    ////dynamic array size and value update
    arr2 = new[3];
    arr2[0] = 2;
    arr2[1] = 2;
    arr2[2] = 2; 
    ////queue 
    arr3.push_front(3);
    arr3.push_front(3);
    //////associative array
    arr4[1] = 4;
    arr4[2] = 4;
    arr4[3] = 4;
    arr4[4] = 4; 
    
  endtask
  
endclass
/////////////////////////////////////
module tb;
  array a;
  
  initial begin 
    a = new("array");
    a.run();
    a.print();
  end 
  
endmodule 
``` 
### Simulation Result 

![alt text](<Simulation Results/15.Using field macros P4 ARRAYS.png>)

</details>

__________________________________________________________________

<details>
 <summary><b>16.Copy and Clone Method</b></summary><br>

### Code

### Copy Method

```systemverilog 

 `include "uvm_macros.svh"
import uvm_pkg::*;

class first extends uvm_object;
  
  rand bit [3:0]data;
  
  function new(string path = "first");
    super.new(path);    
  endfunction
  
  `uvm_object_utils_begin(first)
  `uvm_field_int(data, UVM_DEFAULT);
  `uvm_object_utils_end
 
endclass
//////////////////////////////////////////////////////////////

module tb;
  /////copy method -> new + copy
  first f;
  first s;
  
  
  initial begin
    f = new("first");   
    s = new("second");    //need to add contructor for the instance s to copy into s
    f.randomize();
    s.copy(f);
    f.print();
    s.print();
  end 

endmodule
``` 
### Clone Method

```systemverilog
 `include "uvm_macros.svh"
import uvm_pkg::*;

class first extends uvm_object;
  
  rand bit [3:0]data;
  
  function new(string path = "first");
    super.new(path);    
  endfunction
  
  `uvm_object_utils_begin(first)
  `uvm_field_int(data, UVM_DEFAULT);
  `uvm_object_utils_end
 
endclass
//////////////////////////////////////////////////////////////

module tb;

  first f;
  first s;
  
  ///////////clone method -> clone + create 
  initial begin 
    f = new("first");   //no constructor needed for instance s
    f.randomize();
    //s = f.clone(); //f.clone() returns handle of uvm_object type but handle s is of "first" type so have to use $cast
    $cast(s, f.clone());
    f.print;
    s.print;   
  end
  
endmodule
```
### Simulation Result 

Copy Method 

![alt text](<Simulation Results/16.Copy and Clone Method Part 1.png>)

Clone Method

![alt text](<Simulation Results/16.Copy and Clone Method Part 2.png>)

</details>

__________________________________________________________________

<details>
 <summary><b>17.Shallow copy vs Deep copy</b></summary><br>

In shallow copy, we can observe that if we make any change in the data members in the copy of the nested class , the data members of the original nested class also get changed.This shows shallow copy creates a common handle for the nested class.

### Code

```systemverilog 
`include "uvm_macros.svh";
import uvm_pkg::*;

class first extends uvm_object;
  
  rand bit [3:0]data;
  
  function new(string path = "first");
    super.new(path);  
  endfunction
  
  `uvm_object_utils_begin(first)
  `uvm_field_int(data, UVM_DEFAULT);
  `uvm_object_utils_end
  
endclass

///////////////////////////////////////////////

class second extends uvm_object;
  
  first f;
  
  function new(string path = "second");
    super.new(path);
    f = new("first");
  endfunction
  
  `uvm_object_utils_begin(second)
  `uvm_field_object(f, UVM_DEFAULT);
  `uvm_object_utils_end
  
endclass

//////////////////////////////////////////////

module tb;

  second s1, s2;   //shallow copy
  
  initial begin
    s1 = new("s1");  //always provide instance name as path name 
    s2 = new("s2");  //always provide instance name as path name 
    s1.f.randomize();
    s1.print();
    s2 = s1;
    s2.print();
    
    s2.f.data = 14;   //this f is not an independent handle 
    s1.print();
    s2.print();
    
  end 
  
endmodule
 
```
### Simulation Result 

![alt text](<Simulation Results/17.Shallow copy vs Deep copy.png>)

</details>

__________________________________________________________________

