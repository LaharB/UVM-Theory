# Section 7 : Sequences

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
 
``` 
### Simulation Result 

![alt text](<Simulation Results/50.Creating Sequences.png>)

</details>

__________________________________________________________