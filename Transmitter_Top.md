# Heriarchy of Modules
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/2c6738483fbe499bc84b70610075f8f6ccd746c4/Screenshot%202026-06-12%20041422.png)
# Code of Transmitter Top Module
```verilog
`timescale 1ns / 1ps

module transmitter_top (
    input  wire        clk,
    input  wire        rst_n,
    input  wire        data_in,
    input  wire        valid_in,
    output wire [7:0]  qpsk_I_out,
    output wire [7:0]  qpsk_Q_out,
    output wire [15:0] ifft_out_tdata,
    output wire        ifft_out_tvalid,
    output wire        ifft_out_tlast     // Added missing port matching TB
);

    // Internal wires for 8-bit QPSK symbols
    wire [7:0] w_qpsk_I, w_qpsk_Q;
    wire       w_mapper_valid;
    
    // Internal AXI-Stream bus
    wire [15:0] w_axis_tdata; 
    wire        w_axis_tvalid;
    wire        w_axis_tlast;
    wire        w_ifft_ready;

    // Config TDATA for Xilinx FFT IP Core (8-bit width)
    // 8'h00 -> Inverse FFT (IFFT), 8'h01 -> Forward FFT
    // wire [7:0] w_config_tdata = 8'h00; 
    wire       w_config_tready;

    // Assign Top Level QPSK Outputs
    assign qpsk_I_out = w_qpsk_I;
    assign qpsk_Q_out = w_qpsk_Q;

    // -------------------------------------------------------------
    // 1. QPSK Mapper Instance
    // -------------------------------------------------------------
    qpsk_mapper u_mapper (
        .clk       (clk), 
        .rst_n     (rst_n),
        .data_in   (data_in), 
        .valid_in  (valid_in),
        .qpsk_i    (w_qpsk_I), 
        .qpsk_q    (w_qpsk_Q), 
        .valid_out (w_mapper_valid)
    );

    // -------------------------------------------------------------
    // 2. AXI-Stream Formatter
    // -------------------------------------------------------------
    qpsk_to_axis_formatter u_formatter (
        .clk           (clk), 
        .rst_n         (rst_n),
        .qpsk_I        (w_qpsk_I), 
        .qpsk_Q        (w_qpsk_Q), 
        .mapper_valid  (w_mapper_valid),
        .ifft_ready    (w_ifft_ready),
        .m_axis_tdata  (w_axis_tdata), 
        .m_axis_tvalid (w_axis_tvalid), 
        .m_axis_tlast  (w_axis_tlast)
    );

    // -------------------------------------------------------------
    // 3. Xilinx FFT IP Core Instance (IFFT)
    // -------------------------------------------------------------
    wire [7:0] w_config_tdata = 8'h00; 
    xfft_0 u_ifft (
        .aclk                        (clk),
        
        // Configuration Channel (Slave)
        .s_axis_config_tdata         (w_config_tdata), 
        .s_axis_config_tvalid        (1'b1), 
        .s_axis_config_tready        (w_config_tready),
        
        // Data Input Channel (Slave)
        .s_axis_data_tdata           (w_axis_tdata), 
        .s_axis_data_tvalid          (w_axis_tvalid),
        .s_axis_data_tready          (w_ifft_ready),
        .s_axis_data_tlast           (w_axis_tlast),
        
        // Data Output Channel (Master)
        .m_axis_data_tdata           (ifft_out_tdata),
        .m_axis_data_tvalid          (ifft_out_tvalid),
        .m_axis_data_tready          (1'b1),
        .m_axis_data_tlast           (ifft_out_tlast),

        // Event Signals (Unused)
        .event_frame_started         (),
        .event_tlast_unexpected      (),
        .event_tlast_missing         (),
        .event_status_channel_halt   (),
        .event_data_in_channel_halt  (),
        .event_data_out_channel_halt ()
    );

endmodule
```
