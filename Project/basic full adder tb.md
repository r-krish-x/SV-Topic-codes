# design
```systemverilog
module FA(in1, in2, cin, sum, cout);
  input in1, in2;
  input cin;
  output sum, cout;
  
  assign sum = in1 ^ in2 ^ cin;
  assign cout = (in1 & in2) | (in1 & cin) | (in2 & cin);
  
endmodule
```

# testbench

```systmeverilog
logic end_of_test_flag;

//  ______interface________
interface FA_if;
  
  logic in1;
  logic in2;
  logic cin;
  logic sum;
  logic cout;
  
endinterface

// __________transaction class_________
class FA_item;
  
  rand bit in1;
  rand bit in2;
  rand bit cin;
  bit sum;
  bit cout;
  
endclass

// ___________generator______________
class generator;
  
  mailbox #(FA_item) driver_mbx;
  event driver_done;
  
  int num = 10;
  int ii;
  
  task run();
    
    FA_item item = new();
    
    for(ii = 0; ii < num; ii++)begin
      
      item.randomize();
      
      driver_mbx.put(item);
      @(driver_done);
      
      #20;
//  If the Generator generates the next item immediately after driver_done (without delay), the Driver will pick it up and overwrite the interface signals instantly.
// Thats why without delay you'll get nothing in waveform
 
    end
    
    end_of_test_flag = 1;
    
  endtask
  
endclass

//______________driver________________
class driver;
  
  virtual FA_if vif;
  
  mailbox #(FA_item) driver_mbx;
  event driver_done;
  
  task run();
    
    forever begin
      
      FA_item item;
      
      driver_mbx.get(item);
      
      vif.in1 = item.in1;
      vif.in2 = item.in2;
      vif.cin = item.cin;
      
      -> driver_done;
      
      if(end_of_test_flag) break;
      
    end
    
  endtask
  
endclass


// _______________monitor ______________________
class monitor;
  
  
  virtual FA_if vif;
  
  mailbox #(FA_item)scrbrd_mbx;
  
  int num =  10;
  int i;
  
  task run();  //  this is standard practice, here just a one- 
    monitor1;  //- monitor interface when multiple monitors are-
  endtask     //-  there, it is scalable approch
  
  task monitor1();
    
    FA_item item = new;
    
    for(i = 0; i < num; i++)begin
      
      item.in1 = vif.in1;
      item.in2 = vif.in2;
      item.cin = vif.cin;
      
      item.sum = vif.sum;
      item.cout = vif.cout;
      
      scrbrd_mbx.put(item);
      
      #20;
      
      if(end_of_test_flag) break;
      
    end
  endtask
endclass


// ____________scoreboard_______________
class scoreboard;
  
  mailbox #(FA_item) scrbrd_mbx;
  
  task run();
    
    forever begin
      
      FA_item item;
      scrbrd_mbx.get(item);
      
      if (item.sum == item.in1 ^ item.in2 ^ item.cin) begin
        $display("---------------Test pass for sum at %0t------------------------",$time );
        
       
        
      end
      
      
      if (item.cout == (item.in1 & item.in2) | (item.in1 & item.cin) |  item.in2 & item.cin) begin
        
        $display("_______________Test pass for cout at %0t------------------------", $time);
        
     
        
      end
      
      
      if(end_of_test_flag)break;
    end
    
      endtask
      endclass
      
      
      
//  _______________environment________________     
      class environment;
        
        driver			 drv0;
        monitor	 		 mon0;
        generator		 gen0;
        scoreboard 		 scrbrd0;
        
        
        mailbox #(FA_item)driver_mbx;
        mailbox #(FA_item)scrbrd_mbx;
        event driver_done;
        
        virtual FA_if vif;
        
        function new();
        
          gen0 = new;
          drv0 = new;
          mon0 = new;
          scrbrd0 = new;
          
          driver_mbx = new();
          scrbrd_mbx = new();
          
        drv0.driver_mbx = driver_mbx;
        gen0.driver_mbx = driver_mbx;
        mon0.scrbrd_mbx = scrbrd_mbx;
        scrbrd0.scrbrd_mbx = scrbrd_mbx;
        
        
        drv0.driver_done = driver_done;
        gen0.driver_done = driver_done;
        
        endfunction
        
        
        virtual task run();
          
          drv0.vif = vif;
          mon0.vif = vif;
          
          fork
            
            drv0.run();
            gen0.run();
            mon0.run();
            scrbrd0.run();
          join_any
          
        endtask
      endclass
      
//    ___________________test class________________  
     class test0;
       
       environment env0;
       
       function new();
         env0 = new;
       endfunction 
       
       task run();
         
         env0.run();
       endtask
       
     endclass
    
//       ______________testbench tob module_________________
      module tb_FA;
        
        FA_if if0();
        
        FA dut (
        
          .in1(if0.in1),
          .in2(if0.in2),
          .cin(if0.cin),
          .sum(if0.sum),
          .cout(if0.cout));

        test0 t0;
        
        initial begin
          
        end_of_test_flag = 0;
          
          t0 = new;
          t0.env0.vif = if0;
          t0.run();
          
          #500 $finish;
        end
        
        initial begin
          $dumpvars;
          $dumpfile("FA.vcd");
        end
        
      endmodule
```
