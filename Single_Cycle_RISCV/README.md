# Single-Cycle RISC-V
module single_cycle_riscv ( input logic clk, input logic reset, input logic stall, input logic [31:0] instruction, // from instruction memory (addressed by address) input logic [31:0] mem_out, // from data memory output logic [31:0] rs2_data, // store data output logic [31:0] alu_out, // data memory address output logic mem_write, output logic mem_read, output logic [2:0] fn3, // load / store width output logic [6:0] opcode, output logic [31:0] address // PC );

// Internal wires connecting the datapath stages
logic        and_out_ex;
logic [31:0] pc_ex_out;
logic [31:0] pc_new;

logic [31:0] rs1_data_id;
logic [31:0] rs2_data_id;
logic [31:0] imm_out_id;
logic [6:0]  opcode_d;
logic [6:0]  imm11_5_d;
logic [2:0]  fn3_d;
logic        fn7_5_d;
logic [2:0]  aluop_d;
logic [1:0]  memtoreg_d;
logic        branch_d;
logic        mem_write_d;
logic        mem_read_d;
logic        alu_src_d;
logic        mux_inp_d;

logic [31:0] alu_out_ex;
logic [31:0] wb_data;

// Output assignments to core ports
assign rs2_data  = rs2_data_id;
assign alu_out   = alu_out_ex;
assign mem_write = mem_write_d;
assign mem_read  = mem_read_d;
assign fn3       = fn3_d;
assign opcode    = opcode_d;

// ====================================================================
// 1. Instruction Fetch (IF) Stage
// ====================================================================
if_stage u_if_stage (
    .clk              (clk),
    .reset            (reset),
    .stall            (stall),
    .and_out          (and_out_ex),
    .pc_signed_offset (pc_ex_out),
    .address          (address),
    .pc_new           (pc_new)
);

// ====================================================================
// 2. Instruction Decode (ID) Stage
// ====================================================================
id_stage u_id_stage (
    .clk          (clk),
    .reset        (reset),
    .stall        (stall),
    .instruction  (instruction),
    .wb_data      (wb_data),
    .rs1_data     (rs1_data_id),
    .rs2_data     (rs2_data_id),
    .imm_out      (imm_out_id),
    .opcode_out_d (opcode_d),
    .imm11_5      (imm11_5_d),
    .fn3_out_d    (fn3_d),
    .fn7_5        (fn7_5_d),
    .aluop        (aluop_d),
    .memtoreg     (memtoreg_d),
    .branch       (branch_d),
    .mem_write    (mem_write_d),
    .mem_read     (mem_read_d),
    .alu_src      (alu_src_d),
    .mux_inp      (mux_inp_d)
);

// ====================================================================
// 3. Execution (EX) Stage
// ====================================================================
ex_stage u_ex_stage (
    .rs1_data         (rs1_data_id),
    .rs2_data         (rs2_data_id),
    .imm_out          (imm_out_id),
    .address          (address),
    .branch           (branch_d),
    .alu_src          (alu_src_d),
    .fn7_5            (fn7_5_d),
    .mux_inp          (mux_inp_d),
    .fn3              (fn3_d),
    .imm11_5          (imm11_5_d),
    .aluop            (aluop_d),
    .opcode           (opcode_d),
    .and_out_ex       (and_out_ex),
    .alu_out          (alu_out_ex),
    .pc_ex_out        (pc_ex_out)
);

// ====================================================================
// 4. Write-Back (WB) Multiplexer Logic
// ====================================================================
always_comb begin
    if (opcode_d == 7'b0110111) begin         // LUI
        wb_data = imm_out_id;
    end else if (opcode_d == 7'b0010111) begin // AUIPC
        wb_data = pc_ex_out;
    end else if (memtoreg_d == 2'b01) begin    // Load
        wb_data = mem_out;
    end else if (memtoreg_d == 2'b10) begin    // JAL / JALR (Return Address PC + 4)
        wb_data = pc_new;
    end else begin                             // R-type / I-type ALU
        wb_data = alu_out_ex;
    end
end
endmodule
