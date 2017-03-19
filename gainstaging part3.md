# Creating the gainstaging utility &mdash; Part 3

Initial coding is done (with only some pain &mdash; see [Part 2][part2]), and is available up at its [GitHub page][github].

Download it, and Python 2.7 if you don't have it, and give it a go!

## Real-life Usage Example

Consider a signal path in my [original post][classicalpost] from acoustic space, through a microphone to a voltage, through an amp to another voltage, and finally through an ADC into a digital signal.
Lets call them zones 0 (acoustic), 1, 2, and 3 (digital).

Using a mic and ADC from that post to provide example figures (Schoeps MK21/CMC6, and Merging HAPI), we now need to work out which zone each clipping or noise component belongs to.

```python
micGain = Gain("13mV/Pa")
micNoise = Level("15 dB SPL", zone=0)
micClip = Level("132 dB SPL", zone=0)
adcGain = Gain("+13dBu", "0dBFS")
adcEIN = Level("-128dBu", zone=1)
adcDNR = Level("-119.5dBFS", zone=3)
adcClip = Level("0dBFS", zone=3)
```

From this we can construct the gain list using one of the mic gains, a `Gain()` function for the amp gain (in `V/V` or `dB V/V`), and the ADC's gain.
Initially setting the amp's gain to 1 (or 0dB), the `gainList` becomes:
```python
[ micGain, Gain("0 dB V/V"), adcGain ]
```

### Clipping-level-aligned gain

Now we have this, we can workout the gain needed to align the clipping levels of the mics and ADC.  With the amp gain set to 0dB, the mic clipping level equivalent at zone 3 (the digital output) will be the opposite of the gain needed to align the clipping levels.
```python
levelAtZone([micGain, Gain("0dB V/V"), adcGain], micClip, 3).dB("FS")
```
This returns acoustic clipping points of `-10.48`&hellip;dBFS.

As the Hapi has gain steps of 0.5dB, we can set the amp gain to +10.5dB to match the clipping levels.
The new `gainList` is:
```python
[ micGain, Gain("+10.5 dB V/V"), adcGain ]
```

#### System noise

The dynamic range of this system can now be calculated as follows:
```python
powersum([micGain, Gain("+10.5 dB V/V"), adcGain], [micNoise, adcEIN, adcDNR], 3).dB("FS")
```
giving the system noise of `-114.91`&hellip;dBFS.
We can report this as an equivalent SPL by changing the zone and dB reference:

```python
powersum([micGain, Gain("+10.5 dB V/V"), adcGain], [micNoise, adcEIN, adcDNR], 0).dB("SPL")
```
giving `17.055`&hellip;dB SPL.
Referring back to the datasheets, this is a Noise Factor of 2dB.
A noise factor of 3dB would show the total of the noise sources excluding the mic are equal to the mic noise itself.
A figure under 3dB shows the mic noise is higher than the rest of the system's noise.





[classicalpost]: blog.morfaudio.co.uk/2017/03/classical-concert-mic-gain.html
[part1]: blog.morfaudio.co.uk/2017/03/gainstaging-utility-part-1.html
[part2]: blog.morfaudio.co.uk/2017/03/creating-gainstaging-utility-part-2.html
[github]: https://github.com/heddmorf/gainstaging
