# Section 6 : TLM

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

![alt text](<../Section_6_TLM/Simulation Results/42.Blocking PUT operation part 1.png>)

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

![alt text](<../Section_6_TLM/Simulation Results/43.Adding IMP to Blocking PUT operation.png>)

</details>

__________________________________________________________
