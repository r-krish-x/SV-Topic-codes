## Implication constraint
```systemverilog
//_________________________Implication Constraint____________________


class ABC;
  rand bit [2:0] mode;
  rand bit [3:0] len;
  
  constraint c_mode { mode == 2  ->  len > 10;} 
endclass

module tb;
  
  initial begin
    
    ABC abc = new;
    for(int i = 0; i < 10; i++) begin
      abc.randomize();
      $display("mode = %0d   len = %0d", abc.mode, abc.len);
    end
  end
endmodule


// _______________________Implication operator________________________


class ABC;
  rand bit [3:0] mode;
  rand bit 		 mod_en;
  
  constraint c_mode{ mode inside {[4'h5 : 4'hb]}  -> mod_en == 1;}
endclass

module tb;
  
  initial begin
    
    ABC abc = new;
    for (int i = 0; i < 10; i++) begin
      abc.randomize();
      $display("mode = 0x%0h  mod_en = 0x%0h", abc.mode, abc.mod_en);
    end
  end
  
endmodule


// ________________________if-else constraint______________________


class ABC;
  rand bit [3:0] mode;
  rand bit 		 mod_en;
  
  constraint c_mode { 
    
    if(mode inside{[4'h5:4'hb]})
      mod_en == 1;
    
    else{
      if(mode == 4'h1){
      mod_en == 1;
      				  }
        else{ mod_en == 0;}
    
    	}
  
  					}
          
          endclass
          
         module tb;
           
           ABC abc;
           
           initial begin
              abc = new;
             for (int i = 0; i <10 ; i++) begin
               abc.randomize();
               $display("mode = 0x%0h  mod_en = 0x%0h", abc.mode, abc.mod_en);
             end
           end
           
         endmodule
       ```        

## Inside Constraint
```
//_____________________when used in conditional statements______________________


module tb;
  bit [3:0] m_data;
  bit 		flag;
  
  initial begin
    
    for(int i = 0; i < 10; i++) begin
      m_data = $random;
      
      flag = m_data inside {[4:9]} ? 1:0;
      
      if (m_data inside{[4:9]})
        $display("m_data = %0d INSIDE [4:9], flag = %0d", m_data, flag);
      else
        $display("m_data = %0d outside [4:9], flag = %0d", m_data, flag);
      
    end
    
  end
endmodule



// ________________________When used in constraints_____________________________


class ABC;
  
  rand bit [3:0] m_var;
  
  constraint c_var {m_var inside{[3:7]};}
endclass

module tb;
  
  initial begin
    
    ABC abc = new();
    
    repeat(5) begin
      abc.randomize();
      
      $display("abc.m_var = %0d", abc.m_var);
    end
    
  end
endmodule


// _________________________inverted inside_____________________________________


class ABC;
  
  rand bit [3:0] m_var;
  
  constraint c_var{!(m_var inside {[3:7]});}
  
endclass

module tb;
  
  initial begin
    
    ABC abc = new();
    repeat(5) begin
      
      abc.randomize();
      $display("m_var = %0d", abc.m_var);
    end
    
  end
  
endmodule


// _____________________practical example of inside constraint__________________


class Data;
  rand bit[15:0] m_addr;
  
  constraint c_addr{m_addr inside {[16'h4000 : 16'h4fff]};}
  
endclass

module tb;
  initial begin
    
    Data data =  new();
    
    repeat(5) begin
      data.randomize();
      $display("m_addr = 0x%0h", data.m_addr);
    end
    
  end
  
endmodule
```
