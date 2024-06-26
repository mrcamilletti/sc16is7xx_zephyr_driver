# sc16is7xx_zephyr_driver
Zephyr driver for the NXP I2C SC16IS7XX dual UART

This is a cut and paste driver from the Zephyr 16550 driver (zephyr\drivers\serial\uart_ns16550.c) a few register definitions from the linux kernel (https://github.com/torvalds/linux/blob/master/drivers/tty/serial/sc16is7xx.c) with a few mods to drive the I2C from Zephyr. 

It's a bit messy but hopefully will be enough to get you started, and you can contribute back to make it better for all of us. 

Add this to your overlay file

&i2c0 {
	status = "okay";

	opito_uart:sc16is752@48 {
		compatible = "nxp,sc16is7xx";
		status = "okay";

		reg = <0x48>;
		reg-shift = < 3 >;
		num-ports = <2>; 		
		clock-frequency = < 14745600 >;		
		interrupt-gpios = < &opito_header 19 (GPIO_ACTIVE_LOW)>;		
		serial-mode = "rs232";
		config = <123>;

		ext_uart0: ext_uart@0
		{
			current-speed = < 115200 >;
			//parity = "odd"; 	//NRF9160 doesn't support ODD parity but the sc16is752 does.
		}

		ext_uart0: ext_uart@1
		{
			current-speed = < 9600 >;
		}
	};
};

	
	
	
Then you can use the device as a normal UART like so, (add these functions to main.c or whatever)

	#include "zephyr/drivers/uart.h"
	
	#define FEATHER_UART DT_NODELABEL(opito_uart)
	#if DT_NODE_HAS_STATUS(FEATHER_UART,okay)
		const struct device *uart_dev = DEVICE_DT_GET(FEATHER_UART);
	#else
		#error "uart2 device is disabled."
	#endif
	static uint8_t uart_buf[256];
	static const char *poll_data = "This is a POLL test.\r\n";

	
	
	void uart_cb(struct device *dev)
	{
	    uart_irq_update(dev);
	    int data_length = 0;
	    if (uart_irq_rx_ready(dev)) {
		data_length = uart_fifo_read(dev, uart_buf, sizeof(uart_buf));
		uart_buf[data_length] = 0;
			LOG_WRN("Rx from i2c uart %s", uart_buf);
	    }
	}


	static bool initUART2(){

	    if (uart_dev == NULL) {
		LOG_WRN("Cannot bind %s\n", "opito_uart");
		return false;
	    }
	    uart_irq_callback_set(uart_dev, uart_cb);
		uart_irq_rx_enable(uart_dev);
	    return true;
	}

	void uart2_start(void){
	    	initUART2();
		for (int i = 0; i < strlen(poll_data); i++) {
			uart_poll_out(uart_dev, poll_data[i]);
		}
	}


Currently only supports the first serial port, but should have hooks in there for the second port, without too many issues, Ideally someone will tidy this up and integrate it into the Zephyr mainline code for the benefit of all. 

