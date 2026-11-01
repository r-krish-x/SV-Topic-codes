## This contains code example of all arrays
```systemverilog
// ================verfication series part 1 udemy=================


module tb;
  
  bit arr1[10];
  bit arr2[]={1,0,1,1,1};
  
  initial begin
    $display("Arr1: [%0d]", $size(arr1));
    $display("Arr2: [%0d]", $size(arr2));
  
  $display("Value of first element arr2:[%0d]", arr2[0]);
    $display("value of first element arr1:[%0d]", arr1[0]);
    $display("All element of arr2: %0p", arr2);
  end
endmodule


// ===============================================================

module tb;
  
  int arr[5]= '{1,2,3,45,6};
  
  initial begin
  $display("array: %0p", arr);
  end
  
endmodule


// ============================================================

module tb;
  
  int arr[10];
  int i=0;
  
//   initial begin
//     for(i=0; i<10; i++)begin
//     arr[i]=i;
//     end
//     $display("arr: %0p", arr);
    
//   end
  
//   initial begin
//     foreach(arr[j])begin
//       arr[j]=j;
//       $display("arr: %0d", arr[j]);
//     end
//   end
  
  
  initial begin
    repeat(10)begin
      arr[i]=i;
      i++;
    end
    
    $display("arr: %0p", arr);
  end
endmodule


// =========================================================
/////////// copying array

module tb;
  
  int arr1[5];
  int arr2[5];
//   int i=0;
  
  initial begin
    
    for(int i=0;i<5;i++)begin
    arr1[i]=5*i;
  end
  
  arr2=arr1;
  
  $display("arr2: %0p", arr2);
  end
endmodule


// ===========================================

//////// comparing arrays

module tb;
  int arr1[5] = '{1,2,3,4,5};
  int arr2[5] = '{1,2,3,4,5};
  int status;
  
  initial begin
    status = (arr1==arr2); // status = (arr1!=arr2);
    $display("Status: %0d", status);
  end
  
endmodule


// ===============dynamic array========================

module tb;
  int arr[];
  int farr[10];
  
  initial begin 
    arr = new[5];
    
    for(int i=0; i<5; i++)begin
      arr[i]= 5*i;
    end
    
    $display("Arr: %0p", arr);
    
//     arr = new[10];  
    arr = new[10](arr);   //it can retain old values
    
    for(int i=5; i<10; i++)begin
      arr[i]=i*3;
    end
    
    farr = arr; // assigning farr=arr only if size of both is same else it will throw an compile error
    $display("new farr: %0p", farr);
    
//      arr.delete();
  
//   $display("Arr:%0p", arr);
  end
  
 
    endmodule


// ===========================Queue array========================

module tb;
  
  int arr[$];
  int j =0;
  
  initial begin
    arr ='{1,2,3};
    $display("arr:%0p", arr);
    
    arr.push_front(7);
    $display("arr:%0p", arr);
    
    arr.push_back(9);
    $display("arr:%0p", arr);	
    
    arr.insert(2,10); // 2 is index and 10 is value passed
    $display("arr:%0p", arr);
    
   j= arr.pop_front();
    $display("arr:%0p", arr);
    $display("value of j:%0d", j);
    
    arr.pop_back();
    $display("arr:%0p", arr);
  
    arr.delete(1); // 1 is index
    $display("arr:%0p", arr);
  
  end
endmodule


============= puzzle Example=========

module tb;
  
  int arr[];
  int i = 0;
  
  initial begin
    
    arr= new[7];
    for(i=0; i<7;i++)begin
      arr[i]=(i+1)*7;
    end
    $display("Arr:%0p", arr);

    #20;
    arr=new[20](arr);
    for(i=7;i<20;i++)begin
      arr[i]=(i-6)*5;
    end
    $display("Arr:%0p", arr);
  end
  
endmodule


// ================puzzle example======================

module tb;
  
  int f_arr[20];
  int q_arr[$];
  int i = 0;
  initial begin
    
    for(i=0;i<20; i++)begin

//      f_arr[i] = $urandom_range(1,100);

      // Returns a 32-bit number, mod 100 (0-99), then adds 1 to make it 1-100
      f_arr[i] = ($urandom() % 100) + 1; 
    end
   
    $display("f_arr:%0p", f_arr);
    
    for(i=0;i<20;i++)begin
      q_arr.push_front(f_arr[i]);
    end
    $display("q_arr:%0p", q_arr);
  end
endmodule
    

// =============puzzle assignment========

module tb;
  reg[32:0] arr1[15];
  reg[32:0] arr2[15];
  int i =0;
  
  initial begin
    for(i=0;i<15;i++)begin
      arr1[i] = $urandom_range(1,100);
      arr2[i] = $urandom_range(1,100);
    end
  
    for(i=0;i<15;i++)begin
      $write("arr1:%0p", arr1[i]);
    end
  end
endmodule

// not completed-   i left it coz it was taking too much time



module array_assignment;

  // 1. Create two arrays of reg type capable of storing 15 elements
  // Using 32-bit width (reg [31:0]) since $urandom returns a 32-bit value
  reg [3:0] array1[15];
  reg [3:0] array2[15];    // packed bit of reg was 32 `reg[32:0]` but clear readabilty i reduced size

  initial begin
    for (int i = 0; i < 15; i++) begin
      array1[i] = $urandom();
      array2[i] = $urandom();
    end

    $display("--- Array 1 ---");
    for (int i = 0; i < 15; i++) begin
      $write("%0d ", array1[i]);
    end

    $display("\n--- Array 2 ---");
    for (int i = 0; i < 15; i++) begin
      $write("%0d ", array2[i]);
    end
  end

endmodule

===============run this in cadence====================

class transaction;

rand bit[7:0]din;
randc bit[7:0]addr;
bit wr;
bit [7:0]dout;

constraint addr_c{addr > 10; addr < 18;}

endclass

class generator;

transaction t;
integer i;

task run ();
  
for(i=0 ; i < 100; i++)begin
t = new();
t.randomize;

end
endtask
endclass

class scoreboard;

  bit[7:0] tarr[256] = '{default:0};

transaction t;
task run();


if(t.wr == 1'b1)begin
tarr[t.addr] = t.din;
$display("[SCO]: data stored din: %0d, addr:%0d", t.din, t.addr);
end

if(t.wr == 1'b0)begin
	if(t.dout == 0) 
	$display("[SCO]: No Data written at this location test passed");
	else if(t.dout == tarr[t.addr])
	$display("[SCO]:VAlid data found test passed");
      else
	$display("[SCO]: test failed");
	end
endtask
endclass

// ================================================================





// =================================================
// ===============Chipverify=====================
// =================================================

// ==========Packed Array================
module tb;
  
  bit [7:0] m_data;
  
  initial begin
    
    m_data = 8'hA2;
    
    for(int i=0; i < $size(m_data); i++)begin
      $display("m_data[%0p] = %0b", i, m_data[i]);
    end
  end
endmodule

// ===========================================

module tb;
  bit [3:0][7:0] m_data;
  
  initial begin
    
    m_data = 32'hface_cafe;
    
    $display("m_data = %0h", m_data);
  
    
    for(int i=0; i < $size(m_data); i++)begin
      $display("m_data[%0d] =%b [%0h]", i,m_data[i], m_data[i]);
    end
  end
  
endmodule

// ======================================

module tb;
  bit[2:0][3:0][7:0] m_data;
  
  initial begin
    m_data[0] = 32'hface_cafe;
    m_data[1] = 32'h1234_5678;
    m_data[2] = 32'hc0de_fade;
    
    $display("m_data = %0h", m_data);
    
    foreach(m_data[i])begin
      $display("m_data[%0d] = %h", i, m_data[i]);
    
    foreach(m_data[i][j])begin
      $display("m_data[%0d][%0d] = %0h", i,j,m_data[i][j]);
    end
    end
  end
endmodule


// =====================================================

module tb;
  
  bit [1:0][7:0]channel_data;
  
  initial begin
    channel_data = 16'hA53C;
    for (int i =0; i < $size(channel_data); i++)begin
      $display("channel_data[%0d] = %b(%0h)", i, channel_data[i], channel_data[i]);
    end
    $display("channel_data[0] bit 3 = %b", channel_data[0][3]);  // channel_data[0] = 00111100(3c) → in this 7,6,1,0 index has 0 value where 5,4,3,2 has 1 value
  end
  
endmodule
  

// ======================Unpacked array===========================
//// Signle dimensional unpacked
module tb;
  byte stack[8];
  
  initial begin
    foreach(stack[i])begin
      stack[i] = $urandom;
      
      $display("Assign %0d to index %0d", stack[i], i);
    end
    $display("stack = %p", stack);
  end
endmodule


// ==============================================
//// Multidimensional unpacked

module tb;
  byte stack[2][4];
  
  initial begin
//     foreach(stack[i])
//       foreach(stack[i][j])begin
        foreach(stack[i,j])begin
        stack[i][j] = $random;
        $display("stack[%0d][%0d] = %0d", i, j, stack[i][j]);
      end
    $display("stack =%p", stack);
    
  end
endmodule


// =======================================================
//// packed + unpacked

module tb;
  bit[3:0][7:0]   stack [2][4];
  
  initial begin
    foreach(stack[i,j])begin
      stack[i][j] = $random;
      $display("stack [%0d][%0d] = %0h", i,j,stack[i][j]);
    end
    $display("stack = %p", stack);
    
    $display("stack [0][0][2]= %0h", stack[0][0][2]);
  end
endmodule

// ==========================================================
// ============================================================
// ==========================================================
// =====================================================
// =======================================================
// =========================================================

///// Queue array

module tb;
  
  string fruits[$] = '{"orange", "apple", "kiwi"};
  
  initial begin
    foreach(fruits[i])
      $display("fruits[%0d] = %s", i, fruits[i]);
    
    $display("fruits = %p", fruits);
    
    fruits ={};
    $display("After deletion, frutis = %p", fruits);
  end
endmodule

// ============================================================

module tb;
  
  string fruits[$] = '{"orange", "apple", "lemon", "kiwi"};
  
  initial begin
    
    $display("citrus frutis = %p", fruits[1:2]);
    
    $display("fruits = %p", fruits [1:$]);
    
    fruits[1+$] = "pineapple";
    $display("fruits = %p", fruits);
    
    fruits = fruits[1:$];
    $display("Remove orange, fruits = %p", fruits);
  end
endmodule

//========================================================

module tb;
  
  string fruits[$] = '{"apple", "pear","mango", "banana"};
  
  initial begin
    $display("number of fruits, fruits = %p", fruits.size(), fruits);
    
    fruits.insert(1, "peach");
    $display("insert peach, fruits = %p",fruits.size(), fruits);
    
    fruits.delete(3);
    $display("delete mango, size = %0d, frutis = %p", fruits.size(), fruits);
    
    $display("pop front %s ,size = %0d fruits =%p", fruits.pop_front(), fruits.size(), fruits);
    
    fruits.push_front("apricot");
    $display("push apricot, size =%0d, fruits = %p", fruits.size(), fruits);
    
    $display("pop %s, size = %0d, fruits = %p",fruits.pop_back(), fruits.size(), fruits );
    
    fruits.push_back("plum");
    $display("push plum, size = %0d, fruits = %p", fruits.size(), fruits);
  end 
endmodule

// =================================================================

// create queue in classes

class fruit;
  string name;
  
  function new(string name = "unknown" );
    this.name = name;
  endfunction 
  
endclass

module tb;
  
  fruit list[$];
  
  initial begin
    
    fruit f = new("apple");
    list.push_back(f);
    
    f = new("banana");
    list.push_back(f);
    
    foreach(list[i])
      $display("list [%0d] = %s", i, list[i].name);
    
    $display("size = %0d, list", list.size(), list);
    
    $display("list = %p", list);
  end
endmodule

// ====================================================================

//// Queue of dynamic array

typedef string dyn_a[];

module tb;
  
  dyn_a list[$];
  
  initial begin
    
    dyn_a marvel = '{"spiderman", "ironman", "thor", "doom"};
    dyn_a dc    = '{"batman", "superman", "wonderwoman"};
    
    list.push_back(marvel);
    list.push_back(dc);
    
    foreach(list[i])
      foreach(list[i][j])
        $display("list [%0d][%0d] = %s", i,j, list[i][j]);
    
    $display("list = %p", list);
  end
endmodule

// ================================================================
////////////// puzzle////
module tb;
  int exp_q[$];
  
  initial begin
    exp_q.push_back(10);
    exp_q.push_back(20);
    exp_q.push_back(30);
    
    $display("size = %0d", exp_q.size(), exp_q);
    
//     exp_q.pop_front();
//     $display("pop = %0d", exp_q.size(), exp_q );
  
//     exp_q.pop_front();
//     $display("pop = %0d", exp_q.size(), exp_q );
  
//     exp_q.pop_front();
//     $display("pop = %0d", exp_q.size(), exp_q );
    
    while (exp_q.size() > 0) begin
      $display("Popped %0d, remaining size=%0d", exp_q.pop_front(), exp_q.size());
    end
  
  end
  
endmodule

// ===================================================================
 
// dynamic array

module tb;
  
  int array[];
  int id[];
  
  initial begin
    array = new[5];
    array = '{1,2,3,4,5};
    
    id = array;
    
    $display("id = %p", id);
    id = new[6](id);
    
    id[id.size() - 1] = 6;
    $display("New id = %p", id);
    
    $display("array.size() = %0d, id.size = %0d", array.size(), id.size());
    
  end
endmodule

// ====================================================================

module tb;
  int dyn[];
  
  initial begin
    dyn = new[3];
    
    dyn = '{10,20, 30};
    
    $display("dyn = %p", dyn);
    
//     dyn = new[4](dyn);
    dyn = new[dyn.size + 1](dyn);
    
    dyn[dyn.size() - 1] = 40;
    
    $display("dyn with added element", dyn);
  end
endmodule

```
