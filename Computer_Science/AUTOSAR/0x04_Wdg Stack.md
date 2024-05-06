# Explanation
- A watchdog timer is a crucial component in embedded systems and critical applications, serving as a **countdown timer** that **resets** a microprocessor after a specific interval of time to maintain system stability.
- WDTs enhance reliability and fault tolerance, ensuring that systems remain operational and can recover from failures, thereby preventing malfunctions and improving overall system availability.
- Its primary purpose is to monitor a system’s operation and initiate corrective actions when necessary, ensuring system reliability and preventing malfunctions.
- When the system is powered on or reset, the countdown timer starts and requires the system software to **periodically reset** its countdown value. If the software fails to do so within the predetermined interval, the WDT assumes that the system is **not functioning correctly** and triggers the corrective action.
##### [[0x00_Intro to AUTOSAR|AUTOSAR]] Watchdog stack
- ![[wdg_stack.PNG]]
##### WdgM supervision mechanisms
- **Alive supervision**: 
	- This mechanisms is monitoring **Periodic entities** & It is **Timing focused**. 
	- **Supervised Entity (SE)** is A software entity which is included in the **supervision** of the Watchdog manager. Each supervised entity has exactly **one identifier**.
	- **Supervision cycle (SC)** is the **Period** of WdgM **main** function.
	- **Supervision Reference cycle (SRC)** is The **main** function instance when the **supervision** formula is checked.
	- **Alive indication** are Calls to **WdgM** by the supervised entity (**SE**) through `WdgM_CheckPointReached` 
	- **Expected alive indications (EAI)** is Number of **expected alive** indications.
- **Deadline supervision**: 
	- This mechanisms is monitoring **Sporadic entities** & It is **Timing focused**. 
	- **WdgM** will trigger a reset if the **delay** between two **checkpoints** is less than **Min** deadline or more than **Max** deadline durations.
	 ![[Deadline supervision.PNG]]
- **Logical supervision**: 
	- This mechanisms is monitoring **Program flow** & It is **Sequence focused**. 
	- **WdgM** will trigger a reset if a specific **sequence** of **calling checkpoints** is not achieved.
	 ![[Logical supervision.PNG]]