# Section 8 : Understanding Sequence Library

<details><summary>Code & Simulation</summary>

### Code 

```systemverilog
`include "uvm_macros.svh"
import uvm_pkg::*;

//transaction or sequence_item
class transaction extends uvm_seq_item;
  `uvm_object_utils(transaction) //regis to factory to use macros
  
  bit [3:0] a;
  bit [3:0] b;
  
  //std constr
  function new(input string path);
    super.new(path);
  endfunction  
  
endclass

typedef class seq_library; //reqd only if we use uvm_add_to_seq_lib(SEQ_NAME, seq_library) inside every sequence

//sequences - these are like generator class
class seq1 extends uvm_sequence#(transaction);
  `uvm_object_utils(seq1)
  //`uvm_add_to_seq_lib(seq1, seq_library)
  
  transaction tr;
  
  //std constr
  function new(input string path);
    super.new(path);  
  endfunction
  
  //body task - starting the seqr call this body task
  virtual task body();
    tr = transaction::type_id::create("tr");
    start_item(tr); 
    tr.a = 4;
    tr.b = 4;
    finish_item(tr);
  endtask
  
endclass
  
///another sequence
class seq2 extends uvm_sequence#(transaction);
  `uvm_object_utils(seq2)
  //`uvm_add_to_seq_lib(seq2, seq_library);
  
  transaction tr;
  
  //std constr
  function new(input string path);
    super.new(path);
  endfunction
  
  virtual task body();
    start_item(tr);
    tr.a = 5;
    tr.b = 5;
    finish_item(tr);    
  endtask
    
endclass
  
//SEQUNCE LIBRARY CLASS
class seq_library extends uvm_sequence_library#(transaction);
  `uvm_object_utils(seq_library)
  `uvm_sequence_library_utils(seq_library)
  
  //std contr
  function new(input string path);
    super.new(path);
    //adding my sequences inside the library
    add_typewide_sequence(seq1::get_type());
    add_typewide_sequence(seq2::get_type());
    
  endfunction
    
endclass

//DRIVER
class driver extends uvm_driver#(transaction);
  `uvm_component_utils(driver)   
    
  transaction tr;
  
  //std constr
  function new(input string path, uvm_component parent);
    super.new(path, parent);
  endfunction
  
  virtual task run_phase(uvm_phase phase);
    forever begin //drv needs to be ready all the time - forever
      seq_item_port.get_next_item(tr);
      `uvm_info(get_type_name(), $sformatf("a:%0d, b:%0d", tr.a, tr.b), UVM_NONE); //we can also use "DRV" as tag
       #10; //drive the data to DUT 
      seq_item_port.item_done(); //ack to seq
    end
  endtask 
  
endclass

//AGENT
class agent extends uvm_agent;
  `uvm_component_utils(agent)
  
  //std constr
  function new(input string path, uvm_component parent);
    super.new(path, parent);  
  endfunction
  
  //connect drv and seqr inside agent
  driver d;
  uvm_sequencer#(transaction) seqr;

  //build_phase - function + super
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    d = driver::type_id::create("drv", this);
    seqr = uvm_sequencer#(transaction)::type_id::create("seqr", this);
  endfunction
  
  //connect phase - fucntion + super
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    d.seq_item_port.connect(seqr.seq_item_export); //connected the    
  endfunction

endclass

//ENVIRONMENT 
class env extends uvm_env;
  `uvm_component_utils(env)
  
  //std constr
  function new(input string path, uvm_component parent);
    super.new(path, parent);  
  endfunction
  
  agent a;
  
  //build_phase - function + super 
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    a = agent::type_id::create("a", this);   
  endfunction
    
endclass

//TEST 
class test extends uvm_test;
  `uvm_component_utils(test)
  
  //std constr
  function new(input string path);
    super.new(path);
  endfunction
  
  env e;
  sequence_library seqlib;
  
  //BUILD
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    e = env::type_id::create("e", this);
    seqlib = sequence_library::type_id::create("seqlib");
    seqlib.selection_mode = UVM_SEQ_LIB_RANDC; //choosing the order for creating the sequeces
    seqlib.min_random_count = 5; //how many transactions we want to create 
    seqlib.max_random_count = 10;
    seqlib.init_sequence_library(); //initialize the seq_libr
    seqlib.print(); //field_macro to print the details
  endfunction
  
  //RUN_PHASE
  virtual task run_phase(umv_phase phase);
    phase.raise_objection(this);
    assert(seqlib.randomize()); //start the sequences
    seqlib.start(e.a.seqr); //start the sequence
    phase.drop_objection(this);
  endtask
  
endclass


```





</details>