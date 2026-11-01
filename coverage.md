## coverage 1
```systemverilog
///_______________________________covergroup without event

///						Design Code

module top
  (
    input  [1:0]  a,
    output [1:0]  b  
  );
  
assign b = a;
  
  
endmodule
//////////////Testbench Code



module tb;
 
  reg  [1:0]  a;
  wire [1:0]  b;
  integer i = 0;
  
  top dut (.a(a), .b(b));
  
  
  covergroup cvr_a ; ///// manual sample method
    
    coverpoint a; //// automaitc bins  
    
    coverpoint b;
  
  endgroup 
  
 
  cvr_a ci = new();
 
  
  initial begin
    
    
    for (i = 0; i <10; i++) begin
      a = $urandom();  
      ci.sample();
      
      #10;
    end
    
  end
  
  
  initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    #500;
    $finish();
  end
 
endmodule




// ___________________ covergroup with event____________________________




//////////////////////Design Code

module top
  (
    input  [1:0]  a,
    output [1:0]  b  
  );
  
assign b = a;
  
  
endmodule
//////////////Testbench Code



module tb;
 
  reg  [1:0]  a;
  wire [1:0]  b;
  integer i = 0;
  
  top dut (.a(a), .b(b));
  
  
  covergroup cvr_a @(a); ///// manual sample method
    
    coverpoint a; //// automaitc bins  
    
    coverpoint b;
  
  endgroup 
  
 
  cvr_a ci = new();
 
  
  initial begin
    
    
    for (i = 0; i <10; i++) begin
      a = $urandom();    
      #10;
    end
    
    
  end
  
  
  
  
  initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    #500;
    $finish();
  end
 
endmodule





// ________________________coverage option______________________________


// /////////////////////////////////////Design Code

module top
  (
    input  [1:0]  a,
    output [1:0]  b  
  );
  
assign b = a;
  
  
endmodule


///////////////////////Testbench Code



module tb;
 
  reg  [1:0]  a; //00 01 10 11
  wire [1:0]  b;
  integer i = 0;
  
  top dut (.a(a), .b(b));
  
  
  covergroup cvr_a ; ///// manual sample method
    
    option.per_instance = 1;
    option.name = "COVER_A_B";
    option.goal = 70;
    option.at_least = 4;
    option.auto_bin_max = 2;
    
    coverpoint a; //// automaitc bins  
    
    coverpoint b;
  
  endgroup 
  
 
  cvr_a ci = new();
 
  
  initial begin
    
    
    for (i = 0; i <10; i++) begin
      a = $urandom();  
      ci.sample();
      
      #10;
    end
      end
  

  initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    #500;
    $finish();
  end

endmodule


    //////  for bin auto[2:3] we get 2 times covered, even though value 2 for b signal appeared 3 times in waveform, CLEAR THIS DOUBT LATER
    
//      COVERGROUP COVERAGE
// #     ============================================================
// #     |       Covergroup        |  Hits   |  Goal /  |  Status   |
// #     |                         |         | At Least |           |
// #     ============================================================
// #     | TYPE /tb/cvr_a          | 50.000% | 100.000% | Uncovered |
// #     ============================================================
// #     | INSTANCE COVER_A_B      | 50.000% |  70.000% | Uncovered |
// #     |-------------------------|---------|----------|-----------|
// #     | COVERPOINT COVER_A_B::a | 50.000% | 100.000% | Uncovered |
// #     |-------------------------|---------|----------|-----------|
// #     | bin auto[0:1]           |       7 |        4 | Covered   |
// #     | bin auto[2:3]           |       3 |        4 | Uncovered |      <--- here 3 times
// #     |-------------------------|---------|----------|-----------|
// #     | COVERPOINT COVER_A_B::b | 50.000% | 100.000% | Uncovered |
// #     |-------------------------|---------|----------|-----------|
// #     | bin auto[0:1]           |       7 |        4 | Covered   |
    // #     | bin auto[2:3]           |       2 |        4 | Uncovered |	<--- here hit 2 times
// #     ============================================================






// ______________________________weightage coverage________________


module top
   (
     input  [1:0]  a,
     output [1:0]  b  
   );

 assign b = a;


endmodule

module tb;
 
  reg [1:0] a;
  reg [1:0] b;
  integer i = 0;
  
 
 covergroup c;
   option.per_instance = 1;
   
   coverpoint a
   {
   option.weight = 5;
   }
   
   
   coverpoint b{
   option.weight = 3;
   }
   
 endgroup
 
 
 
 
  initial begin
    c ci = new();
    
    
    
    for (i = 0; i <5; i++) begin
      a = $urandom();
      b = $urandom();
      ci.sample();
      #10;
    end
    
    
  end
  
  
  
  
  initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    #200;
    $finish();
  end
  
 
 
 
 
endmodule




//_______________________example for change option for covegroup, coverpoint, coverbins___________________________


 module top
    (
      input  [1:0]  a,
      output [1:0]  b  
    );

  assign b = a;


 endmodule

module tb;
 
  reg  [1:0]  a;
  reg [1:0]  b;
  integer i = 0;
  
  
  covergroup cvr_a ; ///// manual sample method
    option.per_instance = 1;
    
    
    //////////////////
    ///////////////////////  ASK AI LATER - Elaborate between below two options
    option.goal = 62;
    type_option.goal = 62;
    
    coverpoint a{
    option.goal = 50;
    }
    
    coverpoint b{
    option.goal = 75;
    }
 
  
  endgroup 
  
 
  cvr_a ci = new();
 
  
  initial begin
    
    
    for (i = 0; i < 5; i++) begin
      a = $urandom();  
      b = $urandom();
      ci.sample();
      #10;
    end

  end
  
  initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    #500;
    $finish();
  end
 
endmodule





//___________________________disable coverage_____________________________________

  module top
     (
       input  [1:0]  a,
       output [1:0]  b  
     );

   assign b = a;


  endmodule

module tb;
  reg [1:0] a = 0;
  reg rst = 0;
  integer i = 0;
 
  initial begin
    rst = 1;
    #30;
    rst = 0;
  end
  
  covergroup c;
    option.per_instance = 1;
    coverpoint a iff (!rst); 
  endgroup
  
  initial begin
    c ci = new();
    for(i = 0; i < 10; i++) begin
      a = $urandom();
      ci.sample();
      #10;
    end
  end
  
  initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    #100;
    $finish();
  end
  
endmodule

```

## coverage 2

```systemverilog
//______________________implicit bins____________________________

module tb;
 
  reg  [7:0]  a;
  reg  [7:0]  b;    //256/4 = 64 bins , ,,,,,, each bin of rang 4       
  
  
covergroup cvr_a ;
  
    option.per_instance = 1;
    
  coverpoint b;
    
  coverpoint a {
   option.auto_bin_max = 256;
  }
//     option.auto_bin_max = 64;             ---> default value if not explicitly declared ""64""
 endgroup 
  
 
  cvr_a ci = new();
 
  
  initial begin
    
    
    for (int i = 0; i < 5; i++) begin
      a = $urandom(); 
      b = $urandom();
      ci.sample();
      #10;
    end
    
    
  end
  
  
  
  
  initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    #500;
    $finish();
  end
 
endmodule






// _____________________________________explicit bins_________________________________________________


module tb;
 
  reg  [1:0]  a; /// 00 01 10 11
 
covergroup cvr_a ;
  
    option.per_instance = 1;
    
    
  coverpoint a {
//    bins zero = {0};
//    bins one  = {1};
//    bins two  = {2};
//    bins three = {3};
    
//    bins bin0 = {0,1};  // {[0:1]}
//    bins bin1 = {[2:3]}; // {2,3}
    
    bins bina = {[0:3]};
  
  } 
    
 endgroup 
  
 
  cvr_a ci = new();
 
  
  initial begin
    
    
    for (int i = 0; i < 5; i++) begin
      a = $urandom(); 
      ci.sample();
      #10;
    end
    
    
  end
  
  
  
  
  initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    #500;
    $finish();
  end
 
endmodule




///__________________________________________2. explicit bin______________________________ 


module tb;
 
  reg  [6:0]  a; /// 00 01 10 11
 
covergroup cvr_a ;
  
    option.per_instance = 1;
    
    
  coverpoint a {
 //   bins zero = {0};
 //   bins one  = {1};
 //   bins two  = {2};
 //   bins three = {3};
    
  //  bins bin0 = {0,1};  // {[0:1]}
  //  bins bin1 = {[2:3]}; // {2,3}
    
   // bins bina = {[0:3]};
    
    bins bina[64] = {[0:127]};
  
  } 
    
 endgroup 
  
 
  cvr_a ci = new();
 
  
  initial begin
    
    
    for (int i = 0; i < 5; i++) begin
      a = $urandom(); 
      ci.sample();
      #10;
    end
    
    
  end
  
  
  
  
  initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    #500;
    $finish();
  end
 
endmodule



_________________________________________4) default bins_________________________________________________


module tb;
  reg [3:0] a;  /// 0 -15
 
  
 
  integer i = 0;
  
  
  initial begin
    #100;
    $finish();    
  end
  
 covergroup c;
   
   option.per_instance = 1;
   coverpoint a {
     bins a_values[] = {[0:9]}; 
     bins a_unused = default;
   }
   
   
   
 endgroup
 
    
    
  
  initial begin
    c ci = new();
    for(i = 0; i < 30; i++) begin 
      a = $urandom();
      ci.sample();#1;
    end 
  end
 
  
    initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    #500;
    $finish();
  end
  
endmodule



// // __________________________________mux_________________________________________

// /////////////////////////////////Design Code

module top 
  (
    input a,b,c,d,
    input [1:0] sel,
    output reg y
  );
  
  always@(*)
    begin
      case(sel)
        2'b00: y = a;
        2'b01: y = b;
        2'b10: y = c;
        2'b11: y = d;
        default: y = 1'b0;
      endcase
    end
    
  
endmodule


// ////////////////////////// TB Code



module tb;
 
  reg a,b,c,d;
  reg [1:0] sel;
  wire y;
  
  top dut (a,b,c,d,sel,y);
  
  covergroup cvr_mux;
    
    option.per_instance = 1;
    
    coverpoint a 
    {
     bins a_values[] = {0,1};
    }
    
    
    coverpoint b 
    {
     bins b_values[] = {0,1};
    }
    
    coverpoint c 
    {
     bins c_values[] = {0,1};
    }
    
    coverpoint d 
    {
     bins d_values[] = {0,1};
    }
    
    coverpoint sel 
    {
      bins sel_values[] = {0,1,2,3};
    }
    
    coverpoint y 
    {
     bins y_values[] = {0,1};
    }
     
    
endgroup
 
  
 
  cvr_mux ci = new();
 
  
  initial begin
    
    
    for (int i = 0; i < 20; i++) begin
      a = $urandom(); 
      b = $urandom();
      c = $urandom(); 
      d = $urandom();
      sel = $urandom();
      ci.sample();
      #10;
    end
    
    
  end
  
  
  
  
  initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    #500;
    $finish();
  end
 
endmodule



// ______________________________________FSM example__________________________________________


/////////////////////Design Code

module fsm(input clk, input reset, input din, output reg dout);
 
  reg state, nstate;
  
  parameter S0=0, S1=1;
  
  always @(posedge clk or posedge reset) // always block to update state
      if (reset)
 	state <= S0;
	else
 	state <= nstate;
  
  		always @(state or din) // always block to compute both output & nextstate
			begin
 				dout = 1'b0;
				 case(state)
   					S0: if(din)
 			            	begin
 				            dout = 1; nstate = S1;
 			                end
                        else
                          nstate = S0;
                    S1: if(din)
                          nstate = S0;
                        else
                         begin
                           dout = 1; nstate = S1;
                         end
                        default:
                          nstate = S0;
               endcase
          end
endmodule 


///////////////Testbench Code





module tb;
  
 reg clk = 0;
  reg reset = 0;
  reg din = 0;
  wire dout;
  
  covergroup c ;
    option.per_instance = 1;
    coverpoint dut.state;
  endgroup
  
  fsm dut (clk,reset,din, dout);
  
  always #5 clk = ~clk;
  
  initial begin
    reset = 1;
    #30;
    reset = 0;
    #40;
    din = 1;
  end
  c cia;
  initial begin
    cia = new();
    forever begin
      @(posedge clk);
      cia.sample();
    end
    
  end
  
 initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    #200;
    $finish();
  end

endmodule



```

## coverage 3

```systemverilog
// // _______________________________Illegal Bins_______________________________________________

module tb;
  
  reg [2:0] opcode;
  reg [2:0] a,b;
  reg [3:0] res;
  integer i = 0;
 
  always_comb
    begin
  case (opcode)
    0: res = a + b;
    1: res = a - b;
    2: res = a;
    3: res = b;
    4: res = a & b;
    5: res = a | b;
    default : res = 0;
  endcase
    end
  
  covergroup c;
    option.per_instance  = 1;
    
    coverpoint opcode {
    
      bins valid_opcode[] = {[0:5]};
      illegal_bins invalid_opcode[] = {6,7};
    
    }
    
    
    
  endgroup
  
 
  c ci;
 
  initial begin
     ci = new();
    
    
    
    for (i = 0; i <40; i++) begin
      a = $urandom();
      b = $urandom();
      opcode = $urandom();
      ci.sample();
      #10;
    end
    
    
  end
  
  
  
  
  initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    #400;
    $finish();
  end
  
endmodule




_____________________________ignore bins_____________________________________________________


module tb;
  
  reg [2:0] opcode;
  reg [2:0] a,b;
  reg [3:0] res;
  integer i = 0;
 
  always_comb
    begin
  case (opcode)
    0: res = a + b;
    1: res = a - b;
    2: res = a;
    3: res = b;
    4: res = a & b;
    5: res = a | b;
    default : res = 0;
  endcase
    end
  
  covergroup c;
    option.per_instance  = 1;
    
    coverpoint opcode {
    
      bins valid_opcode[] = {[0:5]};
      ignore_bins invalid_opcode[] = {6,7};
    
    }
    
    
    
  endgroup
  
 
  c ci;
 
  initial begin
     ci = new();
    
    
    
    for (i = 0; i <40; i++) begin
      a = $urandom();
      b = $urandom();
      opcode = $urandom();
      ci.sample();
      #10;
    end
    
    
  end
  
  
  
  
  initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    #400;
    $finish();
  end
  
 
 
 
 
endmodule



// __________________________________________wild card bins____________________________________________

   ////////////////////////////Design Code



module penc(
  input [3:0]y,
  output reg [1:0] a
);
  
  always@(*)
  begin
   
  casez(y)
  4'b0001:a = 2'b00;
  4'b001?:a = 2'b01; // 0010 0011
  4'b01??:a = 2'b10; // 0100 0101 0110 0111
  4'b1???:a = 2'b11; // 1000 1001 1010 1011
 
  default:a = 2'bzz;
  endcase
 end
  
endmodule


/////////////////////////// Testbench Code



module tb;
  
  reg [3:0] y;
  wire [1:0] a;
  integer i = 0;
  
  penc dut (y,a);
  
  covergroup c;
    option.per_instance = 1;
    
    coverpoint y {
      bins zero = { 4'b0001};
      wildcard bins one =   {4'b001?};
      wildcard bins two =   {4'b01??};
      wildcard bins three = {4'b1???};   
    }
    
    coverpoint a; 
    
    
  endgroup
  
  c ci;
  
   
  
  initial begin
    ci = new();
    for(i = 0; i<15; i++) begin 
      y = $urandom();
      $display("value of y : %04b",y);
      ci.sample();
      #10;  
    end
    
  end
  
  
  initial begin
  #3000;
  $finish();
  end
  
  
  
   initial begin
   $dumpfile("dump.vcd"); 
   $dumpvars;
   #400;
   $finish();
     end
  
  
  
endmodule



////_________________________________wild card bins__________________________________________________


/////////////////////////////// Design Code

module penc(
  input [3:0]y,
  output reg [1:0] a
);
  
  always@(*)
  begin
   
  casez(y)
  4'b0001:a = 2'b00;
  4'b001?:a = 2'b01; // 0010 0011
  4'b01??:a = 2'b10; // 0100 0101 0110 0111
  4'b1???:a = 2'b11; // 1000 1001 1010 1011
 
  default:a = 2'bzz;
  endcase
 end
  
endmodule
 
///////////////////////////// Testbench Code

module tb;
  
  reg [3:0] y;
  wire [1:0] a;
  integer i = 0;
  
  penc dut (.y(y),.a(a));
  
  covergroup c;
    option.per_instance = 1;
    
    coverpoint y {
      bins zero = { 4'b0001};
      wildcard bins one =   {4'b001?};
      wildcard bins two =   {4'b01??};
      wildcard bins three = {4'b1???};   
    }
    
    coverpoint a
    {
      bins undef = {/*2'bxx, */2'bx0, 2'b0x, 2'b1x, 2'bx1,2'bzz, 2'bz0, 2'b0z, 2'b1z, 2'bz1};
      bins valid[] = {0,1,2,3};
      
    }
    
    
  endgroup
  
  c ci;
  
   
  
  initial begin
    ci = new();
    for(i = 0; i<15; i++) begin 
      y = $urandom();
      $display("value of y : %04b and a : %02b @%0t ",y, a , $time);
      ci.sample();
      #10;  
    end
    
  end
  
  
  initial begin
  #3000;
  $finish();
  end
  
  
   initial begin
   $dumpfile("dump.vcd"); 
   $dumpvars;
   #400;
   $finish();
 end
  
  
endmodule

//_____________________________inside property____________________________________

// inside property

module tb;
   
  reg rd = 0,wr = 0;
  reg clk = 0;
  reg [4:0] addr;  
  reg [7:0] din;
  reg [7:0] dout;
  
  always #5 clk = ~clk;
 
 
   covergroup c with function sample (reg [4:0] addrin);
    option.per_instance = 1;
    coverpoint addrin{
      bins lower = {[0:7]};
      bins mid = {[15:20]};
      bins high = {[27:31]};
    }
    
  endgroup
 
  c ci;
 
  initial begin
     ci = new();
    
    @(posedge clk);
    addr = 3;
    wr = 1;
    rd = 0;
    din = 12;
    @(posedge clk);
    wr = 0;
    rd = 1;
    addr = 3;
    dout = 12;
    @(posedge clk);
    addr = 17;
    wr = 1;
    rd = 0;
    din = 21;
    @(posedge clk);
    wr = 0;
    rd = 1;
    addr = 17;
    dout = 21;
    @(posedge clk);
    addr = 28;
    wr = 1;
    rd = 0;
    din = 67;
    @(posedge clk);
    wr = 0;
    rd = 1;
    addr = 28;
    dout = 67;
    
    
  end
  
  
  
  
 property p1;
  bit [4:0] addrs;
  bit [7:0] dvar;
    
   @(posedge clk) ( wr |-> (wr, addrs = addr, dvar = din, ci.sample(addrs)) ##[1:50] rd[*1:50] ##0 (addrs == addr) ##0 (dout == dvar) );
  endproperty
  
  A1: assert property ( p1) $info("Suc at %0t",$time);
  
  
  initial begin
    $dumpfile("dump.vcd"); 
    $dumpvars;
    $assertvacuousoff(0); 
    #500;
    $finish();
  end
  
 
 
 
 
endmodule

```

## Coverage -> implicit - explicit

```systemverilog
// ___________________implicit bin_____________________________

module func_coverage;

logic [3:0] addr;
logic [2:0] data;
logic en;

covergroup c_group;

option.per_instance=1;

cp1: coverpoint addr;
cp2: coverpoint data;
cp3: coverpoint en;

endgroup

c_group cg = new();

initial
repeat(20)begin

#1 	addr = $random;
	data = $random;
	en = $random;

	cg.sample();
	$display("addr=%0d, data=%Od en=%0d",addr,data,en);

end
endmodule



_______________________explcit bins_____________________


module func_coverage;

logic [3:0] addr;
logic [2:0] data;
logic en;

covergroup c_group;

option.per_instance=1;

  cp1: coverpoint addr {bins b1				 = {1, 10, 12};
                        bins b2[]            = {[2:9], 11};
                        bins zero_to_seven [4] = { [0:7]};}

  cp2: coverpoint data   {bins b1 = {[5:$]};
						bins b2[] = {2, 3, 6};
                        bins others = default;}



  cp3: coverpoint en {bins one ={1};}

endgroup

c_group cg = new();

initial
repeat(20)begin

#1 addr = $random;
data = $random;
en = $random;

cg.sample();
  $display("addr = %0d, data = %0d, en = %0d", addr, data, en);
end
  
endmodule



//__________________covergroup with clocking event_________-___

module tb;

bit [1:0] mode;
bit [2:0] cfg;

bit clk;
always #20 clk = ~clk;

covergroup cg @ (posedge clk);
option.per_instance=1;

cp_mode		: coverpoint mode;
cp_cfg_10	: coverpoint cfg[1:0];
cp_cfg_lsb  : coverpoint cfg[0];
  cp_sum		: coverpoint (mode + cfg);	 // --> (0,1,2,3) + (0,1,2,3,4,5,6,7)
  //  unique values 0 to 3 are already ready there in 0 to 7 so it'll generate 8 bins	 

endgroup

cg cg_inst;

initial begin
cg_inst= new();

for (int i = 0; i < 5; i++) begin
@(negedge clk);
mode = $random;
cfg  = $random;
  $display ("[%0t] mode=0x0h cfg=0x%0h", $time, mode, cfg);
end
end

initial begin
#500 $finish;
end
  
endmodule
```

## transition, ignore, illegal bins
```systemverilog
////________________________Transition bins_____________________


module coverage;

bit[2:0]data;

covergroup c_g;
  
option.per_instance =	1;
  
coverpoint data
  {
	bins b1 = ( 5[*3]);
	bins b2 = ( 1 => 3 [->4:6] => 1);
										//immediately by a 1
    bins b3 = ( 1 => 2 [= 3:6] => 5);
										//later by a 5
	bins b4 = ( 2 [-> 3:5] ); 
	bins b5 = ( 3 [-> 3:5] ); 
  }
endgroup

  
initial begin
  c_g cg= new();

//   for(int i = 0; i <17; i++) begin
//     #1 data = $random;
//     cg.sample();
//   end
  
  
  #1 data = 1;
  cg.sample();
  
  #1 data = 4;
  cg.sample();
  
  #1 data = 3;
  cg.sample();
  
  #1 data = 2;
  cg.sample();
  
  #1 data = 3;
  cg.sample();
  
  #1 data = 3;
  cg.sample();
  
  #1 data = 2;
  cg.sample();
  
  #1 data = 2;
  cg.sample();
  
  #1 data = 3;
  cg.sample();
  
  #1 data = 2;
  cg.sample();
  
  #1 data = 3;
  cg.sample();
  
  #1 data = 1;
  cg.sample();
  
  #1 data = 5;
  cg.sample();
  
  #1 data = 5;
  cg.sample();
  
  #1 data = 5;
  cg.sample();
  
  #1 data = 5;
  cg.sample();
  
  #1 data = 5;
  cg.sample();
  
  #1 data = 5;
  cg.sample();
  
  
end
  
endmodule 



//______________ignore, illegal & wildcard bins________________


module func_coverage;
  
  logic[3:0] addr;
  
  covergroup c_group;
    
    option.per_instance = 1;
    
    cp1 : coverpoint addr {ignore_bins	exclude =	{8,10};
                           illegal_bins	invalid =	{13,14};
                           bins valid[4]		={[0:3]};
                           wildcard bins wc_valid ={3'b1x?};}
  endgroup
  
  c_group cg = new();
  
  initial begin
    repeat(20) begin
      
      #1 addr = $random;
      		cg.sample();
    	  $display("addr = %0d", addr);
      
    end
  end
    
    endmodule
```

## coverage example -- WE_LSI

```systemverilog
/// ___________Example 1_______________________


class packet;
  
  rand bit [7:0]	addr;
  rand bit [6:0]	data;
  rand bit [7:0]	a,b;
  rand bit [3:0]	x,y;
  
  covergroup class_cg with function sample(bit reset);
    option.per_instance = 1;
    
    coverpoint data iff (!reset)
    {
      bins featureA[] = data with (item % 8 == 0);}
      
     data_range : coverpoint data[6:3]
    {bins featureB[] = data_range with (item[4:3] == 0);}
    
    cp_addr : coverpoint addr{option.auto_bin_max = 8;}
    
    coverpoint a
    
    {
      bins a1 = {[0:63]};
      bins a2 = {[64:127]};
      bins a3 = {[128:191]};
      bins a4 = {[192:255]};
    }
    
    coverpoint b
    {
      bins b1 = {0};
      bins b2 ={[1:84]};
      bins b3 = {[85:169]};
      bins b4 = {[170:255]};
    }
    
    c: cross a,b
    {
      bins c1 = ! binsof(a) intersect {[100:200]};
      bins c2 = binsof(a.a2) || binsof(b.b2);
      bins c3 = binsof(a.a1) && binsof(b.b4);
      
    }
    
    
    d : cross x,y;
    
  endgroup
  
  covergroup c_constructs;
    
    option.per_instance = 1;
    
    cp1 : coverpoint a {
    
      bins x1 = {[0:99]};
      bins x2 = {[100:199]};
      bins x3 = {[200:255]};
    }
    
    cp2 : coverpoint b {
    
      bins y1 = {[0:74]};
      bins y2 = {[75:149]};
      bins y3 = {[155:255]}; 
      
    }
    
    binsof_cp1_x_cp2 : cross cp1, cp2 {
    
      bins xy1 = binsof(cp1.x1);
      bins xy2 = binsof(cp2.y2);  
      bins xy3 = binsof(cp1.x1) && binsof(cp2.y2);  
      bins xy4 = binsof(cp1.x1) || binsof(cp2.y2);  
      
    
    
    intersect_cp1_xcp2 : cross cp1, cp2{
    
    
      bins xy1 = binsof(cp1) intersect {[100:200]};
      bins xy2 = !binsof(cp1) intersect {[100:200]};
      bins xy3 = !binsof(cp1) intersect {99,125,150,170};
    }  
    
  endgroup
  
  function new();
    class_cg 	= new();
   c_constructs = new();
      endfunction
      
      function void sample(bit rst);
        class_cg.sample(rst);
        c_constructs.sample();
      endfunction
      
    endclass
    
    
    module class_coverage;
      
      bit reset;
      
      initial begin
        packet p = new();
        
        for (int i = 0; i < 25; i++) begin
          
          if(i%5==0) reset = 0;
          else		 reset = 1;
          
          p.randomize();
          p.sample(reset);
          $display("reset = %0d  addr = %0d  addr = %0d", reset, p.addr, p.data);
        end
      end
      
    endmodule



///______________Example 2________________________



module coverage_options;
  bit[3:0] a_var, b_var;
  
  covergroup g1 (string instComment);
    
    option.per_instance =1 ;
    option.comment		= instComment;
    
    a : coverpoint a_var;
    b : coverpoint b_var;
    
  endgroup
  
  g1 g1_inst = new("instance_1");
  g1 g2_inst = new("instance_2");
  
  
  initial begin
    
    g1_inst.option.name ="Me";
    g2_inst.option.name ="You";
    
    g1_inst.a.type_option.weight = 2;
    
    repeat(10) begin
      
      {a_var, b_var} = {$urandom, $urandom};
      
      g1_inst.sample ();
      g2_inst.sample ();
      
      #1;
      
    end
  end
endmodule




// _____________________________________________________________         same example as above but corrected by gemini
//___________________________________________________________



module coverage_options;
  bit[3:0] a_var, b_var;
  
  // 1. Updated covergroup to accept a weight argument
  covergroup g1 (string instComment, int specific_weight);
    
    option.per_instance = 1;
    option.comment      = instComment;
    
    // option.name is read-only or set via new(). 
    // You cannot manually assign it to "Me" or "You" arbitrarily 
    // after instantiation, but the instance name in new() handles identification.

    a : coverpoint a_var {
      // 2. We use option.weight (per instance) rather than type_option (static/global)
      // and assign it using the argument passed in.
      option.weight = specific_weight;
    }
    
    b : coverpoint b_var;
    
  endgroup
  
  // 3. Fixed type 'g2' to 'g1'
  // 4. Passing custom weights: Instance 1 has weight 2, Instance 2 has default weight 1
  g1 g1_inst = new("instance_1", 2); 
  g1 g2_inst = new("instance_2", 1); 
  
  
  initial begin
    
    // REMOVED: g1_inst.option.name = "Me"; (Illegal external assignment)
    // REMOVED: g2_inst.option.name = "You"; (Illegal external assignment)
    // REMOVED: g1_inst.a.type_option.weight = 2; (Illegal external assignment)
    
    repeat(10) begin
      
      {a_var, b_var} = {$urandom, $urandom};
      
      g1_inst.sample();
      g2_inst.sample();
      
      #1;
      
    end
    $display("Coverage calculation finished.");
  end
endmodule
```
