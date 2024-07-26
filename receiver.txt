module receiver #(
    parameter CLOCKS_PER_PULSE = 16
)
(
    input logic clk,                  // System clock
    input logic rstn,                 // Active low reset signal
    input logic ready_clr,            // Ready clear signal
    input logic rx,                   // Received data input
    output logic ready,               // Ready signal indicating data reception completion
    output logic [7:0] data_out       // Received data output
);

    enum {RX_IDLE, RX_START, RX_DATA, RX_END} state; // Define reception states

    logic [2:0] bit_counter;           // Bit counter to keep track of received bits
    logic [$clog2(CLOCKS_PER_PULSE)-1:0] pulse_counter; // Counter for clock pulses
    
    logic [7:0] temp_data;             // Temporary storage for received data
    logic rx_sync;                     // Synchronized received data
    
    always_ff @(posedge clk or negedge rstn) begin
        if (!rstn) begin
            // Reset all variables and state to initial values
            pulse_counter <= 0;
            bit_counter <= 0;
            temp_data <= 8'b0;
            ready <= 0;
            state <= RX_IDLE;
        end else begin 
            rx_sync <= rx;  // Synchronize the input signal using a flip-flop
            
            case (state)
                RX_IDLE : begin
                    if (rx_sync == 0) begin
                        state <= RX_START; // Move to RX_START state upon detection of start bit
                        pulse_counter <= 0; // Reset pulse counter
                    end
                end
                RX_START: begin
                    if (pulse_counter == CLOCKS_PER_PULSE/2-1) begin
                        state <= RX_DATA; // Move to RX_DATA state after half a clock period
                        pulse_counter <= 0; // Reset pulse counter
                    end else
                        pulse_counter <= pulse_counter + 1; // Increment pulse counter
                end
                RX_DATA : begin
                    if (pulse_counter == CLOCKS_PER_PULSE-1) begin
                        pulse_counter <= 0; // Reset pulse counter
                        temp_data[bit_counter] <= rx_sync; // Store received bit
                        if (bit_counter == 3'd7) begin
                            state <= RX_END; // Move to RX_END state after all data bits received
                            bit_counter <= 0; // Reset bit counter
                        end else bit_counter <= bit_counter + 1; // Move to the next bit
                    end else pulse_counter <= pulse_counter + 1; // Increment pulse counter
                end
                RX_END : begin
                    if (pulse_counter == CLOCKS_PER_PULSE-1) begin
                        ready <= 1'b1; // Indicate readiness to output data
                        state <= RX_IDLE; // Move back to RX_IDLE state after stop bit reception
                        pulse_counter <= 0; // Reset pulse counter
                    end else pulse_counter <= pulse_counter + 1; // Increment pulse counter
                end
                default: state <= RX_IDLE; // Default state transition to RX_IDLE
            endcase
        end
    end
    assign data_out = temp_data; // Output received data
endmodule
