---
layout:     post
title:      "Balancing an Inverted Pendulum: What a Wobbly Rig Taught Me About Control Theory"
sitemap: false
excerpt_separator: <!--more-->
hide_last_modified: true
comments: true
---

# Balancing an Inverted Pendulum: What a Wobbly Rig Taught Me About Control Theory

## Introduction

The inverted pendulum is one of those problems that sounds almost trivial when you describe it — a stick balanced upright on a moving cart — but turns into something genuinely humbling once you try to keep one upright with a controller you designed yourself. I spent two sessions working with a rig that could be configured in two modes: a **crane** (pendulum hanging below the carriage, stable) and an **inverted pendulum** (pendulum balanced above, unstable). My goal was to design state-feedback controllers for both configurations and compare what the linear theory predicted with what the hardware actually did.

What made the project interesting wasn't just getting the system to work. It was watching the tidy linear theory collide with the messy reality of a physical rig — sometimes confirming it, sometimes contradicting it in ways that turned out to be more instructive than the theory itself.

This post is a write-up of that experience: the design process, the experiments, and a few moments where the maths and the hardware had something to say to each other.

<!--more-->

## The system

The rig is a standard cart-and-pendulum: a carriage of mass *M* = 0.7 kg moves along a horizontal track, driven by a torque motor through a pulley. A pendulum of mass *m* = 0.32 kg is mounted on the carriage and can swing freely.

Linearising the equations of motion around either the hanging or upright equilibrium gives a fourth-order state-space model with state vector **x** = [position, velocity, angle, angular velocity]. The control input is the motor torque, driven by a state-feedback law of the form

$$u = -\mathbf{K}\mathbf{x}, \quad \mathbf{K} = [k_1, k_2, k_3, k_4]$$

The four gains are set physically on the rig by four potentiometers (*p₁*, *p₂*, *p₃*, *p₄*) wired to the analogue summing amplifier. So "designing a controller" meant choosing four pole locations, solving for the gains, and then turning four knobs.

## Pole placement, and the limits of linear theory

The first real design task was to place all four closed-loop poles of the crane at −√78.5 ≈ −8.86 — four coincident real poles, chosen to give a fast, well-damped response.

The theory is straightforward: match coefficients of the desired characteristic polynomial *(s + 8.86)⁴* against the closed-loop polynomial *det(sI − A + BK)*, and solve for **K**. I got *p₁ = 0.13, p₂ = 0.22, p₃ = 0.23, p₄ = 0*, plugged them in, and looked at the actual eigenvalues of the closed-loop system.

They were nowhere near −8.86.

Instead I got poles at **−17.4, −7.04 ± 5.04j, and −4.82** — all stable, but spread out across the left half-plane. The system worked: the step response was well-damped and converged cleanly to the demand. But the poles I'd designed for weren't the poles I got.

This turns out to be a textbook case of **eigenvalue sensitivity for repeated roots**. If your target polynomial is *(s + k)⁴* and the coefficients are perturbed by some small ε (from finite potentiometer precision, in this case), the roots don't move by ε — they spread out by roughly ε^(1/4). For ε = 0.01, that's a perturbation of about 0.3 in the pole locations: tiny in the coefficients, huge in the roots.

The fix is to **separate the poles**. In the next experiment I placed the poles at −15, −12, and −10 ± 10j instead. The computed eigenvalues matched the targets exactly, the response was faster than before, and the simulation tracked the hardware almost perfectly. Distinct poles are *robust* to coefficient perturbation in a way that repeated poles fundamentally aren't.

I'd seen this result derived in theory before. Seeing it on an oscilloscope was different.

## When friction saves you

The most interesting moment of the lab came from a controller I deliberately mis-designed.

For one of the early experiments, the design produced theoretical poles at *1.32 ± 30.5j* — a complex pair with **positive real part**. By every linear-systems argument I had, the system should have been unstable, oscillating at around 30 rad/s with exponentially growing amplitude.

It wasn't. The rig was stable. The pendulum oscillated for about three seconds and settled.

The explanation is that the linear model omits Coulomb friction on the carriage — and the carriage on this rig has *a lot* of it (*F* ≈ 3.6 N of dynamic friction). Using **describing function analysis**, you can show that Coulomb friction contributes an equivalent velocity-feedback gain of

$$N(E) = \frac{4F}{\pi E (M + I/a^2)}$$

where *E* is the oscillation amplitude. The crucial detail is that *N(E)* grows as *E* shrinks. So at the small amplitudes where the unstable mode is trying to grow, friction kicks in with disproportionately strong effective damping and pushes the closed-loop poles back into the left half-plane. The system isn't really unstable — the *frictionless* model is.

I found this genuinely satisfying. It's the sort of result that makes you appreciate why nonlinear analysis exists, and why "the linear model is fine for small signals" is exactly the wrong intuition here: the *smaller* the signal, the *more* the friction matters.

## Pushing the system to instability

The inverted-pendulum half of the project pushed in the opposite direction: deliberately destabilising a working controller to study what happens at the boundary.

Starting from a stable design (*p₁ = 0.23, p₂ = 0.50, p₃ = 0.63, p₄ = 0.40*) and reducing *p₂* — the carriage-velocity feedback — produced sustained, near-sinusoidal oscillations of growing amplitude. At *p₂ = 0.19* the carriage was swinging nearly to the end stops. This is a **limit cycle**, the same describing-function mechanism as before but running in reverse: with weakened velocity feedback, the friction damping is no longer enough to stabilise the system, and oscillations grow until they reach an amplitude where everything balances out.

Increasing *p₂* in the other direction (to 0.86) produced the linear-theory failure mode: unstable complex poles at *0.45 ± 31.8j*, high-frequency oscillations building up until the safety cut-out engaged. The predicted instability frequency (≈ 30 rad/s) matched the measured one to within a few percent — a nice consistency check on the Routh-Hurwitz analysis.

## Reflections

A few things stayed with me after working through this.

**Theory and hardware ask different questions.** The linear model is excellent for *design* — it tells you where to put your poles. It is sometimes terrible for *prediction* — it can declare an unstable system stable, or vice versa, because it omits the nonlinearity (friction) that's actually doing the work. The experiments made that distinction concrete in a way no problem sheet could.

**Numerical conditioning matters in physical systems too.** I tend to think of "ill-conditioned" as a numerical-linear-algebra concept that lives inside a computer. Watching four poles fan out from a target by a factor of two — purely because the potentiometers couldn't be set precisely enough — was a useful reminder that conditioning is a property of the *problem*, not just the algorithm.

**Limit cycles are not failures.** It's tempting to look at sustained oscillations and conclude that the controller is broken. Describing function analysis reframes them as the equilibrium of a nonlinear system — a stable attractor in its own right, with predictable amplitude and frequency. That reframing is genuinely useful when you're trying to diagnose why a system oscillates: the question shifts from "why is it unstable" to "what's the balance condition".

The project itself was four hours at a bench and a Jupyter notebook full of plots. The lessons — about robustness, modelling choices, and where to trust your equations — have outlasted both.
