# Section 7 : Sequence

<details>
 <summary><b>50.Creating Sequences</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
/////////////////////////////////////////////////////////////////////////////

//1.transaction class
class transaction extends uvm_sequence_item;
  
  rand bit [3:0] a;
  rand bit [3:0] b;
   	   bit [4:0]y;
  
  function new(input string path = "transaction"); //only one argument as transaction belongs to uvm_object
    super.new(path);
  endfunction
  
  //register the data members to factory for core methods
  `uvm_object_utils_begin(transaction)
  	`uvm_field_int(a, UVM_DEFAULT);
  	`uvm_field_int(b, UVM_DEFAULT);
  	`uvm_field_int(y, UVM_DEFAULT);
  `uvm_object_utils_end
  
endclass
/////////////////////////////////////////////////////////////////////////////
//2.Sequence
class sequence1 extends uvm_sequence#(transaction); 
  `uvm_object_utils(sequence1);
  
  function new(input string path = "sequence1");  //1 arguement as sequence belongs to uvm_object
    super.new(path);
  endfunction
  
  virtual task pre_body();
    `uvm_info("SEQ1", "PRE-BODY EXECUTED", UVM_NONE);
  endtask
  
  virtual task body();
    `uvm_info("SEQ1", "BODY EXECUTED", UVM_NONE);
  endtask
  
  virtual task post_body();
    `uvm_info("SEQ1", "POST-BODY EXECUTED", UVM_NONE);
  endtask 
  
endclass
//////////////////////////////////////////////////////////////////////////////
//3.driver , we dont need to make a separate class for sequencer 
//we call make sequencer instances directly inside env class
class driver extends uvm_driver#(transaction);   //driver belong to uvm_component 
  `uvm_component_utils(driver)
  
  transaction t;  //transaction instance inside driver to communicate with it and tell when to send the next packet
  
  function new(input string path = "driver", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    t = transaction::type_id::create("t", this);
  endfunction
  
  //run_phase to communicate with the transaction class
  virtual task run_phase(uvm_phase phase);
    forever begin
    //seq_item_port is the port inside driver
      seq_item_port.get_next_item(t);  //using get method to get next packet
      /////apply seq to DUT , here we are not doing so
      seq_item_port.item_done();  //item_done tells thats packet has been sent to DUT
    end
  endtask
  
endclass
/////////////////////////////////////////////////////////////////////////////////////
//4.agent - inside agent we have sequencer, driver and monitor , here mon is omitted
class agent extends uvm_agent;
  `uvm_component_utils(agent)
  //making uvm_sequencer instance directly , we dont need a separate class for it 
  uvm_sequencer #(transaction) seqr;
  driver d;
  
  function new(input string path = "agent", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    seqr = uvm_sequencer #(transaction)::type_id::create("seqr", this); //2 arg as sequencer belongs to uvm_object
    d = driver::type_id::create("d", this);
  endfunction
  
  //connecting driver and sequencer ports 
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    d.seq_item_port.connect(seqr.seq_item_export);
  endfunction

endclass
/////////////////////////////////////////////////////////////////////////////////////
//5.environment class
class env extends uvm_env;
  `uvm_component_utils(env);
  
  agent a;
  
  function new(input string path = "env", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    a = agent::type_id::create("a", this);
  endfunction

endclass
/////////////////////////////////////////////////////////////////////////////////////
//6.test
class test extends uvm_test;
  `uvm_component_utils(test)
  
  function new(input string path, uvm_component parent = null);
    super.new(path, parent);
  endfunction
  //sequence and env instances
  sequence1 seq1;
  env e;
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    seq1 = sequence1::type_id::create("seq1"); //1 arg as uvm_sequence belongs to uvm_object
    e = env::type_id::create("e", this);  //2 arg
  endfunction
  
  //run_phase to start the sequence and connect with sequencer
  virtual task run_phase(uvm_phase phase);
    phase.raise_objection(this);
    seq1.start(e.a.seqr);
    phase.drop_objection(this);
  endtask
  
endclass
/////////////////////////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("test");
  end
  
endmodule 
``` 
### Simulation Result 

![alt text](<Simulation Results/50.Creating Sequences.png>)

</details>

__________________________________________________________

<details>
 <summary><b>51.Undestanding Flow</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
/////////////////////////////////////////////////////////////////////////
class transaction extends uvm_sequence_item;
  
  rand bit [3:0]a;
  rand bit [3:0]b;
  	   bit [4:0]y;
  
  function new(input string path = "transaction");
    super.new(path);
  endfunction
  
  //register the members with field macros
  `uvm_object_utils_begin(transaction);
  `uvm_field_int(a, UVM_DEFAULT);
  `uvm_field_int(b, UVM_DEFAULT);
  `uvm_field_int(y, UVM_DEFAULT);
  `uvm_object_utils_end

endclass
///////////////////////////////////////////////////////////////////////////
class sequence1 extends uvm_sequence#(transaction);
  `uvm_object_utils(sequence1)

  transaction trans;
  
  function new(input string path = "sequence1");
    super.new(path);
  endfunction
  
  virtual task body();
    `uvm_info("SEQ1", "Trans obj created", UVM_NONE);
    //creation of trans object
    trans = transaction::type_id::create("trans");  //instance name as path name
    `uvm_info("SEQ1", "Waiting for Grant from Driver", UVM_NONE);
    wait_for_grant();
    `uvm_info("SEQ1", "Rcvd Grant...Randomizing Data", UVM_NONE);
    assert(trans.randomize());
    `uvm_info("SEQ1", "Randomization Done -> Sent Req to Drv", UVM_NONE);
    send_request(trans);
    `uvm_info("SEQ1", "waiting for Item Done Resp from Driver", UVM_NONE);
    wait_for_item_done();
    `uvm_info("SEQ1", "SEQ1 Ended", UVM_NONE);
  endtask
  
endclass
//////////////////////////////////////////////////////////////////////////////
class driver extends uvm_driver#(transaction);
  `uvm_component_utils(driver);
  
  transaction t; //using trans as a container 
  
  function new(input string path = "driver", uvm_component parent = null);
    super.new(path, parent);
  endfunction
   
  virtual function void build_phase(uvm_phase phase); 
    super.build_phase(phase);
    t = transaction::type_id::create("t");
  endfunction  
  
  //task to connect driver with transaction and communicate 
  virtual task run_phase(uvm_phase phase);
    forever begin   //using forever block as driver has to be always ready for transaction sent from seq and respond 
      `uvm_info("Drv", "Sending Grant for Sequence", UVM_NONE);
      seq_item_port.get_next_item(t); //sending grant to sequence and getting the values inside trans container 
      `uvm_info("DRV", "Applying Seq to DUT", UVM_NONE);
      /////////////////////////
      //apply seq to DUT
      /////////////////////////
      `uvm_info("DRV", "Sending Item Done Resp for Sequence", UVM_NONE);
      seq_item_port.item_done();   //item_done resp to sequence so seq sends the next packet
    end
  endtask

endclass
///////////////////////////////////////////////////////////
class agent extends uvm_agent;
  `uvm_component_utils(agent);
  
  function new(input string path = "agent", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  
  //driver and sequencer are children to agent class
  driver d;
  uvm_sequencer#(transaction) seqr;
  
  //build objects for the driver and seqr instances 
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    d = driver::type_id::create("d", this);
    seqr = uvm_sequencer #(transaction)::type_id::create("seqr", this);//uvm_sequencer belongs to uvm_component , dynamic 
  endfunction
  //connect phase to connect sequencer and driver
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    d.seq_item_port.connect(seqr.seq_item_export);//connect driver and sequencer 
  endfunction
  
endclass
////////////////////////////////////////////////////////////////////
class env extends uvm_env;
  `uvm_component_utils(env);
  
  function new(input string path = "env", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  //agent is child to env class 
  agent a;
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    a = agent::type_id::create("a", this);
  endfunction
  
endclass
////////////////////////////////////////////////////////////////////
class test extends uvm_test;
  `uvm_component_utils(test);
  
  function new(input string path = "test", uvm_component parent = null);
    super.new(path, parent);
  endfunction
  //env is child to test class 
  env e;
  sequence1 s1;
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    e = env::type_id::create("e", this);
    s1 = sequence1::type_id::create("s1", this);
  endfunction
  
  //starting the sequence and sending to sequencer
  //task to start method using s1.start
  virtual task run_phase(uvm_phase phase);
    phase.raise_objection(this);
    s1.start(e.a.seqr); 
    phase.drop_objection(this);
  endtask

endclass
/////////////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("test");  
  end
  
endmodule 
``` 
### Simulation Result 

![alt text](<Simulation Results/51.Undestanding Flow.png>)

</details>

__________________________________________________________

<details>
 <summary><b>52.Sending data to sequencer METHOD 1</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
///////////////////////////////////////////////////////////////
//1.transaction
class transaction extends uvm_sequence_item;
 rand bit [3:0]a;
 rand bit [3:0]b;
      bit [4:0]y;
  
 function new(input string path = "transaction");
   super.new(path);
 endfunction
  
  `uvm_object_utils_begin(transaction);
  `uvm_field_int(a, UVM_DEFAULT);
  `uvm_field_int(b, UVM_DEFAULT);
  `uvm_field_int(y, UVM_DEFAULT);
  `uvm_object_utils_end
  
endclass
////////////////////////////////////////////////////////////////
//2.sequence
class sequence1 extends uvm_sequence#(transaction);
  `uvm_object_utils(sequence1)
    transaction trans;  //making trans handle
  
  function new(input string path = "sequence1") ; //1 arg as uvm_object
    super.new(path);
  endfunction
  
  virtual task body();
    repeat(5)begin
      `uvm_do(trans); //uvm_do creates the object, randomizes the data and also connects trans to the sequencer
      `uvm_info("SEQ1", $sformatf("Data Sent: a: %0d b: %0d", trans.a , trans.b), UVM_NONE);
     end
  endtask
  
endclass
/////////////////////////////////////////////////////////////////////////
//3.driver
class driver extends uvm_driver#(transaction);
  `uvm_component_utils(driver)
  
  transaction trans;
  
  function new(input string path = "driver", uvm_component parent = null);  
    super.new(path, parent);
  endfunction
  
  //task to set up communication between driver and transaction
  virtual task run_phase(uvm_phase phase);
   trans = transaction::type_id::create("trans");  //1 arg as belong to uvm_object
    forever begin 
      seq_item_port.get_next_item(trans); //tell transaction tos end next packet
      `uvm_info("DRV", $sformatf("Data Rcvd: a: %0d, b: %0d",trans.a, trans.b), UVM_NONE);
      //////////////////
      //apply seq to DUT 
      //////////////////
      seq_item_port.item_done();  //send item_done to seq
    end
  endtask
  
endclass
///////////////////////////////////////////////////////////////////////////////////
//4.agent 
class agent extends uvm_agent;
  `uvm_component_utils(agent)
  
  //inside agent , we have driver and sequencer
  uvm_sequencer#(transaction) seqr;
  driver d;
  
  function new(input string path = "agent", uvm_component parent = null);  
    super.new(path, parent);
  endfunction
  
  //build phase
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    seqr = uvm_sequencer#(transaction)::type_id::create("seqr", this);
    d = driver::type_id::create("d", this);
  endfunction
  
  //connect phase to connect driver and seqr
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    d.seq_item_port.connect(seqr.seq_item_export); //connected drv and seqr
  endfunction
  
endclass
/////////////////////////////////////////////////////////////////////////////
//5.env
class env extends uvm_env;
  `uvm_component_utils(env)
  
  sequence1 s1;
  agent a;
  
  function new(input string path = "env", uvm_component parent = null);  
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    s1 = sequence1::type_id::create("s1");  //1 arg as belongs to uvm_object
    a = agent::type_id::create("a", this);
  endfunction
  
  //RUNNING SEQUENCE WITH START METHOD APPROACH 1
  //task to use start method inside env class intead of test class
  virtual task run_phase(uvm_phase phase);
    phase.raise_objection(this);
      s1.start(a.seqr);   
    phase.drop_objection(this);
  endtask

endclass
/////////////////////////////////////////////////////////////////////////////
//6.test 
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

![alt text](<Simulation Results/52.Sending data to sequencer METHOD 1.png>)

</details>

__________________________________________________________

<details>
 <summary><b>53.Sending Data to Driver METHOD 2</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
///////////////////////////////////////////////////////////////
//1.transaction
class transaction extends uvm_sequence_item;
 rand bit [3:0]a;
 rand bit [3:0]b;
      bit [4:0]y;
  
 function new(input string path = "transaction");
   super.new(path);
 endfunction
  
  `uvm_object_utils_begin(transaction);
  `uvm_field_int(a, UVM_DEFAULT);
  `uvm_field_int(b, UVM_DEFAULT);
  `uvm_field_int(y, UVM_DEFAULT);
  `uvm_object_utils_end
  
endclass
////////////////////////////////////////////////////////////////
//2.sequence
class sequence1 extends uvm_sequence#(transaction);
  `uvm_object_utils(sequence1)
    transaction trans;  //making trans handle
  
  function new(input string path = "sequence1") ; //1 arg as uvm_object
    super.new(path);
  endfunction
  /*
  virtual task body();
    repeat(5)begin
      `uvm_do(trans); //uvm_do creates the object, randomizes the data and also connects trans to the sequencer
     end
  endtask
  */
//INSTEAD of uvm_do , lets use start_item and finish_item 
  virtual task body();
    //crate trans object
    repeat(5)begin
     trans = transaction::type_id::create("trans");
     start_item(trans); //start_item and specify the instance name(here trans) 
  //start_item sends thr req to driver and has inbuilt wait_for_grant  
     assert(trans.randomize);
     finish_item(trans); //finish_item has in-built has wait_for_item_done 
  //once we get item_done, we do uvm_info
     `uvm_info("SEQ1", $sformatf("Data Sent: a: %0d b: %0d", trans.a , trans.b), UVM_NONE);
    end
    
  endtask
 
endclass
/////////////////////////////////////////////////////////////////////////
//3.driver
class driver extends uvm_driver#(transaction);
  `uvm_component_utils(driver)
  
  transaction trans;
  
  function new(input string path = "driver", uvm_component parent = null);  
    super.new(path, parent);
  endfunction
  
  //task to set up communication between driver and transaction
  virtual task run_phase(uvm_phase phase);
   trans = transaction::type_id::create("trans");  //1 arg as belong to uvm_object
    forever begin 
      seq_item_port.get_next_item(trans); //tell transaction tos end next packet
      `uvm_info("DRV", $sformatf("Data Rcvd: a: %0d, b: %0d",trans.a, trans.b), UVM_NONE);
      //////////////////
      //apply seq to DUT 
      //////////////////
      seq_item_port.item_done();  //send item_done to seq
    end
  endtask
  
endclass
///////////////////////////////////////////////////////////////////////////////////
//4.agent 
class agent extends uvm_agent;
  `uvm_component_utils(agent)
  
  //inside agent , we have driver and sequencer
  uvm_sequencer#(transaction) seqr;
  driver d;
  
  function new(input string path = "agent", uvm_component parent = null);  
    super.new(path, parent);
  endfunction
  
  //build phase
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    seqr = uvm_sequencer#(transaction)::type_id::create("seqr", this);
    d = driver::type_id::create("d", this);
  endfunction
  
  //connect phase to connect driver and seqr
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    d.seq_item_port.connect(seqr.seq_item_export); //connected drv and seqr
  endfunction
  
endclass
/////////////////////////////////////////////////////////////////////////////
//5.env
class env extends uvm_env;
  `uvm_component_utils(env)
  
  sequence1 s1;
  agent a;
  
  function new(input string path = "env", uvm_component parent = null);  
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    s1 = sequence1::type_id::create("s1");  //1 arg as belongs to uvm_object
    a = agent::type_id::create("a", this);
  endfunction
  
  //RUNNING SEQUENCE WITH START METHOD APPROACH 1
  //task to use start method inside env class intead of test class
  virtual task run_phase(uvm_phase phase);
    phase.raise_objection(this);
      s1.start(a.seqr);   
    phase.drop_objection(this);
  endtask

endclass
/////////////////////////////////////////////////////////////////////////////
//6.test 
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

![alt text](<Simulation Results/53.Sending Data to Driver METHOD 2.png>)

</details>

__________________________________________________________

<details>
 <summary><b>54.Multiple Sequnce in Parallel</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
///////////////////////////////////////////////////////////////
//1.transaction
class transaction extends uvm_sequence_item;
 rand bit [3:0]a;
 rand bit [3:0]b;
      bit [4:0]y;
  
 function new(input string path = "transaction");
   super.new(path);
 endfunction
  
  `uvm_object_utils_begin(transaction);
  `uvm_field_int(a, UVM_DEFAULT);
  `uvm_field_int(b, UVM_DEFAULT);
  `uvm_field_int(y, UVM_DEFAULT);
  `uvm_object_utils_end
  
endclass
////////////////////////////////////////////////////////////////
//2. 1st sequence
class sequence1 extends uvm_sequence#(transaction);
  `uvm_object_utils(sequence1)
    transaction trans;  //making trans handle
  
  function new(input string path = "sequence1") ; //1 arg as uvm_object
    super.new(path);
  endfunction

//INSTEAD of uvm_do , lets use start_item and finish_item 
  virtual task body();
    //crate trans object
     trans = transaction::type_id::create("trans");
     `uvm_info("SEQ1", "SEQ1 Started", UVM_NONE);
     start_item(trans); //start_item and specify the instance name(here trans) 
  //start_item sends the req to driver and has inbuilt wait_for_grant  
      trans.randomize();
     finish_item(trans); //finish_item has in-built has wait_for_item_done
      `uvm_info("SEQ1", "SEQ1 Ended", UVM_NONE);
  endtask
 
endclass
/////////////////////////////////////////////////////////////////////////
//2. 2nd sequence
class sequence2 extends uvm_sequence#(transaction);
  `uvm_object_utils(sequence2)
    transaction trans;  //making trans handle
  
  function new(input string path = "sequence2") ; //1 arg as uvm_object
    super.new(path);
  endfunction

//INSTEAD of uvm_do , lets use start_item and finish_item 
  virtual task body();
    //crate trans object
     trans = transaction::type_id::create("trans");
      `uvm_info("SEQ2", "SEQ2 Started", UVM_NONE);
     start_item(trans); //start_item and specify the instance name(here trans) 
  //start_item sends the req to driver and has inbuilt wait_for_grant  
     trans.randomize();
     finish_item(trans); //finish_item has in-built has wait_for_item_done
      `uvm_info("SEQ2", "SEQ2 Ended", UVM_NONE);
  endtask
 
endclass
//3.driver
class driver extends uvm_driver#(transaction);
  `uvm_component_utils(driver)
  
  transaction trans;
  
  function new(input string path = "driver", uvm_component parent = null);  
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
  	super.build_phase(phase);
    trans = transaction::type_id::create("trans");
  endfunction
  
  //task to set up communication between driver and sequencer 
  virtual task run_phase(uvm_phase phase);
    forever begin 
      seq_item_port.get_next_item(trans); //gives grant to sequence
      //////////////////
      //apply seq to DUT 
      //////////////////
      seq_item_port.item_done();  //send item_done to sequence
    end
  endtask
  
endclass
///////////////////////////////////////////////////////////////////////////////////
//4.agent 
class agent extends uvm_agent;
  `uvm_component_utils(agent)
  
  //inside agent , we have driver and sequencer
  uvm_sequencer#(transaction) seqr;
  driver d;
  
  function new(input string path = "agent", uvm_component parent = null);  
    super.new(path, parent);
  endfunction
  
  //build phase
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    seqr = uvm_sequencer#(transaction)::type_id::create("seqr", this);
    d = driver::type_id::create("d", this);
  endfunction
  
  //connect phase to connect driver and seqr
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    d.seq_item_port.connect(seqr.seq_item_export); //connected drv and seqr
  endfunction
  
endclass
/////////////////////////////////////////////////////////////////////////////
//5.env
class env extends uvm_env;
  `uvm_component_utils(env)
  
  agent a;
  
  function new(input string path = "env", uvm_component parent = null);  
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    a = agent::type_id::create("a", this);
  endfunction 

endclass
/////////////////////////////////////////////////////////////////////////////
//6.test 
class test extends uvm_test;
  `uvm_component_utils(test)
  
  sequence1 s1;
  sequence2 s2;
  env e;
  
  function new(input string path = "test", uvm_component parent = null);  
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    e = env::type_id::create("e", this);
    s1 = sequence1::type_id::create("s1");
    s2 = sequence2::type_id::create("s2");
  endfunction
  
  //task to do start method and for sequence get access to sequencer 
  virtual task run_phase(uvm_phase phase);
    phase.raise_objection(this);
    
    // e.a.seq.set_arbitration(UVM_SEQ_ARB_STRICT_RANDOM); 
    
     fork   //using fork join so that all processes happen parallely
       s1.start(e.a.seqr);
       s2.start(e.a.seqr); //by default , the access to seqr is FIFO fashion, first to get access gets finished first
     join
    phase.drop_objection(this);
  endtask
  
endclass
//////////////////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("test");
  end
  
endmodule 
``` 
### Simulation Result 

![alt text](<Simulation Results/54.Multiple Sequence in Parallel.png>)

</details>

__________________________________________________________

<details>
 <summary><b>54.Multiple Sequnce in Parallel</b></summary><br>

### Code

```systemverilog 
`include "uvm_macros.svh"
import uvm_pkg::*;
///////////////////////////////////////////////////////////////
//1.transaction
class transaction extends uvm_sequence_item;
 rand bit [3:0]a;
 rand bit [3:0]b;
      bit [4:0]y;
  
 function new(input string path = "transaction");
   super.new(path);
 endfunction
  
  `uvm_object_utils_begin(transaction);
  `uvm_field_int(a, UVM_DEFAULT);
  `uvm_field_int(b, UVM_DEFAULT);
  `uvm_field_int(y, UVM_DEFAULT);
  `uvm_object_utils_end
  
endclass
////////////////////////////////////////////////////////////////
//2. 1st sequence
class sequence1 extends uvm_sequence#(transaction);
  `uvm_object_utils(sequence1)
    transaction trans;  //making trans handle
  
  function new(input string path = "sequence1") ; //1 arg as uvm_object
    super.new(path);
  endfunction

//INSTEAD of uvm_do , lets use start_item and finish_item 
  virtual task body();
    //crate trans object
     trans = transaction::type_id::create("trans");
     `uvm_info("SEQ1", "SEQ1 Started", UVM_NONE);
     start_item(trans); //start_item and specify the instance name(here trans) 
  //start_item sends the req to driver and has inbuilt wait_for_grant  
      trans.randomize();
     finish_item(trans); //finish_item has in-built has wait_for_item_done
      `uvm_info("SEQ1", "SEQ1 Ended", UVM_NONE);
  endtask
 
endclass
/////////////////////////////////////////////////////////////////////////
//2. 2nd sequence
class sequence2 extends uvm_sequence#(transaction);
  `uvm_object_utils(sequence2)
    transaction trans;  //making trans handle
  
  function new(input string path = "sequence2") ; //1 arg as uvm_object
    super.new(path);
  endfunction

//INSTEAD of uvm_do , lets use start_item and finish_item 
  virtual task body();
    //crate trans object
     trans = transaction::type_id::create("trans");
      `uvm_info("SEQ2", "SEQ2 Started", UVM_NONE);
     start_item(trans); //start_item and specify the instance name(here trans) 
  //start_item sends the req to driver and has inbuilt wait_for_grant  
     trans.randomize();
     finish_item(trans); //finish_item has in-built has wait_for_item_done
      `uvm_info("SEQ2", "SEQ2 Ended", UVM_NONE);
  endtask
 
endclass
//3.driver
class driver extends uvm_driver#(transaction);
  `uvm_component_utils(driver)
  
  transaction trans;
  
  function new(input string path = "driver", uvm_component parent = null);  
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
  	super.build_phase(phase);
    trans = transaction::type_id::create("trans");
  endfunction
  
  //task to set up communication between driver and sequencer 
  virtual task run_phase(uvm_phase phase);
    forever begin 
      seq_item_port.get_next_item(trans); //gives grant to sequence
      //////////////////
      //apply seq to DUT 
      //////////////////
      seq_item_port.item_done();  //send item_done to sequence
    end
  endtask
  
endclass
///////////////////////////////////////////////////////////////////////////////////
//4.agent 
class agent extends uvm_agent;
  `uvm_component_utils(agent)
  
  //inside agent , we have driver and sequencer
  uvm_sequencer#(transaction) seqr;
  driver d;
  
  function new(input string path = "agent", uvm_component parent = null);  
    super.new(path, parent);
  endfunction
  
  //build phase
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    seqr = uvm_sequencer#(transaction)::type_id::create("seqr", this);
    d = driver::type_id::create("d", this);
  endfunction
  
  //connect phase to connect driver and seqr
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    d.seq_item_port.connect(seqr.seq_item_export); //connected drv and seqr
  endfunction
  
endclass
/////////////////////////////////////////////////////////////////////////////
//5.env
class env extends uvm_env;
  `uvm_component_utils(env)
  
  agent a;
  
  function new(input string path = "env", uvm_component parent = null);  
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    a = agent::type_id::create("a", this);
  endfunction 

endclass
/////////////////////////////////////////////////////////////////////////////
//6.test 
class test extends uvm_test;
  `uvm_component_utils(test)
  
  sequence1 s1;
  sequence2 s2;
  env e;
  
  function new(input string path = "test", uvm_component parent = null);  
    super.new(path, parent);
  endfunction
  
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    e = env::type_id::create("e", this);
    s1 = sequence1::type_id::create("s1");
    s2 = sequence2::type_id::create("s2");
  endfunction
  
  //task to do start method and for sequence get access to sequencer 
  virtual task run_phase(uvm_phase phase);
    phase.raise_objection(this);
    
    // e.a.seq.set_arbitration(UVM_SEQ_ARB_STRICT_RANDOM); 
    
     fork   //using fork join so that all processes happen parallely
       s1.start(e.a.seqr);
       s2.start(e.a.seqr); //by default , the access to seqr is FIFO fashion, first to get access gets finished first
     join
    phase.drop_objection(this);
  endtask
  
endclass
//////////////////////////////////////////////////////////////////////////////
module tb;
  
  initial begin
    run_test("test");
  end
  
endmodule 
``` 
### Simulation Result 

![alt text](<Simulation Results/54.Multiple Sequence in Parallel.png>)

</details>

__________________________________________________________