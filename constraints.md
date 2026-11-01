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
```systemverilog
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

## Inline Constraint

```systemverilog
// ____________________inline constraints_______________________________________


class item;
  rand bit[7:0] id;
  
  constraint c_id{id < 25;}
  
endclass

module tb;
  
  initial begin
    
    item itm = new();
    
    itm.randomize() with { id == 10;};
    
    $display("Item Id = %0d", itm.id);
    
  end
  
endmodule


// ------- conflicting constraint-------------

class item;
  
  rand bit[7:0] id;
  
  constraint c_id { id ==25;}
  
endclass

module tb;
  
  initial begin
    item itm = new();
    if (! itm.randomize() with {id < 10;})
      $display("Randomization failed");
    
    $display("Item ID= %0d", itm.id);
    
  end
endmodule


```

## Solve before constraint

```systemverilog
// _______________________without solve before_____________________________

class ABC;
  
  rand bit 	a;
  rand bit [1:0] b;
  
  constraint c_ab{ a-> b ==3;}
endclass

module tb;
  initial begin
    
    ABC abc = new();
    for (int i =0 ; i < 8; i++ ) begin
      
      abc.randomize();
      $display("a = %0d  b=%0d", abc.a, abc.b);
    end
  end
  
endmodule



// __________________with solve before_______________________________________


class ABC;
  
  rand bit 	a;
  rand bit [1:0] b;
  
  constraint c_ab{ a-> b ==3; solve a before b;}
endclass

module tb;
  initial begin
    
    ABC abc = new();
    for (int i =0 ; i < 8; i++ ) begin
      
      abc.randomize();
      $display("a = %0d  b=%0d", abc.a, abc.b);
    end
  end
  
endmodule

```

## Foreach Constraint

```systemverilog
// _________________________foreach constraint_______________________________

// ----------foreach constraint on dynamic & queue array---------------------

class ABC;
  rand bit[3:0] darray[];
  rand bit[3:0] queue[$];
  
  constraint c_qsize {queue.size() == 5;}
  
  constraint c_darry {foreach (darray[i])
   						 darray[i] == i;
                      foreach(queue[i])
                        queue[i] == i +1;
                     }
  
  function new();
    darray = new[5];
  endfunction
endclass

module tb;
  initial begin
    ABC abc = new();
    abc.randomize();
    $display("darray = %p, queue = %p", abc.darray, abc.queue);
  end
endmodule


// ___________________________multidimensional array__________________________________

class ABC;
  rand bit[4:0][3:0] md_array[3][5];
  
  constraint c_md_array{
    
    foreach (md_array[i]) {
      foreach(md_array[i][j]){
        foreach(md_array[i][j][k]){
      if (k % 2 ==0)
        md_array[i][j][k] == 'hf;
    else
      md_array[i][j][k] == 0;
        						}
      						 }
  						  } 
 					   }
          
          endclass
          
          module tb;
            
            initial begin
              ABC abc = new;
              abc.randomize();
              $display("md_array = %p", abc.md_array);
              
            end
            
          endmodule


// __________________________multidimensional dynamic array_______________________________
// --------------dynamic array of dynamic array---------------------

class ABC;
  
  rand bit[3:0] md_array[][];
  
  constraint c_md_array{
    md_array.size() ==  2;
  
    foreach(md_array[i]){
      md_array[i].size() inside {[1:5]};
    
      foreach(md_array[i][j]){
        md_array[i][j] inside{[1:10]};
      }
    }
  
  }
        
        endclass
        
        module tb;
          
          initial begin
          ABC abc = new;
          abc.randomize();
          $display("md_array = %p", abc.md_array);
          end
          
        endmodule
      
```

## Static Constraint

```systemverilog
// // ______________________________static constraints________________________________

// // -----------------turn-off the non static constraint-----------------------------

class ABC;
  
  rand bit[3:0] a;
  
  constraint c1 { a > 5;}
  static constraint c2{a < 12;}
  
endclass

module tb;
  initial begin
    
    ABC obj1 = new;
    ABC obj2 = new;
    
    obj1.c1.constraint_mode(0);
    
    for (int i = 0; i < 5; i++) begin
      obj1.randomize();
      obj2.randomize();
      
      $display("obj1.a = %0d, obj2.a = %0d", obj1.a, obj2.a);
    end
  end
endmodule

// For obj1: The rule a > 5 is gone. It can generate values like 0, 1, 2, etc.

//For obj2: The rule a > 5 is still active. It was not touched.


// // ------------------------turn-off the static constraint----------------------


class ABC;
  rand bit[3:0] a;
  
  constraint c1{a >5;}
  static constraint c2 {a < 12;}
  
endclass

module tb;
  initial begin
    
    ABC obj1 = new;
    ABC obj2 = new;
    
    obj1.c2.constraint_mode(0);
    
    for(int i = 0; i <5 ; i++)begin
      obj1.randomize();
      obj2.randomize();
      
      $display("obj1.a = %0d, obj2.a = %0d", obj1.a, obj2.a);
    end
  end
endmodule


//For obj1: The rule a < 12 is disabled.

//For obj2: The rule a < 12 is ALSO disabled automatically.
```

## Disable constraint

```systemverilog
// __________________________________disable constraint__________________________________
// ------constraint on------

class fruits;
  rand bit[3:0] num;
  constraint c_num{ num > 4;
                   num < 9;};
endclass

module tb;
  
  initial begin
    fruits f = new;
    
    $display("Before randomization num = %0d", f.num);
    
    
    if(f.c_num.constraint_mode())
      $display("Constraint is enabled");
    else
      $display("Constraint is disabled");
    
    f.randomize();
    
    $display("After randomization num = %0d", f.num);
    
    
  end
  
endmodule
  

// -----constrain off-------


class fruits;
  rand bit[3:0] num;
  constraint c_num{ num > 4;
                   num < 9;};
endclass

module tb;
  
  initial begin
    fruits f = new;
    
    $display("Before randomization num = %0d", f.num);
    
    f.c_num.constraint_mode(0);
    
    if(f.c_num.constraint_mode())
      $display("Constraint is enabled");
    else
      $display("Constraint is disabled");
    
    f.randomize();
    
    $display("After randomization num = %0d", f.num);
    
    
  end
  
endmodule
```

## Constraint Example from Chipverify

```systemverilog
// _________________________simple example______________________________________


class myclass;
  rand bit [3:0] min, max, typ;
  rand bit [3:0] fixed;
  
  constraint my_range { 3 < min;
                       typ < max;
                       typ> min;
                       max < 14;}
  
  constraint c_fixed {fixed == 5;}
  
  function string display();
    return $sformatf("min = %0d  typ = %0d  max =%0d  fixed = %0d", min, typ, max, fixed);
  endfunction
  
endclass

module tb;
  initial begin
    
    for(int i = 0; i<10; i++) begin
      myclass cls = new;
      cls.randomize();
      $display("itr = %0d %s", i, cls.display());
    end
  end
endmodule



// ____________________________Weighted distribution____________________________

//  " := " operator 


class myclass;
  
  rand bit[2:0] typ;
  constraint dist1 { typ dist {0 := 20, [1:5]:= 50, 6:=40, 7:=10};}
endclass

module tb;
  
  initial begin
    for(int i = 0; i < 10; i++)begin
      myclass cls = new();
      cls.randomize();
      $display("itr = %0d  typ = %0d", i, cls.typ);
    end
  end
endmodule



//  " :/ " operator

class myclass;
  rand bit[2:0] typ;
  constraint dist2{ typ dist { 0:/20, [1:5]:/50, 6:/40, 7:/10};}
endclass

module tb;
  initial begin
    for (int i = 0; i<10; i++) begin
      myclass cls = new();
      cls.randomize();
      $display("itr = %0d  typ = %0d", i, cls.typ);
 
    end
  end
  
endmodule



// __________________________Bidirectional constraint______________________

class myclass;
  rand bit [3:0] val;
 
  constraint c1{val > 3; val <12;}
  
  constraint c2{val >= 10;}
  
endclass

module tb;
  
  initial begin
    
    for (int i = 0; i < 10; i++) begin
      myclass cls = new();
      cls.randomize();
      $display("itr = %0d  typ = %0d",i, cls.val);
    end
    
  end
  
endmodule




// __________________Implicit and Explicit constraints_________________________


class ABC;
  rand bit[3:0] mode;
  
 		 constraint c_implicit;		// implicit
  extern constraint c_explicit;		// explicit
endclass;


constraint ABC:: c_implicit { mode > 2;};
constraint ABC:: c_explicit { mode <=6;};

module tb;
  ABC abc;
  
  initial begin
    abc = new();
    
    for(int i = 0; i <5 ; i++)begin
      abc.randomize();
      $display("mode = %0d ", abc.mode);
    end
  end
endmodule

// Implicit : in this constraint after declaring an empty constraint inside class, even if you dont define constraint body outside the class it doesnt throw an error.

// Explicit : in this constraint after declaring an empty constraint using "extern" inside class, its guarantee that outside the class this constraint body will defined. or it will throw an error 

```
