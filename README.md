# np
no parse  HLS SCTE-35 Injection via sidecar file

# There has got to be a better way.

### Here are the facts as I see them.


* For some time now, I have been trying to figure out 
an effective way to inject SCTE-35 into HLS. 

* x9k3, is pretty cool, but scaling it to adaptive bitrate 
has been challenging. 

* sidercar SCTE-35, It was my idea, and I stand by it. 

* I noticed x9k3 now has a lot of options, and I really 
hate that.

* I like how umzz works, spawning a process for each rendition.

* I hate that umzz sucks over network if you don't have fat pipes.

that distills down to:
<br>

Inject SCTE-35 from a sidecar file, 
<br>
on the fly,
<br>
into live ABR  HLS streams,
<br>
in real time,
<br>
over a network,
<br>
make it easy,
<br>
and keep CPU usage to a minimum.

<br>

 I can do that.
<br>
![image](https://github.com/futzu/np/assets/52701496/7c445f4a-4a34-4839-8293-ae2cb36137b3)

<br>

### the game plan.


* read the master.m3u8 and copy it locally.
    * change the rendition URIs to the new local index.m3u8 files.
* load SCTE-35 from a sidecar file for all renditioons, just like umzz.
* spawn a process for each rendition and do the following:
  *  throttle for live output, just like x9k3.
  * read the index.m3u8 all HLS options defined by the index.m3u8.
  *  and write a new one according to these rules.
  * if this the first segment or a segment with a DISCO tag, parse out PTS from the segment.
      * otherwise just add the duration the the first PTS. 
  * for every other segment, don't read the raw segment, no parse.
    * copy over the index.m3u8 data for the segment,
    * change the segment path to the full URI, seg1.ts becomes https://example.com/seg1.ts

### When there are SCTE-35 cues
* if the ad break starts in the middle of a segment:
  * parse the segment, and split it into two pieces to match the SCTE-35 splice point.
  * replace the original segment with the 2 local split segments in the new index.m3u8.
  *  add CONT cue tags to the manifests as needed, but don't parse any other segment until you have a CUE-IN event.

### I've got most of it done, here's what's working.
 I have everything working on a single rendition except the segment splitter.
 I have the segment splitter done, I am merging it in today.  
 When that works, I'll merge in ABR process spawning, which is already done as well.
I expect to have a test first release this week.

__Currently__, I am rounding SCTE-35 Cues to the nearest segment, sort of, but it works 
better than just rounding.
with a 6 second segments noparse is averaging about +/-1.5 seconds off,
with two second segments, it's less than +/- 1 second off.





