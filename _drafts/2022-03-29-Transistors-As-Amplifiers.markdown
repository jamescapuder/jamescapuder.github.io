---
layout: post
title: "Transistors As Amplifiers"
tags: transistors audio
date: 2022-03-29
---

# Transistors As Amplifiers for Audio Signals

Welcome. Previously, I wrote a post about how bipolar junction transistors can be used to implement digital logic. Since writing that post, I discovered the rabbit hole of audio electronics - specifically guitar effects pedals - and along with it a broader understanding of the role of the transistor.

While the content in my previous post isn't necessarily wrong (provides a working understanding of an important role BJT's play in the digital world), it reflects the limited perspective I had on these devices at the time. 

So today, I'll try to fill in those gaps. We'll go over some basics about amplification, different types of transistors, set up a testing rig, and look at some results on an oscilloscope.

As always, I'm not an expert.




# Primer: What does a basic audio signal look like?

Sounds, in simple terms, are waves that travel through the air and get picked up by our ears. The frequency of the wave defines the pitch, and the amplitude defines the volume. Audio signals are a reproduction of these waves as an electrical signal. 

These signals can be sourced from devices that capture vibrations in the real world, like an electric guitar pickup. In these cases, the wave is represented by a change in voltage over time. To keep things simple, however, the example waves below I've generated through code - in this case the wave is represented by a series of numbers (a digital signal). 


## Specific examples

The audio snippets below were generated with a simple python script. Using a mix of numpy and scipy we populate an array with three seconds worth of sample data from a wave we generate, and write this data to a .wav file. Rather than a swinging voltage, a .wav file represents sound waves as a series of 32 bit integer values. The script also produces two plots: one displays the first 50 milliseconds of wave data, the other shows the frequency content of the wave (more on this in a second). 

I've included four examples: sine, triangle, square, and sawtooth waves. Each have a fundamental frequency of 220hz, but sound very different. 

The sine wave is the most "pure" sounding of the bunch. Not only is it smooth (continuous and differentiable everywhere), it contains no additional [harmonic content](https://en.wikipedia.org/wiki/Harmonic), so all we hear is the 220hz tone with no frills. Harmonics are worth their own discussion, but briefly they're additional frequencies which are integer multiples of the fundamental that also contribute to the final wave that reaches our ears. The presence and balance of these harmonics contributes to why the same note played on different instruments will sound different. Triangle and square waves both produce harmonics that are odd-integer multiples of the fundamental frequency, but differ in how quickly the volume of each of these harmonics decreases as the integer coefficient increases. 

Click on the arrows to expand/collapse the individual examples.

<details open>
<summary>Sine Wave Example</summary>

{% include embed-audio.html src="/assets/transistorAudio/signalgen220sin.wav" %}

<img src="/assets/transistorAudio/220sine.svg">

 When we look at the frequency content of each wave, we're plotting volume on the vertical axis, and freqency on the horizontal. As mentioned above, the sine wave doesn't have any additional harmonics above the fundamental, so as expected we only see one peak, at the 220hz mark:

<img src="/assets/transistorAudio/fft220sine.svg">

I'm only showing frequency up to 1.5khz, which is much lower than what we can hear (20khz) but still provides enough information to get the gist. 

</details>


In the next two examples, note the coninciding peaks in the frequency domain. In both cases, they only occur at odd integer multiples of the fundamental: 220, 660, 1100, and 1540 for 1, 3, 5, and 7, respectively. The difference in amplitude falloff and waveshape creates two distinct but not totally dissimilar sounds. 


<details>
<summary>Triangle Wave Example</summary>

{% include embed-audio.html src="/assets/transistorAudio/signalgen220triangle.wav" %}

<img src="/assets/transistorAudio/220triangle.svg">
<img src="/assets/transistorAudio/fft220triangle.svg">

</details>

<details>
<summary>Square Wave Example</summary>

{% include embed-audio.html src="/assets/transistorAudio/signalgen220square.wav" %}

<img src="/assets/transistorAudio/220square.svg">
<img src="/assets/transistorAudio/fft220square.svg">

</details>


Finally, the sawtooth wave is an example containing both even and odd multiple harmonics. You'll see a peak at each multiple of 220, not just the odd ones. 


<details>
<summary>Sawtooth Wave Example</summary>

{% include embed-audio.html src="/assets/transistorAudio/signalgen220sawtooth.wav" %}

<img src="/assets/transistorAudio/220sawtooth.svg">
<img src="/assets/transistorAudio/fft220sawtooth.svg">

</details>



# Amplification

Let's say we have a relatively small analog signal. This could come from almost anywhere: we could build [an oscillator circuit](https://sagittronics.wordpress.com/2019/09/21/yu-synth-vco-exploring-a-much-simpler-sawtooth-oscillator/) to generate analog versions of the waveforms above, or plug in an instrument. 

Amplification is just the process of making this samll signal bigger. Why would we want to do this? There are a multitude of reasons. Let's look to the guitar for some motivation.

A guitar pickup converts the vibration of the strings into an electrical signal, which, if we want to hear, we need to plug in to an amplifier. When plugged in, the signal travels via cable to the amp. When the signal first leaves the guitar, it is very small - 1V peak to peak depending on the pickups - and the amplitude decays quickly. The amplifier increases the amplitude (voltage swing) of the signal to a level sufficient to drive a speaker, converting the electrical signal into air waves. It does this by passing the signal through a series of *gain stages* which each multiply the amplitude of the singal by some amount. The amplification in each stage is carried out by an active device like a vacuum tube, or a transistor, and the gain of each stage (amount by which the input signal is amplified) is determined by these devices and the circuits we build around them. 

You might also want to increase the volume of the guitar during parts of a song for artistic effect, like during a solo, so that the guitar can be more clearly heard as a lead voice. In this case you could turn the volume up on the amplifier, but its hard to do that without taking your hands off the guitar. A more ergonomic solution can be found in *boost* pedals, which are typically one clean stage of amplification. 

An example of a boost pedal is the [LPB-1](http://beavisaudio.com/schematics/Electro-Harmonix-LPB-1-Schematic.htm), which is just a [common-emitter amplifier](https://en.wikipedia.org/wiki/Common_emitter) 