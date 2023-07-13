# 8x8 Bit Sequential Multiplier

The 8x8 Bit Sequential Multiplier is a Verilog design that implements an 8x8 bit multiplier using a sequential approach. This project aims to provide an efficient and modular solution for performing multiplication operations on 8-bit operands, resulting in a 16-bit product. The design supports various features, including multi-cycle computation, output signaling, and efficient pin utilization.

## Sequential Multiplier Specification


The project's goal is to build an 8x8 multiplier that takes two 8-bit multiplicands (`dataa` and `datab`) as inputs and produces a 16-bit product (`product8x8_out`). Additionally, the design provides a done flag (`done_flag`) and seven signals (`seg_a`, `seg_b`, `seg_c`, `seg_d`, `seg_e`, `seg_f`, and `seg_g`) to drive a 7-segment display. The multiplier operates in a multi-cycle fashion, requiring four clock cycles to complete the full multiplication.

### Functionality

The multiplier utilizes a sequential approach to perform the multiplication operation. During each of the four clock cycles, a pair of 4-bit slices from `dataa` and `datab` is multiplied by a 4x4 multiplier. The resulting 4-bit slices are then accumulated to obtain the final 16-bit product. On the fifth cycle, the fully composed 16-bit product is available at the output (`product8x8_out`).


<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/f03c920e-35ae-48f7-a7c8-aca57da20f7f" alt="Binary multiplier">
  <br>
  <em>Binary multiplier</em>
</p>


## Mathematical Principle

The multiplier's functionality is based on the mathematical principles of multiplication. It breaks down the 8-bit multiplicands `a` and `b` into 4-bit slices and performs partial multiplications. The final 16-bit product is obtained by summing the individual partial products according to the following equation:


$$
\text{result}[15..0] &= a[7..0] \times b[7..0] &= ((a[7..4] \times b[7..4]) \times 2^8) + ((a[7..4] \times b[3..0]) \times 2^4) + ((a[3..0] \times b[7..4]) \times 2^4) + (a[3..0] \times b[3..0] \times 2^0)
$$
### Shift-and-Add Multiplication

When designing multipliers, there is always a trade-off between the speed of the multiplication process and the hardware resources used for its implementation. One simple multiplication method that is slow but efficient in terms of hardware utilization is the shift-and-add method.

In the shift-and-add method, the multiplication is performed by considering each bit of one operand (let's call it A) and determining whether to add the other operand (B) to the accumulated partial result, followed by a right shift operation, or to perform only a right shift operation without the addition.

This method is justified by considering how binary multiplication is done manually. When manually multiplying two 8-bit binary numbers, we start by considering the bits of A from right to left. If a bit value is 0, we select 00000000 to be added with the next partial product. If the bit value is 1, we select the value of B. This process repeats, with each selection (00000000 or B) being written one place to the left with respect to the previous value. Once all bits of A are considered, we add all the calculated values to obtain the final multiplication result.

To facilitate the hardware implementation of this procedure, certain modifications are made. First, instead of moving the observation point from one bit of A to another, A is placed in a shift register, allowing the right-most bit to be observed. After each calculation, the shift register is shifted one place to the right, making the next bit accessible.

Second, for the partial products, instead of writing one partial product and the next one to its left, the partial product is moved to the right as it is written, eliminating the need for subsequent shifting.

Finally, instead of calculating all the partial products and adding them up at the end, each partial product is added to the previous partial result, and the newly calculated value becomes the new partial result.

In this modified procedure, if the observed bit of A is 0, 00000000 is added to the previously calculated partial result, and the new value is shifted one place to the right. Since the value being added is 00000000, the addition operation is not necessary, and only shifting the partial result is required. This operation is referred to as a "shift". However, if the observed bit of A is 1, B is added to the previously calculated partial result, and the resulting sum is shifted one place to the right. This is known as an "add-and-shift" operation.

By repeating the above procedure until all bits of A are shifted out, the partial result becomes the final multiplication result. To illustrate this procedure, let's consider a 4-bit example. Figure 11.3 shows the initial state, where A = 1001 and B = 1101 are the operands to be multiplied. At time 0, A is in a shift register with a register for partial results (P) on its left.

<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/5cf3ea8a-c4d0-45c3-b48c-b6bf53a121af" alt="Manual binary multiplication">
  <br>
  <em>Manual binary multiplication</em>
</p>

<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/0296cb44-4010-452b-97f9-3c59fc1729bf" alt="Hardware binary multiplication">
  <br>
  <em>Hardware binary multiplication</em>
</p>

### Hardware-Oriented Multiplication Process

The 8x8 bit sequential multiplier design implements a hardware-oriented multiplication process. Let's explore the steps involved in this process:

1. At time 0, because A[0] is 1, the partial sum of B + P is calculated. This partial sum, represented as 01101 (shown in the upper part of time 1), consists of 5 bits, including a carry. The rightmost bit of this partial sum is shifted into the A register, replacing the old value of A. The remaining bits of the partial sum replace the old value of P. During this shifting operation, a 0 is moved into the A[0] position.

2. At time 1, the new value in A is observed. Since A[0] is 0, the calculation changes to 0000 + P instead of B + P. This results in a new value of 00110. The rightmost bit of this value is shifted into A, while the other bits replace the old value of P.

3. This process repeats for a total of 4 cycles. During each cycle, the multiplication process proceeds as described above. At the end of the fourth cycle, both the A and P registers hold the final multiplication result. The least significant 4 bits of the result are stored in A, while the most significant bits are stored in P.

For example, consider the operation of multiplying 9 and 13. By following the hardware-oriented multiplication process, the result obtained is 117.

This hardware-oriented approach efficiently performs multiplication in the 8x8 bit sequential multiplier, gradually accumulating the result in the A and P registers over multiple cycles.

<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/053b19f3-4d3a-4370-8e96-2c1fc8f947b6" alt="data and control">
  <br>
  <em>Data and Control</em>
</p>

## Sequential Multiplier Design
### Control Data Partitioning
The multiplier consists of two main components: the datapath and the controller. The datapath encompasses registers, logic units, and interconnecting buses, while the controller acts as a state machine responsible for generating control signals that govern the flow of data into the registers.

Both the datapath registers and the controller are synchronized by a common clock signal. Upon the rising edge of the clock, the controller transitions to a new state. In this state, it issues multiple control signals that trigger various actions within the datapath components. The datapath requires a certain amount of time, from one clock edge to the next, for all activities to settle and stabilize.

During this time period, values that need to be processed within the datapath are clocked into the corresponding registers with each clock edge. This ensures the proper synchronization and orderly progression of data throughout the multiplier.

<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/579e12a0-5a2a-4a7d-85ab-badafa51a20c" alt="Timing">
  <br>
  <em>Timing</em>
</p>


### Multiplier Datapath and Controller

<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/cb2b5981-6e38-4951-a2de-af3769700732" alt="8x8 Sequential Multiplier">
  <br>
  <em>8x8 Sequential Multiplier</em>
</p>

<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/c827363b-fd6a-4bc7-8a68-3aeb35f4e387" alt="8x8 Sequential Multiplier Parts">
  <br>
  <em>8x8 Sequential Multiplier Parts</em>
</p>

#### Adder 16-Bit
```
module adder(
  input [15:0] dataa, datab,
  output [15:0] sum
);

  assign sum = dataa + datab;

endmodule
```
<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/6fe111f6-d24d-45d5-adc9-c2a4fa8f8830" alt="adder">
</p>

#### 4x4 Multiplier
```
module mult4x4(
  input [3:0] dataa,
  input [3:0] datab,
  output [7:0] product
);

  assign product = dataa * datab;

endmodule
```
<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/43ace1d4-3ac6-4f4a-8aeb-c70afadac861" alt="multi">
</p>

### Multiplexer
```
module mux4 (
input [3:0] mux_in_a, mux_in_b,
input [1:0] mux_sel,
output reg [3:0] mux_out
);

always @ (*)
begin
	if (mux_sel == 0) begin
	mux_out = mux_in_a;
	end
	else begin
	mux_out = mux_in_b;
	end
end

endmodule
```
<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/914b86aa-58e0-4c7d-b838-849e77e249b7" alt="mux">
</p>

### Shifter
- If shift_cntrl = 0 or 3, then no shift
- If shift_cntrl =1, then 4-bit shift left
- If shift_cntrl = 2, then 8-bit shift left

```
module shifter (
  input [7:0] inp,
  input [1:0] shift_cntrl,
  output reg [15:0] shift_out
);


  always @(*) begin
    case(shift_cntrl)
      2'b00: shift_out = {8'b0000_0000, inp};
      2'b01: shift_out = {4'b0000, inp, 4'b0000};
      2'b10: shift_out = {inp, 8'b0000_0000};
      2'b11: shift_out = {8'b0000_0000, inp};
    endcase
  end

endmodule
```
<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/b06a6c73-95ee-4c21-ac3c-0fd5f13964f1" alt="shift">
</p>


#### Synchronous 16-bit register
- 16-bit register where sclr_n is a reset signal and clk_ena is a clock enable signal
- If clk_ena is high and sclr_n is low then the output of register is cleared.
- If clk_ena is high and sclr_n is high then output of register is set equal to its input.
- If clk_ena is low then do nothing.
  
```
module reg16(
input clk, sclr_n, clk_ena,
input [15:0] datain,
output reg[15:0] reg_out
);

always @(posedge clk) begin
 
  2'b00 : reg_out = reg_out;
  2'b01 : reg_out = reg_out;
  2'b10 : reg_out = 16'b 0000_0000_0000_0000;
  2'b11 : reg_out = datain ;
endcase

end

endmodule
```
<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/a9ca6bb6-b094-4eba-bc15-b3304d626c7e" alt="reg">
</p>

#### Counter with asynchronous control
- 2-bit counter where aclr_n is a reset signal 
- If aclr_n is low, then counter goes to 00 immediately
- If aclr_n is high, output of counter increments by 1 on every rising edge clock

```
module counter (
  input wire clk,
  input wire aclr_n,
  output reg [1:0] count_out
);

  always @(posedge clk or negedge aclr_n) begin
    if (!aclr_n)
      count_out <= 2'b00;
    else
      count_out <= count_out + 1;
  end

endmodule
```
<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/251c1ba5-514b-4034-8ddc-0757ba2bcb4b" alt="reg">
</p>

#### Seven-Segment Display Decoder
<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/778d1fca-cbb4-4f75-9ac1-459e9b67e1d8" alt="table">
</p>

```
module seven_segment_cntrl (
input [2:0] inp,
output reg seg_a, seg_b, seg_c, seg_d, seg_e, seg_f, seg_g
);

always @(*) begin

  case (inp)
  
3'b000: {seg_a, seg_b, seg_c, seg_d, seg_e, seg_f, seg_g} = 7'b1111110; //0
3'b001: {seg_a, seg_b, seg_c, seg_d, seg_e, seg_f, seg_g} = 7'b0110000; //1
3'b010: {seg_a, seg_b, seg_c, seg_d, seg_e, seg_f, seg_g} = 7'b1101101; //2
3'b011: {seg_a, seg_b, seg_c, seg_d, seg_e, seg_f, seg_g} = 7'b1111001; //3

default:
{seg_a, seg_b, seg_c, seg_d, seg_e, seg_f, seg_g} = 7'b1001111; 
  endcase

end

endmodule
```
<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/06c8b88d-4344-4dbf-8ae5-7db80586a62b" alt="7seg">
</p>

#### Multiplier controller
This state machine will manage all the operations that occur within the 8x8 multiplier using 6 defined states: `idle`, `lsb`, `mid`, `msb`, `calc_done`, and `err`.

The state machine in the LSB state multiplies the lowest 4 bits of the two 8-bit multiplicands ($a[3..0] \times b[3..0] \times 2^0$). This intermediate result is saved in an accumulator.

The state machine in the MID state performs cross multiplication ($a[3..0] \times b[7..4] \times 2^4$) and ($a[7..4] \times b[3..0] \times 2^4$). This is done in successive clock cycles. The products of both multiply operations are added to the content of the accumulator as they are completed and clocked back into the accumulator.

The state machine in the MSB state multiplies the highest 4 bits of the two 8-bit multiplicands ($a[7..4] \times b[7..4] \times 2^8$). This product is added with the content of the accumulator and clocked back into the accumulator. This result is the final product:

$$
\text{result}[15..0] = a[7..0] \times b[7..0] \\ = ((a[7..4] \times b[7..4]) \times 2^8) + ((a[7..4] \times b[3..0]) \times 2^4) + ((a[3..0] \times b[7..4]) \times 2^4) + (a[3..0] \times b[3..0] \times 2^0)
$$
The state machine in the CALC_DONE state asserts the `done_flag` output to indicate that the final product has been calculated and is ready for reading by downstream logic.

The state machine in the ERR state indicates that incorrect inputs have been received.

There are two inputs to the state machine: `start` and `count`. The `start` signal is asserted for one clock cycle to begin an 8x8 multiply operation on the next clock cycle. The `start` signal must only be asserted for one clock cycle. The `count` signal is used by the state machine to track the multiplication cycles.

The outputs of `mult_control` state machine control the other blocks in the design.
<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/bf9efa75-66ab-455c-8ed2-002a34a6e73c" alt="fsm">
</p>

```
module mult_control (
    input clk, reset_a, start,
    input [1:0] count,
    output reg done, clk_ena, sclr_n,
    output reg [2:0] state_out,
    output reg [1:0] input_sel, shift_sel
);

    // Define states
    parameter IDLE = 3'b000, LSB = 3'b001, MID = 3'b010, MSB = 3'b011, CALC_DONE = 3'b100, ERR = 3'b101;

    reg [2:0] state;

    always @(posedge clk) begin
        if (~reset_a) begin
            state <= IDLE;
            done <= 0;

        end else begin
            case (state)
                IDLE: begin
                    if (~start & ~count) begin
                        state <= LSB;
                        clk_ena <= 1;
                    end else begin
                        state <= IDLE;
                        clk_ena <= 0;
                    end
			done <= 0;
                        sclr_n <= 1; 
			state_out <= IDLE;
                end

                LSB: begin
                    if (start) begin
                        state <= MID;
                        clk_ena <= 1;
                        sclr_n <= 0;
                        input_sel <= 2'b00;
                        shift_sel <= 2'b00;
                    end else begin
                        state <= IDLE;
                        clk_ena <= 0;
                        sclr_n <= 1;
                    end
			done <= 0;
			state_out <= LSB;
                end

                MID: begin
                    if (count == 2'b10 & ~start) begin
                        state <= MSB;
                        clk_ena <= 1;
                        input_sel <= 2'b10;
                        shift_sel <= 2'b01;
                    end else if (count == 2'b01 & ~start) begin
                        state <= MID;
                        clk_ena <= 1;
                        input_sel <= 2'b01;
                        shift_sel <= 2'b01;
                    end else begin
                        state <= ERR;
                        clk_ena <= 0;
                    end
                        done <= 0;
                        sclr_n <= 1;
			state_out <= MID;
                end

                MSB: begin
                    if(count == 2'b11 & ~start) begin
                        state <= CALC_DONE;
                        clk_ena <= 1;
                        input_sel <= 2'b11;
                        shift_sel <= 2'b10;
                    end else begin
                        state <= ERR;
                        clk_ena <= 0;
                    end
                        done <= 0;
                        sclr_n <= 1;
			state_out <= MSB;
                end

                CALC_DONE: begin
                    if(~start) begin
                        state <= IDLE;
                        done <= 1;
                    end else if (start) begin
                        state <= ERR;
                        done <= 0;
                    end
                        clk_ena <= 0;
                        sclr_n <= 1;
			state_out <= CALC_DONE;
                end

                ERR: begin
                    if(~start) begin
                        state <= ERR;
                        clk_ena <= 0;
                        sclr_n <= 1;
                    end else if (start) begin
                        state <= LSB;
                        clk_ena <= 1;
                        sclr_n <= 0;
                    end
                        done <= 0;
			state_out <= ERR;
                end

                default: state <= IDLE;

            endcase
        end
    end
endmodule
```
<p align="center">
  <img src="https://github.com/islamibr/8x8_seq_mult/assets/49861069/a74f365e-5dc8-4fc9-b435-e4e0ca370d4f" alt="mult-ctrl">
</p>

#### Top-Level Code of the Multiplier
```
module mult8x8(
	input [7:0] dataa, datab,
	input start, reset_a, clk,
	output done_flag, seg_a, seg_b, seg_c, seg_d, seg_e, seg_f, seg_g,
	output [15:0] product8x8_out
);

wire clk_ena ,sclr_n  ;
wire [1:0] sel, shift, count;
wire [2:0] state_out;
wire[3:0] aout, bout;
wire [7:0] product;
wire [15:0] shift_out, sum;

mux4 A(
	.mux_in_a(dataa[3:0]),
	.mux_in_b(dataa[7:4]),
	.mux_sel(sel[1]),
	.mux_out(aout)
);

mux4 B(
	.mux_in_a(datab[3:0]),
	.mux_in_b(datab[7:4]),
	.mux_sel(sel[0]),
	.mux_out(bout[3:0])
);

mult4x4 MA(
	.dataa(aout[3:0]),
	.datab(bout[3:0]),
	.product(product[7:0])
);

shifter SA(
	.inp(product[7:0]),
	.shift_cntrl(shift[1:0]),
	.shift_out(shift_out[15:0])
);

adder AA(
	.dataa(shift_out[15:0]),
	.datab(product8x8_out[15:0]),
	.sum(sum[15:0])
);

reg16 RA(
	.clk(clk),
	.sclr_n(sclr_n),
	.clk_ena(clk_ena),
	.datain(sum[15:0]),
	.reg_out(product8x8_out[15:0])
);

counter CA(
	.aclr_n(~start),
	.clk(clk),
	.count_out(count[1:0])
);

mult_control MCA(
	.clk(clk),
	.reset_a(reset_a),
	.start(start),
	.count(count[1:0]),
	.input_sel(sel[1:0]),
	.shift_sel(shift[1:0]),
	.state_out(state_out[2:0]),
	.done(done_flag),
	.clk_ena(clk_ena),
	.sclr_n(sclr_n)
);

seven_segment_cntrl SSCA(
	.inp(state_out[2:0]),
	.seg_a(seg_a),
	.seg_b(seg_b),
	.seg_c(seg_c),
	.seg_d(seg_d),
	.seg_e(seg_e),
	.seg_f(seg_f),
	.seg_g(seg_g)
);

endmodule
```


