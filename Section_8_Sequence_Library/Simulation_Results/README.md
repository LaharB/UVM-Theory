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
```





</details>