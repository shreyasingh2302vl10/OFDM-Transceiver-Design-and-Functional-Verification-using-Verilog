# DATA TRANSMISSION
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/4fa94a16284469a0bf4263295cdfdcab3678282d/Complete_Transmission_Block.jpeg)
# CODE 
```verilog
`timescale 1ns / 1ps

module tb_ofdm_top;

    reg        clk;
    reg        rst_n;
    reg        tx_data_in;
    reg        tx_valid_in;

    wire [15:0] tx_ifft_out_tdata;
    wire        tx_ifft_out_tvalid;
    wire        tx_ifft_out_tlast;

    wire [15:0] rx_fft_out_tdata;
    wire        rx_fft_out_tvalid;
    wire        rx_fft_out_tlast;

    wire [7:0]  rx_fft_I_out;
    wire [7:0]  rx_fft_Q_out;

    // --- Demapper Signals ---
    wire [7:0]  mapped_I_out;
    wire [7:0]  mapped_Q_out;
    wire [1:0]  rx_demap_2bit_out; // Updated 2-bit wire
    wire        rx_demap_valid;

    // Unit Under Test (UUT)
    ofdm_top uut (
        .clk                 (clk),
        .rst_n               (rst_n),
        .tx_data_in          (tx_data_in),
        .tx_valid_in         (tx_valid_in),
        .tx_ifft_out_tdata   (tx_ifft_out_tdata),
        .tx_ifft_out_tvalid  (tx_ifft_out_tvalid),
        .tx_ifft_out_tlast   (tx_ifft_out_tlast),
        .rx_fft_out_tdata    (rx_fft_out_tdata),
        .rx_fft_out_tvalid   (rx_fft_out_tvalid),
        .rx_fft_out_tlast    (rx_fft_out_tlast),
        .rx_fft_I_out        (rx_fft_I_out),
        .rx_fft_Q_out        (rx_fft_Q_out),

        .mapped_I_out        (mapped_I_out),
        .mapped_Q_out        (mapped_Q_out),
        .rx_demap_2bit_out   (rx_demap_2bit_out), // Port mapped to 2-bit
        .rx_demap_valid     (rx_demap_valid)
    );

    always #5 clk = ~clk;

    integer i;
    reg [15:0] test_bits = 16'b10_11_10_01_10_11_00_11;

    initial begin    
        clk         = 0;
        rst_n       = 0;
        tx_data_in  = 0;
        tx_valid_in = 0;
    
        #200;          // let's end it
        rst_n = 1;
        #100;

        $display("\n=======================================================");
        $display("   STARTING TOP OFDM (TX IFFT + RX FFT + DEMAP) SIM");
        $display("=======================================================\n");

        for (i = 15; i >= 0; i = i - 1) begin
            @(posedge clk);
            tx_data_in  <= test_bits[i];
            tx_valid_in <= 1'b1;
        end

        @(posedge clk);
        tx_data_in  <= 1'b0;
        tx_valid_in <= 1'b0;

        #2500;
        $display("\n=======================================================");
        $display("   SIMULATION FINISHED ");
        $display("=======================================================\n");
        $finish;
    end

    always @(posedge clk) begin
        if (rx_demap_valid) begin
            $display("[DEMAP OUT] Time: %0t ns | Mapped (+8/-8): (I=%d, Q=%d) | Decoded 2-Bit Symbol: %b", 
                     $time, 
                     $signed(mapped_I_out), 
                     $signed(mapped_Q_out), 
                     rx_demap_2bit_out);
        end
    end

endmodule
```
# INPUT
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/39d9ce4e0652a90e04abe3ea82ab185ccafc4d52/Transmitter.png
)
### Transmitter I/Q Symbols:
        I  Q
       11 11
       01 01
       11 11
       11 01
       11 11
       11 01
       01 01
       01 11
# OUTPUT
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/39d9ce4e0652a90e04abe3ea82ab185ccafc4d52/OUTPUT.png)
### Recieved Output:
       0000ff80
       ff2500db
       ff80ff00
       00dbfe6f
       ff00ff80
       ffdb0025
       ff800000
       00250091

