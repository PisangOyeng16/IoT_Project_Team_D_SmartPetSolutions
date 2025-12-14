# ESP32 Machine Logic Code Explanation

*NOTE: the code for this can be found in the main_workinglogic_setup.cpp file

====== Main logic =====
 can be found at the end of the loop() section.
 how does it functions?

 there is a couple of if statement present in this section, here is what they do.
 
 - if (settingState == NOT_SETTING && manualState == MANUAL_IDLE) {checkScheduledFeeding(); }, this statement basically says that the machine will only auto-feed when the user is not editing the settings and is not choosing the manual feed option.

 - if (feedingActive) {monitorFeeding();}, this statement tells the device to continously monitor the machine state, every loop.

 -  if (millis() - lastScreenUpdate > 1000), this statement updates the LCD once per second, if it is in setting mode and/or idle -- the monitor will refresh, and also prevents LCD flicker & slowdown.

===== Schedule Feeding Check ======
 for this function, the program will do these several things:
 - it will keep the feeder in check whether is currently feeding or not, if not it will tell to feed if the conditions are correct and not feed if it is already feeding.
 - get current time using RTC, if RTC not working tells it to use millis() to simulate seconds
 - loop through all feeding slots to check
 - validate if the slots are usable and actually has food weight assigned, to precent empty feeds ad disabled schedules triggering.
 - it will match hour and minute exactly.
 - prevent multiple triggers in the same minute, ensuring only one feeding per slot per minute.
 - store last trigger time as a memory checkpoint.
 - and start feeding

===== startFeeding =====
 this section is to start initialisizing the scheduled feeding option.
 it does not dispense food itself, monitor weight changes, nor decide when to stop.

 for this function, the program will do these several things:
 - identify feeding type choice.
 - set feeding state flags.
 - compute target weight and compute the correct amount of food needed to be dispensed.
 - initialize timing and safety tracking, this is to detect that food actually increasing and if there is any stuck feeder / jammed
 - debug output in serial monitor
 - with openfeeder() it tells the logic to start feeding
 - update UI.

===== start manual feeding =====
 this function is called when the user selected the manual feeding option. its role is to initialize the feeding session without any scheduled slot
 this function is basically simillar to what the scheduled feeding is, but with the manual feeding conditions.

===== feeding monitor (manual + scheduled)
 this functions job is to continously read bowl weight, decide when to stop feeding, detect jams or empty hopper, enforced hard safety timeout, and finilize feeding cleanly.

 for this function, the program will do these several things:
 - guard clause -- only run if feeding, which prevents wasted CPU cycles and logic running when nothing is happening.
 - cache key values, which avoids repeated global reads and removes negative noise from load cell
 - primary stop feeding condition, if targret is reached 
 - stuck / jam detection logic, which only checks while the feeder is open.
 - safety timeouts, to stop everything if errors arises.
 - periodic status logging.

===== finish feeding =====
 this function is used as the function to close all logic when feeding is completed / jam detected / timeout triggered.
 its responsibilites is to finalize the feeding session safely, log activities that happened, reset all feeding-related state, and update the user interface.

 for this function, the program will do these several things:
 - capture feeding context (before reset), saving the machines state.
 - log the feeding event, record all events.
 - reset feeding logic flags
 - debug feedback / state validation
 - user feedback on LCD.

===== reset system state =====
 this function is used as the logic to factory reset / clean reboot the esp32 feeder system state.
 the purpose is to force the system into a known-safe-idle state, remove any leftover feeding/scheduling/UI state, restore default schedules, clear feeding history, reinitialize weight sensing, and ensure hardware is closed.

 for the author's function, the program wiil do these several things:
 - reset all feeding slots schedulling, weight, and state value to default.
 - clear feeding history -- to reset history count, marks all entries unused, and prevents old feeding data from appearing.
 - reset all weight sensors.
 - force hardware & UI into safe state.

===== handle setting mode =====
 this functions take on the logic side of the scheduled feeding setting mode.
 this function will only appear if you are in the setting slots screen and pressed the green button.
 the flow of the function is as follows:
 enter setting mode --> set hour --> set minute --> set weight --> save options --> exit setting mode.

===== adjust setting value =====
 this function is directly correlated to the handle setting mode. it is basically the one that deals with the value adjustment change inside said mode.

===== save current slot =====
 this function serves as the section to save the values edited in the setting mode.
 it also displays your selected slot and set value once saved.

===== add feed log =====
 this function records one completed feeding event into teh feeders history.
 
 for the author's function, the program wiil do these several things:
 - shift old logs down to make space for newest logs.
 - get the current time from RTC or from millis simulated time.
 - logs feeding metadata.
 - logs feeding time.
 - stores target weight and final weight.
 - maintain log count safely, which prevents overflow, ensures UI knows how many valid entries exist, and stops incrementing score once buffer is full.