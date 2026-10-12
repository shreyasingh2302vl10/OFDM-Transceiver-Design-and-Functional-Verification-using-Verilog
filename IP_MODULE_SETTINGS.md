# CONFIGURATION SECTION
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/171fa5c0c799acf9319bcc0895387b0571e8821d/Screenshot%202026-06-12%20033718.png)
# IMPLEMENTATION SECTION
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/a7b1d37f572ce664a6e09cbf6fcf882ac03234a7/IP_Module_Settings.png)
# IFFT CODE 
```scheme 
 xfft_0 u_ifft (
        .aclk(clk),
        .s_axis_config_tdata(8'b00000000), // '0' means Inverse FFT (IFFT)
        .s_axis_config_tvalid(1'b1),
        .s_axis_data_tdata(w_axis_tdata),
        .s_axis_data_tvalid(w_axis_tvalid),
        .s_axis_data_tready(w_ifft_ready),
        .s_axis_data_tlast(w_axis_tlast),
        .m_axis_data_tdata(ifft_out_tdata),
        .m_axis_data_tvalid(ifft_out_tvalid),
        .m_axis_data_tready(1'b1)
    );
    
