---
layout:     post
title:      "Building a Superheterodyne AM Radio Receiver"
sitemap: false
excerpt_separator: <!--more-->
hide_last_modified: true
comments: true
---

# Building a Superheterodyne AM Radio Receiver: From RF Signal to Audio

Radio receivers are everywhere — in your phone, your car, your Wi-Fi router — but the elegant signal-processing chain that turns a faint electromagnetic wave into intelligible sound is rarely visible to the people who use it. I recently spent time building and characterising a **superheterodyne AM radio receiver** on the medium-wave band, working through each stage from the antenna to the audio output. This post walks through what the architecture does, what I measured at each stage, and what I took away from designing around real-world non-idealities.

<!--more-->

## Why Superheterodyne?

A radio receiver has one fundamental problem to solve: extract a narrow band of information from a crowded electromagnetic spectrum, then recover the audio modulated onto it. A naive design might try to filter and amplify directly at the carrier frequency, but this is impractical — sharp filters at megahertz frequencies are expensive, and any tunable filter would need to maintain its selectivity across the entire reception band.

The **superheterodyne architecture**, invented by Edwin Armstrong in 1918, sidesteps this by translating every incoming station down to a single fixed **intermediate frequency (IF)** before doing the heavy lifting. Once the signal is at the IF, all the filtering and amplification can be optimised for that one frequency. It's a beautiful piece of engineering, and almost every radio receiver built in the last century uses some variant of it.

The signal chain I worked with follows the classic block diagram:

```
Antenna → LC Tank → RF Buffer → Mixer → IF Filter → IF Amplifier → AM Demodulator → Audio Amp → Speaker
                                  ↑
                          Local Oscillator
```

The IF was set to **455 kHz**, the long-standing industry standard for AM broadcast receivers.

## Stage-by-Stage: What I Measured

### The LC Tank Circuit

The front end is a parallel LC resonant circuit coupled to a ferrite coil antenna. The capacitance is tunable, which is how you "tune" to a station — sweeping the resonant frequency across the medium-wave band selects which carrier the receiver responds to.

Using the oscilloscope's Frequency Response Analyzer, I swept ten capacitor settings and identified the resonant peaks. The tank tuned from **519 kHz to 1510 kHz**, comfortably covering the medium-wave band. Back-solving for capacitance using the resonance equation:

$$C = \frac{1}{4\pi^2 f_0^2 L}$$

with an assumed inductance of 450 µH, the capacitor varied between **24.7 pF and 209 pF** — about an 8.5× range.

The quality factor of the tank determines how sharply it discriminates between adjacent stations. At a resonant frequency of 680.1 kHz, I measured a 3 dB bandwidth of 20.4 kHz, giving a **Q-factor of around 33**. That's enough selectivity to attenuate adjacent channels but not so much that it distorts the audio sidebands — a deliberate trade-off.

### Local Oscillator Tracking

For the mixer to translate every station to exactly 455 kHz, the local oscillator must always run at *carrier frequency + 455 kHz*. This means as you tune the LC tank, the LO has to track in lockstep — a problem mechanical engineers and circuit designers have wrestled with for decades.

I measured the LO frequency at each of the ten tuning settings and computed the offset. The result was instructive:

| Setting | 0   | 2   | 4   | 6   | 8   | 10  |
|---------|-----|-----|-----|-----|-----|-----|
| LO − LC (kHz) | 534 | 520 | 503 | 495 | 458 | 347 |

The offset drifts from 534 kHz at the low end to 347 kHz at the high end, against an ideal target of 455 kHz. The tracking is reasonable in the middle of the band but degrades at the extremes — a consequence of imperfect mechanical matching between the two ganged capacitors and the inherently non-linear way capacitance maps to frequency. In production radios, this is mitigated with carefully designed **padder and trimmer capacitors**; the version I worked with had a simpler implementation, and the resulting tracking error is a textbook example of why this calibration matters.

### The Mixer

The mixer is where the frequency translation actually happens. It multiplies the RF input by the LO signal, producing sum and difference components by the trigonometric identity:

$$\cos(\omega_1 t) \cdot \cos(\omega_2 t) = \tfrac{1}{2}\left[\cos((\omega_1 - \omega_2)t) + \cos((\omega_1 + \omega_2)t)\right]$$

Feeding a 1200 kHz test signal in while the LO ran at 1655 kHz, the FFT of the mixer output showed a clear peak at **455 kHz (the difference)** at −56.5 dBV, alongside the **sum at 2855 kHz** at −66.6 dBV. Both predicted components were exactly where the maths said they would be.

I also caught the LO's harmonic distortion — visible second and third harmonics at roughly 2× and 3× the fundamental. That non-linearity is a reminder that even a "sine wave" oscillator in practice contains a spectrum of unwanted components, all of which become potential sources of spurious responses downstream.

### The IF Filter

This is where the architecture earns its keep. With every station translated to 455 kHz, a single sharply tuned ceramic filter can do the channel selection that would otherwise need a tunable RF filter.

The IF filter response showed a clean resonance peak at 455 kHz with about **3–5 dB of insertion loss** — corresponding to a voltage transmission of 56–70%. Off-band rejection was steep, exactly what you want for separating one 9 kHz-wide AM channel from its neighbours.

### IF Amplifier and AM Demodulator

The IF amplifier provided a mid-band gain of **11.04** at 455 kHz, slightly above its nominal ×10 spec, with a 3 dB bandwidth of about 882 kHz. The wide bandwidth confirms the amplifier isn't doing any selectivity itself — that job belongs to the IF filter.

The AM demodulator was the most interesting non-ideality to characterise. Plotting DC output against input amplitude revealed clear non-linear behaviour below about **700 mVpp**, where the input diode isn't sufficiently forward-biased. Above that threshold, the response became cleanly linear. In a real radio, this translates to weak stations sounding distorted — which is exactly the behaviour anyone who has used an AM radio in fringe reception will recognise.

## Bringing It All Together

With each block characterised, the final test was the most satisfying: connecting headphones to the audio output, tuning the LC tank, and listening to actual broadcast radio. The FFT of the demodulated audio showed a bandwidth of roughly 5–10 kHz, which matches the standard channel allocation for medium-wave AM.

## Reflections

A few things stuck with me after this build:

- **The architecture is the design.** Almost every stage of a superhet exists to push a hard problem (selectivity, gain, tuning) onto a more tractable one. Once you internalise *why* the IF exists, the rest of the system explains itself.
- **Real components misbehave in instructive ways.** Imperfect LO tracking, harmonic distortion at the mixer, and the demodulator's dead zone aren't bugs — they're the gap between textbook equations and physical reality, and characterising them is most of what RF design actually involves.
- **Measurement discipline matters more than I expected.** Frequency response analysis, FFT views of mixer outputs, and Bode plots all told different stories about the same signal chain. Picking the right tool for each question was its own skill to develop.

Working through this end-to-end gave me an intuition for RF system design that I don't think I could have built from theory alone — and a lasting appreciation for how much engineering is compressed into the little radio in everyone's pocket.