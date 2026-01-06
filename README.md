This repository consists of all the basic concepts and practices required to learn UVM coding 

- Section 1 : UVM Reporting Macros
- Section 2 : Base Classes Part 1 : UVM_OBJECT
- Section 3 : Base Classes Part 2 : UVM_COMPONENT
- Section 4 : config_db
- Section 5 : UVM_PHASES
- Section 6 : TLM
- Section 7 : Sequence

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




</details>

