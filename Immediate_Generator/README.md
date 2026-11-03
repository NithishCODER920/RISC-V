# Immediate Generator
module imm_generator( input logic [11:0] imm_input, // decoder imm input logic [2:0] fn3, input logic [6:0] opcode, input logic [19:0] imm_input_uj, // decoder imm_uj output logic [31:0] imm_output );

// RISC-V Opcodes
localparam logic [6:0] OP_BRANCH  = 7'b1100011;
localparam logic [6:0] OP_JAL     = 7'b1101111;
localparam logic [6:0] OP_LUI     = 7'b0110111;
localparam logic [6:0] OP_AUIPC   = 7'b0010111;
localparam logic [6:0] OP_JALR    = 7'b1100111;
localparam logic [6:0] OP_I_ALU   = 7'b0010011;

always_comb begin
    case (opcode)
        OP_BRANCH: begin
            // sign_extend(imm_input) << 1
            imm_output = {{20{imm_input[11]}}, imm_input} << 1;
        end

        OP_JAL: begin
            // 20-bit sign extension of imm_input_uj, then << 1
            imm_output = {{12{imm_input_uj[19]}}, imm_input_uj} << 1;
        end

        OP_LUI, OP_AUIPC: begin
            imm_output = {imm_input_uj, 12'b0};
        end

        OP_I_ALU: begin
            if ((fn3 == 3'b001) || (fn3 == 3'b101)) begin
                // Shift amount (shamt) for SLLI/SRLI/SRAI
                imm_output = {27'b0, imm_input[4:0]};
            end else begin
                imm_output = {{20{imm_input[11]}}, imm_input};
            end
        end

        default: begin // Load, Store, R-type, JALR, etc.
            imm_output = {{20{imm_input[11]}}, imm_input};
        end
    endcase
end
endmodule
