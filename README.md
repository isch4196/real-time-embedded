# Overview #
The project was to implement a rate monotonic scheduler (RMS) to perform 1Hz analog clock detection through a USB camera. I utilized rate monotonic scheduling, asymmetric multiprocessing (AMP), threading, and ring buffers in order to balance tasks (see high-level diagram for different tasks). Only results are uploaded since this is a class-wide project (to prevent cheating).</br>

## 3 Minutes of Saved Ticks Without Skips/Repeats/Blurs (Sped-up) ##
![](https://github.com/isch4196/real-time-embedded/blob/master/3mindiff.gif)

## High-level Diagram ##
Basically, a sequencer schedules when each task should run by releasing a sempahore. Task 1 is ran on core 1 to only acquire frames at 30 FPS. Task 2 and task 3, which are frame differencing and frame trasnformation, are ran on core 2 and are made sure that they can be scheduled based on the rate-monotonic theory. Task 4 is a best-effort service to save the transformed frames to flash.
</br>
![](https://github.com/isch4196/real-time-embedded/blob/master/5318_high_level_diagram.png)

See [syslog.txt](https://github.com/isch4196/real-time-embedded/blob/master/syslog.txt) for syslog statements during the execution of the system. It includes data such as the time it took for a task to run, when it ran, how many frames a task has processed so far, etc.
