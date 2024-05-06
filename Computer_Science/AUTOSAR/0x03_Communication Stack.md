# Explanation
- ECU sends messages over different networks (Can , Lin , Flexray .. ) these messages are called **PDUs (Protocol Data Unit)**.
- Each PDU consists of different signals.
	![[pdu_signals.png]]
- Data items sent or received by applications are mapped to signals.
	 ![[app_signals.png]]
- AUTOSAR Com Stack modules role :
	- Com module : **Packs** signals from **application** into **PDUs**.
	- PduR module : **Routes PDUs** to different communication protocols.
	- CanTp module : Part of [[0x02_Diagnostics Stack#Protocol TP|Protocol TP]] that performs **segmentation** of large PDUs.
	- CanIf module : **Maps PDUs** to their corresponding specific **Can IDs**.
	- Can module : Access microcontroller registers in order to send actual Can frame on physical bus.
	 ![[communication_stack.png|1100]]
##### Communication Modes
- **Transmission modes (Pdu level)**
	- **Direct**: When application request to send a PDU, this PDU will be **transferred directly**.
	- **Periodic**: COM module will send some **PDUs** **periodically**.
	- **Mixed**: Mode that use both Direct & Periodic modes.
- **Transfer property (Signal level)**
	- **Triggered**: When a signal is configured as triggered signal, COM will send the whole  PDU that contain this signal and other signals **directly** at the moment the application request to send a signal (`COM_sendSignal(signalID, *data)`).
	- **Triggered (On change)**: When a signal is configured as triggered signal, COM will send the whole  PDU that contain this signal and other signals **if the value of this signal is changed**.
	- **Pending**: If application requested to send a signal, it will **not be transferred**.
	 ![[data_transimition.png|1000]]
	![[COMReceivingSimpleData.png|1000]]
##### Filtering
- Filter **control** passage of signals **from/to the application**.
- Filtering is done by COM module.
- Different kinds of filtering algorithms are used by Com :
	- **ALWAYS**: No filter is used.
	- **NEW_IS_WITHIN**: Signal value is within some range.
	- **NEW_IS_OUTSIDE**: Signal value is outside some range.
	- **MASKED_NEW_EQUALS_X**: Signal value is equal to some value.
	- **MASKED_NEW_DIFFERS_X**: Signal value is not equal to some value.
##### Update Bit
- Allows the receiver to **identify** whether the sender has **updated the data** in the signal before sending the PDU or not.
- An update bit can be configured for each **signal** or **signal group** through `ComUpdateBitPostion`.
- On the sender side , AUTOSAR COM shall set the update bit if the application updates the value of the signal or the signal group.
- On the receiver side , AUTOSAR COM shall only process the signal or the signal group if the update is set otherwise it is discarded.
##### Signal Groups
- Signal group are some signals, grouped together, in the same PDU. 
- To send this group, `com_sendSignalGroup(groupID)` is used.
- Each signal in this group is configured to **pending** transfer property.
- Each signal group has the **same transfer properties** options as a single signal.
	![[signal_group.png]]
	![[send_signalGroup.png|1000]]
	![[receiving_signalGroup.png|1000]]
##### Deadline Monitoring
- **Reception Deadline Monitoring**: Used to verify that **periodic PDUs** are received within the allowed **time frame**.
- **Transmission Deadline Monitoring**: Used to verify that transmission requests are **acknowledged** by other ECUs on the network (TxConfirmation is received) within a given **time frame**.
- **TimeOut**:
	-  Amount of seconds waited until a sender application receive an acknowledgment. 
		- If TxConfirmation is **not received** in this time frame, COM will **notify** the application.
			 ![[Timeout_TX.png|800]]
	- Amount of seconds waited until a receiver application receive a periodic signal.
		- If periodic signal is **not received** in this time frame, COM will **notify** the application.
			 ![[timeout_RX.png|800]]
- **FirstTimeOut**:
	- It is time frame for receiving a signal after **initialization** of an ECU. If this first signal is **not received** in this time frame, COM will **notify** the application.
	- It is for **RX PDUs only**.
- Configuration parameters:
	- **ComTimeout**.
	- **ComFirstTimeout**.
	- **ComRxDataTimeoutAction**.
	- **ComTimeoutNotification**.
##### PDU Groups
- PDU group is A number of PDUs that can be **logically** grouped.
- Those PDUs are treated by Com as a group that can be activated (started) or deactivated (stopped) together.![[pduGroups.png]]
##### Signal Invalidation
- Indicates that the sender is not able to provide a valid value for a signal , for example in case a sensor is faulty ( parameter : `ComSignalDataInvalidValue`)
- Transmission
	- `Com_InvalidateSignal(Signal ID)`.
	- `Com_InvalidateSignalGroup(Signal Group ID)`.
- Reception
	- Configuration parameter ( `ComDataInvalidAction`)
		- **Notify**: Send a notification to the app to decide what to do.
		- **Replace**: Replace the invalid value with another value.
		- **None**.
##### Minimum Delay Timer
- A minimum delay time `ComMinimumDelayTime` between transmissions can be configured per PDU.
- If a transmission is requested before MDT expires , the next transmission is postponed until the delay time expires.
- If `ComMinimumDelayTime` is configured with 0 , no minimum delay time monitoring shall be performed.
#####  Network & Mode management
- ==Note: Always review the video in this part till U master it==
- **ComM**:
	- ComM handles the user communication **mode requests**.
	- ComM simplifies the resource management for the users.
- **NmIf**:
	- Nm is just the generic interface that works as a **dispatcher** between **ComM** and **BusNm**.
- **BusNM**:
	- BusNM coordinates the different NM state and the transition between them.
	- BusNm is responsible for sending/receiving Bus NM messages.
- **BusSM**:
	- BusSM handles the communication system dependent **startup** and **shutdown** features.
	![[Capture.PNG]]
- **Network states & transitions**
	![[comm_states.png]]
- **Passive Wakeup**
	![[Passive wakeup.PNG|1100]]
- **Active Wakeup**
	 ![[Active wakeup.PNG|1100]]
- **Network Shutdown**
	![[Network shutdown.PNG|1100]]