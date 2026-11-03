# IF Stage (Instruction Fetch)
module if_stage ( input logic clk, input logic reset, // Asynchronous, active high input logic stall, // Hold the PC input logic and_out, // 1 = take branch/jump target input logic [31:0] pc_signed_offset, // Branch / jump target address output logic [31:0] address, // Current PC output logic [31:0] pc_new // PC + 4 );

// Calculate PC + 4
assign pc_new = address + 32'd4;

// Next PC selection: branch/jump target if and_out is high, otherwise pc_new
logic [31:0] next_pc;
assign next_pc = and_out ? pc_signed_offset : pc_new;

// Program Counter register with asynchronous active-high reset and stall control
always_ff @(posedge clk or posedge reset) begin
    if (reset) begin
        address <= 32'h8000_0000;
    end else if (!stall) begin
        address <= next_pc;
    end
    // If stall is high, address holds its current value automatically
end
endmodule
