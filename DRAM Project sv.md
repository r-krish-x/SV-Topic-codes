# design
```systemverilog
`timescale 1ns / 1ps
////////////////////////////////////////////////////////

module dram_dut(
    input reset_n,          // Active Low 
    input clock,            // Clock Input is 100MHz to 400MHz     
    input [7:0] data_in,
    input [7:0] col_addr,   // col_addr[0] and col_addr[1] must be 0x00 for Read/Write for BURST of 4 
    input [1:0] row_addr,
    input [1:0] bank_sel,
    input act_cmd,
    input pre_cmd,
    input ref_cmd,
    input read_cmd,
    input write_cmd,
    //inout dqs_inout,   	     // active high 
    input dqs_in,
    output dqs_out,
    input data_mask,  // not used in dut 
    output reg [7:0] data_out
    );

//reg [7:0] data_out; // commented this because it showed error

enum int unsigned { IDLE=0, ACTIVATE=1, WRITE=3,  READ=4, PRECHARGE=5, REFRESH=6, WR_DATA=7, RD_DATA=8, ACTOPRE=9, ERROR=10 } state, next_state;

reg  state_error_flag ;
reg  timing_error_flag;  

reg act_c,rd_c,wr_c,pre_c,ref_c;
reg [1:0]  rowAdr;  // changed it from 2:0 to 1:0
reg [7:0]  colAdr; 
reg [7:0]  data_reg[16]; 
reg [7:0]  datareg; 
reg [1:0]  bankSel; 
reg [1:0]  a1a0; 

// Delay in Cycles  (all are min timing unless define as max)

int  tact_to_rdwract = 2;  // not used in dut 
int  trd_to_rddata   = 4;
int  twr_to_wrdata   = 4;
int  trdwr_to_rdwr   = 10;  // not used in dut
int  trdwe_to_pre    = 10;
int tpre_to_refact   = 16;  // not used in dut
int tref_to_act      = 64;
int tref_to_ref_max  = 2048;  // not used in dut   
int burst_count      = 4; 

int delay_count     = 0;  // reg delay_count wasn't working
reg write_operation = 0;
reg read_operation  = 0; 

reg [3:0]bankactive;

// Control State Machine 

always@(*) begin: next_state_logic  // always_comb results in error because delay_count and a1a0 are assigned value at other place too

    next_state = IDLE; // default
   
    if($onehot0({act_cmd,write_cmd,read_cmd,pre_cmd,ref_cmd})) begin
        case(state) 
        IDLE:   begin 
                bankactive  = 4'b0000; // All bank inactive 
                state_error_flag = 0; 
                write_operation = 0;
                read_operation = 0; 
                if(act_cmd) begin 
                    next_state = ACTIVATE; 
                    bankactive[bank_sel] = 1'b1;     
                end 
                else if(ref_cmd) begin next_state = REFRESH; delay_count = tref_to_act; end  
                else if (!read_cmd | !write_cmd | !pre_cmd) next_state = state;  
                else begin next_state = ERROR; state_error_flag = 1;  
                    $display("ERROR:  READ,WRITE,PRECHARGE not valid  in IDLE STATE");
                end
                end
         
        ACTIVATE: begin 
                
                if(read_cmd) begin  next_state = READ; delay_count =  trd_to_rddata; end
                else if(write_cmd) begin next_state = WRITE; delay_count = twr_to_wrdata; end
                else if(pre_cmd) begin next_state = ACTOPRE ; delay_count = trdwe_to_pre ; end 
                else if(!act_cmd | !ref_cmd) next_state = state;
                else begin  next_state = ERROR; state_error_flag = 1;
                     $display("ERROR:  ACT_CMD or REF_CMD not valid  in ACTIVE STATE");
                end
        end
               
        WRITE: begin   
               if(bankactive[bank_sel] != 1)begin
                    next_state  = ERROR; 
                    state_error_flag = 1; 
                    $display("ERROR:  Bank bank_sel not active for WRITE"); 
               end 
               else if(delay_count == 0) begin 
                        next_state = WR_DATA; 
                        delay_count = burst_count;
                        write_operation = 1'b1; 
                        a1a0 = 0;
                end
                else next_state = state; 
                         
          end
          WR_DATA: begin
                if(delay_count == 0) begin 
                    next_state = ACTIVATE;  
                    write_operation = 1'b0; 
                end
                else next_state = state; 
          
          end
         
          READ: begin   
               if(bankactive[bank_sel] != 1)begin
                    next_state  = ERROR; 
                    state_error_flag = 1; 
                    $display("ERROR:  Bank bank_sel not active for READ"); 
               end 
               else if(delay_count == 0) begin 
                        next_state=RD_DATA; 
                        delay_count= burst_count;
                        read_operation = 1'b1; 
                        a1a0 = 0;
                end
                else next_state = state; 
                         
          end
          RD_DATA: begin
                if(delay_count == 0) begin 
                    next_state = ACTIVATE;  
                    read_operation = 1'b0; 
                    
                end
                else next_state = state; 
          
          end 
          // Following is RdWR or ACT to PRE delay 
          ACTOPRE:  begin
                if(delay_count == 0)begin
                    next_state = PRECHARGE; 
                    bankactive  = 4'b0000; // All bank inactive 
                end else 
                begin
                    next_state = state; 
                end
          
          end
          PRECHARGE: begin 
                next_state = IDLE ;   
          end 
          
          REFRESH: begin 
                if(delay_count == 0) begin 
                    next_state = IDLE;
                end else next_state = state; 
          end 
          ERROR: begin  
                $display("ERROR:  Test Env. Fails to generate proper sequence in proper time");
                $display("MODLE is Returing to IDLE state for next sequence DUE to ERROR");
                next_state = IDLE;  
          end
          default: next_state <= state;  
                
        endcase 
    end else begin
        next_state = ERROR; 
        state_error_flag = 1;
        $display("ERROR: Multiple Commands active in single clock Cycle");
    end

end: next_state_logic 

always @(posedge clock or negedge reset_n)
begin
    if(!reset_n) state <= IDLE;
    else begin
        state <= next_state; 
    end
end 

always @(posedge clock or negedge reset_n)
begin
 if(!reset_n) delay_count <= 0;
 else begin
    if(delay_count > 0) delay_count <= delay_count -1; 
 end
 
end

always @(posedge clock or negedge reset_n)
begin
 if(!reset_n) a1a0 <= 0;
 else begin
    if (read_operation | write_operation)  a1a0 <= a1a0 + 1; 
 end
 
end



assign dqs_out = (read_operation) ? clock : 1'b0;

//assign dqs_inout = (read_operation) ? dqs_inout : 8'bz;  // spelling error because there's no variable named "dqs_input"
//assign dqs_inout = (write_operation) ? clock : 1'b0 ;     // commented and excluded the inout type data strobe because it wasn't giving proper values in waveforms

always @(posedge clock or negedge reset_n) begin
    if(!reset_n) begin
        act_c <= 1'b0;
        pre_c <= 1'b0;
        ref_c <= 1'b0;
        rd_c  <= 1'b0;
        wr_c  <= 1'b0;
    
        rowAdr  <= 'h0;
        bankSel <= 'h0; 
        colAdr  <= 'h0;
        
    end else begin 
       
        act_c <= act_cmd;
        pre_c <= pre_cmd;
        ref_c <= ref_cmd;
        rd_c  <= read_cmd;
        wr_c  <= write_cmd;
       
        rowAdr       <= (act_cmd) ? row_addr : rowAdr ;
        bankSel      <= (act_cmd) ? bank_sel : bankSel ;
        colAdr[7:2]  <= (read_cmd | write_cmd) ? col_addr[7:2]: colAdr[7:2]; 
        colAdr[1:0]  <= a1a0;  
        //if(act_cmd) bankactive[bank_sel] <= 1'b1;  // commented because it was being repeated
    end  
      
end

    
// Control block state machine


// Memory Block addrss map 
// Size : 256 from top level ( c0 and c1 is used but with local logic) 
//                 ( c2 to c7 = 6bits used for (256 address) 
// generate with 4 bank * 4 Row = 16 device)


wire r0_bk0_sel = (rowAdr == 'b00) && (bankSel == 'b00);
wire r1_bk0_sel = (rowAdr == 'b01) && (bankSel == 'b00);
wire r2_bk0_sel = (rowAdr == 'b10) && (bankSel == 'b00);
wire r3_bk0_sel = (rowAdr == 'b11) && (bankSel == 'b00);
wire r0_bk1_sel = (rowAdr == 'b00) && (bankSel == 'b01);
wire r1_bk1_sel = (rowAdr == 'b01) && (bankSel == 'b01);
wire r2_bk1_sel = (rowAdr == 'b10) && (bankSel == 'b01);
wire r3_bk1_sel = (rowAdr == 'b11) && (bankSel == 'b01);
wire r0_bk2_sel = (rowAdr == 'b00) && (bankSel == 'b10);
wire r1_bk2_sel = (rowAdr == 'b01) && (bankSel == 'b10);
wire r2_bk2_sel = (rowAdr == 'b10) && (bankSel == 'b10);
wire r3_bk2_sel = (rowAdr == 'b11) && (bankSel == 'b10);
wire r0_bk3_sel = (rowAdr == 'b00) && (bankSel == 'b11);
wire r1_bk3_sel = (rowAdr == 'b01) && (bankSel == 'b11);
wire r2_bk3_sel = (rowAdr == 'b10) && (bankSel == 'b11);
wire r3_bk3_sel = (rowAdr == 'b11) && (bankSel == 'b11);


assign data_out =   (r0_bk0_sel) ? data_reg[0]:
                    (r1_bk0_sel) ? data_reg[1]:
                    (r2_bk0_sel) ? data_reg[2]:
                    (r3_bk0_sel) ? data_reg[3]:
                    (r0_bk1_sel) ? data_reg[4]:
                    (r1_bk1_sel) ? data_reg[5]:
                    (r2_bk1_sel) ? data_reg[6]:
                    (r3_bk1_sel) ? data_reg[7]:
                    (r0_bk2_sel) ? data_reg[8]:
                    (r1_bk2_sel) ? data_reg[9]:
                    (r2_bk2_sel) ? data_reg[10]:
                    (r3_bk2_sel) ? data_reg[11]:
                    (r0_bk3_sel) ? data_reg[12]:
                    (r1_bk3_sel) ? data_reg[13]:
                    (r2_bk3_sel) ? data_reg[14]:
                    (r3_bk3_sel) ? data_reg[15]: 8'bz;

Mem256x32  Mem_U0(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r0_bk0_sel),     
    .data_out(data_reg[0]) 
    );
 
Mem256x32  Mem_U1(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r1_bk0_sel),     
    .data_out(data_reg[1]) 
    );

Mem256x32  Mem_U2(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r2_bk0_sel),     
    .data_out(data_reg[2]) 
    ); 

Mem256x32  Mem_U3(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r3_bk0_sel),     
    .data_out(data_reg[3]) 
    ); 
  
  Mem256x32  Mem_U4(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r0_bk1_sel),     
    .data_out(data_reg[4]) 
    );
 
Mem256x32  Mem_U5(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r1_bk1_sel),     
    .data_out(data_reg[5]) 
    );

Mem256x32  Mem_U6(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r2_bk1_sel),     
    .data_out(data_reg[6]) 
    ); 

Mem256x32  Mem_U7(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r3_bk1_sel),     
    .data_out(data_reg[7]) 
    ); 
Mem256x32  Mem_U8(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r0_bk2_sel),     
    .data_out(data_reg[8]) 
    );
 
Mem256x32  Mem_U9(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r1_bk2_sel),     
    .data_out(data_reg[9]) 
    );

Mem256x32  Mem_U10(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r2_bk2_sel),     
    .data_out(data_reg[10]) 
    ); 

Mem256x32  Mem_U11(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r3_bk2_sel),     
    .data_out(data_reg[11]) 
    ); 
  
  Mem256x32  Mem_U12(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r0_bk3_sel),     
    .data_out(data_reg[12]) 
    );
 
Mem256x32  Mem_U13(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r1_bk3_sel),     
    .data_out(data_reg[13]) 
    );

Mem256x32  Mem_U14(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r2_bk3_sel),     
    .data_out(data_reg[14]) 
    ); 

Mem256x32  Mem_U15(
    .reset_n(reset_n),
    .clock(clock),
    .data_in(data_in),
    .wr_op(write_operation),
    .rd_op(read_operation),
    .address(colAdr),
    .dev_select(r3_bk3_sel),     
    .data_out(data_reg[15]) 
    );   
  
    
// ######################  PENDING TASK  ####################################
    
// To BE DONE BY Verification TEAM  
// Timing Check and ERROR Logic using c using assertion at latter stage 
    
// ######################  PENDING TASK  ####################################    
    
    
endmodule

module Mem256x32(
    input reset_n,
    input clock,
    input [7:0] data_in,
    input wr_op,
    input rd_op,
    input dev_select, 
    input [7:0] address,
    output reg [7:0] data_out
    );

//reg [7:0] data_out; // commented because it was giving error
reg [7:0] memory[0:256];  // 6bit address

always @(posedge clock or negedge reset_n) begin
    if(!reset_n) begin
       data_out <= 8'h00; 
    end else begin
        if(wr_op & dev_select) begin
            `ifdef DEBUG_MEM
            $display("Write Data %d at Mem Address= %d", data_in,address);     
            `endif 
            memory[address] <= data_in;
        end else if(rd_op & dev_select)begin
            `ifdef DEBUG_MEM
            $display("Read Data %d at Mem Address= %d", data_in,address);     
            `endif 
            data_out <= memory[address];
        end
    end
end
    
endmodule

```

# testbench

```systemverilog
// Code your testbench here
// or browse Examples
`timescale 1ns/1ps 

// end_of_test_flag is for breaking driver's forever loop when generator work is done

logic end_of_test_flag;

// read_active is used for if statements of monitor and scoreboard classes

logic read_active;

// end_of_read_flag is for breaking forever loops of monitor and scoreboard classes

logic end_of_read_flag;

// interface class

// data_mask is not used in this testbench

interface dram_dut_if(input bit clk);
  
  logic reset_n;
  logic clock;
  logic [7:0] data_in;
  logic [7:0] col_addr;
  logic [1:0] row_addr;
  logic [1:0] bank_sel;
  logic act_cmd;
  logic pre_cmd;
  logic ref_cmd;
  logic read_cmd;
  logic write_cmd;
  logic dqs_in;
  logic dqs_out;
  logic data_mask;
  logic [7:0] data_out;
  
endinterface: dram_dut_if

// transaction packet class

// only data_in is randomized and other inputs are manually stimulated in the top module testbench

// data_out is included in the transaction packet for using it in the scoreboard for comparison

class dram_dut_item;
  
  rand bit [7:0] data_in;
  bit [7:0] col_addr;
  bit [1:0] row_addr;
  bit [1:0] bank_sel;
  bit reset_n;
  
  bit [7:0] data_out;
  
  function void print(string tag = "");
    $display(" T= %0t, %s, data_in = %0d, col_addr = %0d, row_addr = %0d, bank_sel = %0d, data_out = %0d",$time, tag,data_in,col_addr,row_addr,bank_sel,data_out);
  endfunction
  
endclass

// generator class

// mailbox is for putting the data in it from the generator class and getting it in driver class

class generator;
  
  mailbox #(dram_dut_item) driver_mbx;
  event driver_done;
  
  task run();
    
// an object of transaction packet class is created as new data is generated
    
    dram_dut_item item = new();
    item.randomize();
    
    driver_mbx.put(item);
    @(driver_done);
    
    end_of_test_flag = 1;
    $display(" T= %t Generator work is done", $time);
    
  endtask
  
endclass

// driver class

class driver;
  
  virtual dram_dut_if _vif;
  
  mailbox #(dram_dut_item) driver_mbx;
  event driver_done;
  
  task run();
    
    @(posedge _vif.clk);
    
    forever begin
      
      dram_dut_item item;
      driver_mbx.get(item);
      item.print("Driver");
      
// driver sends the randomized data_in values to the interface
      
      _vif.data_in <= item.data_in;
      
      @(posedge _vif.clk);
      
// driver_done event is triggered
      
      -> driver_done;
      if(end_of_test_flag) break;
      
    end
    
  endtask
  
endclass

// monitor class

// mailbox is for putting the data in it from the monitor class and getting it in scoreboard class

class monitor;
  
  virtual dram_dut_if _vif;
  
  mailbox #(dram_dut_item) scrbrd_mbx;
  semaphore semaP;
  
  function new();
    semaP = new(1);
  endfunction
  
  task run();
    
    $display ("%t Monitor: ....  ",$time );
    
    fork
      
      sample_ifPort("Thread_0");
      sample_ifPort("Thread_1");
      
    join
    
  endtask
  
  task sample_ifPort(string tag = "");
    
    forever begin
      
      @(posedge _vif.clk);
      
      if(read_active) begin
        
// an object of transaction packet class is created as new output data is received from the dut
        
        dram_dut_item item = new;
        semaP.get();
        
// monitor's item object receives input data from the interface
        
        item.data_in = _vif.data_in;
        item.col_addr = _vif.col_addr;
        item.row_addr = _vif.row_addr;
        item.bank_sel = _vif.bank_sel;
        item.reset_n = _vif.reset_n;
        
        $display("%t Monitor: Collected the Driven packet %s ",$time,tag);
        @(posedge _vif.clk);
        semaP.put();
        
// monitor's item object receives output data from the interface
        
        item.data_out = _vif.data_out;
        $display("%t Monitor: Collected the Output packet %s ",$time,tag);
        
        scrbrd_mbx.put(item);
        item.print({"Monitor__",tag});
        
      end
      
      if(end_of_read_flag) break;
      
    end
  
  endtask
  
endclass

// scoreboard class

// scoreboard shows whether there is error of dut or success of dut

class scoreboard;
  
  mailbox #(dram_dut_item) scrbrd_mbx;
  
// memdata is a register internal to scoreboard for comparison
  
// memdata is used as expected data and data_out is observed data
  
  reg [7:0] memdata;
  
  task run();
    
    forever begin
      
      dram_dut_item item;
      scrbrd_mbx.get(item);

// expected data is stored in memdata for comparison by using if statement      
      
      memdata = item.data_in;
      
      item.print("#### Scoreboard ####");
      
      if(read_active) begin
        
        if(item.data_out == memdata) begin
          
          $display("#### #### #### ####");
          $display("#### %t RESULT : SUCCESS of DUT",$time);
          $display("#### #### #### ####");
          
// success of dut is due to observed data being the same as expected data
          
        end
        
        else if(item.data_out != memdata) begin
          
          $display("#### #### #### ####");
          $display("#### %t RESULT : ERROR of DUT",$time);
          $display("#### #### #### ####");
          
// error of dut is due to observed data not being the same as expected data
          
        end
        
      end
      
      if(end_of_read_flag) break;
    
    end
    
  endtask
  
endclass

// environment class

// objects of generator, driver, monitor and scoreboard are created in environment class

// objects for mailboxes of these classes are connected to mailboxes declared in environment

class environment;
  
  driver drv0;
  generator gen0;
  monitor mon0;
  scoreboard scrbd0;

  mailbox #(dram_dut_item) driver_mbx;
  mailbox #(dram_dut_item) scrbrd_mbx;
  event driver_done;	

  virtual dram_dut_if _vif;

  function new();
    
    drv0 = new;
    gen0 = new;
    mon0 = new;
    scrbd0 = new;

    driver_mbx = new();
    scrbrd_mbx = new();
    
    drv0.driver_mbx = driver_mbx;
    gen0.driver_mbx = driver_mbx;
    mon0.scrbrd_mbx  = scrbrd_mbx;
    scrbd0.scrbrd_mbx = scrbrd_mbx;
   
// driver_done event of driver and generator classes is connected to driver_done event declared in environment
    
    drv0.driver_done = driver_done;
    gen0.driver_done = driver_done;
	
  endfunction

  virtual task run();

// interface objects of driver and monitor classes are connected to interface object created in environment
    
    drv0._vif = _vif;
    mon0._vif = _vif;

// run tasks for generator, driver, monitor and scoreboard are executed within fork join_any statement
    
    fork
      drv0.run();
      gen0.run();
      mon0.run();
      scrbd0.run();	
    join_any
  
  endtask
  
endclass

// test class for environment

// an onject of environment class is created in the test class

class test0;
  
  environment env0;

  function new();
    env0 = new;
  endfunction 

// run task of environment class is executed
  
  task run();
    env0.run();
  endtask
  
endclass

// top module testbench for dram

module dram_dut_tb;
  
// write_active is high when data is being written
  
  reg write_active;
 
// w, i2 and num2 are variables used for the for loops
  
  int w;
  int i2;
  int num2 = 9;
  
  reg clk;
  
  always #2.5 clk = ~clk;
  
  dram_dut_if __if(clk);
  
  dram_dut dut_u0 (
    .reset_n(__if.reset_n),
    .clock(clk),
    .data_in(__if.data_in),
    .col_addr(__if.col_addr),
    .row_addr(__if.row_addr),
    .bank_sel(__if.bank_sel),
    .act_cmd(__if.act_cmd),
    .pre_cmd(__if.pre_cmd),
    .ref_cmd(__if.ref_cmd),
    .read_cmd(__if.read_cmd),
    .write_cmd(__if.write_cmd),
    .dqs_in(dqs_in),
    .dqs_out(dqs_out),
    .data_mask(__if.data_mask), 
    .data_out(__if.data_out)
  );
  
// object of test class is created
  
  test0 t0;
  
  initial begin
    
// dut is in state 0 (IDLE) at the start and the cycle for testing functionalities of dut starts
    
// all the delays are decided as per requirements
    
    end_of_test_flag = 0;
    end_of_read_flag = 0;
    clk <= 0;
    __if.dqs_in <= 0;
    __if.reset_n <= 0;
    read_active <= 0;
    write_active <= 0;
    
    #20;
    
    __if.act_cmd <= 1;
    __if.bank_sel <= 2'b00;
    
    __if.read_cmd <= 0;
    __if.write_cmd <= 0;
    __if.pre_cmd <= 0;
    __if.ref_cmd <= 0;
    
    __if.reset_n <= 1;
    
    #5;
    
// run task of test class is executed
    
    t0 = new;
    t0.env0._vif = __if;
    t0.run();
    
    __if.row_addr <= 2'b00;
    
    #25;
    
    __if.act_cmd <= 0;
    __if.write_cmd <= 1;
    __if.col_addr <= 8'b00000000;
    
// data will be written 4 cycles after write_cmd is set to high
    
    #20;
    
    fork
      for(w=0; w<8; w++) begin
        __if.dqs_in <= ~__if.dqs_in;
        #2.5;
      end
    join_none
    
    write_active <= 1;
    
    #5;
    #15;
    
    write_active <= 0;
    
// data is not being written after write_active is set to low
    
    #5;
    
    __if.write_cmd <= 0;
    __if.read_cmd <= 1;
    
// data will be read 5 cycles after read_cmd is set to high
    
    #22.5;
    
// read_active is set to high so that if statements of monitor and scoreboard classes are executed
    
    read_active <= 1;
    
    #2.5;
    #15;
    #5;
    
    read_active <= 0;
    end_of_read_flag = 1;
    
// monitor and scoreboard loops break
    
    __if.read_cmd <= 0;
    __if.pre_cmd <= 1;
    
    #50;
    #5;
    #5;
    
    __if.pre_cmd <= 0;
    __if.ref_cmd <= 1;
    
    #320;
    
    __if.ref_cmd <= 0;
    
    #10;
    
// dut is in state 0 (IDLE) again and the functionalities of dut have been tested, one cycle is completed
    
// the entire cycle above is repeated 10 times by using for loop with random values of bank_sel, row_addr and col_addr
    
    for(i2 = 0; i2 < num2; i2++) begin
      
      end_of_test_flag = 0;
      end_of_read_flag = 0;
      
      __if.act_cmd <= 1;
      __if.bank_sel <= $urandom;
    
      __if.read_cmd <= 0;
      __if.write_cmd <= 0;
      __if.pre_cmd <= 0;
      __if.ref_cmd <= 0;
    
      __if.reset_n <= 1;
  
      #5;
    
      t0 = new;
      t0.env0._vif = __if;
      t0.run();
    
      __if.row_addr <= $urandom;
    
      #25;
    
      __if.act_cmd <= 0;
      __if.write_cmd <= 1;
      __if.col_addr[1:0] <= 2'b00;
      
// only 6 bits of column address can be actually decided by the testbench
      
// last 2 bits of column address are decided by the dut internally for a burst of 4
      
      __if.col_addr[7:2] <= $urandom;
    
      #20;
      
      fork
        for(w=0; w<8; w++) begin
          __if.dqs_in <= ~__if.dqs_in;
          #2.5;
        end
      join_none
      
      write_active <= 1;
      
      #5;
      #15;
      
      write_active <= 0;
      
      #5;
      
      __if.write_cmd <= 0;
      __if.read_cmd <= 1;
    
      #22.5;
    
      read_active <= 1;
    
      #2.5;
      #15;
      #5;
      
      read_active <= 0;
      end_of_read_flag = 1;
    
      __if.read_cmd <= 0;
      __if.pre_cmd <= 1;
    
      #50;
      #5;
      #5;
    
      __if.pre_cmd <= 0;
      __if.ref_cmd <= 1;
    
      #320;
    
      __if.ref_cmd <= 0;
    
      #10;
      
    end
    
    #10;
    
// simulation is terminated
    
    $finish;
    
  end
  
  initial begin
    
    $dumpvars;
    $dumpfile("dram.vcd");
    
  end
  
endmodule
```
