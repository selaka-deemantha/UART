module uart #(
    parameter CLOCKS_PER_PULSE = 5208
)
(
    input logic [3:0] data_in,         // Input data to be transmitted
    input logic data_en,                // Data enable signal
    input logic clk,                    // System clock
    input logic rstn,                   // Active low reset signal
    output logic tx,                    // Transmitted data output
    output logic tx_busy,               // Transmission status output
    input logic ready_clr,             // Ready clear signal for receiver
    input logic rx,                     // Received data input
    output logic ready,                 // Ready signal indicating data reception completion
    output logic [3:0] led_out,         // Output for LED display
    output logic [6:0] display_out      // Output for 7-segment display
);
    logic [7:0] data_input;             // Input data for transmitter
    logic [7:0] data_output;            // Output data from receiver

    // Instantiate transmitter module
    transmitter #(.CLOCKS_PER_PULSE(CLOCKS_PER_PULSE)) uart_tx (
        .data_in(data_input),
        .data_en(data_en),
        .clk(clk),
        .rstn(rstn),
        .tx(tx),
        .tx_busy(tx_busy)
    );
    
    // Instantiate receiver module
    receiver #(.CLOCKS_PER_PULSE(CLOCKS_PER_PULSE)) uart_rx (
        .clk(clk),
        .rstn(rstn),
        .ready_clr(ready_clr),
        .rx(rx),
        .ready(ready),
        .data_out(data_output)
    );
    
    // Instantiate binary-to-7-segment converter module
    binary_to_7seg converter (
        .data_in(data_output[3:0]),
        .data_out(display_out)
    );
    
    // Connect input data to transmitter
    assign data_input = {4'b0, data_in};
    
    // Connect LED output to lower nibble of received data
    assign led_out = data_output[3:0];
    
endmodule
