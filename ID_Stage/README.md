# ID Stage (Instruction Decode)
module id_stage( input logic clk, input logic reset, input logic stall, input logic [31:0] instruction, input logic [31:0] wb_data, // data to write back (from the WB stage) output logic [31:0] rs1_data, output logic [31:0] rs2_data, output logic [31:0] imm_out, output logic [6:0] opcode_out_d, output logic [6:0] imm11_5, output logic [2:0] fn3_out_d, output logic fn7_5, output logic [2:0] aluop, output logic [1:0] memtoreg, output logic branch, output logic mem_write, output logic mem_read, output logic alu_src, output logic mux_inp );

// Internal wires
logic [4:0]  rs1, rs2, rd;
logic [6:0]  opcode;
logic [2:0]  fn3;
logic [11:0] imm;
logic [19:0] imm_uj;
logic        reg_write;
logic        reg_write_gated;

// Pass-through outputs from decoder
assign opcode_out_d = opcode;
assign fn3_out_d    = fn3;

// Gated register write enable: no register is written while stalled
assign reg_write_gated = reg_write && !stall;

// 1. Instruction Decoder
decoder u_decoder (
    .instruction (instruction),
    .rs1         (rs1),
    .rs2         (rs2),
    .rd          (rd),
    .opcode      (opcode),
    .imm11_5     (imm11_5),
    .fn3         (fn3),
    .imm         (imm),
    .imm_uj      (imm_uj),
    .fn7_5       (fn7_5)
);

// 2. Immediate Generator
imm_gen u_imm_gen (
    .imm         (imm),
    .imm_uj      (imm_uj),
    .opcode      (opcode),
    .imm_out     (imm_out)
);

// 3. Register File
register_file u_register_file (
    .clk         (clk),
    .reset       (reset),
    .reg_write   (reg_write_gated),
    .rs1_addr    (rs1),
    .rs2_addr    (rs2),
    .rd_addr     (rd),
    .rd_data     (wb_data),
    .rs1_data    (rs1_data),
    .rs2_data    (rs2_data)
);

// 4. Main Control
main_control u_main_control (
    .opcode      (opcode),
    .branch      (branch),
    .memtoreg    (memtoreg),
    .aluop       (aluop),
    .mem_write   (mem_write),
    .mem_read    (mem_read),
    .alu_src     (alu_src),
    .mux_inp     (mux_inp),
    .reg_write   (reg_write)
);
endmodule

// ============================================================================ // Immediate Generator Module // ============================================================================ module imm_gen ( input logic [11:0] imm, input logic [19:0] imm_uj, input logic [6:0] opcode, output logic [31:0] imm_out ); always_comb begin unique case (opcode) 7'b0010011, // I-type ALU 7'b0000011, // Load 7'b1100111: // JALR imm_out = {{20{imm[11]}}, imm};

        7'b0100011: // Store
            imm_out = {{20{imm[11]}}, imm};

        7'b1100011: // Branch
            imm_out = {{20{imm[11]}}, imm};

        7'b1101111: // JAL
            imm_out = {{12{imm_uj[19]}}, imm_uj};

        7'b0110111, // LUI
        7'b0010111: // AUIPC
            imm_out = {imm_uj, 12'b0};

        default:
            imm_out = 32'b0;
    endcase
end
endmodule

// ============================================================================ // Register File Module // ============================================================================ module register_file ( input logic clk, input logic reset, input logic reg_write, input logic [4:0] rs1_addr, input logic [4:0] rs2_addr, input logic [4:0] rd_addr, input logic [31:0] rd_data, output logic [31:0] rs1_data, output logic [31:0] rs2_data ); logic [31:0] rf [31:0];

// Read ports (Asynchronous read, x0 is hardwired to 0)
assign rs1_data = (rs1_addr == 5'b0) ? 32'b0 : rf[rs1_addr];
assign rs2_data = (rs2_addr == 5'b0) ? 32'b0 : rf[rs2_addr];

// Write port (Synchronous write)
always_ff @(posedge clk or posedge reset) begin
    if (reset) begin
        integer i;
        for (i = 0; i < 32; i = i + 1) begin
            rf[i] <= 32'b0;
        end
    end else if (reg_write && (rd_addr != 5'b0)) begin
        rf[rd_addr] <= rd_data;
    end
end
endmodule

// ============================================================================ // Main Control Module // ============================================================================ module main_control ( input logic [6:0] opcode, output logic branch, output logic [1:0] memtoreg, output logic [2:0] aluop, output logic mem_write, output logic mem_read, output logic alu_src, output logic mux_inp, output logic reg_write ); always_comb begin // Default control signals branch = 1'b0; memtoreg = 2'b00; aluop = 3'b000; mem_write = 1'b0; mem_read = 1'b0; alu_src = 1'b0; mux_inp = 1'b0; reg_write = 1'b0;

    case (opcode)
        7'b0110011: begin // R-type
            reg_write = 1'b1;
            aluop     = 3'b000;
        end
        7'b0010011: begin // I-type ALU
            reg_write = 1'b1;
            alu_src   = 1'b1;
            aluop     = 3'b001;
        end
        7'b0000011: begin // Load
            reg_write = 1'b1;
            alu_src   = 1'b1;
            mem_read  = 1'b1;
            memtoreg  = 2'b01;
            aluop     = 3'b010;
        end
        7'b0100011: begin // Store
            alu_src   = 1'b1;
            mem_write = 1'b1;
            aluop     = 3'b011;
        end
        7'b1100011: begin // Branch
            branch    = 1'b1;
            aluop     = 3'b100;
        end
        7'b1101111: begin // JAL
            reg_write = 1'b1;
            memtoreg  = 2'b10;
            aluop     = 3'b110;
        end
        7'b1100111: begin // JALR
            reg_write = 1'b1;
            alu_src   = 1'b1;
            mux_inp   = 1'b1;
            memtoreg  = 2'b10;
            aluop     = 3'b110;
        end
        7'b0110111, // LUI
        7'b0010111: begin // AUIPC
            reg_write = 1'b1;
            alu_src   = 1'b1;
            aluop     = 3'b110;
        end
        default: begin
            // Unsupported instructions
        end
    endcase
end
endmodule
