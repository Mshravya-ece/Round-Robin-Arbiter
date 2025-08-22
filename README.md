# Round-Robin-Arbiter

In this repository, we shall discuss the implementation of Round Robin Arbiter using AMD Vivado tool.

The round-robin arbiter is used to prevent starvation and provide statistical fairness in a system. Starvation occurs when a request is repeatedly denied access to the shared resource, even though other requests are being granted access. The round-robin arbiter solves this problem by granting access to each request in a circular fashion, ensuring that no request is consistently denied access to the resource.

So, scheduling arbiters are used when multiple devices require access to a shared resource.

In a shared bus, the arbiter determines which master uses the bus. When a master relinquishes the bus, the switch is turned to the next position and the bus is granted to the master on level. So. All masters are granted bus on an equal basis.

<img width="362" height="208" alt="image" src="https://github.com/user-attachments/assets/be4f6f59-091a-4898-9b47-3e5cd77a965f" />

Arbiter Architecture:

<img width="500" height="414" alt="image" src="https://github.com/user-attachments/assets/56210731-7842-4b18-bc87-ee4554210e87" />

Arbiter logic:

<img width="422" height="248" alt="image" src="https://github.com/user-attachments/assets/4ee840e7-05c8-4855-a5db-6d8d9a6e12e4" />

When the requests: REQ0, REQ1, REQ2, REQ3 arrive, the arbiter grants signal corresponding to the request signal.

COMREQ logic:

COMREQ indicates whether the bus is free or busy. It is active high when the bus is granted or requested, i.e., when it is busy.

<img width="365" height="220" alt="image" src="https://github.com/user-attachments/assets/592ad5af-bb4d-43df-a744-2dd33c4f4a2d" />

Encoder logic:

It indicates which master has been granted bus.

<img width="278" height="185" alt="image" src="https://github.com/user-attachments/assets/54745660-f034-4e63-abe3-798444a40fce" />

<img width="294" height="194" alt="image" src="https://github.com/user-attachments/assets/4366ace0-af64-408e-beab-214b463a0d34" />

Begin Logic:

<img width="601" height="172" alt="image" src="https://github.com/user-attachments/assets/696e4fae-0eeb-4c46-a290-4e017641e53b" />

BEG = (REQ0 | REQ1 | REQ2 | REQ3) & (~COMREQ)

Register Block:

Request level is saved in a register that latches the state of grant signals. The output of register is lmask0 and lmask1.

<img width="393" height="259" alt="image" src="https://github.com/user-attachments/assets/13ef8ed1-f081-41fd-9297-8fd5c3e1b086" />

Based on the begin signal (BEG) and current state (REQ), the grant signal is activated and the next state is decided by the arbiter. 

State Diagram:

<img width="793" height="303" alt="image" src="https://github.com/user-attachments/assets/c0a0b37f-4867-4d8c-8705-adf1b4ef9e4d" />

Verilog Code of Round Robin Arbiter:

module Arbiter(clk,rst,req3,req2,req1,req0,gnt3,gnt2,gnt1,gnt0);

// PORT DECLARATION

input clk;
input rst;

//REQUEST SIGNALS

input req3;
input req2;
input req1;
input req0;

// GRANT SIGNALS

output gnt3; 
output gnt2;
output gnt1;
output gnt0;

// INTERNAL REGISTERS

wire [1:0]gnt;
wire comreq;

wire beg;       //BEGIN SIGNAL

wire [1:0]lgnt; //LATCHED ENCODED GRANT

wire lcomreq;   // BUS STATUS

 // LATCHED GRANTS
 
reg lgnt0;      
reg lgnt1;
reg lgnt2;
reg lgnt3;
reg mask_enable;
reg lmask0;
reg lmask1;
reg ledge;

always@(posedge clk) 

if (rst) //if reset is true

begin

  lgnt0 <= 0;
  lgnt1 <= 0;
  lgnt2 <= 0;
  lgnt3 <= 0;
  
end

else

 begin
 
 lgnt0 <= (~lcomreq & ~lmask1 & ~lmask0 & ~req3 & ~req2 & ~req1 & req0)
 
          | (~lcomreq & ~lmask1 & lmask0 & ~req3 & ~req2 & req0)
          
          | (~lcomreq & lmask1 & ~lmask0 & ~req3 & req0)
          
          | (~lcomreq & lmask1 & lmask0 & req0)
          
          | (lcomreq & lgnt0);
         
 lgnt1 <= (~lcomreq & ~lmask1 & ~lmask0 & req1)
 
          | (~lcomreq & ~lmask1 & lmask0 & ~req3 & ~req2 & req1 & ~req0)
          
          | (~lcomreq & lmask1 & ~lmask0 & ~req3 & req1 & ~req0)
          
          | (~lcomreq & lmask1 & lmask0 & req1 & ~req0)
          
          | (lcomreq & lgnt1);
     
  lgnt2 <= (~lcomreq & ~lmask1 & ~lmask0 & req2 & ~req1)
  
          | (~lcomreq & ~lmask1 & lmask0 & req2)
          
          | (~lcomreq & lmask1 & ~lmask0 & ~req3 & req2 & ~req1 & ~req0)
          
          | (~lcomreq & lmask1 & lmask0 & req2 & ~req1 & ~req0)
          
          | (lcomreq & lgnt2);
          
  lgnt3 <= (~lcomreq & ~lmask1 & ~lmask0 & req3 & ~req2 & ~req1)
  
          | (~lcomreq & ~lmask1 & lmask0 & req3 & ~req2)
          
          | (~lcomreq & lmask1 & ~lmask0 & req3)
          
          | (~lcomreq & lmask1 & lmask0 & req3 & ~req2 & ~req1 & ~req0)
          
          | (lcomreq & lgnt3);
   
  end

// BEGIN SIGNAL

assign beg = (req3 | req2 | req1 | req0) & ~lcomreq;

// comreq logic (BUS STATUS)

assign lcomreq = (req3 & lgnt3) | (req2 & lgnt2) | (req1 & lgnt1) |(req0 & lgnt0);
            
// Encoder Logic

assign lgnt = {(lgnt3 | lgnt2),(lgnt3 | lgnt1)};

//lmask register

always @ (posedge clk)

 if (rst)
 
  begin
  
   lmask1 <= 0;
   lmask0 <= 0;
   
  end 
  
 else if (mask_enable)
 
  begin
  
   lmask1 <= lgnt[1];
   lmask0 <= lgnt[0];
   
  end 
  
 else
 
  begin
  
   lmask1 <= lmask1;
   lmask0 <= lmask0;
   
  end 

assign comreq = lcomreq;

assign gnt = lgnt;

// Drive the outputs

assign gnt3 = lgnt3;

assign gnt2 = lgnt2;

assign gnt1 = lgnt1;

assign gnt0 = lgnt0;

endmodule

Testbench Code:

module Arbiter_tb();

reg clk;
reg rst;

reg req3;
reg req2;
reg req1;
reg req0;

wire gnt3;
wire gnt2;
wire gnt1;
wire gnt0;

//CLOCK GENERATOR

always #1 clk = ~clk;

initial begin

 $dumpfile ("Arbiter.vcd");
 
 $dumpvars();
 
 clk=0;
 rst=1;
 
 req0=0;
 req1=0;
 req2=0;
 req3=0;
 
 #10 rst=0;
 
 repeat(1) @ (posedge clk);
 
 req0 <= 1;
 
 repeat(1) @ (posedge clk);
 
 req0 <= 0;
 
 repeat(1) @ (posedge clk);
 
 req0 <= 1;
 req1 <= 1;
 
 repeat(1) @ (posedge clk);
 
 req2 <= 1;
 req1 <= 0;
 
 repeat(1) @ (posedge clk);
 
 req3 <= 1;
 req2 <= 0;
 
 repeat(1) @ (posedge clk);
 req3 <= 0;
 
 repeat(1) @ (posedge clk);
 req0 <= 0;
 
 repeat(1) @ (posedge clk);
 #10 $finish;
 
end

Schematic after RTL Analysis:

<img width="861" height="319" alt="image" src="https://github.com/user-attachments/assets/1b72f592-8311-4a15-87cd-1abf1ccfcb55" />

Schematic after Synthesis:

<img width="940" height="258" alt="image" src="https://github.com/user-attachments/assets/922de92f-ea79-47bc-9af4-9f36a27c2a3c" />

Output after Simulation:

<img width="1011" height="329" alt="image" src="https://github.com/user-attachments/assets/0762cedf-6db9-449c-97fc-a43ab09b882a" />

This code was virtually dumped on the FPGA Board: Kria K26C SOM and 4 registers, 10 IOBs ,1 clk buffer and 5 LUTs were used during the hardware execution.









