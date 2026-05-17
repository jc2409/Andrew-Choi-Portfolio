---
layout:     post
title:      "Why Pilots Cause Oscillations: A Hands-On Exploration of Flight Control"
sitemap: false
excerpt_separator: <!--more-->
hide_last_modified: true
comments: true
---

# Why Pilots Cause Oscillations: A Hands-On Exploration of Flight Control

There's a counterintuitive result in control theory: sometimes the human trying to stabilise an aircraft is the reason it becomes unstable. The phenomenon is called **pilot-induced oscillation (PIO)**, and it's been responsible for some genuinely alarming moments in aviation history — including, famously, one of the early space shuttle landings.

I recently spent some time working through a series of flight-control experiments that build up from "human pilot trying to keep a plane level" to a full PID autopilot with anti-windup protection. The exercise turned out to be one of the cleanest demonstrations I've encountered of why classical control theory is the way it is — every concept (phase margin, time delay, integrator wind-up) showed up as a *thing I could feel* through a joystick rather than an abstract inequality on a page.

This post walks through what I built, what surprised me, and what each stage taught me about the gap between a model and reality.

<!--more-->

## The Setup

The simulated aircraft is reduced to its simplest form: a transfer function with two integrators in series, representing the relationship between control input and aircraft position:

$$G(s) = \frac{10}{s(s + 10)}$$

A disturbance enters the loop. My job — first as a human pilot, then as the designer of an autopilot — was to drive the error back to zero.

The exercises stepped through increasingly hostile scenarios: a benign impulse disturbance, then sustained sinusoidal buffeting, then an unstable aircraft that diverges on its own, and finally a full PID controller with realistic actuator saturation.

## Stage 1: Modelling the Pilot

A human pilot is, to a first approximation, a proportional controller with a reaction delay:

$$u(t) = k \cdot e(t - D)$$

By holding a joystick and responding to a step disturbance, I could extract my own gain $k$ and delay $D$ from the response trace. The numbers came out at roughly $k \approx 1.77$ and $D \approx 0.31$ s. That delay is huge in control terms — about a third of a second between seeing an error and responding to it.

Running the Bode analysis on this pilot-plus-aircraft loop gave a **phase margin of around 47°**, with a gain crossover at 1.8 rad/s. That's a comfortable margin in the classical-control sense. The system handled impulse disturbances cleanly: the error spiked and decayed back to zero within a couple of seconds.

It's worth noting what's *not* in this model. There's no integral action in the pilot — yet the error still settles to zero, because the aircraft itself contains an integrator (a pole at the origin). Real pilots almost certainly do something more sophisticated than pure proportional response — they learn, adapt, and develop something like integral behaviour for unfamiliar conditions — but the model captured enough to be useful.

## Stage 2: When the Pilot Becomes the Problem

The interesting failure mode arrived with sustained sinusoidal disturbances at 0.66 Hz.

With no control input at all, the aircraft just oscillated passively, error amplitude around 3. But when I tried to actively suppress the disturbance, **things got worse**. The error amplitude grew to over 5. My reaction delay of 0.31 s introduced about 75° of phase lag at the disturbance frequency, which meant my inputs were arriving at almost exactly the wrong moment — adding energy to the oscillation instead of cancelling it.

This is the essence of PIO. The Bode plot makes it precise: at $\omega \approx 2.56$ rad/s, the loop gain crosses unity exactly when the phase reaches $-180°$. That's the textbook marginal-stability condition, and the system sits on a knife edge of self-sustained oscillation.

The theoretical period at marginal stability is $T = 2\pi / \omega \approx 2.45$ s. My observed period was closer to 1.7 s — a real discrepancy, likely from the fact that my actual inputs weren't a clean $k \cdot e^{-sD}$ model, with saturation and higher-frequency components muddying the picture. But the qualitative result was unmistakable: I was the source of the instability.

The practical rule for designers that falls out of this:

> **If you want to avoid PIO, keep the loop's phase margin healthy at the frequencies where the pilot might be active.** Faster actuators, lower latency, lead compensation, and rate limiting on controls all push the dangerous frequency band away from where the pilot operates.

## Stage 3: An Unstable Aircraft

Things got dramatically more demanding with an aircraft that's unstable open-loop — a pole at $s = +2$ that makes the plant diverge exponentially without active control:

$$G_2(s) = \frac{1}{s - 2}$$

The Nyquist criterion gives a clean stability condition: with one unstable open-loop pole ($P = 1$), the closed-loop system needs exactly one clockwise encirclement of the critical point to be stable. Working through the geometry, this requires a proportional gain greater than 0.5. Below that threshold, the aircraft is genuinely uncontrollable.

Flying this thing manually was an experience. Where the stable aircraft tolerated lazy inputs, this one punished hesitation. Any moment where I let the error grow gave the unstable pole time to do its exponential thing. I managed to stabilise it for short periods, but each save felt like catching a falling vase.

## Stage 4: Letting a PID Controller Do It

The natural next step was to hand control over to a proper autopilot. I used the Ziegler–Nichols method: crank up proportional gain until the closed loop oscillates at marginal stability, measure the critical gain $K_c$ and period $T_c$, then plug them into the standard formulas to get PID parameters.

For my system, $K_c \approx 16$ and $T_c \approx 2$ s, giving:

| Parameter | Value |
|-----------|-------|
| $K_p$ | 9.6 |
| $T_i$ | 1.0 s |
| $T_d$ | 0.25 s (adjusted to 0.35 s) |

The PID controller handled a composite disturbance — step plus impulse plus sinusoid — far better than I could manually. Error settled within a couple of seconds, no oscillations, no drama. A satisfying result, until I poked at it harder.

## Stage 5: Integrator Wind-Up

This was the moment that turned theory into instinct.

When the disturbance was large enough to saturate the actuator (the control signal clipped at ±10), the integral term kept accumulating error even though the controller couldn't actually act on it. By the time the error finally came down, the integrator had built up a massive bias — and the control signal slammed full-negative for several seconds before it could unwind.

The cure is **integrator clamping**: cap the accumulated integral at a bound $Q$ chosen to be just large enough to reject the worst expected disturbance. For a step disturbance of magnitude 2 with my plant's DC gain near 1, the integrator needs to reach roughly $u_\infty / K_p = 2 / 9.6 \approx 0.21$. Setting $Q = 2/K_p$ gave the integrator enough capacity to do its job, but not enough to wind up catastrophically when the actuator saturated.

The before-and-after was vivid. Without clamping, the controller swung from full-positive to full-negative output and back, taking nearly 8 seconds to settle. With clamping, the same disturbance was handled in under 4 seconds, with one clean overshoot. **Same controller, same gains — one extra line of code.**

## Reflections

A few things stayed with me after this:

- **Phase margin isn't an abstraction; it's how much delay you can absorb.** Every concept in classical control has a physical analogue, and feeling them through the joystick made the maths click in a way that no textbook proof had.
- **The most dangerous systems are the ones where you're part of the loop.** A pilot trying their hardest can be the destabilising element. This generalises: any time you have a human and an automated system sharing authority — driver-assist, surgical robotics, algorithmic trading — the interaction matters more than either component alone.
- **Saturation breaks linear analysis.** Every Bode plot, every Nyquist diagram, every phase margin calculation assumes the actuators have infinite range. They don't, and the moment they hit a limit, all the elegant frequency-domain reasoning stops applying. The fix (anti-windup, gain scheduling, model-predictive control) is where modern practice diverges from the textbook.

The thing I appreciated most about the progression was how each stage *broke* something the previous stage had built. The pilot model worked until disturbances got fast; proportional control worked until the aircraft was unstable; PID worked until the actuator saturated. Every fix opened up the next problem. That's control engineering in miniature, and it's why the discipline rewards careful thinking over clever tricks.