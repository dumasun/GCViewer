GCViewer 1.36
=============

[![Build Status](https://app.travis-ci.com/chewiebug/GCViewer.svg?branch=develop)](https://app.travis-ci.com/chewiebug/GCViewer)
[![codecov.io](https://codecov.io/github/chewiebug/GCViewer/coverage.svg?branch=develop)](https://codecov.io/github/chewiebug/GCViewer?branch=develop)

GCViewer is a little tool that visualizes verbose GC output
generated by Sun / Oracle, IBM, HP and BEA Java Virtual Machines. It
is free software released under GNU LGPL.

You can start GCViewer (gui) by simply double-clicking on gcviewer-1.3x.jar
or running java -jar gcviewer-1.3x.jar (it needs a java 1.8 vm to run).

For a cmdline based report summary just type the following to generate a report (including optional chart image file): 
`java -jar gcviewer-1.3x.jar gc.log summary.csv [chart.png] [-t PLAIN|CSV|CSV_TS|SIMPLE|SUMMARY]`
When logfile rotation (-XX:+UseGCLogFileRotation) is enabled, the logfiles can be read at once: 
`java -jar gcviewer-1.3x.jar gc.log.0;gc.log.1;gc.log.2;gc.log.current summary.csv [chart.png] [-t PLAIN|CSV|CSV_TS|SIMPLE|SUMMARY]`


Supported verbose:gc formats are:

- some support for OpenJDK 9 / 10 unified logging format -Xlog:gc:<file>, the following configurations will work
  - -Xlog:gc:file="path-to-file" (uses defaults)
  - -Xlog:gc=info:file="path-to-file":tags,uptime,level (minimum configuration needed)
  - -Xlog:gc*=trace:file="path-to-file":tags,time,uptime,level
    (maximum configuration supported, additional tags ok, but ignored; additional decorations will break parsing)
- Oracle JDK 1.8 -Xloggc:<file> [-XX:+PrintGCDetails] [-XX:+PrintGCDateStamps]
- Sun / Oracle JDK 1.7 with option -Xloggc:<file> [-XX:+PrintGCDetails] [-XX:+PrintGCDateStamps]
- Sun / Oracle JDK 1.6 with option -Xloggc:<file> [-XX:+PrintGCDetails] [-XX:+PrintGCDateStamps]
- Sun JDK 1.4/1.5 with the option -Xloggc:<file> [-XX:+PrintGCDetails]
- Sun JDK 1.2.2/1.3.1/1.4 with the option -verbose:gc
- IBM JDK 1.3.1/1.3.0/1.2.2 with the option -verbose:gc
- IBM iSeries Classic JVM 1.4.2 with option -verbose:gc
- HP-UX JDK 1.2/1.3/1.4.x with the option -Xverbosegc
- BEA JRockit 1.4.2/1.5/1.6 with the option -verbose:memory [-Xverbose:gcpause,gcreport] [-Xverbosetimestamp]

Best results for non unified gc logging Oracle JDKs are achieved with: -Xloggc:<file> -XX:+PrintGCDetails -XX:+PrintGCDateStamps.
A few other options are supported, but most of the information generated is ignored by GCViewer
(the javadoc introduction of 
https://github.com/chewiebug/GCViewer/blob/master/src/main/java/com/tagtraum/perf/gcviewer/imp/DataReaderSun1_6_0.java
shows the details).

Hendrik Schreiber wrote GCViewer up to 1.29. What you are seeing here is based 
on his very good work.
Links to detailed descriptions of many JVM parameters relevant to garbage collection
can be found in the links section of https://github.com/chewiebug/GCViewer/wiki

Results of log analysis
=======================
There are two sections, where the results of the log analysis are shown.
One is the left side with the chart, the other the data panel on the right side.
In the following, the content of these sections is explained.

Chart
-----

GCViewer shows a number of lines etc. in a chart (first tab). These are:

- Full GC Lines:
  - Black vertical line at every Full GC
- Inc GC Lines:
  - Cyan vertical line at every Incremental GC
- GC Times Line:
  - Green line that shows the length of all GCs
- GC Times Rectangles:
  - black rectangle at every Full GC
  - blue rectangle at every initial mark event
  - orange rectangle at every remark event
  - red rectangle at every vm operation event ("application stopped...")
  - Grey rectangle at every 'normal' GC
  - Light grey rectangle at every Incremental GC
- Total Heap:
  - Red line that shows heap size
- Tenured Generation:
  - Magenta area that shows the size of the tenured generation
      (not available without PrintGCDetails)
- Young Generation:
  - Orange area that shows the size of the young generation 
      (not available without PrintGCDetails)
- Used Heap:
  - Blue line that shows used heap size
- Initial mark level:
  - Yellow line that shows the heap usage at "initial-mark" event 
      (only available when the gc algorithm uses concurrent 
      collections, which is the case for CMS and G1)
- Concurrent collections
  - Cyan vertical line for every begin (concurrent-mark-start) 
      and pink vertical line for every end (CMS-concurrent-reset /
      G1: concurrent-cleanup-end) of a concurrent collection cycle

Event details
-------------
In the second tab it shows details about the events it parsed:
E.g. events like the following

24.187: [GC 24.188: [ParNew: 93184K->5464K(104832K), 0.0442895 secs] \
93184K->5464K(1036928K), 0.0447149 secs] \
[Times: user=0.39 sys=0.07, real=0.05 secs]

are shown in one line as
GC ParNew: <number of events parsed>, <min duration>, <max duration>...

Events like these

4183.962: [Full GC 4183.962: [CMS: 32957K->40326K(932096K), 2.3313389 secs] \
76067K->40326K(1036928K), [CMS Perm : 43837K->43453K(43880K)], 2.3339606 secs] \
[Times: user=2.33 sys=0.01, real=2.33 secs] 
 
are shown as
Full GC; CMS; CMS Perm <number of events parsed> ...

So for every line the text is extracted (not always every part of it). This allows
a user which is familiar with the text log files to find out more details about
the events that occurred.

### median, 75th...
These columns show the median, 75th percentile etc.

### Gc pauses
This are shows all stop-the-world pauses, that are not full gc pauses.

### Full gc pauses
In this area all pauses are shown, which GCViewer considers as "full gc"
pauses. The current definition of a "full gc" is: Either the gc algorithm
prints "full gc" in its event name, or more than one generation (young,
old, permgen / metaspace) were involved during collection.

### VM operations overhead (safepoint pauses)
This area is only shown, if the gc log was written with the option
-XX:+PrintGCApplicationStoppedTime. To understand the meaning of this
metric, it is important to know about safepoints (see e.g.
http://blog.ragozin.info/2012/10/safepoints-in-hotspot-jvm.html).

If GCViewer finds gc log lines like the following:
2017-03-29T14:37:12.812+0200: 8.832: [GC (Allocation Failure) \
  [PSYoungGen: 29146K->3457K(29184K)] 78228K->52539K(116736K), 0.0009340 secs] \
  [Times: user=0.00 sys=0.00, real=0.00 secs]
2017-03-29T14:37:12.813+0200: 8.833: Total time for which application \
  threads were stopped: 0.0010682 seconds, Stopping threads took: 0.0000155 seconds

GCViewer will report one event in the "Gc pauses" area and one in this area.
The pause duration reported for "Total time..." will be 0.0001342s
(0.0010682 (duration of safepoint pause) - 0.0009340 (duration of gc pause))
So GCViewer only calculates the additional overhead needed for the whole
safepoint on top of the gc pause.

If the event immediately before the "Total time..." event was not a any
kind of gc pause, but another "Total time..." event, then the whole pause
for "Total time..." will be recorded for this event. In this case the
safepoint was not caused by a gc pause.

### Concurrent GCs
This are contains information about concurrent collection cycles, if
the gc algorithm used them. The time reported here is spent while the
application threads are running. It is possible to read here, how long
concurrent gc operations took until they finished.

Parser
------
In the third tab the output of the parser is shown. If there were warnings
during the parsing process or other output, you can check there.

Data Panel
==========
GCViewer provides some metrics to help you interpret the chart.
Note that some metrics based on averages are shown along with
their standard deviation. If it is obvious that the standard
deviation is fairly big in comparison to the average, the values
are grayed out, indicating that actual values are much smaller
or bigger than the average.

Summary
-------

- Footprint:
  - Maximal amount of memory allocated
- Max heap after conc GC:
  - Max used heap after concurrent gc.
- Max tenured after conc GC:
  - Max used tenured heap after concurrent gc 
    (followed by % of max tenured / % of max total heap).
- Max heap after full GC:
  - Max used heap after full gc. Indicates max live object 
      size and can help to determine heap size.
- Freed Memory:
  - Total amount of memory that has been freed
- Freed Mem/Min:
  - Amount of memory that has been freed per minute
- Total Time:
  - Time data was collected for (only if timestamp was present in log)
- Acc Pauses:
  - Sum of all pauses due to GC
- Throughput:
  - Time percentage the application was NOT busy with GC
- Full GC Performance:
  - Performance of full collections. Note that all collections 
      that include a collection of the tenured generation or are 
      marked with "Full GC" are considered Full GC.
- GC Performance:
  - Performance of minor collections. These are collections that 
      are not full according to the definition above.

Memory
------

- Total heap (usage / alloc max):
  - Max memory usage / allocation in total heap (the last 
    is the same as "footprint" in Summary)
- Tenured heap (usage / alloc max):
  - Max memory usage / allocation in tenured space
- Young heap (usage / alloc max):
  - Max memory usage / allocation in young space
- Perm heap (usage / alloc max):
  - Max memory usage / allocation in perm space
- Max tenured after conc GC:
  - see in "summary" section
- Avg tenured after conc GC:
  - average size of tenured heap after concurrent collection
- Max heap after conc GC:
  - see in "summary" section
- Avg heap after conc GC:
  - average size of concurrent heap after concurrent collection
- Max heap after full GC:
  - see in "summary" section
- Avg after full GC:
  - The average heap memory consumption after a full collection
- Avg after GC:
  - The average heap memory consumption after a minor collection
- Freed Memory:
  - Total amount of memory that has been freed
- Freed by full GC:
  - Amount of memory that has been freed by full collections
- Freed by GC:
  - Amount of memory that has been freed by minor collections
- Avg freed full GC:
  - Average amount of memory that has been freed by full collections
- Avg freed GC:
  - Average amount of memory that has been freed by minor
      collections
- Avg rel inc after FGC:
  - Average relative increase in memory consumption between full 
      collections. This is the average difference between the memory 
      consumption after a full collection to the memory consumption 
      after the next full collection.
- Avg rel inc after GC:
  - Average relative increase in memory consumption between minor 
      collections. This is the average difference between the
      memory consumption after a minor collection to the memory
      consumption after the next minor collection. This can be used
      as an indicator for the amount of memory that survives
      minor collections and has to be moved to the survivor spaces
      or the tenured generation. This value added to "Avg freed GC"
      gives you an idea about the size of the young generation in case
      you don't have PrintGCDetails turned on.
- Slope full GC:
  - Slope of the regression line for the memory consumption after
      full collections. This can be used as an indicator for the
      increase in indispensable memory consumption (base footprint)
      of an application over time.
- Slope GC:
  - Average of the slope of the regression lines for the memory
      consumption after minor collections in between full collections.
      That is, if you have two full collections and many minor
      collections in between, GCViewer will calculate the slope for
      the minor collections up to the first full collection, then the
      slope of the minor collections between the first and the second
      full collection. Then it will compute a weighted average (each
      slope wil be weighted with the number of measuring points it was
      computed with).
- initiatingOccFraction (avg / max)
  - CMS GC kicks in before tenured generation is filled.
      InitiatingOccupancyFraction tells you the avg / max usage in % of the
      tenured generation, when CMS GC started (initial mark).
      This value can be set manually using 
      -XX:CMSInitiatingOccupancyFraction=<value>. 
- avg promotion
  - Promotion means the size of objects that are promoted from young
      to tenured generation during a young generation collection.
      Avg promotion shows the average amount of memory that is promoted
      from young to tenured with each young collection (only available
      with PrintGCDetails)
- total promotion
  - Total promotion shows the total amount of memory that is promoted
      from young to tenured with all young collections in a file (only 
      available with PrintGCDetails)


Pause
-----

- Acc Pauses:
  - Sum of all pauses due to any kind of GC
- Number of Pauses:
  - Count of all pauses due to any kind of GC
- Avg Pause:
  - Average length of a GC pause of any kind
- Min / max Pause:
  - Shortest /longest pause of any kind
- Avg pause interval:
  - avg interval between two pauses of any kind
- Min / max pause interval:
  - Min / max interval between two pauses of any kind
  
* * *
- Acc full GC:
  - Sum of all pauses due to full collections
- Number of full GC pauses:
  - Count of all pauses due to full collections
- Acc GC:
  - Sum of all full GC pauses
- Avg full GC:
  - Average length of a full GC pause
- Min / max full GC pause:
  - Shortest / longest full GC pause
- Min / max full GC pause interval:
  - Min / max interval between two pauses due to full collections
  
* * *
- Acc GC:
  - Sum of all pauses due to minor collections
- Number of GC pauses:
  - Count of all pauses due to minor collections
- Avg GC:
  - Average length of a minor collection pause
- Min / max GC pause:
  - Shortest / longest minor GC pause


Notes
=====

This is not a perfect tool. However, GCViewer can help you
getting a grip on finding out what's going on in your application
with regards to garbage collection.

Here are some known limitations.


IBM JDKs
--------

If you have problems with the IBM format, please check that
every line of information is really in one line and not wrapped.

The IBM format actually provides a lot more information than is
visualized.


Sun JDK 1.3.1/1.4 with -verbose:gc
----------------------------------

Sun JDK 1.3.1/1.4 with -verbose:gc does not provide a timestamp.
Therefore values like 'Total Time', 'Throughput', and 'Freed Mem/Min'
cannot be calculated.


Sun / Oracle JDK 1.6 / 1.7 / 1.8 (a.k.a. Java 6 / 7 / 8)
---------------------------

CMS and G1 collector sometimes mix concurrent events with stop the world 
collections in the output. In some cases the parser can recover from 
such mixed lines, sometimes it can't and will show an error message.


BEA JRockit 1.4.2/1.5/1.6
-------------------------

Concurrently collected garbage may not be reflected correctly in the
data panel.

Export formats
--------------
**CSV** Comma Separated Values
The CSV format is quite useful for importing the data to a
spreadsheet application. However, it does not export all
data.

**CSV_TS** Comma Separated Values
CSV format using unix timestamp and one line per gc event.

**PLAIN** Plain Data
Plain text representation of the gc log. If written from Sun / Oracle gc log
it is usually compatible with HPjmeter.

**SIMPLE** Simple GC Log
Very simple representation of a gc log in the format
<name of event> <secondes since start of log> <pause time>.
This format is compatible with gchisto (http://java.net/projects/gchisto)

**SUMMARY**
Detailed summary exporting all details about a gc log file (same as shown in data panel). 

Internationalization
--------------------

Provided are a German, an English and a Swedish localStrings.properties
file. If someone feels the need to translate these to another
language, please do so. I will be more than glad, to include it
in a future version of this tool.


Start of log / absolute times
-----------------------------

If you happen to know when the application and GC log was started, you
can specify this time by right-clicking on the time ruler and entering
a start time.
Sun / Oracle VMs: If -XX:+PrintGCDateStamps was used, the proposed start time is 
read from the gc log file.

Bug reports
-----------

If you are a developer, you may fork (http://help.github.com/fork-a-repo/)
the repository on http://github.com/chewiebug/GCViewer and send me a 
pull request (http://help.github.com/send-pull-requests/). If you plan a bigger 
change I'd appreciate a notice in advance.

To file a bug report, please open an issue on 
http://github.com/chewiebug/GCViewer/issues or send an email to 
gcviewer-info@googlegroups.com with a description of the error, the 
name of the JVM that produced the GC data and all used flags along 
with a sample GC log file.


Building GCViewer from Source
-----------------------------

Download and install Maven3 from http://maven.apache.org/
Download the src distribution of GCViewer.
Execute from the GCViewer base directory (same as pom.xml):

    mvn clean install

The executable jar will be placed in the target directory.


Enjoy!

Joerg Wuethrich  
http://github.com/chewiebug/GCViewer  
gcviewer@gmx.ch

aa
bb
cc
dd
ee
