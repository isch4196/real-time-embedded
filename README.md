# Overview #
The project was to implement a rate monotonic scheduler (RMS) to perform 1Hz analog clock detection through a USB camera on a raspberry pi 3b+ in linux. I utilized rate monotonic scheduling, asymmetric multiprocessing (AMP), threading, and ring buffers in order to balance tasks (see high-level diagram for different tasks). Only results are uploaded since this is a class-wide project (to prevent cheating).</br>

## 3 Minutes of Saved Ticks Without Skips/Repeats/Blurs (Sped-up gif) ##
![](https://github.com/isch4196/real-time-embedded/blob/master/3mindiff.gif)<br/>
Note that these are individual frames combined together to form this gif. This is not a video.

## High-level Diagram ##
Basically, a sequencer schedules when each task should run by releasing a semaphore. Task 1 is ran on core 1 to only acquire frames at 30 FPS. Task 2 and task 3, which are frame differencing and frame trasnformation, are ran on core 2 and are made sure that they can be scheduled based on the rate-monotonic theory. Task 4 is a best-effort service to save the transformed frames to flash.
</br>
![](https://github.com/isch4196/real-time-embedded/blob/master/5318_high_level_diagram.png)

See [syslog.txt](https://github.com/isch4196/real-time-embedded/blob/master/syslog.txt) for syslog statements during the execution of the system. It includes data such as the time it took for a task to run, when it ran, how many frames a task has processed so far, etc.

## RMS (Rate-monotonic scheduler) ##
### Why Scheduling Algorithms are Important ###
First off, scheduling algorithms are important because you want to be able to meet certain deadlines. To give a layman's example, if you have 3 homework assignments, it would be smarter to prioritize the one that is due the next day instead of, for example, the one that is the easiest to finish but due in 2 days. This way, you will be able to finish each of the assignments before each deadline. Likewise, in programming, there are tasks they need to be finished before a certain deadline, hence why scheduling algorithms are so important. If there were not some sort of scheduling algorithm in place, a program for example may be starved of cpu time and feel unresponsive. 

### What is RMS? ###
RMS was used in this project. RMS stands for rate-monotonic scheduler, and it is a static priority preemptive scheduling algorithm. By static, it means that priorities are determined before run time and by preemptive, it means that the processing of a task can be interrupted by a higher priority task. Its static nature makes it much easier to analyze in comparison with dynamic scheduling algorithms, hence its commonplace in the industry. There are two important properties of RMS that make it popular: it is both optimal and stable. By optimal, it means that if a set of tasks can be scheduled by some other static scheduling algorithm, it can also be scheduled by RMS. This also implies that if a set of tasks cannot be scheduled by RMS, it can not be scheduled by some other static scheduling algorithm either. And by stable, it means that in an overload situation, it has a predictable failure (higher-priority tasks will keep running, and lower-priority tasks will miss their deadlines).<br/>

### How RMS works ###
The algorithm for RMS is quite simple. It assigns static priorities based on the period of a task, or how long it takes for a job to run. The shorter the period, the higher priority of the task, and vice versa. Since RMS is based on the period of a task, this is probably the most difficult thing about RMS: determining the worst case execution time of a task. General ways to approach this include using timers (which are available in Linux or an RTOS) to measure the execution time, or more reliably, by using GPIO pins and an oscilloscope. For the latter, functions to toggle the GPIO pin can be put at the start and end of the respective function, and the oscilloscope can measure how long it takes for the pin to toggle back to its original value. See [link](https://www.geeksforgeeks.org/rate-monotonic-scheduling/) for a great example.

### Potential Negatives of Using a Scheduling Algorithm ###
Although scheduling algorithms are very helpful, they can introduce a couple of problems. The main problem is overhead, both in terms of CPU and memory usage. Switching between tasks consumes both time and resources, and the presence of a scheduler itself consumes additional memory. Of course, this is all very application-dependent. Some systems may need to run bare metal code, some may need to use an RTOS, and others perhaps even embedded Linux.

## Software Design ##
There are two, or perhaps three, general designs that can work for this project. First, one design could be a "shotgun start" in which a stable tick is detected, and then the program simply periodically acquires and saves a frame every second (which is the time for a tick to occur). Another design would be always detecting a stable tick, so this would consume more cpu processing power. The last design could be a hybrid version of both, in which the tick could be detected every minute or so, but the frames saved in between would be based on that initial timestamp. I chose to go with the second design of always detecting a stable tick since it seemed to be more reliable.

## Software Concepts/Tools used in Project ##
1) RMS (Rate-monotonic scheduler)<br/>
See above.
2) Asymmetric Multiprocessing<br/>
Asymmetric multiprocessing was used to designate a core to a certain activity. This was necessary because with symmetric multiprocessing, work is divided and balanced between each core. This does not work well for a time-critical system. Therefore, three cores were used to process tasks in parallel, and at different rates. Core 0 is reserved for the Linux operating system. Core 1 was used for task 1, or frame acquisition. Core 2 was used for tasks 2 and 3, frame differencing and frame transformation. Core 3 was used for saving a frame into flash memory.
3) V4L2 (Video4Linux)<br/>
V4L2 is an API supporting realtime video capture on linux systems. This was used as opposed to opencv due to the perceived overhead of opencv (although this project also works with opencv) and because of the want to learn and interact with lower-level Linux. One of the nice things about V4L2 is its support for mmap which allows mapping device memory to the application address space. This means there wasn't CPU cycles copying memory from the kernel to userspace. Camera frames were captured using V4L2 and copied into a ring buffer for further processing between tasks.
4) Ring Buffers<br/>
Ring buffers played a large part in the software design of this project. Ring buffers allowed decoupling between different tasks that ran at different rates. For example, task 1 frame acquisition simply needed to worry about acquiring frames and dumping the frames into the ring buffer that would then be processed by task 2. This way, task 1 doesn't need to worry about waiting for task 2 to finish processing the data before continuing to fill the buffer with new frames. Of course, task 2 still needs to consume the data at a rate equal to or greater than the rate task 1 produces data; otherwise, data would be overwritten.
5) Mutexes<br/>
A mutex was used for each the count variable of each ring buffer that is shared between tasks. This is to ensure the count variable (for example, the number of frames not yet processed by the frame differencing thread) to be stable between different tasks, which add to or subtract from the current count.
6) Profiling<br/>
Syslog profiling was used in order to get and analyze the WCET (worst case execution time) of a task. Note that syslog entries were analyzed only after the system was done running. Reading syslog entries over the network proved to add too much latency, affecting the real-time requirements of the system.
7) Cheddar<br/>
Cheddar is a real-time scheduling simulator tool, to check the schedulability of tasks using RMS. It helps to determine theoretically whether a set of tasks are feasible or not.

## Challenges ##
1) One of the biggest challenges for this project was maintaining a stable frame rate. Tests were conducted at different frame rates, for example at 30 FPS and 25 FPS, with only the task of frame acquisition running. However, it wasn't able to maintain a stable frame rate. The problem was found to be the lighting: webcams need to be under the correct lighting to be run at their max capabilities.

## Project Retrospection/Learning Outcomes ##
The project worked, which is fine. However, I would probably approach the project from a different perspective if I were to redo it. RMS is something that doesn't need to be applied to every task, but rather the most critical set of tasks in a system. So for example task #3, which was used to transform a frame into the correct format so that it could be saved, should not be considered as a critical task but a post-process task that could be best-effort. As for the proper critical set of tasks, I would define them as one task for calculating the difference between frames and one task to choose a stable frame. The other tasks, which can be best-effort, can be considered as post-processing, and I would define them as task for transforming a frame, and another task for saving the frame into flash.

## Future Considerations ##
Future considerations include adding more tasks to be balanced with RMS on core 2. This can involve tasks such as doing some sort of post-processing of the frame itself. Adding another task will tighten the bounds, meaning there will be less slack time.
