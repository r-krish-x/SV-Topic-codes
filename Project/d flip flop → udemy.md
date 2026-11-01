# design

```systemverilog
module dff(dff_if vif);
  
  always @(posedge vif.clk)
    begin
      
      if(vif.rst == 1'b1)
        vif.dout <= 1'b0;
      
      else
        vif.dout <=vif.din;
    end
  
endmodule

interface dff_if;
  logic clk;
  logic rst;
  logic din;
  logic dout;
  
endinterface

```

# testbench

```systemverilog
class transaction;
  
  rand bit din;
  bit dout;
  
  function transaction copy();
    copy = new();
    copy.din = this.din;
    copy.dout = this.dout;
  endfunction
  
  function void display(input string tag);
    $display("[%s] → din:%0b, dout:%0b", tag, din, dout);
  endfunction
  
endclass

class generator;
  
  transaction trans;
  mailbox #(transaction)gen2drv;
  mailbox #(transaction)gen2sco;
  event sconext;
  event done;
  int count;
  
  function new(mailbox #(transaction)gen2drv, mailbox #(transaction)gen2sco);
    this.gen2drv = gen2drv;
    this.gen2sco = gen2sco;
    trans = new();
  endfunction
  
  task run();
    repeat(count)begin
      assert(trans.randomize) else $error ("[GEN]:Randomization failed");
      gen2drv.put(trans.copy);
    gen2sco.put(trans.copy);
      trans.display("GEN");
      @(sconext);
    end
    ->done;
  endtask
endclass

class driver;

  transaction trans;
  mailbox #(transaction) gen2drv;
  virtual dff_if vif;
  
  function new(mailbox #(transaction)gen2drv);
    this.gen2drv = gen2drv;
  endfunction
  
  task reset();
      vif.rst <= 1'b1;
      repeat(5) @ (posedge vif.clk);
      vif.rst <= 1'b0;
      @(posedge vif.clk);
      $display("[DRV]: RESET DONE");
  endtask
  
  task run();
    forever begin
    gen2drv.get(trans);
    vif.din <= trans.din;
    @(posedge vif.clk);
    trans.display("DRV");
    vif.din <= 1'b0;
    @(posedge vif.clk);
// //------------------------------------------------------
//       gen2drv.get(trans);
//         vif.din <= trans.din;
//         @(posedge vif.clk);
//         vif.din <= 1'b0;
//         @(posedge vif.clk);
//         trans.display("DRV");// what diff. does it make to put this display after asserting reset or deassertinig reset
// //------------------------------------------------------
    end
  endtask
endclass

class monitor;
  
  transaction trans;
  mailbox #(transaction) mon2sco;
  virtual dff_if vif;
  
  function new(mailbox #(transaction) mon2sco);
    this.mon2sco = mon2sco;
  endfunction
  
  task run();
    forever begin
    trans = new();
      repeat(2) @ (posedge vif.clk);
      trans.dout = vif.dout;
      mon2sco.put(trans);
      trans.display("MON");
    end
  endtask
endclass

// // ----------------------------------------
//  task run();
//       forever begin
//       trans = new();
//         mon2sco.put(trans);// this line should be after 133 line, bcoz if i put before vif giviing data to trans, it doesnt matter if i put the data or not
//       trans.dout = vif.dout;// forgot to add delay here
//       trans.display("MON");
//       end 
//     endtask
// // ---------------------------------------------
      
class scoreboard;
  
  transaction trans;
  transaction trgen;
  
  mailbox #(transaction) mon2sco;
  mailbox #(transaction) gen2sco;
  event sconext;
  
  function new(mailbox #(transaction)mon2sco, mailbox #(transaction) gen2sco);
    this.mon2sco = mon2sco;
    this.gen2sco = gen2sco;
  endfunction
  
  task run();
    forever begin
      mon2sco.get(trans);
      gen2sco.get(trgen);
      trans.display("SCO");
      trgen.display("REFERENCE DATA");
      if(trans.dout == trgen.din)
        $display("DATA MATCHED");
      else 
        $display("DATA MISMATCHED");
    ->sconext;
    end   
  endtask
endclass
    

 class environment;
    
    generator gen;
    driver    drv;
    monitor   mon;
    scoreboard sco;
    
    mailbox #(transaction)gen2drv;
    mailbox #(transaction)gen2sco;
    mailbox #(transaction)mon2sco;
    
    event next;
    
    virtual dff_if vif;
    
    function new(virtual dff_if vif);
      
      gen2drv = new();
      gen2sco = new();
      mon2sco = new();
      
      gen = new(gen2drv, gen2sco);
      drv = new(gen2drv);
      mon = new(mon2sco);
      sco = new(mon2sco, gen2sco);
      
      this.vif = vif;
      drv.vif = this.vif;
      mon.vif = this.vif;
      
      gen.sconext = next;
      sco.sconext = next;
      
    endfunction
      
      task pre_test();
        drv.reset();
      endtask
      
      task test();
        fork
        gen.run();
        drv.run();
        mon.run();
        sco.run();
        join_any
      endtask
    
      task post_test();
        wait(gen.done.triggered);
        $finish;
      endtask
      
      task run();
        pre_test();
        test();
        post_test();
      endtask      
 endclass
        
        
```
