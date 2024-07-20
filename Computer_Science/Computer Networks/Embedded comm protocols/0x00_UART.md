# Explanation
- First we need to introduce types of communication:
	1. **Serial Communication:** 
		- The data will be sent (bit by bit), One bit at a time.
		- Sequentially transmission.
		- Needs one channel or wire for transmitting.
		- Protocol Examples : UART – [[0x02_I2C|I2C]] – [[0x01_SPI|SPI]] – USB – Flex Ray – Controller Area Network “[[0x03_CAN|CAN]]”  – [[0x04_LIN|LIN]].
		 ![[serial_comm.png]]
	2.  **Parallel Communication:**
		- More than bit can be transmitted at the same time. 
		- Needs more than channel or wire for transmitting.
		- Protocol Examples : PCI Bus.
		 ![[parallel_comm.png|450]]
	3. **Wireless Communication:**
		- This done using IR or RF.
		- Protocol Examples : Bluetooth – IEEE 802.11 – ZigBee - WIFI - 2G (GSM) - 3G & 4G, … etc.
- Types of communication systems:
	1.  **Simplex:** 
		-  One device transmits and the other devices receive.
	2.  **Half-Duplex:**
		- Each device can both transmit and receive, but not at the same time.
	3. **Full-Duplex:**
		- Each device can both transmit and receive at the same time.
- Types of communication Mechanisms:
	1.  **Asynchronous:** 
		- The transmission of the data **don’t** use **external clock**.
		- The transmission is synchronized over **synchronization bits**.
		- The receiver and the transmitter has known **Buadrate**.
			- Buadrate is determined by the lowest one of the devices.
		- Byte Oriented communication (Data is a group of “Bytes”).
		- Protocol Examples: UART
	2.  **Synchronous:**
		- Transmission of the data uses external clock.
		- The transmission is synchronized by a **common** clock.
		- The synchronization can be at **positive** or **negative** edge.
		- Protocol Examples : SPI - I2C – USART – USB.
##### What is UART?
- UART stands for **universal asynchronous receiver / transmitte**r and defines a protocol, or set of rules, for exchanging serial data between two devices.
- UART is very simple and only uses two wires between transmitter and receiver to transmit and receive in both directions.
- Communication in UART can be **simplex**, **half-duplex**, or **full-duplex**.
- Data in UART is transmitted in the form of frames.
##### Timing and synchronization of UART protocols
- One of the big advantages of UART is that it is asynchronous. Although this greatly simplifies the protocol, it does place certain requirements on the transmitter and receiver. 
- Since they do not share a clock, both ends must transmit at the same, prearranged speed in order to have the same bit timing. 
- The most common UART baud rates in use today are 4800, 9600, 19.2K, 57.6K, and 115.2K. 
- In addition to having the same baud rate, both sides of a UART connection also must use the same frame structure and parameters.
##### UART frame format
- Because UART is asynchronous, the transmitter needs to signal that data bits are coming. This is accomplished by using the **start bit**. 
- The start bit is a transition from the **idle high** state to a **low** state, and immediately followed by user data bits.
- After the data bits are finished, the **stop bit** indicates the end of user data. 
- The stop bit is either a transition back to the **high or idle** state or **remaining** at the high state for an additional bit time. 
- A second (optional) stop bit can be configured, usually to give the receiver time to get ready for the next frame, but this is uncommon in practice.
- A UART frame can also contain an optional parity bit that can be used for **error detection**. 
- This bit is inserted between the end of the data bits and the stop bit. The value of the parity bit depends on the type of parity being used (**even or odd**):
	- In **even parity**, this bit is set such that the total number of 1s in the frame will be even.
	- In **odd parity**, this bit is set such that the total number of 1s in the frame will be odd.
- For Example: Capital “S” (1 0 1 0 0 1 1) contains a total of three zeros and 4 ones. 
	- If using even parity, the parity bit is zero because there already is an even number of ones. 
	- If using odd parity, then the parity bit has to be one in order to make the frame have an odd number of 1s.  
- The parity bit can only detect a single flipped bit. If more than one bit is flipped, there’s no way to reliably detect these using a single parity bit.
- Note that in the idle state (where no data is being transmitted), the line is held high. This allows an easy detection of a damaged line or transmitter.
	 ![[uart_frameFormat.png]]
##### WHAT IS THE DIFFERENCE BETWEEN USART and UART
- USART and UART are both communication protocols used for serial communication, but they have some differences:
	1. **Functionality:**
		- UART is a hardware block that is responsible for transmitting and receiving serial data. 
		- It only supports asynchronous communication. 
		- USART is a more advanced version of UART that can support both asynchronous and synchronous communication. 
	2. **Synchronization:** 
		- UART only supports asynchronous communication, which means that there is no clock signal that synchronizes the data transfer between the transmitter and receiver.
		- In contrast, USART supports both asynchronous and synchronous communication, which means that there is a clock signal that synchronizes the data transfer.
	3. **Transmission speed:** 
		-  USART can typically operate at higher speeds than UART.
	4. **Data framing:**
		- USART supports more flexible data framing options, such as using 7, 8, or 9 data bits per frame, while UART typically only supports 8 data bits per frame.