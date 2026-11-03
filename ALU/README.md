# ALU
module alu #(parameter int DATA_WIDTH = 32)( input logic [4:0] alu_control, input logic [DATA_WIDTH-1:0] rs1_data, input logic [DATA_WIDTH-1:0] rs2_data, output logic [DATA_WIDTH-1:0] alu_out, output logic zero // branch condition true );

// Internal signed representations for signed comparisons and shifts
logic signed [DATA_WIDTH-1:0] rs1_signed;
logic signed [DATA_WIDTH-1:0] rs2_signed;

assign rs1_signed = rs1_data;
assign rs2_signed = rs2_data;

always_comb begin
    // Default outputs
    alu_out = '0;
    zero    = 1'b0;

    casez (alu_control)
        5'b00000: begin // ADD
            alu_out = rs1_data + rs2_data;
        end
        5'b00001: begin // SUB
            alu_out = rs1_data - rs2_data;
        end
        5'b00010: begin // XOR
            alu_out = rs1_data ^ rs2_data;
        end
        5'b00011: begin // OR
            alu_out = rs1_data | rs2_data;
        end
        5'b00100: begin // AND
            alu_out = rs1_data & rs2_data;
        end
        5'b00101: begin // SLL (Shift Left Logical)
            alu_out = rs1_data << rs2_data[4:0];
        end
        5'b00110: begin // SRL (Shift Right Logical)
            alu_out = rs1_data >> rs2_data[4:0];
        end
        5'b00111: begin // SRA (Shift Right Arithmetic)
            alu_out = rs1_signed >>> rs2_data[4:0];
        end
        5'b01000: begin // SLT (Set Less Than - Signed)
            alu_out = (rs1_signed < rs2_signed) ? {{DATA_WIDTH-1{1'b0}}, 1'b1} : '0;
        end
        5'b01001: begin // SLTU (Set Less Than - Unsigned)
            alu_out = (rs1_data < rs2_data) ? {{DATA_WIDTH-1{1'b0}}, 1'b1} : '0;
        end
        5'b01010: begin // BEQ (Branch Equal)
            alu_out = '0;
            zero    = (rs1_data == rs2_data);
        end
        5'b01011: begin // BNE (Branch Not Equal)
            alu_out = '0;
            zero    = (rs1_data != rs2_data);
        end
        5'b01100: begin // BLT (Branch Less Than - Signed)
            alu_out = '0;
            zero    = (rs1_signed < rs2_signed);
        end
        5'b01101: begin // BGE (Branch Greater Than or Equal - Signed)
            alu_out = '0;
            zero    = (rs1_signed >= rs2_signed);
        end
        5'b01110: begin // BLTU (Branch Less Than - Unsigned)
            alu_out = '0;
            zero    = (rs1_data < rs2_data);
        end
        5'b01111: begin // BGEU (Branch Greater Than or Equal - Unsigned)
            alu_out = '0;
            zero    = (rs1_data >= rs2_data);
        end
        default: begin // 1xxxx - Unused
            alu_out = '0;
            zero    = 1'b0;
        end
    endcase
end
endmodule
