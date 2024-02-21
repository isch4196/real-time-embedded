# Overview #
The project was to implement a rate monotonic scheduler (RMS) to perform 1Hz analog clock detection through a USB camera on a raspberry pi 3b+ in linux. I utilized rate monotonic scheduling, asymmetric multiprocessing (AMP), threading, and ring buffers in order to balance tasks (see high-level diagram for different tasks). Only results are uploaded since this is a class-wide project (to prevent cheating).</br>

## 3 Minutes of Saved Ticks Without Skips/Repeats/Blurs (Sped-up) ##
![](https://github.com/isch4196/real-time-embedded/blob/master/3mindiff.gif)<br/>
Note that these are individual frames combined together to form this gif. This is not a video.

## High-level Diagram ##
Basically, a sequencer schedules when each task should run by releasing a semaphore. Task 1 is ran on core 1 to only acquire frames at 30 FPS. Task 2 and task 3, which are frame differencing and frame trasnformation, are ran on core 2 and are made sure that they can be scheduled based on the rate-monotonic theory. Task 4 is a best-effort service to save the transformed frames to flash.
</br>
![](https://github.com/isch4196/real-time-embedded/blob/master/5318_high_level_diagram.png)

See [syslog.txt](https://github.com/isch4196/real-time-embedded/blob/master/syslog.txt) for syslog statements during the execution of the system. It includes data such as the time it took for a task to run, when it ran, how many frames a task has processed so far, etc.

## Software Design ##
1) Asymmetric Multiprocessing<br/>
Asymmetric multiprocessing was used to designate a core to a certain activity. This was necessary because with symmetric multiprocessing, work is divided and balanced between each core. This does not work well for a time-critical system. Therefore, three cores were used to process tasks in parallel, and at different rates. Core 0 is reserved for the Linux operating system. Core 1 was used for task 1, or frame acquisition. Core 2 was used for tasks 2 and 3, frame differencing and frame transformation. Core 3 was used for saving a frame into flash memory.
2) Ring Buffers<br/>
Ring buffers played a large part in the software design of this project. Ring buffers allowed decoupling between different tasks that ran at different rates. For example, task 1 frame acquisition simply needed to worry about acquiring frames and dumping the frames into the ring buffer that would then be processed by task 2. This way, task 1 doesn't need to worry about waiting for task 2 to finish processing the data before continuing to fill the buffer with new frames. Of course, task 2 still needs to consume the data at a rate equal to or greater than the rate task 1 produces data. 
3) V4L2 (Video4Linux)<br/>
V4L2 is an API supporting realtime video capture on linux systems. This was used as opposed to opencv due to the perceived overhead of opencv (although this project also works with opencv). One of the nice things about V4L2 is its support for mmap which allows mapping device memory to the application address space. This means there won't be wasted CPU cycles copying memory from the kernel to userspace.

## Challenges ##
1) One of the biggest challenges for this project was maintaining a stable frame rate. Tests were conducted at different frame rates, for example at 30 FPS, 25 FPS, and 20 FPS, with only the task of frame acquisition running. However, it wasn't able to maintain a stable frame rate. The problem was found to be the lighting: webcams need to be under the correct lighting to be run at their max capabilities.
2) Another challenge was the initial design. There were two, or perhaps three, general designs that could have worked. First, one design could be a "shotgun start" in which a stable tick is detected, and then the program simply periodically acquires and saves a frame every second. Another design would be always detecting a stable tick, so this would consume more cpu processing power. The last design could be a hybrid version of both, in which the tick could be detected every minute or so, but the frames saved in between would be based on that initial timestamp.

## Future Considerations ##
Future considerations include adding more tasks to be balanced with RMS on core 2. This can involve tasks such as doing some sort of post-processing of the frame itself. Adding another task will tighten the bounds, meaning there will be less slack time.
