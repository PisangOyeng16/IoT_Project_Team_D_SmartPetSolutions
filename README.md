# ESP32 Machine Logic Code Explanation

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
