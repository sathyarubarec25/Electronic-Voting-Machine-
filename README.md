# Electronic Voting Machine
Electronic Voting Machine (EVM) using Verilog HDL and Finite State Machine (FSM) on FPGA. Supports three candidates, vote registration, vote counting, and real-time vote count display using a multiplexed seven-segment display.

# Electronic Voting Machine using FSM on FPGA
# Project Overview

This project implements an Electronic Voting Machine using Verilog HDL based on a Finite State Machine (FSM) architecture. The system allows users to cast votes for three candidates (A, B, and C), registers valid votes, maintains individual vote counts, and displays the selected candidate's vote count on a multiplexed seven-segment display.

# Features

- FSM-based voting control.

- Supports three candidates.

- Valid vote registration.

- Individual vote counters for each candidate.

- Seven-segment display for vote count visualization.

- Reset functionality.

- Synthesizable Verilog RTL design.

# FSM States

- IDLE – Waits for the start signal.

- VOTING – Accepts candidate selection.

- REGISTER – Registers and counts the vote.

- CONFIRM – Confirms vote registration and returns to IDLE

 # System Architecture

The system consists of the following major modules:

- Input: Accepts reset, start, and candidate selection signals through FPGA buttons and switches.

- FSM Controller: Controls the voting process through four states: IDLE, VOTING, REGISTER, and CONFIRM.

- Vote Registration: Identifies the selected candidate and registers the vote.

- Vote Counter: Maintains separate 8-bit counters for Candidates A, B, and C (0–255).

- Display Controller: Selects the candidate whose count should be displayed and converts the count into seven-segment signals.

- Seven-Segment Display: Displays the selected candidate's vote count using three multiplexed digits.


# State Transitions

- IDLE → VOTING: When the start signal is enabled.

- VOTING → REGISTER: When exactly one candidate is selected.

- VOTING → VOTING: When no valid candidate selection is detected.

- REGISTER → CONFIRM: After registering the vote.

- CONFIRM → IDLE: Returns to the initial state.


# FSM State Diagram
<img width="800" height="435" alt="Screenshot 2026-11-02 180552" src="https://github.com/user-attachments/assets/cc77f51b-9b11-4c22-928b-be88ee7b8376" />



# Hardware and Tools

# Hardware:

- Boolean Spartan-7 FPGA Development Board

- Slide Switches

- Push Buttons

- Multiplexed Seven-Segment Display

# Software and Design Tools:

- Xilinx Vivado

- Verilog HDL

# State Table
<img width="962" height="502" alt="image" src="https://github.com/user-attachments/assets/2508f890-965a-4d66-a13d-4a589011c254" />

# Input and Output Signals
<img width="771" height="693" alt="image" src="https://github.com/user-attachments/assets/887f26f3-679c-4189-92a9-18bc2f068a47" />

# Verilog RTL


## 💻 Verilog RTL

Save as voting_machine.v.
```verilog
module voting_machine(count_en,valid,count_a,count_b,count_c,clk,rst,start,vote_a,vote_b,vote_c);
output reg count_en;
output reg valid;
output reg [7:0]count_a;
output reg [7:0]count_b;
output reg [7:0]count_c;
input clk,rst,start;
input vote_a;
input vote_b;
input vote_c;
localparam idle=2'b00;
localparam voting=2'b01;
localparam register=2'b10;
localparam confirm=2'b11;
reg [1:0]ps,ns;
reg [3:0]selected_candidate;

always@(posedge clk or posedge rst)
begin
  if(rst)
  ps<=idle;
  else
  ps<=ns;
end

always@(*)
begin
  case(ps)
  idle:
    begin
    if(start==1)
    ns=voting;
    else
    ns=idle;
    end
  voting:
    begin
    if(vote_a && !vote_b && !vote_c)
    ns=register;
    else if(!vote_a && vote_b && !vote_c)
    ns=register;
    else if(!vote_a && !vote_b && vote_c)
    ns=register;
    else
    ns=voting;
    end
  register:
    begin
    ns=confirm;
    end
  confirm:
    begin
    ns=idle;
    end
    default:
    begin
    ns=idle;
    end
  endcase
end

always@(posedge clk or posedge rst)
begin
  if(rst)
  selected_candidate<=4'b0000;
  else if(ps==voting)
  begin
  if(vote_a && !vote_b && !vote_c)
  selected_candidate<=4'b0001;
  else if(!vote_a && vote_b && !vote_c)
  selected_candidate<=4'b0010;
  else if(!vote_a && !vote_b && vote_c)
   selected_candidate<=4'b0011;
   end
 end

 always@(posedge clk or posedge rst)
begin
  if(rst)
  begin
  count_a<=0;
  count_b<=0;
  count_c<=0;
  end
  else if(ps==register)
  begin
   case(selected_candidate)
     4'b0001:count_a<=count_a+1'b1;
     4'b0010:count_b<=count_b+1'b1;
     4'b0011:count_c<=count_c+1'b1;
     default:
       begin
       count_a<=count_a;
       count_b<=count_b;
       count_c<=count_c;
       end
   endcase
  end
end

always@(*)
begin
  count_en=0;
  valid=0;
  if(ps==register)
   count_en=1;
  if(ps==confirm)
  valid=1;
end
endmodule
```


# Testbench
## 🧪 Testbench

Save as `voting_machine_tb.v`.

```verilog
module voting_machine_tb;
wire count_en;
wire valid;
wire [7:0]count_a;
wire [7:0]count_b;
wire [7:0]count_c;
reg clk,rst,start;
reg vote_a;
reg vote_b;
reg vote_c;
voting_machine uut(
.count_en(count_en),.rst(rst),.valid(valid),.count_a(count_a),.count_b(count_b),.count_c(count_c),
.clk(clk),.start(start),.vote_a(vote_a),. vote_b( vote_b),.vote_c(vote_c));

always #5 clk = ~clk;
initial begin
$monitor($time ,"clk=%d,start=%d,rst=%d,count_a=%b,count_b=%b,count_c=%b",clk,start,rst,count_a,count_b,count_c);
    clk = 0;
    rst = 1;
    start = 0;
    vote_a = 0;
    vote_b = 0;
    vote_c = 0;
#10 rst=0;
 #10;
    start = 1;

 #10;
    start = 0;

 #10;
    vote_a = 1;

 #10;
    vote_a = 0;

#30;
    // VOTE FOR CANDIDATE B
start = 1;

 #10;
    start = 0;

 #10;
    vote_b = 1;

 #10;
    vote_b = 0;

 #30;
 // VOTE FOR CANDIDATE C
  start = 1;

 #10;
    start = 0;

 #10;
    vote_c = 1;

 #10;
    vote_c = 0;

 #30;
$finish;
end 

endmodule
```


🚀 Future Improvements

- Voter Authentication: Add a secure voter identification system.

- Vote Verification: Provide confirmation after successful vote registration.

- Result Display: Display the final election results automatically.

- Data Security: Improve vote data protection and prevent unauthorized access.

- Enhanced Display: Add an LCD or OLED display for better user interaction.

  ## 🎓 Learning Outcomes

Through this project, I explored:

- Designing an FSM using Verilog HDL.
- Implementing state transitions for vote registration.
- Understanding sequential logic and counters.
- Displaying vote counts using a seven-segment display.
- Simulating and debugging RTL using Xilinx Vivado.
- Implementing digital designs on a Spartan-7 FPGA board.

## 🏁 Conclusion

Through this project, I successfully designed and implemented an FPGA-based Electronic Voting Machine using Verilog HDL and FSM.

The system enables vote registration, accurate vote counting, and real-time display of results.

This project strengthened my understanding of digital design, FSM implementation, and FPGA development.

**A small step towards smarter digital systems!**
