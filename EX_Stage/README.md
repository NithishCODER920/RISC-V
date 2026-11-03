# EX Stage (Execute)
module ex_stage ( input logic [31:0] rs1_data, input logic [31:0] rs2_data, input logic [31:0] imm_out, input logic [31:0] address, // current PC input logic branch, input logic alu_src, // 0 = rs2_data, 1 = imm_out input logic fn7_5, input logic mux_inp, // 1 = JALR input logic [2:0] fn3, input logic [6:0] imm11_5, input logic [2:0] aluop, input logic [6:0] opcode, output logic and_out_ex, // branch / jump taken output logic [31:0] alu_out, output logic [31:0] pc_ex_out // branch / jump target );

// Internal signals
logic [4:0]  alu_ctrl;
logic [31:0] alu_operand_b;
logic        zero;
logic [31:0] pc_signed_offset;

// ALU operand B selection: 1 = imm_out, 0 = rs2_data
assign alu_operand_b = alu_src ? imm_out : rs2_data;

// Instantiate ALU Control Decoder
alu_control u_alu_control (
    .alu_op      (aluop),
    .fn3         (fn3),
    .imm11_5     (imm11_5),
    .fn7_5       (fn7_5),
    .control_out (alu_ctrl)
);

// Instantiate ALU
alu #(
    .DATA_WIDTH(32)
) u_alu (
    .alu_control (alu_ctrl),
    .rs1_data    (rs1_data),
    .rs2_data    (alu_operand_b),
    .alu_out     (alu_out),
    .zero        (zero)
);

// PC target calculations
assign pc_signed_offset = address + imm_out;
assign pc_ex_out        = mux_inp ? {alu_out[31:1], 1'b0} : pc_signed_offset;

// Branch / Jump Taken logic (JAL = 7'h6F, JALR = 7'h67)
assign and_out_ex       = (branch && zero) || (opcode == 7'h6F) || (opcode == 7'h67);
endmodule
