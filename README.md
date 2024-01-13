# Overview #
The project was to implement a rate monotonic scheduler to perform 1Hz analog clock detection through a USB camera. Utilized RMS, assymetric multiprocessing, threading, and ring buffers to balance tasks.</br>
Only results are uploaded since this is a class-wide project (to prevent cheating).</br>

## 3 Minutes of Tick Detection Without Skips or Blurs (Sped-up) ##
![](https://github.com/isch4196/real-time-embedded/blob/master/3mindiff.gif)

## High-level Diagram ##
Basically, a sequencer schedules when each task should run by releasing a sempahore. Task 1 is ran on core 1 to only acquire frames at 30 FPS. Task 2 and task 3, which are frame differencing and frame trasnformation, are ran on core 2 and are made sure that they can be scheduled based on the rate-monotonic theory. Task 4 is a best-effort service to save the transformed frames to flash.
</br>
![](https://github.com/isch4196/real-time-embedded/blob/master/5318_high_level_diagram.png)
