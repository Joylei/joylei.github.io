+++
title="Windows timer resolution"
date=2021-10-10

[taxonomies]
categories=["Programming"]
tags=["windows","timer", "resolution", "performance"]
+++

Recently I noticed that the author of libplctagresolved an [issue]((https://github.com/libplctag/libplctag/issues/260)) with `timeBeginPeriod()`. It's interesting to learn what happened and timer resolution.

## Timer resolution

The system timer resolution determines how frequently Windows performs two main actions:

- Update the timer tick count if a full tick has elapsed.
- Check whether a scheduled timer object has expired.

On Windows 7, the default timer resolution is 15.6 ms.

Let's say sleep for 50ms, it would have its time delay rounded to some multiple of the tick frequency.


## View system clock resolution

To view your system clock resolution, you need the [clockres](https://docs.microsoft.com/en-us/sysinternals/downloads/clockres) tool by sysinternals.

```
Clockres v2.1 - Clock resolution display utility
Copyright (C) 2016 Mark Russinovich
Sysinternals

Maximum timer interval: 15.625 ms
Minimum timer interval: 0.500 ms
Current timer interval: 0.997 ms
```

It said that it uses `GetSystemTimeAdjustment()` function.

## `timeBeginPeriod()`

As [the doc](https://docs.microsoft.com/en-us/windows/win32/multimedia/timer-resolution?redirectedfrom=MSDN) said, `timeBeginPeriod()` allows you requests a minimum resolution for periodic timers. And you must call this function before using timer services. `timeBeginPeriod()` should be paired with `timeEndPeriod()`, and they can be nested.

The effect of this function:

- Before Windows 10, version 2004

affects global windows setting

- From Windows 10, version 2004

for processes that call this function, windows use lowest value requested by any process; For processes which have not called this function, Windows does not guarantee a higher resolution than the default system resolution

- From Windows 11

if a window-owning process becomes fully occluded, minimized, or otherwise invisible or inaudible to the end user, Windows does not guarantee a higher resolution than the default system resolution

The doc also suggest to look at `SetProcessInformation()`.

## `SetProcessInformation()`

`SetProcessInformation()` is available from Windows 8.

> ProcessPowerThrottling enables throttling policies on a process, which can be used to balance out performance and power efficiency in cases where optimal performance is not required

- when a process enabling PROCESS_POWER_THROTTLING_EXECUTION_SPEED
  the process will be classified as EcoQoS (before Windows 11, labeled as LowQoS).

- if does not explicitly enable PROCESS_POWER_THROTTLING_EXECUTION_SPEED
  the system will use its own heuristics to automatically infer a Quality of Service level.

- In Windows 11, if a window-owning process becomes fully occluded, minimized, or otherwise invisible or inaudible to the end user, Windows may automatically ignore the timer resolution request.

## High-precision timing

Another topic related to timer resolution is high-precision timing.
Some referenced articles described how to get high-precision timing.

| Function                       | precision                                      | accuracy                     | comments                                   |
| ------------------------------ | ---------------------------------------------- | ---------------------------- | ------------------------------------------ |
| GetTickCount                   | 1ms                                            | 10ms-55ms                    | -                                          |
| GetSystemTimeAsFileTime        | 100ns                                          | not better than GetTickCount | -                                          |
| QueryPerformanceCounter        | 1 microsecond or better                        | -                            |
| GetSystemTimePreciseAsFileTime | the highest possible level of precision (<1us) | -                            | in Coordinated Universal Time (UTC) format |

Someone said QPC is not reliable, and must run on the same processor core with thread affinity. But microsoft claims that [thread affinity is  not need as this might affect your application's performance](https://docs.microsoft.com/en-us/windows/win32/sysinfo/acquiring-high-resolution-time-stamps#faq-about-programming-with-qpc-and-tsc). Windows has evolved to improve this, see the next section.

### QueryPerformanceCounter

QueryPerformanceCounter depends on hardware timer ( TSC or other motherboard timer) or platform timer. The precision depends on hardware timer resolution and access time, Precision = MAX [ Resolution, AccessTime].

| Version                      | QPC basis                                                                                                                                          | Sync across cores                                                                                                                                                                           |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| XP and 2000                  | TSC (some hardware systems' BIOS didn't indicate the hardware CPU characteristics correctly)                                                       | some multi-core or multi-processor systems used processors with TSCs that couldn't be synchronized across cores                                                                             |
| Vista and Server 2008        | used a platform counter (High Precision Event Timer (HPET)) or the ACPI Power Management Timer (PM timer)                                          | shared between multiple processors                                                                                                                                                          |
| Windows 7 and Server 2008 R2 | The majority of Windows 7 and Windows Server 2008 R2 computers have processors with constant-rate TSCs and use these counters as the basis for QPC | Windows use TSCs as the basis of QPC on single-clock domain systems where the operating system (or the hypervisor) is able to tightly synchronize the individual TSCs across all processors |
| Windows 8, Server 2012 R2    | TSC                                                                                                                                                | The TSC synchronization algorithm was significantly improved to better accommodate large systems with many processors                                                                       |


## Other solutions

Someone discussed using QueryPerformanceCounter and timeGetTime at the same time , so that if the result of QueryPerformanceCounter is not acceptable, use timeGetTime to fixup. 

Another solution is using Windows kernel events, such as CreateEvent/SetEvent/WaitForSingleObject. WaitForSingleObject doesn't use a timer if you set infinite timeout. It depends on kernel scheduling inside Windows. [Code](https://gist.github.com/GoaLitiuM/aff9fbfa4294fa6c1680)

## Summary

Windows has evolved to be more power-efficient.
Use high time resolution only when it's necessary. 
High time resolution might hurt overall system performance and reduce battery run time.

If your application is performance critical, it should call `timeBeginPeriod()` to request high timer resolution before using timer service; moreover, your application should not have any window in Windows 11. 

QPC is typically the best method to use for high resolution timing.

## References

- [Timer Resolution](https://docs.microsoft.com/en-us/windows/win32/multimedia/timer-resolution?redirectedfrom=MSDN)
- [timeBeginPeriod function (timeapi.h)](https://docs.microsoft.com/en-us/windows/win32/api/timeapi/nf-timeapi-timebeginperiod)
- [SetProcessInformation function (processthreadsapi.h)](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-setprocessinformation)
- [Quality of Service](https://docs.microsoft.com/en-us/windows/win32/procthread/quality-of-service)
- [High-Resolution Timers](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/high-resolution-timers)
- [Implement a Continuously Updating, High-Resolution Time Provider for Windows](https://docs.microsoft.com/en-us/archive/msdn-magazine/2004/march/implementing-a-high-resolution-time-provider-for-windows)
- [Windows Timer Resolution: Megawatts Wasted](https://randomascii.wordpress.com/2013/07/08/windows-timer-resolution-megawatts-wasted/)
- [usleep() for Windows (C/C++) + set timer resolution to lowest possible supported by system](https://gist.github.com/GoaLitiuM/aff9fbfa4294fa6c1680)
- [Precision is not the same as accuracy](https://devblogs.microsoft.com/oldnewthing/20050902-00/?p=34333)
- [Timers, Timer Resolution, and Development of Efficient Code](https://download.microsoft.com/download/3/0/2/3027d574-c433-412a-a8b6-5e0a75d5b237/timer-resolution.docx)
- [Better on the inside: under the hood of Windows 8](https://arstechnica.com/information-technology/2012/10/better-on-the-inside-under-the-hood-of-windows-8/2/)
- [Acquiring high-resolution time stamps](https://docs.microsoft.com/en-us/windows/win32/sysinfo/acquiring-high-resolution-time-stamps)
- [Windows 系统时钟间隔](http://yiiyee.cn/blog/2013/09/01/clock-interval/)
