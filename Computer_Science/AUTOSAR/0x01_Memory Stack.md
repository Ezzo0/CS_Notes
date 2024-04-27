# Explanation
- Memory stack provides access to Non-volatile memory to store information.
- Information are stored in Blocks and Each application defines the blocks it needs:
	- Length.
	- Protection.
- Memory stack main functionalities are:
	- Scheduling of accesses to any NV block for data saving/loading.
	- Access to blocks through BlockID, with optional queuing and priority management.
	- NV data safety through:
		- CRC checking.
		- Redundancy management.
		- Default data recovery.
	- Automatic multi-block loading/saving for ECU startup/shutdown modes.
	- NV Jobs priority management.
	- Explicit NV block invalidation services.
- Memory stack consists of 8 modules:
	1. **Non volatile memory manger (NVM):** Exists in services layer.
	2. **Memory interface (MEMIF):** Exists in ECU layer.
	3. **EEPROM abstraction (EA) & Flash EEPROM Emulation (FEE):** Exists in ECU layer.
	4. **External EEPROM driver (Ext EEP) & External Flash driver (Ext FLS):** Exists in ECU layer and are optional.
	5. **EEPROM driver (EEP) & Flash driver (FLS):** Exists in MCAL layer.
	![[Structure of memorystack.png|1000]]
##### EEP module
- It is responsible for abstraction of MC registers used to control on-chip EEPROM peripheral.
- It provides services for reading , writing, erasing to/from EEPROM.
- It also provides a service for comparing data.
- The EEPROM driver shall not buffer data.
##### FLS module
- It is responsible for abstraction of MC registers used to control on-chip FLASH peripheral.
- It provides services for reading , writing, erasing to/from FLASH.
- It also provides a service for comparing data.
- The Flash driver shall not buffer data.
##### EA module
- It is responsible of providing an abstraction of internal/external EEPROM devices.
- It provides the upper layers with a virtual addressing scheme.
- It provides “virtually” unlimited number of erase cycles.
##### FEE
- It is responsible of providing an abstraction of internal/external FLASH devices.
- It provides the upper layers with a virtual addressing scheme.
- It provides “virtually” unlimited number of erase cycles.
##### MEMIF
- It allows the NVRAM manager to access several memory abstraction modules ( FEE or EA modules).
##### NVM
- Scheduling of accesses to any NV block for data saving/loading.
- Access to blocks through BlockId , with optional queuing and priority management.
- Nv data safety through:
	- CRC checking.
	- Redundancy management.
	- Default data recovery.
- Automatic multi-block loading/saving for ECU startup/shutdown modes.
- Explicit NV block invalidation services.
##### NV blocks
- The NV block is a basic storage object in NV memory.
- The NV block consist of
	- Optional NV block header (Static block ID).
	- Data.
	- Optional CRC.
- Types of NV block management:
	- Native NV block.
	- Redundant NV block.
	- Dataset NV block.
- The **ROM block**, which consists of **constant data**, provides default data in case of an empty or damaged NV BLOCK.
##### Priority management Feature
- The memory stack supports a **priority** based job processing (in case of multiple write/read requests from application).
- Two [[Priority Queues|priority queues]] exists in memory stack
	- One for **immediate** write jobs.
	- Another for all other jobs.
- A write with **immediate** priority only **preempt** the running job.
- The preempted job shall be resumed/restarted by the memory stack.
##### Polling and Callbacks 
- The memory stack can use either **polling or callback** to get the status of current write/read job requested from application.
- Mixed configuration can be used along the memory stack.
##### Write verification Feature
- When a Ram block is written to NV memory the NV block shall be immediately read back and compared with the original content in RAM block.
- Write verification can be performed on some number of bytes (e.g. not all stored data will be verified).
- If write verification failed then write retires shall be performed.
##### Protection of NV block Feature
- Memory stack provides functionality of **protecting** the NV block from being overwritten.
- Application could use **“Nvm_SetBlockProtection”** API to **activate**/**deactivate** block protection during runtime.
##### Write all blocks / Read all blocks Feature
- **Nvm_WriteAll()** writes selected blocks on shutdown.
- **Nvm_ReadAll()** writes selected blocks on startup.
