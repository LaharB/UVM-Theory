This repository consists of all the basic concepts and practices required to learn UVM coding 

- Section 1 : UVM Reporting Macros
- Section 2 : Base Classes Part 1 : UVM_OBJECT
- Section 3 : Base Classes Part 2 : UVM_COMPONENT
- Section 4 : config_db
- Section 5 : UVM_PHASES
- Section 6 : TLM
- Section 7 : Sequence

_________________________________________________________________

<details><summary><b>Section 1 : UVM Reporting Mechanisms</b></summary>
------------------------------------------------------------------
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

<details><summary><b>Section 2 : UVM Base Classes P1 : UVM_OBJECT</b></summary>
------------------------------------------------------------------
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

________________________________________________________________________

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

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/12.Using field macros P1 INT continued.png>)

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

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/13.Using field macros P2 ENUM, REAL.png>)

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

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/14.Using field macros P3 OBJECT.png>)

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

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/15.Using field macros P4 ARRAYS.png>)

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
  ///////////clone method -> create + clone  
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

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/16.Copy and Clone Method Part 1.png>)

Clone Method

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/16.Copy and Clone Method Part 2.png>)

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

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/17.Shallow copy vs Deep copy.png>)

</details>

__________________________________________________________________

<details>
 <summary><b>18.Copy and Clone Method - deep copy or shallow copy ?</b></summary><br>

### Code

### Copy Method

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

  second s1, s2; 
  
  initial begin
    s1 = new("s1");  //always provide instance name as path name 
    s2 = new("s2");  //no need of constructor for instance s for clone method
    s1.f.randomize();
    
  
   //copy method - we need a constructor for instace s2 
    s2.copy(s1);  //deep copy 
    
    s1.print();
    s2.print();
    
    s2.f.data = 10;
    s1.print();
    s2.print();
  
  end 
  
endmodule
``` 
### Clone Method

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

  second s1, s2;   
  
  initial begin
    s1 = new("s1");  //always provide instance name as path name 
    //s2 = new("s2");  //no need of constructor for instance s for clone method
    s1.f.randomize();
    
    //clone method - no need of constructor for instance s2
    $cast(s2, s1.clone());
    
    s1.print();
    s2.print();
    
    s2.f.data = 15;
    s1.print();
    s2.print();
    
  end 
  
endmodule 

```
### Simulation Result 

Copy Method 

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/18.Copy and Clone Method Part 1.png>)

Clone Method

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/18.Copy and Clone Method Part 2.png>)

We can observe that copy and clone method performs a deep copy for nested object class

</details>

__________________________________________________________________

<details>
 <summary><b>19.Compare Method</b></summary><br>

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

//////////////////////////////////////////////

module tb;
  
  first f1,f2;
  int status = 0;
  
  initial begin
    f1 = new("f1");  //always give instance name as path name 
    f2 = new("f2");
    
    f1.randomize();
    //f2.randomize();
    f2.copy(f1); //copying f1 content into f2
    f1.print();
    f2.print();
    status = f1.compare(f2);
    $display("Status : %0d", status);
    
  end
    
endmodule 
``` 
### Simulation Result 

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/19.Compare Method.png>)

The copy method uses automation and directly allows to copy the contents of one object into another. 

</details>

__________________________________________________________________

<details>
 <summary><b>20.Create Method</b></summary><br>

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

//////////////////////////////////////////////

module tb;
  
  first f1,f2;
  
  initial begin 
    //create(arguments) -> for uvm_object , only 1 arguement : path 
          //-> for uvm_component , 2 arguements : path and component name 
    //always use the instance name as path name 
    f1 = first::type_id::create("f1"); 
    f2 = first::type_id::create("f2");
    
    f1.randomize();
    f2.randomize();
    
    f1.print();
    f2.print();
    
  end 
    
endmodule 
``` 
### Simulation Result 

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/20.Create Method.png>)

</details>

_________________________________________________________________

<details>
 <summary><b>21.Factory Override - new vs create method</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
 
 
class first extends uvm_object; 
  
  rand bit [3:0] data;
  
  function new(string path = "first");
    super.new(path);
  endfunction 
  
  `uvm_object_utils_begin(first)
  `uvm_field_int(data, UVM_DEFAULT);
  `uvm_object_utils_end
  
endclass
/////////////////////////////////////
class first_mod extends first;
  rand bit ack;
  
  function new(string path = "first_mod");
    super.new(path);
  endfunction 
  
  `uvm_object_utils_begin(first_mod)
  `uvm_field_int(ack, UVM_DEFAULT);
  `uvm_object_utils_end
  
  
endclass
 
////////////////////////////////////////////
 
class comp extends uvm_component;
  `uvm_component_utils(comp)
  
  first f;
  
  function new(string path = "second", uvm_component parent = null);
    super.new(path, parent);
    f = new("f"); // new method - we wont be able to use override 
    //f = first::type_id::create("f");
    f.randomize();
    f.print();
  endfunction 
  
  
endclass
 
 
/////////////////////////////////////////////
 
module tb;
 
  comp c;
  
  initial begin
    //whereever we use the first object will be override by first_mod object
    c.set_type_override_by_type(first::get_type, first_mod::get_type); 
    c = new("c", null);
    //c = comp::type_id::create("comp", null); 
  end
  
endmodule
``` 
### Simulation Result 

### Using new() method
![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/21.Factory Override - new vs create method Part 1.png>)

### Using create() method without factory override
![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/21.Factory Override - new vs create method Part 2.png>)

### Using create() method + factory override
![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/21.Factory Override - new vs create method Part 3.png>)

</details>

_________________________________________________________________

<details>
 <summary><b>22.do_print method</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::* ;

class obj extends uvm_object;
  //need to register the class to factory while using DO METHOD
  `uvm_object_utils(obj)
  
  function new(string path = "obj");
    super.new(path);
  endfunction
  
  ///no need to register data members with factory while using DO METHOD
  //need to do it using CORE METHOD
  bit [3:0]a = 4;    
  string b = "UVM";
  real c = 9.11;
  
  virtual function void do_print(uvm_printer printer);
    super.do_print(printer);
    
    printer.print_field_int("a", a, $bits(a), UVM_HEX);
    printer.print_string("b", b);
    printer.print_real("c", c);
  endfunction
  
endclass

module tb;

  obj o;
  
  initial begin
    
    o = obj::type_id::create("o");
    o.print();
    
  end 
  
endmodule
``` 
### Simulation Result 

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/22.do_print method.png>)

</details>

_________________________________________________________________

<details>
 <summary><b>23.convert2string</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

class obj extends uvm_object;
  `uvm_object_utils(obj)
  
  function new(string path = "obj");
    super.new(path);
  endfunction
  
  bit [3:0] a = 4;
  string b = "UVM";
  real c = 12.34;
  
  virtual function string convert2string();
    
    string s = super.convert2string;
   
    s = {s, $sformatf("a : %0d ", a)};
    s = {s, $sformatf("b : %0s ", b)};
    s = {s, $sformatf("c : %0f ", c)};
    ////a : 4 b : UVM c : 12.3400
    return s;
  endfunction
  
endclass

module tb;
  
  obj o;
  
  initial begin
    o = obj::type_id::create("o");
    `uvm_info("TB_TOP", $sformatf("%0s", o.convert2string()), UVM_NONE);  
  end
  
endmodule
``` 
### Simulation Result 

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/23.convert2string method.png>)

</details>

_________________________________________________________________

<details>
 <summary><b>24.do_copy method</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
 
class obj extends uvm_object;
  `uvm_object_utils(obj)
  
  function new(string path = "obj");
    super.new(path);
  endfunction
  
  rand bit [3:0] a;
  rand bit [4:0] b;
   
  virtual function void do_print(uvm_printer printer);
    super.do_print(printer);
    
    printer.print_field_int("a :", a, $bits(a), UVM_DEC);
    printer.print_field_int("b :", b, $bits(b), UVM_DEC);
  endfunction
  
  virtual function void do_copy(uvm_object rhs);
    obj temp;
    $cast(temp, rhs);
    super.do_copy(rhs);
    
    this.a = temp.a;
    this.b = temp.b;
    
  endfunction
  
endclass

module tb;
  
  obj o1,o2;
  
  initial begin 
    o1 = obj::type_id::create("o1");
    o2 = obj::type_id::create("o2");
    
    o1.randomize();
    o1.print(); //same style for calling print in case core and do_print methods
    o2.copy(o1);
    o2.print();
  end
  
endmodule
``` 
### Simulation Result 

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/24.do_copy method.png>)

</details>

_________________________________________________________________

<details>
 <summary><b>25.do_compare method</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
 
class obj extends uvm_object;
  `uvm_object_utils(obj)
  
  function new(string path = "obj");
    super.new(path);
  endfunction
  
  rand bit [3:0] a;
  rand bit [4:0] b;
   
  virtual function void do_print(uvm_printer printer);
    super.do_print(printer);
    
    printer.print_field_int("a :", a, $bits(a), UVM_DEC);
    printer.print_field_int("b :", b, $bits(b), UVM_DEC);
  endfunction
  
  virtual function void do_copy(uvm_object rhs);
    obj temp;
    $cast(temp, rhs);
    super.do_copy(rhs);
    
    this.a = temp.a;
    this.b = temp.b; 
  endfunction
  
  virtual function bit do_compare(uvm_object rhs, uvm_comparer comparer);
    obj temp;
    int status;
    $cast(temp, rhs);
    status = super.do_compare(rhs, comparer) && (a == temp.a) && (b == temp.b);
    return status;
       
  endfunction
  
endclass

module tb;
  
  obj o1,o2;
  int status;
  
  initial begin
    o1 = obj::type_id::create("o1");
    o2 = obj::type_id::create("o2");
    
    o1.randomize();
    o1.print();
    o2.copy(o1); 
    status = o2.compare(o1);
    $display("Status : %0d", status);
    
  end
   
endmodule 
``` 
### Simulation Result 

![alt text](<Section_2_Base_classes_P1_UVM_OBJECT/Simulation Results/25.do_compare method.png>)

</details>

</details>

_________________________________________________________________

<details><summary><b>Section 3 : Base Classes Part 2 : UVM_COMPONENT</b></summary>
------------------------------------------------------------------
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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/26.Creating UVM Component part 1.png>)

If we use system verilog method , we get a warning 

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/26.Creating UVM Component part 2.png>)

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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/27.UVM_TREE.png>)

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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/28.Understanding typical format of config_db.png>)

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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/29.Use case of config_db.png>)

</details>

__________________________________________________________

<details>
 <summary><b>30.How to override phases</b></summary><br>

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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/30.How to override phases.png>)

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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/31.Execution of build_phase in multiple components.png>)

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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/32.Understanding execution of connect phase.png>)

</details>

__________________________________________________________

<details>
 <summary><b>33.Execution of multiple instance phases</b></summary><br>

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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/33.Execution of multiple instance phases P1.png>)

Using d for monitor and m for driver so monitor connect phase gets executed first then driver's.

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/33.Execution of multiple instance phases P2.png>)

</details>

__________________________________________________________

<details>
 <summary><b>34.Raising Objection</b></summary><br>

To hold the simulator till this task is completed, we use raise and drop objection.

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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/34.Raising Objection.png>)

</details>

__________________________________________________________

<details>
 <summary><b>35.How time consuming phases work in a single component ?</b></summary><br>

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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/35.How time consuming phases work in a single component.png>)

</details>

__________________________________________________________

<details>
 <summary><b>36.Time consuming phases in multiple components</b></summary><br>

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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/36.Time consuming phases in multiple components.png>)

</details>

__________________________________________________________

<details>
 <summary><b>37.Timeout</b></summary><br>

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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/37.Timeout.png>)

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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/38.Drain Time - Individual component.png>)

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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/39.Drain Time - Multiple component.png>)

</details>

__________________________________________________________

<details>
 <summary><b>40.Phase Debug Switch</b></summary><br>

We have to use the commands in run options to see the phase trace and objection trace report:
+UVM_PHASE_TRACE
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

### +UVM_PHASE_TRACE

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/40.Phase Debug Switch P1 - +UVM_PHASE_TRACE 1.png>)
![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/40.Phase Debug Switch P1 - +UVM_PHASE_TRACE 2.png>)
[text](README.md) 
![text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/40.Phase Debug Switch P1 - +UVM_PHASE_TRACE 3.png>) 
![text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/40.Phase Debug Switch P1 - +UVM_PHASE_TRACE 4.png>)

### +UVM_OBJECTION_TRACE

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/40.Phase Debug Switch P2 - UVM_OBJECTION_TRACE.png>)

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

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/41.Objection Debug Switch.png>)

</details>

__________________________________________________________

<details>
 <summary><b>42.Blocking PUT operation part 1</b></summary><br>

This code will give error because export is not the point where a transaction ends and there is no proper method inside port or export to send the data , it is done by "implementation".

### Code

```systemverilog 
//This code will give error because export is not the point where a transaction ends and there is no proper method inside port or export to send the data , it is done by "implementation" 

`include "uvm_macros.svh"
import uvm_pkg::*;

class producer extends uvm_component;
  `uvm_component_utils(producer)
  
  int data = 12;
  
  //declare the port 
  uvm_blocking_put_port #(int) send;
  
  //add the constructor for this class 
  function new(input string path = "producer", uvm_component parent = null);
    super.new(path, parent);
    //add the constructor for this port  
    send = new("send", this);
    
  endfunction
  
endclass
/////////////////////////////////////////////////////////////////
class consumer extends uvm_component;
  `uvm_component_utils(consumer)
  
  //declare the export 
  uvm_blocking_put_export #(int) recv;
  
  //add the constructor for this class 
  function new(input string path = "consumer", uvm_component parent = null);
    super.new(path, parent);
    
    //add the constructor for "recv" port 
    recv = new("recv", this);
    
  endfunction
    
endclass
//////////////////////////////////////////////////////////////////////
class env extends uvm_env;
  `uvm_component_utils(env)
  
  producer p;
  consumer c;
  
  function new(input string path = "env", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    p = producer::type_id::create("p", this);
    c = consumer::type_id::create("c", this);
  endfunction
  
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase); 
    p.send.connect(c.recv);   //access "send" using p handle and "recv" using c handle and connect them using .connect  
  endfunction
  
endclass
////////////////////////////////////////////////////////////////////////
class test extends uvm_test;
  `uvm_component_utils(test)
  
  env e;
  
  function new(input string path = "test", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    e = env::type_id::create("e", this);
  endfunction
  
endclass
////////////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("test");
  end
  
endmodule 
``` 
### Simulation Result 

![alt text](<Section_3_Base_classes_P2_UVM_COMPONENT/Simulation Results/42.Blocking PUT operation part 1.png>)

</details>

</details>

____________________________________________________________________

<details><summary><b>Section 4 : config_db</b></summary>
------------------------------------------------------------------
<details>
 <summary><b></b></summary><br>

 

### Code

```systemverilog 

``` 
### Simulation Result 



</details>

______________________________________________________________


</details>

_______________________________________________________________

<details><summary><b>Section 5 : UVM_PHASES</b></summary>
------------------------------------------------------------------
<details>
 <summary><b></b></summary><br>

 

### Code

```systemverilog 

``` 
### Simulation Result 



</details>

______________________________________________________________


</details>

_______________________________________________________________

<details><summary><b>Section 6 : TLM</b></summary>
------------------------------------------------------------------
<details>
 <summary><b>42.Blocking PUT operation</b></summary><br>

This code will give error because export is not the point where a transaction ends and there is no proper method inside port or export to send the data , it is done by "implementation".

### Code

```systemverilog 
//This code will give error because export is not the point where a transaction ends and there is no proper method inside port or export to send the data , it is done by "implementation" 

`include "uvm_macros.svh"
import uvm_pkg::*;

class producer extends uvm_component;
  `uvm_component_utils(producer)
  
  int data = 12;
  
  //declare the port 
  uvm_blocking_put_port #(int) send;
  
  //add the constructor for this class 
  function new(input string path = "producer", uvm_component parent = null);
    super.new(path, parent);
    //add the constructor for this port  
    send = new("send", this);
    
  endfunction
  
endclass
/////////////////////////////////////////////////////////////////
class consumer extends uvm_component;
  `uvm_component_utils(consumer)
  
  //declare the export 
  uvm_blocking_put_export #(int) recv;
  
  //add the constructor for this class 
  function new(input string path = "consumer", uvm_component parent = null);
    super.new(path, parent);
    
    //add the constructor for "recv" port 
    recv = new("recv", this);
    
  endfunction
    
endclass
//////////////////////////////////////////////////////////////////////
class env extends uvm_env;
  `uvm_component_utils(env)
  
  producer p;
  consumer c;
  
  function new(input string path = "env", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    p = producer::type_id::create("p", this);
    c = consumer::type_id::create("c", this);
  endfunction
  
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase); 
    p.send.connect(c.recv);   //access "send" using p handle and "recv" using c handle and connect them using .connect  
  endfunction
  
endclass
////////////////////////////////////////////////////////////////////////
class test extends uvm_test;
  `uvm_component_utils(test)
  
  env e;
  
  function new(input string path = "test", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    e = env::type_id::create("e", this);
  endfunction
  
endclass
////////////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("test");
  end
  
endmodule 
``` 
### Simulation Result 

![alt text](<Section_6_TLM/Simulation Results/42.Blocking PUT operation part 1.png>)

</details>

__________________________________________________________

<details>
 <summary><b>43.Adding IMP to Blocking PUT operation</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

class producer extends uvm_component;
  `uvm_component_utils(producer)
  
  int data = 12;
  
  uvm_blocking_put_port #(int) send;
  
  function new(input string path = "producer", uvm_component parent = null);
    super.new(path, parent);  
    
    send = new("send", this);
    
  endfunction
  
  //Step2 writing a method to send data to the consumer
  task main_phase(uvm_phase phase);
  //have to hold the simulator till data is sent so use raise objection
    phase.raise_objection(this);
     send.put(data);
    `uvm_info("PROD", $sformatf("Data Sent: %0d", data), UVM_NONE);
    phase.drop_objection(this);
  endtask
  
endclass
//////////////////////////////////////////////////////////////////////////////
class consumer extends uvm_component;
  `uvm_component_utils(consumer)
  
  uvm_blocking_put_export #(int) recv;
  uvm_blocking_put_imp #(int, consumer) imp; //2nd argument is the class where u receive data
    
  function new(input string path = "consumer", uvm_component parent = null);
    super.new(path, parent);  
    
    //make the constructors
    recv = new("recv", this);
    imp =  new("imp", this);
    
  endfunction
  //task to display the value received
  task put(int datar);
    `uvm_info("CONS", $sformatf("Data Rcvd : %0d", datar), UVM_NONE);
  endtask
  
endclass
//////////////////////////////////////////////////////////////////////////////
class env extends uvm_env;
  `uvm_component_utils(env)
 
   producer p;
   consumer c;
  
  function new(input string path = "env", uvm_component parent = null);
    super.new(path, parent);  
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    p = producer::type_id::create("p", this);
    c = consumer::type_id::create("c", this);
  endfunction
  
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    p.send.connect(c.recv);  //connection from port to export
    //Step1
    c.recv.connect(c.imp);   //from export ending at implementation
  endfunction
  
endclass
//////////////////////////////////////////////////////////////////////////////
class test extends uvm_test;
  `uvm_component_utils(test)
 
   env e;
  
  function new(input string path = "test", uvm_component parent = null);
    super.new(path, parent);  
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    e = env::type_id::create("e", this);   
  endfunction
  
endclass
//////////////////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("test");
  end 
  
endmodule 
``` 
### Simulation Result 

![alt text](<Section_6_TLM/Simulation Results/43.Adding IMP to Blocking PUT operation.png>)

</details>

________________________________________________________________

<details>
 <summary><b>44.Port to IMP</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
 
class producer extends uvm_component;
  `uvm_component_utils(producer)
  
  int data = 12;
  
  uvm_blocking_put_port #(int) send;
  
  function new(input string path = "producer", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    send  = new("send", this);
  endfunction
  //task for using a put method 
  task main_phase(uvm_phase phase);
    phase.raise_objection(this);
   `uvm_info("PROD", $sformatf("Data Sent : %0d", data), UVM_NONE); 
    send.put(data);
    phase.drop_objection(this);
 endtask

endclass
////////////////////////////////////////////////
 
class consumer extends uvm_component;
  `uvm_component_utils(consumer)
  
  uvm_blocking_put_imp #(int, consumer) imp; //2 arguments - data type of the data beinf sent and the class to which put method is being mentioned
  function new(input string path = "consumer", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    imp  = new("imp", this);
  endfunction
  
  //function to display data received 
  function void put(int datar);
    `uvm_info("Cons", $sformatf("Data Rcvd : %0d", datar), UVM_NONE);
  endfunction
  
endclass
///////////////////////////////////////////////////////////////////////
class env extends uvm_env;
 `uvm_component_utils(env)
 
 producer p;
 consumer c;
 
  function new(input string path = "env", uvm_component parent = null);
    super.new(path, parent);
  endfunction
 
  virtual function void build_phase(uvm_phase phase);
   super.build_phase(phase);
   p = producer::type_id::create("p",this);
   c = consumer::type_id::create("c", this);
  endfunction
 
  virtual function void connect_phase(uvm_phase phase);
   super.connect_phase(phase);
   p.send.connect(c.imp);
  endfunction
  
endclass
 
///////////////////////////////////////////////////
class test extends uvm_test;
`uvm_component_utils(test)
 
 env e;
 
  function new(input string path = "test", uvm_component parent = null);
   super.new(path, parent);
 endfunction
 
 virtual function void build_phase(uvm_phase phase);
  super.build_phase(phase);
  e = env::type_id::create("e",this);
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

![alt text](<Section_6_TLM/Simulation Results/44.Port to IMP.png>)

</details>

__________________________________________________________

<details>
 <summary><b>45.Port-Port to IMP</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;

class subproducer extends uvm_component;
  `uvm_component_utils(subproducer)
  
  int data = 12;
  
  uvm_blocking_put_port #(int) subport; 
  
  function new(input string path = "subproducer", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    subport  = new("subport", this);
  endfunction
  //task for using a put method 
  task main_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("SUBPROD", $sformatf("Data Sent : %0d", data), UVM_NONE); 
    subport.put(data);
    phase.drop_objection(this);
 endtask

endclass
////////////////////////////////////////////////
 
class producer extends uvm_component;
  `uvm_component_utils(producer)
  
  subproducer s;
  
  uvm_blocking_put_port #(int) port;
  
  function new(input string path = "producer", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    port  = new("send", this);
    s = subproducer::type_id::create("s", this);
  endfunction 
  
  //subproducer is inside porducer only , so we connect them in porducer class itself , not in env class
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    s.subport.connect(port);
  endfunction

endclass
////////////////////////////////////////////////
 
class consumer extends uvm_component;
  `uvm_component_utils(consumer)
  
  uvm_blocking_put_imp #(int, consumer) imp; //2 arguments - data type of the data being sent and the class to which put method is being mentioned
  function new(input string path = "consumer", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    imp  = new("imp", this);
  endfunction
  
  //function to display data received 
  function void put(int datar);
    `uvm_info("Cons", $sformatf("Data Rcvd : %0d", datar), UVM_NONE);
  endfunction
  
endclass
///////////////////////////////////////////////////////////////////////
class env extends uvm_env;
 `uvm_component_utils(env)
 
 producer p;
 consumer c;
 
  function new(input string path = "env", uvm_component parent = null);
    super.new(path, parent);
  endfunction
 
  virtual function void build_phase(uvm_phase phase);
   super.build_phase(phase);
   p = producer::type_id::create("p",this);
   c = consumer::type_id::create("c", this);
  endfunction
 
  virtual function void connect_phase(uvm_phase phase);
   super.connect_phase(phase);
   p.port.connect(c.imp);
  endfunction
  
endclass
 
///////////////////////////////////////////////////
class test extends uvm_test;
`uvm_component_utils(test)
 
 env e;
 
 function new(input string path = "test", uvm_component parent = null);
   super.new(path, parent);
 endfunction
 
 virtual function void build_phase(uvm_phase phase);
   super.build_phase(phase);
  e = env::type_id::create("e",this);
 endfunction
  
 //just to see the heirachy
  virtual function void end_of_elaboration_phase(uvm_phase phase);
    super.end_of_elaboration_phase(phase);
    uvm_top.print_topology();
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

![alt text](<Section_6_TLM/Simulation Results/45.Port-Port to IMP Part 1.png>)

![alt text](<Section_6_TLM/Simulation Results/45.Port-Port to IMP Part 2.png>)

</details>

__________________________________________________________

<details>
 <summary><b>46.Port to Export-IMP</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
//////////////////////////////////////////////////////////
class producer extends uvm_component;
  `uvm_component_utils(producer)
  
  int data = 12;
 
  uvm_blocking_put_port #(int) port;
  
  function new(input string path = "producer", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    port = new("port", this);
  endfunction
  
  task main_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("PROD", $sformatf("Data Sent : %0d", data), UVM_NONE);
    port.put(data);  //using put method to send data 
    phase.drop_objection(this);  
  endtask
   
endclass
///////////////////////////////////////////////////////
class subconsumer extends uvm_component;
  `uvm_component_utils(subconsumer)
 
  uvm_blocking_put_imp #(int, subconsumer) imp;
  
  function new(input string path = "subconsumer", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    imp = new("imp", this);
  endfunction
  
  function void put(int datar);
    `uvm_info("SUBCONS", $sformatf("Data Rcvd : %0d", datar), UVM_NONE);
  endfunction
   
endclass
///////////////////////////////////////////////////////
class consumer extends uvm_component;
  `uvm_component_utils(consumer)
 
  uvm_blocking_put_export #(int) expo;
  subconsumer s;
  
  function new(input string path = "consumer", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    expo = new("expo", this);
    s = subconsumer::type_id::create("s", this);
  endfunction
  
  //connect subconsumer port with parent class consumer
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    expo.connect(s.imp);
  endfunction
   
endclass
///////////////////////////////////////////////////////
class env extends uvm_env;
  `uvm_component_utils(env)
 
  producer p;
  consumer c;
  
  function new(input string path = "env", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    p = producer::type_id::create("p", this);
    c = consumer::type_id::create("c", this);
  endfunction
  
  //connect subconsumer port with parent class consumer
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    //connect the port in producer to the export in consumer in the env class
    p.port.connect(c.expo);
  endfunction
   
endclass
///////////////////////////////////////////////////////
class test extends uvm_test;
  `uvm_component_utils(test)
  
  env e;
 
  function new(input string path = "test", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    e = env::type_id::create("e", this);
  endfunction
  
  //just to see the heirarchy
  virtual function void end_of_elaboration_phase(uvm_phase phase);
    super.end_of_elaboration_phase(phase);
    uvm_top.print_topology();
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

![alt text](<Section_6_TLM/Simulation Results/46.Port to Export-IMP Part 1.png>)

![alt text](<Section_6_TLM/Simulation Results/46.Port to Export-IMP Part 2.png>)

</details>


</details>

_______________________________________________________________

<details><summary><b>Section 7 : Sequence</b></summary>
------------------------------------------------------------------
<details>
 <summary><b></b></summary><br>

 

### Code

```systemverilog 

``` 
### Simulation Result 



</details>

______________________________________________________________


</details>

_______________________________________________________________