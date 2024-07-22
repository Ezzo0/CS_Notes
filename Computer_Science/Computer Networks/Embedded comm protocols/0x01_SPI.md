# Explanation
- Serial peripheral interface is a synchronous, full duplex main-subnode-based interface. The data from the main or the subnode is synchronized on the rising or falling clock edge.
- Both main and subnode can transmit data at the same time. The SPI interface can be either 3-wire or 4-wire.
##### Interface
- 4-wire SPI devices have four signals:
	- **Clock** (SPI CLK, SCLK).
	- **Chip select** (CS).
	- **Main out, sub-node in** (MOSI).
	- **Main in, sub-node out** (MISO).
- The device that generates the clock signal is called the main. Data transmitted between the main and the sub-node is synchronized to the clock **generated** by the main.
- SPI devices support much higher clock frequencies compared to [[0x02_I2C|I2C]] interfaces. Users should consult the product data sheet for the clock frequency specification of the SPI interface.
- SPI interfaces can have only one main and can have one or multiple sub-nodes.
	 ![[spi.png|550]]
- The chip select signal from the main is **used to select** the sub-node. This is normally an active low signal and is pulled high to disconnect the sub-node from the SPI bus.
- When multiple sub-nodes are used, an individual chip select signal for each sub-node is required from the main.
- MOSI and MISO are the **data lines**. MOSI transmits data **from the main to the sub-node** and MISO transmits data **from the sub-node to the main**.
##### Data Transmission
- To begin SPI communication, the main must send the clock signal and select the sub-node by enabling the CS signal.
- During SPI communication, the data is simultaneously transmitted (**shifted out** serially onto the MOSI / SDO bus) and received (the data on the bus (MISO / SDI) is **sampled** or read in).
- The serial clock edge synchronizes the shifting and sampling of the data.
##### Clock Polarity and Clock Phase
- The Clock Polarity bit sets the polarity of the clock signal during the idle state. The idle state is defined as the period when CS is high and transitioning to low at the start of the transmission and when CS is low and transitioning to high at the end of the transmission.
- The Clock Phase bit selects the clock phase. Depending on the bit, the rising or falling clock edge is used to sample and/or shift the data.
- The main must select the clock polarity and clock phase, as per the requirement of the sub-node. Depending on the Clock Polarity and Clock Phase bits selection, four SPI modes are available. Table 1 shows an example of the four SPI modes.
	 ![[SPI Modes.png]]
- In these examples, the data is shown on the MOSI and MISO line. The start and end of transmission is indicated by the dotted green line, the sampling edge is indicated in orange, and the shifting edge is indicated in blue.
	 ![[spimode0.png|1000]]
	 ![[spimode1.png|1000]]
	 ![[spimode2.png|1000]]
	 ![[spimode3.png|1000]]
##### Multi-Sub-node Configuration
- Multiple sub-nodes can be used with a single SPI main. The sub-nodes can be connected in regular mode or daisy-chain mode.
	1. **Regular SPI Mode:**
		- In regular mode, an individual chip select for each sub-node is required from the main. 
		- Once the chip select signal is enabled (pulled low) by the main, the clock and data on the MOSI/MISO lines are available for the selected sub-node. 
			 ![[Multi-subnode SPI.png|900]]
		- As can be seen from Figure 6, as the number of sub-nodes increases, the number of chip select lines from the main increases.
		- This can quickly add to the number of inputs and outputs needed from the main and limit the number of sub-nodes that can be used. 
		- There are different techniques that can be used to increase the number of sub-nodes in regular mode; for example, using a multiplexer to generate a chip select signal. 
	2. **Daisy-Chain Method:**
		- In daisy-chain mode, the sub-nodes are configured such that the chip select signal for all sub-nodes is tied together and data propagates from one sub-node to the next.
		- In this configuration, all sub-nodes receive the same SPI clock at the same time. The data from the main is directly connected to the first sub-node and that sub-node provides data to the next sub-node and so on.
		- The number of clock cycles required to transmit data is proportional to the sub-node position in the daisy chain.
			 ![[daisy_chain.png]]
# Sources
- [Introduction to SPI Interface](https://www.analog.com/en/resources/analog-dialogue/articles/introduction-to-spi-interface.html).