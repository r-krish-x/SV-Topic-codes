## Memory Partition
```systemverilog
//______________________________memory block randomization________________________________

class MemoryBlock;
  
  bit [31:0]	m_ram_start;
  bit [31:0]	m_ram_end;
  
  rand bit [31:0]	m_start_addr;
  rand bit [31:0]	m_end_addr;
  rand int			m_block_size;
  
  constraint c_addr { m_start_addr >= m_ram_start;
                     m_start_addr < m_ram_end;
                    m_start_addr % 4 == 0;
                     m_end_addr == m_start_addr + m_block_size - 1;};
  
  constraint c_blk_size {m_block_size inside {64, 128, 512};};
  
  function void display();
    
    $display("----------------MEMORY BLOCK----------------");
    $display("RAM Start_Addr	= 0x%0h"	, m_ram_start);
    $display("RAM END_Addr		= 0x%0h"	, m_ram_end);
    $display("Block Start_Addr	= 0x%0h"	, m_start_addr);
    $display("Block Start_Addr	= 0x%0h"	, m_end_addr);
    $display("Block Size        = %0d bytes", m_block_size);
    
  endfunction
endclass

module tb;
  
  initial begin
    MemoryBlock mb = new();
    mb.m_ram_start = 32'h0;
    mb.m_ram_end   = 32'h7ff;
    mb.randomize();
    mb.display();
  end
endmodule


//___________________equal partitions of memeory_____________________________


class MemoryBlock;
  
  bit[31:0]		m_ram_start;
  bit[31:0]		m_ram_end;
  
  rand int 		m_num_part;
  rand bit[31:0]m_part_start[];
  rand int		m_part_size;
  rand int		m_tmp;
  
  constraint c_parts { m_num_part > 4; m_num_part <10;}
  
  constraint c_size  {m_part_size == (m_ram_end - m_ram_start)/m_num_part;}
  
  constraint c_part {m_part_start.size() == m_num_part;
                     foreach (m_part_size[i]){
                       if(i)
                         m_part_start[i] == m_part_start[i-1] + m_part_size;
                     else
                       m_part_start[i] == m_ram_start;
                     }
                    
                    }
                       
                       function void display();
                     
                     $display("---------------MEMORY BLOCK----------------------");
                     $display("RAM StartAddr		= 0x%0h", m_ram_start);
                     $display("RAM EndAddr			= 0x%0h", m_ram_end);
                     $display("# Partitions 		=  %0d",  m_num_part);
                     $display("Partition Size 		=  %0d bytes",  m_part_size);
                     $display("------------Partitions-------------------");
                     foreach (m_part_start[i])
                       $display("Partition %0d start = 0x%0h", i, m_part_start[i]);
                     endfunction
                     endclass
                     
                     
                     module tb;
                       initial begin
                         MemoryBlock mb = new;
                         mb.m_ram_start = 32'h0;
                         mb.m_ram_end   = 32'h7ff;
                         mb.randomize();
                         mb.display();
                       end
                     endmodule
                     
                     
//                      2047/6 = 341 bytes
                     
//                       decimal 341 in hex  0x155
//                      decimal 682 in hex  0x2aa
//                      decimal 1023 in hex 0x3ff
//                      decimal 1364 in hex 0x554
                     
                     
                     
                     
//__________________________Variable Memory Partition__________________________
                     

          class MemoryBlock;
            
            bit[31:0]		m_ram_start;
            bit[31:0]		m_ram_end;
            
            rand int 		m_num_part;
            rand bit[31:0]	m_part_start[];
            rand int		m_part_size[];
            rand int 		m_tmp;
            
            
            constraint c_parts {m_num_part > 4; m_num_part <10;}
            
            constraint c_size {m_part_size.size() == m_num_part;
                               m_part_size.sum() == m_ram_end - m_ram_start + 1;
                               foreach (m_part_size[i])
                                 m_part_size[i] inside {16,32,64,128,512,1024};
                              }
            
            constraint c_part { m_part_start.size() == m_num_part;
                               foreach(m_part_start[i]){
                                 if(i)
                                   m_part_start[i] == m_part_start[i-1] + m_part_size[i-1];
                                 else
                                   m_part_start[i] == m_ram_start;
            
            }
                              }
                                 
                                 function void display();
                               $display("-------------Memory Block-------------------------");
                               $display("RAM StartAddr		= 0x%0h", m_ram_start);
                               $display("RAM EndAddr 		= 0x%0h", m_ram_end);
                               $display("# Partitions 		= %0d",m_num_part);
                               $display("-------------Partitions-----------------");
                               foreach(m_part_start[i])
                                 $display("Partition %0d -> start = 0x%0h  -> size = %0d bytes", i, m_part_start[i], m_part_size[i]);
                               endfunction
                               endclass
                               
                               
                               module tb;
                                 
                                 initial begin
                                   MemoryBlock mb = new;
                                   mb.m_ram_start = 32'h0;
                                   mb.m_ram_end	  = 32'h7ff;
                                   mb.randomize();
                                   mb.display();
                                 end
                               endmodule
```
                    
