# BOARD TOP 
This module is used to connect switches and led with the fgpa board and there are around 70 input and output but we consider only important input and output 

```verilog
`timescale 1ns / 1ps

module board_top (
    input  wire        clk,     // 100 MHz
    input  wire        btnC,    // reset
    input  wire        btnU,    // send
    input  wire        btnD,    // hold = status on LEDs
    input  wire [15:0] sw,      // 16-bit data input
    output wire [15:0] led      // 16-bit received data
);

    // ---- button synchronizers ----
    reg [1:0] rst_s = 2'b11, up_s = 2'b00, dn_s = 2'b00;
    reg       up_prev = 1'b0;
    always @(posedge clk) begin
        rst_s   <= {rst_s[0], btnC};
        up_s    <= {up_s[0],  btnU};
        dn_s    <= {dn_s[0],  btnD};
        up_prev <= up_s[1];
    end
    wire rst_n       = ~rst_s[1];
    wire start_pulse = up_s[1] & ~up_prev;

    // ---- serializer: 16 switches -> 1 bit per clock ----
    localparam IDLE = 2'd0, SEND = 2'd1, COOL = 2'd2;
    reg [1:0]  state;
    reg [4:0]  idx;
    reg [21:0] cool_cnt;
    reg [15:0] tx_word;
    reg        tx_data, tx_valid;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            state <= IDLE; idx <= 5'd15; cool_cnt <= 22'd0;
            tx_word <= 16'd0; tx_data <= 1'b0; tx_valid <= 1'b0;
        end else begin
            case (state)
                IDLE: begin
                    tx_valid <= 1'b0;
                    idx      <= 5'd15;
                    if (start_pulse) begin
                        tx_word <= sw;
                        state   <= SEND;
                    end
                end
                SEND: begin
                    tx_data  <= tx_word[idx[3:0]];
                    tx_valid <= 1'b1;
                    if (idx == 5'd0) state <= COOL;
                    else             idx   <= idx - 1'b1;
                end
                COOL: begin
                    tx_valid <= 1'b0;
                    cool_cnt <= cool_cnt + 1'b1;
                    if (&cool_cnt) state <= IDLE;
                end
                default: state <= IDLE;
            endcase
        end
    end

    // ---- your OFDM design (unchanged) ----
    wire [1:0] demap_2bit;
    wire       demap_valid;

    ofdm_top u_ofdm (
        .clk(clk), .rst_n(rst_n),
        .tx_data_in(tx_data), .tx_valid_in(tx_valid),
        .tx_ifft_out_tdata(), .tx_ifft_out_tvalid(), .tx_ifft_out_tlast(),
        .rx_fft_out_tdata(),  .rx_fft_out_tvalid(),  .rx_fft_out_tlast(),
        .rx_fft_I_out(), .rx_fft_Q_out(),
        .mapped_I_out(), .mapped_Q_out(),
        .rx_demap_2bit_out(demap_2bit),
        .rx_demap_valid(demap_valid)
    );

    // ---- collect 8 symbols -> 16 bits ----
    reg [15:0] rx_bits;
    reg [3:0]  rx_cnt;
    reg        done;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            rx_bits <= 16'd0; rx_cnt <= 4'd0; done <= 1'b0;
        end else if (state == IDLE && start_pulse) begin
            rx_bits <= 16'd0; rx_cnt <= 4'd0; done <= 1'b0;
        end else if (demap_valid && rx_cnt < 4'd8) begin
            rx_bits <= {rx_bits[13:0], demap_2bit};
            rx_cnt  <= rx_cnt + 1'b1;
            if (rx_cnt == 4'd7) done <= 1'b1;
        end
    end

    wire match = done && (rx_bits == tx_word);

    // normal: received bits | hold btnD: led[2]=match, led[1]=done
    assign led = dn_s[1] ? {13'd0, match, done, 1'b0} : rx_bits;

endmodule
```
