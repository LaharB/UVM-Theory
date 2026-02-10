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

![alt text](<Simulation Results/44.Port to IMP.png>)

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

![alt text](<Simulation Results/45.Port-Port to IMP Part 1.png>)

![alt text](<Simulation Results/45.Port-Port to IMP Part 2.png>)

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

![alt text](<Simulation Results/46.Port to Export-IMP Part 1.png>)

![alt text](<Simulation Results/46.Port to Export-IMP Part 2.png>)

</details>

__________________________________________________________

<details>
 <summary><b>47.Get Operation</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
 
class producer extends uvm_component;
  `uvm_component_utils(producer)
  
  uvm_blocking_get_port #(int) port;
  
  int data = 0; //variable to store the data that we receive from consumer
  
  function new(input string path = "producer", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    port   = new("port", this);
  endfunction
  
  //task to get the data from consumer
  task main_phase(uvm_phase phase);
    phase.raise_objection(this);
    port.get(data);  //get method 
    `uvm_info("PROD", $sformatf("Data Rcvd : %0d",data), UVM_NONE);
    phase.drop_objection(this);
  endtask

endclass
/////////////////////////////////////////////////////////////////////
class consumer extends uvm_component;
  `uvm_component_utils(consumer)
  
  int data = 12;
  
  uvm_blocking_get_imp#(int, consumer) imp;
  
  function new(input string path = "consumer", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    imp  = new("imp", this);
  endfunction
  
  virtual task get(output int datar ); //output as consumer is sending data to producer 
    `uvm_info("CONS", $sformatf("Data Sent : %0d", data), UVM_NONE);
    datar = data; //passing the data = 12 to datar which will be sent to producer
  endtask

endclass
///////////////////////////////////////////////////////////////////////////
 
class env extends uvm_env;
`uvm_component_utils(env)
 
 producer p;
 consumer c;
 
 
 function new(input string path = "env", uvm_component parent = null);
   super.new(path, parent);
 endfunction
 
 virtual function void build_phase(uvm_phase phase);
  super.build_phase(phase);
  c = consumer::type_id::create("c", this);
  p = producer::type_id::create("p",this);
 endfunction
 
  //conencting producer and consumer in env class as env is the parent 
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
 
endclass
///////////////////////////////////////////////////////////
module tb;
  
 initial begin
  run_test("test");
 end
 
endmodule 
``` 
### Simulation Result 

![alt text](<Simulation Results/47.Get Operation.png>)

</details>

__________________________________________________________

<details>
 <summary><b>48.Transport Port</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
////////////////////////////////////////////////////////////////////
class producer extends uvm_component;
 `uvm_component_utils(producer)

//Two arguments for transport port -
//1.Datatype to be sent 2.Datatype to be received
  uvm_blocking_transport_port #(int, int) port;

  int datas = 12;  //data to be sent 
  int datar = 0;   //variable to store the data received
  
  function new(input string path = "producer", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    port = new("port", this);
  endfunction
  
  //main_phase
  task main_phase(uvm_phase phase);
    phase.raise_objection(this);
    port.transport(datas, datar); //calling transport task
    `uvm_info("PROD", $sformatf("Data Sent : %0d, Data Rcvd : %0d", datas, datar), UVM_NONE);
    phase.drop_objection(this);
  endtask
   
endclass
////////////////////////////////////////////////////////////////////////////
class consumer extends uvm_component;
  `uvm_component_utils(consumer)

//Two arguments for transport port + 1 from implementation port
  uvm_blocking_transport_imp #(int, int, consumer) imp;

  int datas = 13;  //data to be sent 
  int datar = 0;   //variable to store the data received
  
  function new(input string path = "consumer", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    imp = new("imp", this);
  endfunction
  
  virtual task transport(input int datar, output int datas);
   datas = this.datas;
    `uvm_info("CONS", $sformatf("Data Sent : %0d, Data Rcvd : %0d", datas, datar), UVM_NONE);
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
    p.port.connect(c.imp);
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

![alt text](<Simulation Results/48.Transport Port.png>)

</details>

__________________________________________________________

<details>
 <summary><b>49.Analysis Port</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
 
class producer extends uvm_component;
  `uvm_component_utils(producer)
  
  uvm_analysis_port #(int) port;
  
  int data = 12;
  
  function new(input string path = "producer", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    port   = new("port", this);
  endfunction
  
  task main_phase(uvm_phase phase);
  phase.raise_objection(this);
  `uvm_info("PROD", $sformatf("Data Broadcasted : %0d", data), UVM_NONE);
  port.write(data);
  phase.drop_objection(this);
 endtask
    
endclass
////////////////////////////////////////////////
 
class consumer1 extends uvm_component;
  `uvm_component_utils(consumer1)
  
  
  uvm_analysis_imp#(int, consumer1) imp;
  
  function new(input string path = "consumer1", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    imp  = new("imp", this);
  endfunction
  
  virtual function void write(int datar);
    `uvm_info("CONS1", $sformatf("Data Recv : %0d", datar), UVM_NONE); 
  endfunction
  
endclass
 
/////////////////////////////////////////////////////////////////
class consumer2 extends uvm_component;
  `uvm_component_utils(consumer2)
  
  
  uvm_analysis_imp#(int, consumer2) imp;
  
  function new(input string path = "consumer2", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    imp  = new("imp", this);
  endfunction
  
  virtual function void write(int datar);
    `uvm_info("CONS2", $sformatf("Data Recv : %0d", datar), UVM_NONE); 
  endfunction
   
endclass  
   
/////////////////////////////////////////////////////////////////
 
class env extends uvm_env;
`uvm_component_utils(env)
 
producer p;
consumer1 c1;
consumer2 c2;  
 
 
function new(input string path = "env", uvm_component parent = null);
    super.new(path, parent);
endfunction
 
virtual function void build_phase(uvm_phase phase);
super.build_phase(phase);
  c1 = consumer1::type_id::create("c1", this);
  c2 = consumer2::type_id::create("c2", this);
  p = producer::type_id::create("p",this);
endfunction
 
virtual function void connect_phase(uvm_phase phase);
super.connect_phase(phase);
  p.port.connect(c1.imp);
  p.port.connect(c2.imp);
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
 
//////////////////////////////////////////////
module tb;
 
initial begin
  run_test("test");
end
  
endmodule 
``` 
### Simulation Result 

![alt text](<Simulation Results/49.Analysis Port.png>)

</details>

__________________________________________________________

