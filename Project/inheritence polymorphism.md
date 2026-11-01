## inheritenec & polymorphism

```systemverilog
//_________________________ Behavior of Inherited class____________________________________

class baseclass;
  bit[15:0] data;
  
  function new();
    data = 16'hface;
  endfunction   
endclass

class subclass extends baseclass;
  bit[3:0] id;
  bit[2:0] mode = 3;
  
  function new(int val = 2);
    
//     super.new();
    super.new;
    
    id = val;
    
  endfunction
  
endclass

module tb;
  initial begin
    
    subclass sc1, sc2;
    
//     sc1 = new();
    
    sc1 = new;
 
    $display("data = 0x%0h -> id = %0d -> mode = %0d", sc1.data, sc1.id, sc1.mode);
    
    sc2 = new(4);
    $display("data = 0x%0h -> id = %0d -> mode = %0d", sc2.data, sc2.id, sc2.mode);
    
  end
endmodule
    
    

// __________________________Inheritence_________________________________________________

// .......................same class............................

class Packet;
  int addr;
  
  function new(int addr);
    this.addr = addr;
  endfunction
  
  function display();
    $display("[Base] addr = 0x%0h", addr);
  endfunction
endclass
  
  class Extpacket extends Packet;
    
    int data;
    
    function new(int addr, data);
      super.new(addr);
      this.data = data;
    endfunction
    
    function display();
      $display("[Child] addr = 0x%0h -> data = 0x%0h", addr, data);
    endfunction
  endclass

// ......................same class........................................
  module tb;
    Packet bc;
    Extpacket sc;
    
    initial begin
      bc = new(32'hface_cafe);
      bc.display();
      
      sc= new(32'hface_cafe, 32'h1234_5678);
      sc.display();
      
    end
  endmodule



// ____________________________________Polymorphism_________________________________________________
// -------------------------Assign child class to base class----------------------------------------

// .................................Same class as above..................................
class Packet;
  int addr;
  
  function new(int addr);
    this.addr = addr;
  endfunction
  
  function display();
    $display("[Base] addr = 0x%0h", addr);
  endfunction
endclass
  
  class Extpacket extends Packet;
    
    int data;
    
    function new(int addr, data);
      super.new(addr);
      this.data = data;
    endfunction
    
    function display();
      $display("[Child] addr = 0x%0h -> data = 0x%0h", addr, data);
    endfunction
  endclass
// .................................Same class as above..................................

module tb;
	Packet      bc; 	
	Extpacket   sc; 	

	initial begin
		sc = new (32'hfeed_feed, 32'h1234_5678);

		// Assign sub-class to base-class handle
    
		bc = sc;

//   -> This copies the address held in sc into bc.
//   -> both bc and sc point to the exact same object in memory.
//   -> bc is strictly a "Packet" type handle. It cannot see the data variable inside that object. It acts like a mask, hiding the child's features.
      
      
		bc.display ();
		sc.display ();
	end
endmodule


```

## static variable and function
```systemverilog
// // _________________________Static Variable______________________________________

class Packet;
  bit[15:0] addr;
  bit[7:0] data ;
  static int static_ctr = 0;  // static variable
  		 int ctr = 0;		// non static variable
  
  function new (bit[15:0] ad, bit [7:0] d);
    addr = ad;
    data =d;
    static_ctr ++;		// in output this will remember the previous value
    ctr ++;				// in output this will not remember the previous value and
    				->	// start with 0 -> 1
    $display("static_ctr = %0d -> ctr = %0d -> addr = 0x%0h -> data = 0x%0h", static_ctr, ctr, addr, data);
    
  endfunction
endclass

module tb;
  initial begin
    Packet p1, p2, p3;
    
    p1 = new(16'hdead, 8'h12);
    p2 = new(16'hface, 8'hab);
    p3 = new(16'hcafe, 8'hfc);
    
  end
  
endmodule


// __________________________Static Function_________________________________________________

class Packet;
  static int ctr = 0;
  
  function new();
    ctr ++;
  endfunction
  
  static function get_pkt_ctr();
    $display("ctr = %0d", ctr);
  endfunction
  
endclass


module tb;
  Packet pkt[6];
  
  initial begin
    for(int i = 0; i < $size(pkt) ; i++ ) begin
      pkt[i] = new ;
    end
    
    Packet::get_pkt_ctr();

    pkt[5].get_pkt_ctr();
/////Since get_pkt_ctr is defined as static, it does not look inside the specific object pkt[5]. It ignores the specific data of object #5. Instead, the simulator just checks the type of pkt[5], whether it is pkt[3] or anything, it sees that it is a Packet, and goes directly to the shared Packet class memory to run the function
    
  end 
  
endmodule



//------------------> Let's add in a non-static member called mode and try to call that from our static function.

class Packet;
  static int ctr = 0;
  bit[1:0] mode;
  
  function new();
    ctr ++;
  endfunction
  
  static function get_pkt_ctr();
    $display("ctr =%0d  -->  mode = %0d", ctr, mode);
  endfunction
  
endclass

// this code will throw an error bcoz mode is non-static and it is called from static function

```

## parameterized class

```systemverilog
// __________________Parameterized Class_______________________________
class something#(int size = 8);
  bit[size-1:0] out;
endclass
module tb;
  
  something #(16) sth1;
  something #(.size(32)) sth2;
  typedef something #(4) td_nibble;
  td_nibble nibble;
  
  initial begin
    
    sth1 = new;
    sth2 = new;
    nibble = new;
    
    $display(" sth1.out	  = %0d bits", $bits(sth1.out));
    $display(" sth2.out   = %0d bits", $bits(sth2.out));
    $display(" nibble.out = %0d bits", $bits(nibble.out));
  end
  
endmodule


// __________________Pass data types as parameter________________________

class stack #(type T = int);
  T item;
  
  function T add_a(T a);
    return item + a;
  endfunction
  
endclass


module tb;
  
  stack				st;
  stack #(bit[3:0])	bs;
  stack #(real)		rs;
  
  initial begin
    
    st = new;
    bs = new;
    rs = new;
    
    st.item = -256;
    $display("st.item = %0d", st.add_a(10));
    
    bs.item = 8'ha1;
    $display("bs.item = %0d", bs.add_a(16));
    
    rs.item = 3.14;
    $display("rs.item = %0.2f", rs.add_a(10));
//      $display("rs.item = %0.f", rs.add_a(10));
  end
 
endmodule


```
