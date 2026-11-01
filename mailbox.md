```systemverilog
//----------Bounded Generic mailbox-----------
module mailbox_example();
  mailbox mb = new(3);
  
  task process_A();
    int value = 5;
    string name = "STRING";
    mb.put(value);
    $display("Put data = %0d", value);
    mb.put("STRING");
    $display("Put data = %s", name);
  endtask

  task process_B();
    int value;
    string name;
    mb.get(value);
    $display("Retrieved data = %0d", value);
    mb.get(name);
    $display("Retrieved data = %s", name);
  endtask
  
  initial begin
    fork
      process_A();
      process_B();
    join
  end
endmodule


----------Unbounded  mailbox--------------

module mailbox_example();
  mailbox mb = new();
  
  task process_A();
    int value;
    repeat(10) begin
      value = $urandom_range(1, 50);
      mb.put(value);
      $display("Put data = %0d", value);
    end
    $display("----------------------");
  endtask

  task process_B();
    int value;
    repeat(10) begin
      mb.get(value);
      $display("Retrieved data = %0d", value);
    end
  endtask
  
  initial begin
    fork
      process_A();
      process_B();
    join
  end
endmodule



// //--------------Parameterized mailbox---------------


module mailbox_example();
  mailbox #(string) mb = new(3);
  
  task process_A();
    string name = "Alex";
    mb.put(name);
    $display("Put data = %s", name);
    name = "Robin";
    mb.put(name);
    $display("Put data = %s", name);
  endtask

  task process_B();
    string name;
    mb.get(name);
    $display("Retrieved data = %s", name);
    mb.get(name);
    $display("Retrieved data = %s", name);
  endtask
  
  initial begin
    fork
      process_A();
      process_B();
    join
  end
endmodule



----------------try_get, try_put-----------


module mailbox_example();
  mailbox mb = new(3);
  
  task process_A();
    int value;
    repeat(5) begin
      value = $urandom_range(1, 50);
      
      if(mb.try_put(value))
        $display("successfully try_put data = %0d", value);
      else begin
        $display("failed while try_put data = %0d", value);
        $display("Number of messages in the mailbox = %0d", mb.num());
      end
    end
    $display("---------------------------------------");
  endtask

  task process_B();
    int value;
    repeat(5) begin
      
      if(mb.try_get(value))
        $display("Successfully retrieved try_get data = %0d", value);
      else begin
        $display("Failed in try_get data");
        $display("Number of messages in the mailbox = %0d", mb.num());
      end
    end
  endtask
  
  initial begin
    fork
      process_A();
      process_B();
    join
  end
endmodule


// //------------------peek and try_peek example--------------------


module mailbox_example();
  mailbox mb = new(3);
  
  task process_A();
    int value;

    repeat(3) begin
      value = $urandom_range(1, 50);
      mb.put(value);
      $display("put data = %0d", value);
    end
    $display("----------------------------------");

  endtask

  task process_B();
    int value;

    mb.peek(value); // message is not removed
    $display("peek data = %0d", value);
    mb.peek(value); // message is not removed
    $display("peek data = %0d", value);

    if(mb.try_peek(value))
      $display("Successful try_peek data = %0d", value);
    else begin
      $display("Failed in try_peek");
    end

    $display("----------------------------------");
    repeat(3) begin
      mb.get(value);
      $display("get data = %0d", value);
    end
   $display("----------------------------------");

   if(mb.try_peek(value))
      $display("Successful try_peek data = %0d", value);
    else begin
      $display("Failed in try_peek");
    end
  endtask
  
  initial begin
    fork
      process_A();
      process_B();
    join
  end
endmodule

///_________________Bounded Generic Mailbox___________________ 


module mailbox_example();
  
  mailbox mb = new(3);
  
  task process_A();
    
    int value = 5;
    string name = "STRING" ;
    mb.put(value);
    $display("PUT data = %0d", value);
    mb.put("STRING");
    $display("PUT data = %0s", name);
    
  endtask
  
  task process_B();
    
    int value;
    string name;
    mb.get(value);
    $display("Retrieved data = %0d", value);
    mb.get(name);
    $display("Retrieved data = %0d", name);
    
  endtask
  
  initial begin
    fork
      process_A();
      process_B();
    join
  end
endmodule



///_________________Parameterized mailbox_____________________


module mailbox_example();
  
  mailbox #(string) mb = new(3);
  
  task process_A();
    
    string name = "ALEX" ;
    mb.put(value);
    $display("PUT data = %0d", value);
    
    name = "ROBIN"
    mb.put("STRING");
    $display("PUT data = %0s", name);
    
  endtask
  
  task process_B();
    

    string name;
    mb.get(value);
    $display("Retrieved data = %0d", value);
    
    mb.get(name);
    $display("Retrieved data = %0d", name);
    
  endtask
  
  initial begin
    fork
      process_A();
      process_B();
    join
  end
endmodule

```
