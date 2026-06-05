# Mission 1 - Activity & Development Log

This log tracks the chronological execution timeline, software modifications, and task milestones for **Mission 1 (Receptionist Challenge)**.

| Date & Time | State / Phase | Action Taken / Milestone Achieved | Status / Result | Notes / Technical Details |
| :--- | :--- | :--- | :--- | :--- |
| **2026-05-28**<br>19:00 | Repository Setup | Initialized git and successfully opened the GitHub repository site. | Success | Remote tracking branches configured. |
| **2026-05-28**<br>20:35 | Audio & Sensors | Added a specialized background function to handle **doorbell sound detection**. | Success | Integrated **`librosa`** for acoustic feature processing and **`sounddevice`** for real-time stream listening. |
| **2026-05-29**<br>10:55 | Vision Processing | Added a frame-cropping capability during navigation state adjustments. | Success | Automatically **crops the middle frame** immediately when the robot turns to face the center/middle position. This ensures cleaner visual tracking input. |
| **2026-06-05**<br>19:00 | State 0 Tuning | Modified State 0 (bell ring check) execution code and updated the `detect_bell_from_file` mechanism. | Success | Set **`BELL_THRESHOLD = 62000`** based on empirical data collected during environmental audio testing. |
