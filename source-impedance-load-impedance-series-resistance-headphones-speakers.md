# Notes: Source Impedance, Headphone Loads, and the Effect of Series Resistance

**Module:** System Interfacing and Reactive Loads

**By** Hydraulik

---

## Abstract

A common claim in audio hobbyist discussions is that adding a series resistor to a headphone output should reduce volume uniformly across the entire audio band because the resistor is a linear, passive component. This claim is only true if the load behaves as a constant resistance. Real headphones and loudspeakers do not generally satisfy that condition. They are electroacoustic transducers with frequency-dependent impedance and electromechanical behavior. As a result, any nonzero source impedance, including an added series resistor, can change the voltage delivered to the load as a function of frequency, modify frequency response, affect electrical damping, waste power, and in some cases increase distortion. These lecture notes use a practical case study to develop the relevant circuit theory and engineering interpretation.

---

## 1. Introductory Case Study

> An audio enthusiast modifies a standard 3.5 mm headphone cable by soldering a \(16\,\Omega\) resistor in series with each signal path. The enthusiast argues:
>
> *"The resistor is linear and passive, so it should attenuate the signal equally at all audio frequencies. Therefore, it should reduce loudness without changing tonal balance."*
>
> **Question:** Is this claim correct?

### Short Answer

**Not in general.**

The claim would be correct **only if** the headphone behaved like a constant, purely resistive load over frequency. Real headphones are not ideal resistors. Their impedance usually changes with frequency because of voice-coil resistance, voice-coil inductance, mechanical resonance, and other electromechanical effects. Therefore, a series resistor and a real headphone typically form a **linear but frequency-dependent** network.

### Why This Case Matters

This case is pedagogically valuable because it exposes several important engineering ideas at once:

1. The difference between **resistance** and **impedance**.
2. The difference between **linearity** and **flat frequency response**.
3. The importance of **source impedance** in audio interfaces.
4. The fact that electroacoustic transducers are **electromechanical systems**, not simple resistors.
5. The engineering reason why a design that looks harmless in DC or dummy-load analysis may behave differently with a real headphone or loudspeaker.

---

## 2. Why the Claim Sounds Reasonable at First

### 2.1 The DC Intuition

In elementary DC circuits:

- a resistor has a fixed resistance \(R\);
- a series resistor reduces current;
- if the load is also a resistor, the circuit behaves as a simple voltage divider.

Under those assumptions, a resistor does indeed produce a predictable attenuation.

### 2.2 The Hidden Assumption

The hidden assumption is:

> **The headphone is being treated as a constant resistor.**

That is the critical mistake. A real headphone is not just "\(16\,\Omega\)" in the same sense that a laboratory resistor is \(16\,\Omega\). The value printed on the headphone is only a **nominal impedance**, not a complete description of its behavior.

### 2.3 Linearity Does Not Mean Frequency Neutrality

A second conceptual error is the confusion between two different ideas:

- **Linear system**
- **Frequency-independent system**

A system can be perfectly linear and still have a non-flat transfer function. A basic RC low-pass filter is linear. A basic RL high-pass filter is linear. Neither is frequency neutral.

Therefore, even if the resistor itself is linear, the **combined source-load network** can still alter spectral balance.

---

## 3. Theoretical Foundation: Resistance, Reactance, and Impedance

To analyze audio interfaces correctly, we use sinusoidal steady-state AC analysis and phasor notation.

### 3.1 Resistance

For an ideal resistor:

\[
Z_R = R
\]

where \(R\) is real-valued and frequency independent.

### 3.2 Reactance

For an inductor:

\[
Z_L = j\omega L
\]

For a capacitor:

\[
Z_C = \frac{1}{j\omega C}
\]

where:

- \(j = \sqrt{-1}\)
- \(\omega = 2\pi f\)
- \(f\) is frequency in hertz

### 3.3 Impedance

The general impedance is written as:

\[
Z(j\omega) = R + jX
\]

where:

- \(R\) is the real part
- \(X\) is the reactance

In many audio problems, what matters is not just the magnitude \(|Z|\), but the full **complex impedance**, including phase.

### 3.4 Why Impedance Matters in Audio

Audio signals vary over frequency. Therefore, if the load has frequency-dependent impedance, the current drawn from the source and the voltage delivered to the load will also be frequency dependent.

This is the core reason the series-resistor claim fails.

---

## 4. Modeling the Audio Source: The Thévenin View

A practical amplifier output can be modeled as a **Thévenin source**:

- an ideal voltage source \(V_s\)
- in series with an output impedance \(Z_s\)

This is one of the most useful abstractions in analog engineering.

### 4.1 Source Current

If the load is \(Z_L(j\omega)\), then the current is:

\[
I(j\omega) = \frac{V_s(j\omega)}{Z_s + Z_L(j\omega)}
\]

### 4.2 Load Voltage

The voltage across the load is:

\[
V_L(j\omega) = I(j\omega)\,Z_L(j\omega)
\]

Substituting for current gives:

\[
V_L(j\omega) = V_s(j\omega)\frac{Z_L(j\omega)}{Z_s + Z_L(j\omega)}
\]

So the transfer function is:

\[
H(j\omega) = \frac{V_L(j\omega)}{V_s(j\omega)} = \frac{Z_L(j\omega)}{Z_s + Z_L(j\omega)}
\]

This is the central equation of the lecture.

### 4.3 Special Case: Constant Resistive Load

If \(Z_L(j\omega) = R_L\), where the load is a constant pure resistance:

\[
H(j\omega) = \frac{R_L}{Z_s + R_L}
\]

If \(Z_s\) is also purely resistive and constant, the attenuation is frequency independent.

This is the **only** situation in which the hobbyist's claim is valid.

---

## 5. Why a Headphone Is Not a Pure Resistor

A headphone driver is an **electroacoustic transducer**. Its electrical behavior is coupled to mechanical motion and acoustic loading.

### 5.1 Physical Structure of a Dynamic Headphone Driver

A typical dynamic headphone driver includes:

- a **voice coil**
- a **magnetic field**
- a **diaphragm**
- a **suspension system**
- air loading and enclosure effects

When current flows through the voice coil, a force is generated, the diaphragm moves, and sound is produced.

### 5.2 Small-Signal Equivalent Model

A simplified small-signal model may be written as:

\[
Z_{\text{drv}}(j\omega) = R_e + j\omega L_e + Z_m(j\omega)
\]

where:

- \(R_e\) is the DC resistance of the voice coil
- \(L_e\) is the voice-coil inductance
- \(Z_m(j\omega)\) is the motional impedance reflected into the electrical domain

A more explicit electromechanical form is:

\[
Z_{\text{drv}}(j\omega) =
R_e + j\omega L_e +
\frac{(Bl)^2}{R_{ms} + j\omega M_{ms} + \frac{1}{j\omega C_{ms}}}
\]

where:

- \(Bl\) is the force factor
- \(R_{ms}\) is mechanical damping
- \(M_{ms}\) is moving mass
- \(C_{ms}\) is mechanical compliance

This model is approximate, but it captures the essential physics.

### 5.3 Interpretation of the Model

The model tells us three important things:

1. **The voice coil has resistance**
   This gives a real-valued baseline impedance.

2. **The voice coil has inductance**
   This causes impedance to rise at high frequency.

3. **The moving system resonates mechanically**
   Around resonance, the diaphragm motion and back electromotive force can produce a pronounced impedance peak.

Therefore, the driver's impedance is not flat across frequency.

---

## 6. Special Case: Balanced Armature (BA) Earphones

Balanced-armature (BA) earphones are not an exception to the source-impedance rule. A BA driver is still an electro-mechano-acoustic transducer rather than a fixed resistor. Knowles describes the BA driver as a tiny reed balanced between magnets, with its motion transferred to a stiff diaphragm, while a JASA paper models the balanced-armature speaker as a coupled electromechanoacoustical system. Therefore, the same transfer-function logic used earlier in this lecture still applies:

\[
H(j\omega)=\frac{V_L(j\omega)}{V_s(j\omega)}=\frac{Z_L(j\omega)}{Z_s+Z_L(j\omega)}
\]

If the load impedance \(Z_L(j\omega)\) varies with frequency, then a series resistor or any nonzero source impedance can still alter the voltage delivered to the earphone as a function of frequency.

### 6.1 What Is Electrically Different About a BA Driver?

Compared with a moving-coil dynamic driver, a BA driver has a different physical construction and different impedance behavior. In Knowles' technical description, the coil is stationary in the balanced-armature driver, whereas the coil moves in a dynamic speaker. In a Knowles white paper comparing one BA family against a small dynamic speaker, the BA device transitions from predominantly resistive to inductive behavior at much lower frequencies—typically around 100 Hz to 300 Hz—while the compared dynamic speaker remains predominantly resistive throughout the audio band and beyond. This does **not** mean every BA earphone behaves identically, but it does mean that BA earphones should not be treated as simple flat resistive loads.

### 6.2 Single-BA, Multi-BA, and Hybrid IEMs Must Be Distinguished

Not all BA earphones are electrically similar. Official product information shows a wide range of architectures and nominal impedances: Etymotic ER3XR is a **single-BA** design rated at **22 \(\Omega\) @ 1 kHz** and explicitly marketed as avoiding the added complexity of crossovers; Etymotic ER4SR is also **single-BA**, rated at **45 \(\Omega\) @ 1 kHz**; Shure AONIC 3 is a **single balanced-armature** model rated at **26 \(\Omega\) @ 1 kHz**; Shure AONIC 5 is a **triple balanced-armature** model rated at **36 \(\Omega\) @ 1 kHz**, with two woofers and a separate tweeter; and Shure AONIC 4 is a **hybrid** earphone using one dynamic driver plus one balanced-armature driver, rated at **7 \(\Omega\) @ 1 kHz**. Knowles also states that its TWFK series enables customized crossover systems for pro-audio in-ear applications. So "BA earphone" is not one electrical category but a family of topologies with substantially different interface behavior.

### 6.3 Why Multi-BA and Hybrid Earphones Can Be More Source-Impedance-Sensitive

A reasonable engineering inference from the cited product architectures is that many **multi-BA** and **hybrid** earphones may be more sensitive to source impedance than a single-driver design, because the load seen by the amplifier may include multiple driver branches and crossover elements rather than a single transducer alone. A simplified abstraction for one possible multi-branch load is

\[
Z_L^{-1}(j\omega)=\sum_k \frac{1}{Z_{\mathrm{drv},k}(j\omega)+Z_{\mathrm{xo},k}(j\omega)}
\]

where each branch contains a driver and its associated crossover elements. As the load becomes more structured, the source-to-load transfer function becomes more dependent on \(Z_s\), and a series resistor may change not only the overall level but also the relative electrical balance between low-, mid-, and high-frequency branches. This is one reason why the phrase "a resistor just turns the volume down" is especially unsafe for multi-driver BA or hybrid IEMs.

### 6.4 Practical Engineering Guidance for BA IEMs

For BA earphones—especially low-impedance, multi-driver, or hybrid IEMs—the safest design rule is to keep source impedance very low. TI states that nonzero output impedance can cause some frequencies to be artificially attenuated or accentuated, and ADI explicitly recommends making the series output resistance \(R_S\) low from a performance perspective. This recommendation is consistent with current earphone-amplifier practice: Shure's SHA900 portable listening amplifier specifies **0.35 \(\Omega\)** output impedance, and TI's headphone-amplifier reference design reports a calculated closed-loop output impedance of about **0.0317 \(\Omega\)** at 1 kHz. If a series resistor is added for hot-plug protection, current limiting, or amplifier stability, it should be presented as a deliberate engineering trade-off rather than as an acoustically neutral attenuator.

---

## 7. Typical Impedance Behavior of Dynamic Transducers

A dynamic headphone or loudspeaker often shows the following broad pattern:

### 7.1 Low-Frequency Region

Near the fundamental mechanical resonance \(f_s\):

- diaphragm motion is large;
- back EMF increases;
- the electrical input impedance often shows a pronounced peak.

### 7.2 Midband Region

Away from resonance:

- the impedance is often closer to the voice-coil resistance plus smaller motional contributions;
- this region is often more stable and less dramatic than the resonant region.

### 7.3 High-Frequency Region

At higher frequencies:

- the inductive term \(j\omega L_e\) becomes more important;
- input impedance often rises again.

### 7.4 Important Engineering Caution

The term **nominal impedance** is only a label. It does **not** mean the transducer behaves like a fixed resistor equal to that number at all frequencies.

A headphone labeled as \(16\,\Omega\) may be close to that value over part of the midband, but it can be significantly higher near resonance and may rise again toward high frequencies.

---

## 8. Returning to the Case Study: What the Series Resistor Actually Does

Let the added series resistor be \(R_s = 16\,\Omega\).
Let the headphone load be \(Z_L(j\omega)\).

Then the transfer function becomes

\[
H(j\omega) = \frac{Z_L(j\omega)}{16 + Z_L(j\omega)}
\]

where the \(16\) is treated as a real-valued resistance in ohms.

### 8.1 Why the Attenuation Is Not Constant

The load voltage depends on the ratio of the load impedance to the total impedance. If the load impedance changes with frequency, then this ratio changes with frequency.

Therefore, the resistor does **not** simply "remove the same amount" of signal at every frequency.

### 8.2 A Magnitude-Only Approximation

For intuition, suppose the load impedance magnitude behaves approximately as follows:

- near \(1\,\text{kHz}\): \(|Z_L| \approx 16\,\Omega\)
- near low-frequency resonance: \(|Z_L| \approx 32\,\Omega\)
- at a higher frequency: \(|Z_L| \approx 24\,\Omega\)

If we ignore phase for the moment and use a magnitude-only approximation:

#### Midband

\[
|H| \approx \frac{16}{16+16} = 0.50
\]

So the voltage attenuation is

\[
20\log_{10}(0.50) \approx -6.02\,\text{dB}
\]

#### Resonance Region

\[
|H| \approx \frac{32}{16+32} = \frac{2}{3} \approx 0.667
\]

So the voltage attenuation is

\[
20\log_{10}(0.667) \approx -3.52\,\text{dB}
\]

#### Higher-Frequency Example

\[
|H| \approx \frac{24}{16+24} = 0.60
\]

So the voltage attenuation is

\[
20\log_{10}(0.60) \approx -4.44\,\text{dB}
\]

### 8.3 Interpretation

Relative to the midband example:

- the low-frequency resonant region is attenuated **less**;
- the higher-frequency region may also be attenuated **less**, depending on the impedance rise;
- the final tonal effect depends on the actual impedance curve and acoustic transfer function of the headphone.

Therefore, the correct statement is **not**:

> "A series resistor always makes the sound bass-heavy."

The correct statement is:

> A series resistor makes the output **load dependent**.
> The tonal change depends on the headphone's actual impedance-versus-frequency behavior.

For many dynamic headphones, low-frequency resonance and high-frequency inductive rise mean that the midband may be attenuated more strongly than bass or upper frequencies. But the exact spectral effect is device specific.

---

## 9. Complex Transfer Functions and Why Phase Also Matters

The previous example used impedance magnitudes for intuition. Strictly speaking, the exact calculation requires the full complex impedance.

### 9.1 Exact Expression

\[
H(j\omega) = \frac{Z_L(j\omega)}{Z_s + Z_L(j\omega)}
\]

The magnitude response is:

\[
|H(j\omega)| = \left|\frac{Z_L(j\omega)}{Z_s + Z_L(j\omega)}\right|
\]

The phase response is:

\[
\angle H(j\omega) = \angle Z_L(j\omega) - \angle\bigl(Z_s + Z_L(j\omega)\bigr)
\]

### 9.2 Why Phase Is Relevant

If the headphone impedance contains significant reactive content, then:

- the magnitude of attenuation changes with frequency;
- the phase of the voltage transfer can also change;
- the source current is frequency dependent in both amplitude and phase.

In many introductory discussions, only the magnitude effect is emphasized, but the complete engineering analysis is complex-valued.

---

## 10. Source Impedance and the Meaning of Voltage Drive

A conventional high-quality headphone amplifier is usually designed to behave approximately as a **voltage source**, which means:

\[
|Z_s| \ll |Z_L(j\omega)|
\]

over the relevant frequency range.

### 10.1 Why Low Source Impedance Is Usually Preferred

If \(Z_s\) is very small compared with \(Z_L(j\omega)\), then

\[
H(j\omega) \approx 1
\]

and the delivered voltage becomes relatively insensitive to variations in the load.

In other words, the source "holds" the voltage more firmly.

### 10.2 What Happens When We Add a Large Series Resistor

Adding a series resistor increases source impedance. This shifts the drive condition away from an ideal voltage source and somewhat toward **current-source behavior**.

That shift matters because many conventional headphones and loudspeakers are designed and voiced assuming **low source impedance**.

---

## 11. Damping Factor and Electromechanical Control

One of the most frequently discussed consequences of source impedance is **damping factor**.

### 11.1 Basic Definition

In simplified engineering practice:

\[
DF = \frac{Z_{L,\text{nom}}}{Z_s}
\]

or, in a purely resistive approximation:

\[
DF = \frac{R_L}{R_s}
\]

### 11.2 Physical Interpretation

When a driver moves inside a magnetic field, it generates **back EMF**. A low source impedance provides a stronger electrical path through which that back EMF can be dissipated. This contributes to **electrical damping** of the moving system.

A higher source impedance reduces that electrical damping contribution.

### 11.3 Important Nuance

Damping factor is a useful shorthand, but it is **not** a complete theory of sound quality.

Why?

Because the final behavior depends on:

- the transducer's mechanical damping;
- the acoustic loading;
- the electrical impedance curve;
- the operating level;
- the degree of nonlinearity;
- the frequency region being discussed.

So the statement

> "If damping factor is low, the sound is ruined"

is too absolute for a formal lecture.

A better statement is:

> Lower damping factor generally reduces electrical control over the driver and can worsen load-dependent response errors, especially near resonance, but the audible result depends on the full electromechanical system.

---

## 12. Advanced Engineering Note: Source Resistance and Resonance \(Q\)

For loudspeaker theory, added series resistance is often discussed using the electrical quality factor \(Q_{es}\).

If external series resistance \(R_s\) is added to a driver with voice-coil resistance \(R_e\), a common approximation is

\[
Q_{es}' = Q_{es}\left(1+\frac{R_s}{R_e}\right)
\]

This means the effective electrical \(Q\) increases as source resistance increases.

If the mechanical quality factor is \(Q_{ms}\), then the total quality factor becomes

\[
Q_{ts}' = \frac{Q_{ms}Q_{es}'}{Q_{ms}+Q_{es}'}
\]

### Interpretation

As series resistance increases:

- electrical damping is reduced;
- the resonance can become less controlled;
- low-frequency behavior may change.

This framework is especially familiar in loudspeaker design, but the underlying principle also helps explain why source resistance can matter in headphone systems.

---

## 13. Distortion and Nonlinear Effects

So far, we have mostly discussed small-signal linear theory. Real transducers are not perfectly linear.

### 13.1 Why Real Drivers Are Nonlinear

In practice, several parameters may vary with operating point:

- force factor \(Bl\)
- suspension compliance
- voice-coil position in the magnetic gap
- inductance with excursion
- thermal resistance effects

### 13.2 Why Source Impedance Can Interact with Nonlinearity

If the load current is nonlinear and the source impedance is nonzero, then the terminal voltage delivered to the transducer can also be modulated by that current variation.

That means source impedance can influence not only frequency response but also:

- harmonic distortion,
- intermodulation behavior,
- large-signal dynamics.

### 13.3 Engineering Interpretation

A low source impedance is generally preferred when the design goal is:

- predictable voltage drive,
- minimal load interaction,
- reduced sensitivity to the driver's nonlinear current behavior.

---

## 14. Power Dissipation and Efficiency

A series resistor does not merely reshape the transfer function. It also dissipates power.

### 14.1 Power in the Series Resistor

\[
P_R = I^2 R_s
\]

or, in RMS terms,

\[
P_R = I_{\text{rms}}^2 R_s
\]

### 14.2 Engineering Consequences

This has several practical implications:

- reduced electrical efficiency;
- lower maximum achievable acoustic output for a given source voltage;
- unnecessary heating in the resistor;
- possible mismatch between expected and actual listening level.

Thus, the series resistor is not a "free" attenuator.

---

## 15. Why Engineers Sometimes Still Use Output Series Resistors

It is important not to overcorrect into another myth.

The statement

> "A series resistor is always bad engineering"

is also false.

### 15.1 Legitimate Reasons for Using a Series Resistor

Engineers may intentionally use output series resistance for reasons such as:

- protection of the amplifier;
- current limiting;
- improved robustness under shorts or hot-plugging;
- stabilization against difficult loads or cable capacitance;
- intentional system voicing in special designs.

### 15.2 The Correct Engineering View

A series resistor is not inherently "wrong."
It is a **trade-off**.

What matters is whether the designer understands and accepts the consequences:

- more source impedance,
- more load dependence,
- lower damping factor,
- more power loss,
- potentially more distortion.

---

## 16. Better Ways to Reduce Listening Level

If the design goal is simply "lower the headphone volume without changing tonal balance," then placing a large resistor in series with the final transducer is usually not the best first choice.

### 16.1 Preferred Approaches

More controlled options include:

- attenuation at line level before the power stage;
- volume control inside the amplifier's gain structure;
- digital attenuation, provided adequate headroom and noise performance are maintained;
- a properly engineered attenuator network designed with the actual load behavior in mind.

### 16.2 Practical Rule

For transparent reproduction, engineers typically prefer:

- low output impedance,
- known gain structure,
- measurement with the real load rather than only a resistor dummy load.

---

## 17. Common Misconceptions

| Misconception | Correction |
|---|---|
| A linear passive resistor must attenuate all frequencies equally. | Only if the load is frequency independent. The combined network can be linear and still frequency selective. |
| A headphone labeled \(16\,\Omega\) behaves like a \(16\,\Omega\) resistor at all frequencies. | The label is nominal. Real transducers usually have frequency-dependent impedance. |
| If the same current flows through all series elements, the attenuation must be the same at all frequencies. | The current itself depends on the total complex impedance, which varies with frequency. |
| Damping factor alone determines sound quality. | Damping factor is useful, but the full electromechanical system determines the outcome. |
| A series resistor always makes the sound bass-heavy. | Not necessarily. The direction of change depends on the load impedance curve and the acoustic transfer function. |
| A series resistor is always bad design. | Not always. It may be used intentionally for protection or stability, but it introduces trade-offs. |

---

## 18. Summary

The introductory claim in the case study is **not generally correct**.

The correct engineering conclusions are:

1. A real headphone or loudspeaker is not a constant resistor; it is an electroacoustic transducer with frequency-dependent impedance.
2. A series resistor and a transducer form a voltage divider whose transfer function is

   \[
   H(j\omega)=\frac{Z_L(j\omega)}{Z_s + Z_L(j\omega)}
   \]

3. If the load impedance varies with frequency, the delivered voltage also varies with frequency.
4. Therefore, a series resistor may alter tonal balance rather than acting as a perfectly neutral volume control.
5. Higher source impedance also reduces electrical damping, wastes power, and may worsen distortion in practical systems.
6. For transparent voltage-drive operation, low source impedance is usually preferred.
7. If a series resistor is used, it should be treated as a deliberate engineering trade-off, not as an acoustically neutral assumption.

This warning applies not only to dynamic drivers but also to balanced-armature, multi-balanced-armature, and hybrid IEMs; in those systems, the dominant mechanism may be source-impedance interaction with a complex load and any passive crossover network, rather than only classical moving-coil damping.

---

## 19. Suggested Further Reading

- Douglas Self, *Audio Power Amplifier Design*
- Leo L. Beranek and Tim Mellow, *Acoustics: Sound Fields and Transducers*
- Application notes on headphone driver output impedance, load interaction, and loudspeaker impedance measurement from major analog IC manufacturers