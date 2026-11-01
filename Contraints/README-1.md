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
               
