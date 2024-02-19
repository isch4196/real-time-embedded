# Overview #
The project was to implement a rate monotonic scheduler (RMS) to perform 1Hz analog clock detection through a USB camera on a raspberry pi 3b+ in linux. I utilized rate monotonic scheduling, asymmetric multiprocessing (AMP), threading, and ring buffers in order to balance tasks (see high-level diagram for different tasks). Only results are uploaded since this is a class-wide project (to prevent cheating).</br>

## 3 Minutes of Saved Ticks Without Skips/Repeats/Blurs (Sped-up) ##
![](https://github.com/isch4196/real-time-embedded/blob/master/3mindiff.gif)

## High-level Diagram ##
Basically, a sequencer schedules when each task should run by releasing a semaphore. Task 1 is ran on core 1 to only acquire frames at 30 FPS. Task 2 and task 3, which are frame differencing and frame trasnformation, are ran on core 2 and are made sure that they can be scheduled based on the rate-monotonic theory. Task 4 is a best-effort service to save the transformed frames to flash.
</br>
![](https://github.com/isch4196/real-time-embedded/blob/master/5318_high_level_diagram.png)

See [syslog.txt](https://github.com/isch4196/real-time-embedded/blob/master/syslog.txt) for syslog statements during the execution of the system. It includes data such as the time it took for a task to run, when it ran, how many frames a task has processed so far, etc.

## Challenges ##
1) One of the biggest challenges for this project was maintaining a stable frame rate. Tests were conducted at different frame rates, for example at 30 FPS, 25 FPS, and 20 FPS, with only the task of frame acquisition running. However, it wasn't able to maintain a frame rate. The problem was found to be the lighting: webcams need to be under the correct lighting to be run at their max capabilities.
2) Another challenge was the initial design. There were two, or perhaps three, general designs that could have worked. First, one design could be a "shotgun start" in which a stable tick is detected, and then the program simply periodically acquires and saves a frame every second. Another design would be always detecting a stable tick, so this would consume more cpu processing power. The last design could be a hybrid version of both, in which the tick could be detected every minute or so, but the frames saved in between would be based on that initial timestamp.
