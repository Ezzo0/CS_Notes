# Explanation
- Diagnostics in the Automotive domains is the analysis of different functionalities of various components of the vehicle , which is normally done to find out if the components are operating properly or are faulty and require repair.
- Standardization committees presented some standards that specify the operations and services that can be used , in order to serve different use cases of automotive diagnostics
	- Unified Diagnostics Protocol (**UDS**) - ISO - 14229 (Commonly used in all ECUs).
	- Onboard Diagnostics Protocol (OBD) - ISO - 15031 (Only used in few ECUs).
- [[0x00_Intro to AUTOSAR|AUTOSAR]] provides standard , well-defined and modular specifications for implementing most commonly used Diagnostics protocols (e.g. DCM , DEM , …).
	![[Overview of diagnostics modules.png|1000]]
- Diagnostic system model consists of:
	- Diagnostic tool.
	- Ecu(s) to be diagnosed.
	- Communication channel between the diagnostics tool and diagnosed ECU(s) e.g. CAN , LIN ….
- The diagnostic tool requests a service and the ECU responds with results.
- The response can be positive or negative.
- UDS Protocol Frames :
	- Service Request
		![[service_request.png|500]]
	- Service Positive Response
		![[Service_Positive_Response.png|500]]
	- Service Negative Response
		![[Service_Negative_Response.png|500]]
##### UDS Services
- Most Common UDS Services are:
	1. Diagnostic Session Control (0x10)
	2. ECU Reset (0x11)
	3. Security Access (0x27)
	4. Tester Present (0x3E)
	5. Read Data By Identifier (0x22)
	6. Write Data By Identifier (0x2E)
	7. Read DTC Information (0x19)
	8. Clear Diagnostic Information (0x14)
	9. Control DTC Settings (0x85)
	10. Routine Control (0x31)
- You can determine each session services and its sub-services.
##### Diagnostic Session Control
- Session is a mode of operation with a specific timing parameters and specific services that can be executed.
- There are 3 common sessions: **Default** Session, **Extended** Session, and **Programming** Session.
- Programming session : Enables all diagnostics services related to memory programming of the server (ECU) for **software update**.
- Only one session can be active at a time in the server.
- During system startup , the default session is started by default.
- Session transition can be done either by service 0x10 or **S3Server** timeout (5 secs).
- **S3Server**: Time for the server to keep a diagnostic session other than the default session active while not receiving any diagnostic request message.
- **DCM** is responsible for notifying the application layer with the session transition.
- Request Format for session transition is:
	![[Request_Format.png|300]]
	- Sub-functions are :
		- Default Session (0x01).
		- Programming Session (0x02).
		- Extended Session (0x03).
- The Response Format is:
	![[Response_Format.png|500]]
##### Ecu Reset
- Ecu Reset service allows the external tester to request server reset.
- DCM is responsible for requesting system reset from application layer.
- DCM shall send the service positive response before performing the server reset.
- After a successful reset , the DCM shall activate the default session.
- Most common sub-services of Ecu Reset are:
	- **Hard Reset** (0x01): Simulates the power on sequence typically performed after the server got connected from the power supply.
	- **Key OFF/ON Reset** (0x02).
	- **Soft Reset** (0x03): This is implementation specific reset type , typically used to restart the application layer without re-initializing the basic software.
- Request Format for ECU reset is:
	![[ecu_Request_Format.png]]
- The Response Format is:
	![[ecu_Response_Format.png|400]]
##### Security Access
- Access to some services needs to be restricted for security matters.
- Security Access service is used to grant access to such protected services and sub-services.
- The following procedures shall be followed to unlock security levels:
	1. Client requests the “seed” i.e. random number.
	2. Server sends the “seed”.
	3. Client sends the “key” (corresponding for the “seed” received by using some algorithm).
	5. Server receive the "key" value from client and compare it with the computed the key.
	6. If key was valid, it will unlock itself.
-  Only one security level can be unlocked at a time.
- Request Format for security access control is - Request Seed:
	![[security_Request_Format.png|300]]
	- Sub-functions are :
		- “Request Seed” for security level 1 (0x01).
		- “Request Seed” for security level 2 (0x03).
		- ....
- The Response Format is - Request Seed:
	![[security_Response_Format.png]]
- Request Format for security access control is - Send Key:
	![[key_request_format.png|500]]
	- Sub-functions are :
		- “Request Seed” for security level 1 (0x01).
		- “Request Seed” for security level 2 (0x03).
		- ....
- The Response Format is - Send Key:
	![[key_response_format.png|300]]
- With every **session transition**, any unlocked security level will be locked.
##### Read Data By Identifier
- Client uses RDBI service to request one or more (DID) that identify data records.
- DID is two bytes long.
- DIDs with **read access** type can be used with RDBI service and vice versa.
- DIDs can be used to represent data from application or from BSW modules.
- Request Format for RDBI:
	![[RDBI_request_format.png|500]]
- The Response Format is:
	![[RDBI_response_format.png|500]]
##### Write Data By Identifier
- Client uses WDBI service to request one or more (DID) that identify data records.
- DIDs with **write access** type can be used with WDBI service.
- DIDs can be used to represent data from application or from BSW modules.
- Request Format for WDBI:
	![[WDBI_request_format.png|500]]
- The Response Format is:
	![[WDBI_response_format.png|500]]
##### DCM
- The Diagnostic Communication Manager (**DCM**) is an AUTOSAR-Basic SW module responsible for implementing a common APIs for using diagnostics services.
- The DCM module depends on other Com stack modules to be able to receive/transmit diagnostics requests/responses.
	- The Dcm module is network independent , it works over different types of networks (CAN , FlexRay … ) by using PduR module independent interfaces.
- DCM Module Functionality:
	- Supervise and guarantee protocol timings ( e.g. **S3Server** , **P2Sever** , **P2\*Server** …).
	- Manage diagnostic sessions.
	- Manage security levels locking/unlocking.
	- Check the validity of an incoming request.
	- Forward the request to application.
	- Assemble and transmit response.
- **P2Server** is the Maximum time value before which the **response** should be available at **client** side.
- **P2\*Server** is the Maximum time value (or Maximum number of P2Server) before which the **NRC 0x78** (Negtive response) should be available at the client side.
	![[DCM Timing Parameters.png|500]]
##### Protocol TP
- Protocol TP is a module or modules under DCM (e.g. CANTP, LINTP ....).
- Transport protocol has 4 kinds of frames:
	1. Single Frame:
		- It is the regular frame that data to be sent fits in the one frame.
		- The first byte of any protocol used is called **protocol information byte**.
			- The most 4 significant bits are zeros, which is the ID of the frame. It is called **frame type**.
			- The least 4 significant bits are the length of data.
	2. First Frame:
		-  It is used if data to be sent will fit in more than one frame of the used protocol.
		- The first 2 bytes of the frame has the information of the first frame and length of the total bytes to be sent.
			- Frame type bits store value 1.
			- The least 4 significant bits are the number of bytes of data with the second byte store the number of total bytes of data to be sent.
	3. Consecutive Frame:
		-  It is next frame or frames after First Frame.
		- The first byte of the frame has the information of the frame.
			- Frame type bits store value 2.
			- The least 4 significant bits store the sequence number, which is the order of the frame (0, 1, 2 .....).
		- The rest of the frame will have data.
	4. Flow Control Frame:
		- After first frame, client will send this frame to confirm that he received a frame successfully and inform ECU with some information.
		- It consists of 3 bytes.
			- The first byte of the frame has the information of the frame.
				- Frame type bits store value 3.
				- The least 4 significant bits store the **Flow Status (FS)**.
			- The second byte is called **Block Size (BS)**.
				- It is the **maximum number of consecutive frames** to be sent from ECU, before waiting for another flow control frame.
			- The third byte is called **ST MIN**.
				- The minimum time between two consecutive frame.
##### Development Error Tracer
- BSW modules and SW components report all detected development errors to DET.
- DET reporting is disabled in production phases.
- DET provides the API `Det_ReportErro`, which is an infinite loop without any instructions in it, for tracing the occurrence of the error and it’s related information.
- Error related information are passed as arguments to the API as follows :
	- Module where error has been detected (Module-ID).
	- API where error has been detected (API-ID).
	- Error ID (type of error).
##### Diagnostic Trouble Codes
- DTC is a code defined by the manufacturer to identify the error, it is the interface between DEM and DCM.
- DTCs are configured, managed, and stored in the DEM module.
- Diagnostics tool uses DCM module to communicate with the DEM module.
- DCM offers the following DTC related services :
	- Read DTC Information (0x19).
	- Clear DTC Information (0x14).
	- Control DTC Settings (0x85).
- DCM role in DTC related services is to receive the request, validate the request, call the DEM module for the request execution, assemble the output into the response message and transmit it.
- Only service and sub-service needs to be configured in DCM, other DTC configurations are done in DEM module.
##### Diagnostic Event Manager
- SW-C and BSW modules **monitor the run-time behavior** and **report** the monitoring results to the **DEM** module.
- DEM module is responsible for **processing and storing** diagnostic events reported by SW-C and BSW modules in non-volatile memory.
- Diagnostic tools use DCM services to read fault information from DEM module using DTCs.
- DEM provides API for reporting Diagnostic events `Dem_SetEventStatus(EventID, EventStatus)`.
- DEM event is linked to operation cycle that should be started before reporting event status
	- Power cycle.
	- Ignition cycle.
	- Driving cycle.
- Result of the event monitoring is reported through Event status
	- Passed.
	- Failed.
	- Pre-Passed.
	- Pre-Failed.
- **Passive** event is an event that was reported with failed status, and passed status after the failed status.
- Each event has status which is one byte.
	- First bit -**Test Failed bit**- is determining the time of failure. If it is one, the event failure is right now. Else, the event failure is not right now.
	- Third bit -**Confirmed bit**- is determining the status of storing event in memory. If it is one, the event is stored in memory. Else, the event is not stored in memory.
- DEM gives **priority** to each event.
- When an event, that is in its operation cycle, is reported to DEM, DEM will collect some data related to this event. These data are called **Environmental Data**, **Snapshot Data**, or **Freeze Frame**.
- **Enable conditions** feature means that DEM can receive a reported event when all conditions in the group are set.
- **Function Inhibition manger -FIB-** is disabling some functions that are related to an event when DEM has a reported for that event.
##### DEM Event Displacement Algorithm
- If memory that store events is full, DEM will or will not store an event based on its priority:
	- If event's priority is **less than** all priorities of events in memory, it will **not be stored** in memory.
	- If event's priority is **higher than** all priorities or equal to some priorities of events in memory, it will replace the oldest **passive** event that has lesser priority.
		- If there isn't a lesser priority event, the reported event will replace the oldest **passive** event that has equal priority. 
	- In the previous case: if there isn't a passive event, the reported event will replace the oldest **non-passive** event that has lesser priority. 
		- If there isn't a lesser priority event, the reported event will replace the oldest **non-passive** event that has equal priority.
	- If event is passive for an **aging threshold** consecutive cycles, then DEM will **remove** this event.
##### Event debouncing
- If a module report an event with pre-failed status, DEM will start one of the debouncing algorithms.
- **Time-Based debouncing**
	- DEM starts the internal **debounce time** to qualify events as failed, when monitors reported PRE-FAILED status.
	- When the configured **DemDebounceTimeFailedThreshold** value is reached, the event is qualified as **failed**.
	- Dem starts the internal **debounce time** to qualify events as **passed**, when monitors report **PRE-PASSED** status.
	- When the configured **DemDebounceTimePassedThreshold** value is reached , the event is then qualified as **passed**.
- **Counter-Based debouncing**
	- It is like time-based except it is using a **counter** instead of timer.
- Each event can be debounced or not (AKA **debouncing internal**).
