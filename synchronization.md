# Synchronization
Synchronizing clocks plays a key role in coordination, loggging, etc.

## Date and Time
[UTC (Universal Coordinated Time)}(https://en.wikipedia.org/wiki/Coordinated_Universal_Time) must be used.
[NPT (Network Time Protocol)}(https://en.wikipedia.org/wiki/Network_Time_Protocol) must be used to synchronize clocks.

## GSecond counter
When a second counter is needed, e.g. for coordinated mode, the current UTC time is converted to [Unix Time](https://en.wikipedia.org/wiki/Unix_time) is used. 

When a leap seconds occur, the Unix Time number will jump one second forward or backwards.
Care must be taken to handle this correctly, e.g. by reshifting the cycle offset.

