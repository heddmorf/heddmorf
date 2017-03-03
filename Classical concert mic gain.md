144dB.
That's HUGE.
The oft-quoted dynamic range of a 24&nbsp;bit digital signal is much greater than a normal orchestral concert.
So why do mic inputs still need adjustable gain?
Can't you just do the gain digitally after the recorder, leaving as much headroom as possible for overzealous percussionists etc.?

That's what I found myself wondering while setting up for an orchestral recording recently.

I was using only two brands of microphone, with one model of mic amp / ADC, so thought it shouldn't be too difficult to work out what's going on.

Time to dig out the data-sheets.

The microphones I had were DPA 4006ER, Schoeps CMM 4 and Schoeps CMM 21.
The ADCs (with integral mic amps) were Merging HAPIs with ADA8 analogue cards.

Some pertinent specifications are reproduced below.

>| Microphone              |4006ER|CMM 4|CMM 21|
>| ----------------------- |-----:|----:|-----:|
>| sensitivity, mV/Pa      |  40  |  13 |  13  |
>| self noise, dB SPL A-wgt|  15  |  15 |  14  |
>| clipping level, dB SPL  | 132  | 132 | 132  |

and for the HAPI ADA8:

>|                          |                      |
>|--------------------------|:--------------------:|
>|0 dBFS at 0 dB gain*      | +13   dBu            |
>|Dynamic Range, ref +13 dBu| 119.5 dB A-wgt       |
>|Gain settings             |   0   dB to +66 dB   |
>|Equivalent Input Noise    |< &minus;128 dBu A-wgt|
>*ignoring pad<br/>

On initially reading these specs we can see the dynamic range of the microphones is 117 or 118dB, and the ADC reaches 119.5dB.


To do anything more involved with these numbers, we need to units;
we currently have values in dBu, milliVolts, dB&nbsp;SPL, Pascals, and noises spec'd absolutely and relatively: a right mess!
Time to crack open a spreadsheet and convert these all into consistent units.

---

To keep things simple, I'm going to convert everything to dB relative to SI units,
with the additional dBFS (relative to Full Scale sine wave) for the digital domain.
A purer approach would be to convert everything to SI units themselves,
but I'm very much a dB person, and find the maths easier adding dB values rather than multiplying scalar values.
For display I like my SPL relative to 20&nbsp;µPa and
my voltage relative to 0.775&nbsp;V (i.e. dBu).
But SI units will make the sums much easier, so here we go.

Field quantities are converted to deciBells thus: 20&times;log<sub>10</sub>(field quantity), and back again thus: 10<sup>(dB value)/20</sup>.

>Aside: I say 'field quantity' to distinguish from the equations for power ratios, on which dBs are based
&mdash; _deci_ for a tenth of, and _Bell_, the log<sub>10</sub> of the power ratios.
Since power is proportional to the square of field quantity (Volts, Pascals, etc.), the log<sub>10</sub> value is doubled for a field quantity change compared to the same ratio change in power.
Hope that's cleared up the 3dB v 6dB for a doubling debate.

The conversions for different reference levels can be made either in the linear values by multiplying or in dB by adding the following conversion factors:

| From |  To  |linear | dB        |
|------|------|-------|----------.|
|dB SPL|dB(Pa)|0.00002|&minus;94  |
|mV    |dB(V) |0.001  |&minus;60  |
|dBu   |dB(V) |0.775  |&minus; 2.2|

So now we can restate the specs in the more consistent units.


>|Microphone           |4006ER                 |CMM 4                    |CMM 21                   |
>|-------------------- |-----------------------|-------------------------|-------------------------|
>|sensitivity          |&minus;28&nbsp;dB(V/Pa)|&minus;37.7&nbsp;dB(V/Pa)|&minus;37.7&nbsp;dB(V/Pa)|
>|self noise, A-wgt    |&minus;79&nbsp;dB(Pa)  |&minus;79&nbsp;dB(Pa)    |&minus;80&nbsp;dB(Pa)    |
>|clipping level, A-wgt|      +38&nbsp;dB(Pa)  |      +38&nbsp;dB(Pa)    |      +38&nbsp;dB(Pa)    |

and for the HAPI ADA8:

>|                              |                         |
>|------------------------------|------------------------:|
>|sensitivity*, (at 0dB gain)   |&minus;10.8&nbsp;dB(FS/V)|
>|Output noise, (0dB gain) A-wgt|&minus;119.5&nbsp;dB(FS) |
>|EIN, (high gain) A-wgt        |< &minus;130.2&nbsp;dB(V)|
>*ignoring pad<br/>

---

Now we have the specs in SI-dB units, we can work out the sensitivity of the system from pressure through to digital by adding the gains: &minus;38.8&nbsp;dB(FS/Pa) for the DPA at 0dB gain, and &minus;48.5&nbsp;dB(FS/Pa) for the Schoeps.

By adding the clipping levels and the sensitivities, we can also now work out some non-sensible gain settings:
at 0dB gain, **the microphones will clip before the ADC**, i.e. below 0dBFS.
For the DPA, matching the clipping levels requires only +0.8dB gain, but for the Schoeps any setting below +10.5dB will cause mic clipping below 0dBFS.

At these gain settings, the mic self-noise becomes:

|mic    |amp gain|system gain dB(FS/Pa)|noise dB(FS)|
|-------|--------|---------------------|------------|
|DPA    |+0.8 dB |&minus;38 dB         |&minus;116.9|
|Schoeps|+10.5 dB|&minus;38 dB         |&minus;116.9|

Both mics have the same dynamic range, so adjusting the amp's gain has aligned both their clipping points and therefore also their noise floors.

Time for a recap of where we are.
We have seen that the dynamic range of the different microphones and the ADC are similar, and by applying the correct gain, we can eliminate wasted headroom in the ADC.
We don't have enough dynamic range in the ADC to apply a single gain for all microphones and expect optimal performance.

Next we must look into the noise floors, and how the different noise sources contribute to the system noise floor, to decide if aligning gain for clipping levels gives adequate noise performance, or if we will need to compromise between head- and foot-room.

---

### handling noise levels/sources

The immediate naive view at the start is that as the dynamic range of the HAPI is greater than the microphones, it must "fit" in.

However, by the output of the ADC, there are more sources of noise than just the mic's self-noise that need to be considered.
These noise sources add together proportional to their power (or Root-Mean-Square of the field quantity).

>Aside: To find the RMS sum, we could either
convert the dB values to field quantities, then square each field quantity, sum them, take it's square-root, and finally convert back to dB:
$$dB_{Total} = 20 \times \log_{10} \sqrt {\sum (10^\frac{dB_{NoiseComponent}}{20} )^2 }$$
or simplify by converting the dB values to powers, which can simply be summed and converted back:
$$dB_{Total} = 10 \times \log_{10} \sum 10^\frac{dB_{NoiseComponent}}{10}$$

A slightly less naive view would therefore be to see what the RMS sum of the mic noise and the HAPI is.
For low gains, the dynamic range spec figure for a mic amp / ADC unit will represent the noise (w.r.t. clipping, i.e. 0dBFS).
This gives a new noise floor figure of &minus;115&nbsp;dBFS
&mdash; 2dB noisier than the mic's self-noise alone.

As gain increases, a mic amp's noise starts to increase too.
We can model this as a noise source at the output and another noise source at the input, before the gain.

These can be taken (simplistically) or derived (more thoroughly) from the spec sheet figures of EIN (Equivalent Input Noise) and  Dynamic Range.  The EIN is calculated from the noise floor at high gain, where the amplified input noise dominates, and the Dynamic Range will be measured at low gain to get the best number for the marketeers.

> Aside: the Dynamic Range spec represents the 'output' noise only as long as the EIN is vanishingly small by comparison at minimum gain.
For the HAPI ADA8, the EIN at 0dB gain contributes &minus;142dB(FS).
>Compared to the &minus;119.5dB(FS) total noise of the ADA8, this is minuscule, and contributes only about 0.01dB.  Within the tolerances we are working with, it's fair to say the output noise is &minus;119.5dB(FS).
>
>If the numbers were closer, we should subtract the EIN power from system noise floor to find the output-associated noise.

Summing all the noise sources at the clipping-aligned gains, we get:

>DPA&hellip; PowerSum of<br/>
mic self-noise: -79dB(Pa) + -28dB(V/Pa) + 0.8dB + -10.8dB(FS/V)<br/>
amp EIN: -130.2dB(V) + 0.8dB + -10.8dB(FS/V)<br/>
amp output noise: -119.5dB(FS)<br/>
= -115.0dB(FS)

>Schoeps&hellip; PowerSum of<br/>
-79dB(Pa) + -37.7dB(V/Pa) + 10.5dB + -10.8dB(FS/V)<br/>
-130.2dB(V) + 10.5dB + -10.8dB(FS/V)<br/>
-119.5dB(FS)<br/>= -114.9dB(FS)

So, at these low gains, the HAPI ADA8 dynamic range hasn't changed from the spec sheet value.
For a 3dB increase, the amplified input noise would have to equal the output noise, so -130.2dB(V) + -10.8dB(FS/V) + GAIN = -119.5dB(FS)<br/>
&there4; GAIN = 21.5dB

The Noise Factor (NF) is still about 2dB.
Applying that back to the acoustic noise level, that's a system equivalent acoustic noise level of about 17dB&nbsp;SPL.
Given that the ambient noise of a concert hall is probably in the order of NR&nbsp;20 at best, I'm probably happy to take the hit of the slightly raised system noise floor, for the convenience of knowing that any clipping is down to the _force mejeure_ of exuberant timpanists clipping the microphone itself.
