##
```systemverilog
// ===example:1=== Type A: An event is triggered after waiting for the event trigger
// ====================================================================================

module event_example();
  event e1;
  
  task process_A();
    #10;
    $display("@%0t: Before triggering event e1", $time);
    ->e1;
    $display("@%0t: After triggering event e1", $time);
  endtask
  
  task process_B();
    $display("@%0t: waiting for the event e1", $time);
    @e1;
   // wait(e1.triggered);
    $display("@%0t: event e1 is triggered", $time);
  endtask

  initial begin
    fork
      process_A();
      process_B();
    join
  end
endmodule

//===Example:2===Type B: An event is triggered before waiting for event trigger
// // ====================================================================================

module event_example();
  event e1;
  
  task process_A();
    $display("@%0t: Before triggering event e1", $time);
    ->e1;
    $display("@%0t: After triggering event e1", $time);
  endtask
  
  task process_B();
    #10;
    $display("@%0t: waiting for the event e1", $time);
    @e1;
//    wait(e1.triggered);
    $display("@%0t: event e1 is triggered", $time);
  endtask

  initial begin
    fork
      process_A();
      process_B();
    join
  end
endmodule


// ===Example:3============================= @ and wait()==============================

module event_example();
  event e1;
  
  task process_A();
    $display("@%0t: Process A: Before triggering event e1", $time);
    ->e1;
    $display("@%0t: Process A: After triggering event e1", $time);
  endtask
  
  task process_B();
    $display("@%0t: Process B: waiting for the event e1 using @", $time);
    @e1;
    $display("@%0t: Process B: event e1 is triggered using @", $time);
  endtask

  task process_C();
    $display("@%0t: Process C: waiting for the event e1 using wait(e1.triggered)", $time);
    wait(e1.triggered);
    $display("@%0t: Process C: event e1 is triggered using wait(e1.triggered)", $time);
  endtask
  
  initial begin
    fork
      process_A();
      process_B();
//       process_A();
      process_C();
    join
  end
endmodule

===example:4------------------------- using ->> operator-------------------------
module event_example();
  event e1;

  task process_A();
    #10; $display("@%0t: process_A: Before triggering event e1 using ->>", $time);
    ->>e1;
    $display("@%0t: process_A: After triggering event e1 using ->>", $time);
  endtask
  
  task process_B();
    #5; $display("@%0t: process_B: waiting for the event e1", $time);
    
    @(e1);
    //wait(e1.triggered);
    $display("@%0t: process_B: event e1 is triggered", $time);
  endtask

  initial begin
    fork
      process_A();
      process_B();
      //process_A();

    join
  end
endmodule


//     @(e1.triggered); 
// this is invalid syntax, it doesnt give error which does not mean its correct

/*
//----------------using @ operator--------------
@0: waiting for the event e1
@10: Before triggering event e1
@10: After triggering event e1
@10: event e1 is triggered
------------------------------------------------
@0: Before triggering event e1
@0: After triggering event e1
@10: waiting for the event e1
------------------------------------------------
@0: Before triggering event e1
@0: After triggering event e1
@0: waiting for the event e1

//----------------------------------------------
--------------------using wait() operator-------
//----------------------------------------------

@0: waiting for the event e1
@10: Before triggering event e1
@10: After triggering event e1
@10: event e1 is triggered
---------------------------------------------
@0: Before triggering event e1
@0: After triggering event e1
@10: waiting for the event e1
-----------------------------------------------
@0: Before triggering event e1
@0: After triggering event e1
@0: waiting for the event e1
@0: event e1 is triggered
*/


```

* In 2nd example, race condition will happen bcoz process_A is executing first then process_B.
  The blocking event trigger -> should be executed later, firstly @e1 should be executed but its
  executing in opposite order so e1 will wait forever for trigger and trigger will never come.
* Here this race condition could be avoided by wait construct but it can only avoid if both process
  are executing in same time unit. For example on the line 45 `wait(e1.triggered);` is replaced by `@e1`
  also timeunit must be same then it can resolve it.

* In example 4 : ->> non blocking event trigger is used, so @ will not block any statement, all the
  threads will execute.

* I misunderstood `->>` solves any kind of delay but it solves only when both are having same
  timeunit or trigger `->>` must be executed after `@` meaning, in this code block process_A must have
  higher delay than process_B coz if process_B's delay is more than process_A then event e1 will not
  be executed(line 106).

 `task process_A();
    #10; $display("@%0t: process_A: Before triggering event e1 using ->>", $time);
    ->>e1;
    $display("@%0t: process_A: After triggering event e1 using ->>", $time);
  endtask
  
  task process_B();
    #5; $display("@%0t: process_B: waiting for the event e1", $time);
			//@(e1.triggered);
    @(e1);
    //wait(e1.triggered);
    $display("@%0t: process_B: event e1 is triggered", $time);
  endtask`
