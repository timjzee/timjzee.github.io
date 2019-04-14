---
layout: post
title:  "Extract part of a Pitch object in Praat"
date:   2019-4-14 23:00:00 +0100
categories: phonetics praat
---

For some reason, [*Praat*](http://www.fon.hum.uva.nl/praat/) does not have an option to extract parts of *Pitch* objects and other objects that are derived from a pitch analysis. However, in certain cases it is necessary to do the pitch analysis before extracting a part of the signal. Anyone familiar with acoustic phonetic analysis will know that phonetic segments are often shorter than 50 ms. Let's say we wanted to analyse the pitch of such segments. Using *Praat*'s standard options, our only option would be to first extract the relevant 50 ms from our *Sound* object and then apply *Praat*'s pitch detection algorithm.

Now consider the following quote from [David Weenink's *Praat* manual](http://www.fon.hum.uva.nl/david/LOT/sspbook.pdf) (p. 98):

> For periodicity detection we need a minimum of three periods in an analysis window. The lower the pitch, the longer a period will be and the longer the analysis window length needs to be. Three periods of a 75 Hz periodic signal last 3/75 = 0.04 s.

Given this information, we cannot really expect our pitch analysis of the 50 ms segment to be very good. It would be much better to first do the pitch analysis on a much longer part of speech before extracting the relevant segment. This difference in quality is exemplified in the figures below:

![alt text](/assets/extracted_pitch.png "Comparison of pitch measurements")

As you can see, extracting part of the *Pitch* object rather than part of the *Sound* object results in a much more complete pitch contour.

The right-most figure was based on a *Pitch* object that was constructed by first converting the original *Pitch* object to a *Matrix*, extracting the relevant columns and converting it back to a *Pitch* object:

```python
# Converting the selected Pitch object to a Matrix
To Matrix

# Creating a new Matrix based on previously assigned variables
Create Matrix: "'object_name$'_part", part_start, part_end,
    num_frames, 'full_name$'.dx, t_start_frame, 1, 1, 1, 1, 1,
    "Matrix_'object_name$'[1, start_frame - 1 + col]"

# Converting the Matrix back to a Pitch object
To Pitch
```

The full script can be found on [my GitHub page](https://github.com/timjzee/praatzee).
