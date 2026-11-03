# Decoder
module decoder ( input logic [31:0] instruction, output logic [4:0] rs1, output logic [4:0] rs2, output logic [4:0] rd, output logic [6:0] opcode, output logic [6:0] imm11_5, // instruction[31:25] (I-type ALU only) output logic [2:0] fn3, output logic [11:0] imm, // I / S / B immediate bits output logic [19:0] imm_uj, // U / J immediate bits output logic fn7_5 // instruction[30] (R-type only) );

// Always extract the opcode
assign opcode = instruction[6:0];

// RISC-V Opcodes
localparam logic [6:0] OP_R     = 7'b0110011;
localparam logic [6:0] OP_I_ALU = 7'b0010011;
localparam logic [6:0] OP_LOAD  = 7'b0000011;
localparam logic [6:0] OP_STORE = 7'b0100011;
localparam logic [6:0] OP_BR    = 7'b1100011;
localparam logic [6:0] OP_JAL   = 7'b1101111;
localparam logic [6:0] OP_JALR  = 7'b1100111;
localparam logic [6:0] OP_LUI   = 7'b0110111;
localparam logic [6:0] OP_AUIPC = 7'b0010111;

always_comb begin
    // Default all other outputs to 0
    rs1     = '0;
    rs2     = '0;
    rd      = '0;
    imm11_5 = '0;
    fn3     = '0;
    imm     = '0;
    imm_uj  = '0;
    fn7_5   = '0;

    case (opcode)
        OP_R: begin
            rd    = instruction[11:7];
            fn3   = instruction[14:12];
            rs1   = instruction[19:15];
            rs2   = instruction[24:20];
            fn7_5 = instruction[30];
        end

        OP_I_ALU: begin
            rd      = instruction[11:7];
            fn3     = instruction[14:12];
            rs1     = instruction[19:15];
            imm     = instruction[31:20];
            imm11_5 = instruction[31:25];
        end

        OP_LOAD: begin
            rd    = instruction[11:7];
            fn3   = instruction[14:12];
            rs1   = instruction[19:15];
            imm   = instruction[31:20];
        end

        OP_STORE: begin
            fn3   = instruction[14:12];
            rs1   = instruction[19:15];
            rs2   = instruction[24:20];
            imm   = {instruction[31:25], instruction[11:7]};
        end

        OP_BR: begin
            fn3   = instruction[14:12];
            rs1   = instruction[19:15];
            rs2   = instruction[24:20];
            imm   = {instruction[31], instruction[7], instruction[30:25], instruction[11:8]};
        end

        OP_JAL: begin
            rd     = instruction[11:7];
            imm_uj = {instruction[31], instruction[19:12], instruction[20], instruction[30:21]};
        end

        OP_JALR: begin
            rd    = instruction[11:7];
            fn3   = instruction[14:12];
            rs1   = instruction[19:15];
            imm   = instruction[31:20];
        end

        OP_LUI, OP_AUIPC: begin
            rd     = instruction[11:7];
            imm_uj = instruction[31:12];
        end

        default: begin
            // Unsupported instructions leave everything except opcode as 0
        end
    endcase
end
endmodule
