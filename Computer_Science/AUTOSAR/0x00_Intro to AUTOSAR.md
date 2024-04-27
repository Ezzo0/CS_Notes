# Explanation
- Automotive open system Architecture is an open and standardized automotive software architecture.
- AUTOSAR key features:
	- Modularity and configurability.
	- Standardized interfaces.
	- Application is separated from utilities.
- AUTOSAR architecture consists of 3 main layers:
	1. **Basic software.**
	2. **RTE.**
	3. **Application.**
	![[autosar_layers.png]]
##### BSW Layer
- Provide service to the application.
- Responsible for running the functional part of the software like communication, I/O management, network management, memory management, and OS.
- **BSW** consists of 3 layers:
	- Mircro-controller Abstraction Layer.
	- ECU Abstraction Layer.
	- Service Layer.
	 ![[bsw_layers.png]]
- **Service layer** is providing basic services for application and basic software modules, which are:
	- OS functionality.
	- Vehicle network communication & management services.
	- Memory services.
	- Diagnostic services.
	- ECU state management & mode management.
- **Complex Device Drivers (CDD)** provides the possibility to integrate special purpose functionality like drivers for devices:
	- which are not specified within AUTOSAR.
	- with very high timing constrains.
- **AUTOSAR libraries** like CRC are collection of functions that can be called by any layer.
##### RTE Layer
- It is a communication center for:
	- Intra: SWC/BSW, SWC/SWC.
	- Inter: ECU information exchange.
- It provides a communication abstraction to application AUTOSAR software components by providing the same interface whether:
	- inter-ECU communication channels are used (e.g. CAN, LIN, ...).
	- communication stays intra ECU.
##### AUTOSAR Stacks
- AUTOSAR architecture consists of 4 stacks:
	- Communication stack.
	- [[0x01_Memory Stack|Memory stack]].
	- System stack.
	- I/O stack.
	![[autosar_stacks.png]]
- **System Stack** is responsible for:
	- Scheduling management.
	- Error management.
	- Mode management.
	- Watchdog management.
- **[[0x01_Memory Stack|Memory Stack]]** is responsible for:
	- Services for reading / writing to NV memory.
- **Communication Stack** is responsible for:
	- Message transmission / reception through different communication protocols.
	- Diagnostics.
	- Network management.
- **I/O Stack** is responsible for:
	- Management of I/O peripherals.
  ![[layers_stack_comm.png|1060]]
##### Types of interfaces
- Each stack provide 3 types of interfaces which are:
	- **Public Interface:** in which, an upper layer call a function in a lower layer.
	- **Scheduled Interface:** in which, OS call a function periodically.
	- **Callback Interface:** in which, a lower layer call a function in an upper layer.
