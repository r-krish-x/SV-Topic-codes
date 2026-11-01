# Design

```systemverilog

module full_adder(
  input a,b,cin,
  output reg s,c);

//   input [1:0] a,b,cin;
// output reg s,c;

always@(a,b,cin)begin
 s = a^b^cin;
 c= a&b | b&cin | a&cin;
 end
 
 endmodule

```

# testbench
```systemverilog
logic end_of_test_flag;

//===============================interface=========================
interface adder_if;

  logic a;
  logic b;
  logic cin;
  logic s;
  logic c;
  
endinterface

// ===========================Transaction===========================
class transaction;

  rand bit  a,b,cin;
  bit 		s,c;
 
  function void display(string name);
    $display("[%s] a=%0b, b=%0b, cin=%0b | s=%0b, c=%0b", name, a,b,cin,s,c);
  
  endfunction
  
endclass

// -------------------------------------------------------------------
// questions

//`function void `is used when no return value, but in generator, driver, monitor is used for `trans.display` using this function from transaction. This is return by this void's function. Explain

// -------------------------------------------------------------------


// ===========================generator=============================

class generator;
  
transaction trans;
  mailbox mbx;
  
  function new(mailbox m);
  mbx = m;
  
  endfunction
  
  task main();
    repeat(5) begin
      trans = new();
      void'( trans.randomize());  // -> void'( trans.randomize) -> void 
      trans.display("Generator"); //    added bcoz it was showing error
      mbx.put(trans);
      
    end
    end_of_test_flag = 1;
  endtask

endclass

// -------------------------------------------------------------------
// questions

// why in `function new(mailbox m)` mailbox m mean what, then mailbox's pointer is mbx = m, how??

// in task, repeat(5) then trans.new mean 5 unit space is for trans handle, then randomized it and that randomized data put in mailbox to send it to driver maybe ?
// -- -----------------------------------------------------------------


// =========================driver===================================
class driver;
  
  transaction trans;
  mailbox mbx;
  virtual adder_if vif;
  
  function new(mailbox m, virtual adder_if v);
    mbx = m;
    vif = v;
  endfunction
  
  
  task main();
  forever begin
    mbx.get(trans);
    vif.a   = trans.a;
    vif.b   = trans.b;
    vif.cin = trans.cin;
    trans.display("Driver");
    #5;
  end
  endtask
   
endclass

// -------------------------------------------------------------------
// questions

// same question-in function why mailbox m and virtual adder_if v then mbx = m and vif = v again.

// get the data from genrator and virtual interface's input is equal to randomized data of a,b,c.

// why 5 unit delay, whats need ?

// -------------------------------------------------------------

// ==========================monitor===============================
class monitor;
transaction trans;
mailbox mbx;
  virtual adder_if vif;
  
  function new(mailbox m, virtual adder_if v);
    mbx= m;
    vif=v;
  endfunction
  
  task main();
    forever begin
      #5;
      
      trans = new();
      trans.a = vif.a;
      trans.b = vif.b;
      trans.cin = vif.cin;
      trans.s = vif.s;
      trans.c = vif.c;
      
      trans.display("Monitor");
      
      mbx.put(trans);
if(end_of_test_flag) break;      
    end
  endtask
endclass

// -------------------------------------------------------------------
// questions

// variable and function same as driver and then new constructor for trans and that pin wiggled data will gonna store into this new space.

// que-> that randomize data for input and output is eqauled  to virtual interface's in-out puts why

// ans-> the equals sign (=) takes whatever is on the RIGHT side, and 		copies it into the LEFT side.

// mbx.put is for data put for scoreboard

// -------------------------------------------------------------------

// =======================scoreboard==================================
class scoreboard;
transaction trans;
  mailbox mbx;
  
  function new(mailbox m);
    mbx = m;
  endfunction
  
  task main();
    forever begin
      mbx.get(trans);
      
      if({trans.c, trans.s} ==  trans.a + trans.b + trans.cin) begin
   
      $display("RESULT MATCH");
      end
   else begin
      $display("RESULT MISMATCH");
   
    end
    end
  endtask

endclass

// -------------------------------------------------------------------
// questions

// in scoreboard there no need of vif coz here to compare data we need real values which will compared to expected output.

// in task firstforemost need mbx.get coz thats the data will be compare to expected result

// -------------------------------------------------------------------

// ========================environment==============================
class environment;
generator gen;
  driver drv;
  monitor mon;
  scoreboard sco;
  
  mailbox m_gen2drv;
  mailbox m_mon2sco;
  
  virtual adder_if vif;
  
  function new(virtual adder_if v);
    vif = v;
    m_gen2drv = new();
    m_mon2sco = new();
    
    gen = new(m_gen2drv);
    drv = new(m_gen2drv, vif);
    mon = new(m_mon2sco, vif);
    sco = new(m_mon2sco);
    
  endfunction
  
  task start();
    fork
    gen.main();
    drv.main();
    mon.main();
    sco.main();
    join_any
    
    #50;
    $finish;  
  endtask


endclass

// -------------------------------------------------------------
// questions

// why we need all 4 gen,drv,sco, mon here

// why gen2drv and mon2sco is needed. also it gave the space to 2 of them. why?

//if real values need now then why used virtual now?? 

// `gen = gen2drv` `drv = gen2drv, vif`
// `mon = mon2sco, vif`  `sco = mon2sco` why is that i did not get it.

// what all parts with main  

// delay why

// $finish why

// -------------------------------------------------------------------

// ======================test class==================================
class test;
  
  environment env;
  function new(virtual adder_if v);
    env = new(v);
  endfunction
  
  task run();
    env.start();
    
  endtask
    
    endclass



// ========================top module==================================
module top;
    adder_if pif();
    
    full_adder dut (.a(pif.a),
               .b(pif.b),
               .cin(pif.cin),
               .s(pif.s),
               .c(pif.c));
    
    test tst;
    
    initial begin
      tst = new(pif);
      tst.run();
    end

endmodule
```

1. Monitor should not be continuously ON, we should have an "end of test flag", in order to break its forever loop after generator is done 
  
2. And, in generator, we're supposed to assign 1 to end of test flag, but we cannot do it without giving time delay in generator, so make sure to give time delay in generator
  
3. Since we're using time delay in generator and forever loop in driver, use event triggering mechanism for synchronization All 3 points can be understood from MY older full adder testbench.
			- Order of execution of components
			- exploration of testbench, flow of pointers source and destination find out
			- counter question for everything
			- You can ask AI for testbench, what improvement can be done in this,
			- 2-3 testbenches are enough, for basic understanding.


- monitor
  **que**→ that randomize data for input and output is eqauled  to virtual interface's in-out puts why

  **ans**-> the equals sign (=) takes whatever is on the RIGHT side, and 		copies it into the LEFT side.
					In the Driver you wrote: vif.a = trans.a;
					The Driver reads the random number from the paper (trans), and  pushes it onto the hardware pin (vif). (Software -> Hardware)
					In the Monitor you wrote: trans.a = vif.a;
					This is exactly the opposite! The Monitor looks at the hardware pin (vif), and copies the number onto its blank piece o	paper (trans).
