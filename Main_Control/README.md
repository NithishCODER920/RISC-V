# Main Control
module main_control ( input logic [6:0] opcode, input logic [2:0] fn3, output logic [1:0] memtoreg, // write-back select output logic branch, output logic mux_inp, // 1 = JALR (target comes from the ALU) output logic memread, output logic memwrite, output logic alusrc, // 0 = rs2, 1 = immediate output logic reg_write, output logic [2:0] aluop // operation class for the ALU control );

always_comb begin
    // Default control signals (other / unsupported)
    memtoreg  = 2'b00;
    branch    = 1'b0;
    mux_inp   = 1'b0;
    memread   = 1'b0;
    memwrite  = 1'b0;
    alusrc    = 1'b0;
    reg_write = 1'b0;
    aluop     = 3'b000;

    case (opcode)
        7'b0110011: begin // R-type
            reg_write = 1'b1;
            aluop     = 3'b000;
        end
        7'b0010011: begin // I-type ALU
            alusrc    = 1'b1;
            reg_write = 1'b1;
            aluop     = 3'b001;
        end
        7'b0000011: begin // Load
            memtoreg  = 2'b01;
            memread   = 1'b1;
            alusrc    = 1'b1;
            reg_write = 1'b1;
            aluop     = 3'b010;
        end
        7'b0100011: begin // Store
            alusrc    = 1'b1;
            memwrite  = 1'b1;
            aluop     = 3'b011;
        end
        7'b1100011: begin // Branch
            branch    = 1'b1;
            aluop     = 3'b100;
        end
        7'b1101111: begin // JAL
            memtoreg  = 2'b10;
            alusrc    = 1'b1;
            reg_write = 1'b1;
            aluop     = 3'b110;
        end
        7'b1100111: begin // JALR
            memtoreg  = 2'b10;
            mux_inp   = 1'b1;
            alusrc    = 1'b1;
            reg_write = 1'b1;
            aluop     = 3'b110;
        end
        7'b0110111, // LUI
        7'b0010111: begin // AUIPC
            alusrc    = 1'b1;
            reg_write = 1'b1;
            aluop     = 3'b110;
        end
        default: begin
            // other / unsupported
        end
    endcase
end
endmodule

