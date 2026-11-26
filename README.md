## Primary System Function and Design

The system uses three LEDs (red, infrared, and green) and a photodiode. The photodiode acts as the transducer: it converts the light returning from the tissue into an electrical signal (a photocurrent). A 3-axis accelerometer measures motion, so that motion artifacts can be cancelled from the PPG signal, or the affected segments flagged as unreliable.

The photocurrent has two components. The **DC component** comes from light absorbed by skin, tissue, bone, and venous blood, which stays roughly constant over time. The **AC component** comes from arterial blood. With each heartbeat, the volume of arterial blood in the tissue rises and falls, which changes how much light is absorbed and produces a small pulsatile signal on top of the DC level.

Red and infrared light are both needed to calculate blood oxygen saturation (SpO2). Oxygenated and deoxygenated hemoglobin absorb these two wavelengths differently, so comparing the pulsatile signal at each wavelength gives the ratio:

```math
R = \frac{AC_{red} / DC_{red}}{AC_{IR} / DC_{IR}}
```

which is converted to SpO2 through a calibration curve.

Green light is used for heart rate because it is more robust to motion. Blood absorbs green light strongly, so the AC component makes up a larger fraction of the total signal. Green light also only reaches about 1 mm into the skin, so it mostly samples surface capillaries and avoids much of the motion noise from deeper tissue.

## Analog Front End Architecture

The photodiode produces a photocurrent proportional to the light it receives. A transimpedance amplifier (TIA) converts this current into a voltage:

```math
V_{out} = I_{PD} R_f
```

Because the AC and DC components have not yet been separated, the TIA gain is kept moderate so that the DC level sits around 50–70% of the ADC's full scale. This leaves headroom so that motion does not cause clipping.

The ADC digitizes the TIA output. Because the system uses time-division multiplexing, the firmware sorts each sample into its channel (green, red, IR, or ambient) in the following order:

| Time Slot     | LEDs    | What You Sample        |
|---------------|---------|------------------------|
| 1             | Green   | Green signal + ambient |
| 2             | Red     | Red signal + ambient   |
| 3             | IR      | IR signal + ambient    |
| 4             | All off | Ambient only           |
| Rest of frame | All off | Nothing (sleep)        |

Here, *ambient* is the light reaching the photodiode that was not emitted by the LEDs. Subtracting the ambient sample from each LED channel removes this background light.

## Motion Artifact Removal

Motion artifacts are removed in firmware using a **normalized least mean squares (NLMS) adaptive filter**, with the accelerometer as the noise reference. The measured PPG signal contains both the heart signal $`s[n]`$ and the motion artifact $`v[n]`$. The filter reshapes the accelerometer signal $`\mathbf{x}[n]`$ into an estimate of the motion artifact, $`\mathbf{w}^T[n]\,\mathbf{x}[n]`$, and subtracts it:

```math
e[n] = s[n] + v[n] - \mathbf{w}^T[n]\,\mathbf{x}[n]
```

The output $`e[n]`$ is the cleaned heart signal. The filter weights are updated every sample to minimize the remaining motion-related error:

```math
\mathbf{w}[n+1] = \mathbf{w}[n] + \frac{\mu \, e[n] \, \mathbf{x}[n]}{\lVert \mathbf{x}[n] \rVert^2 + \epsilon}
```

Here $`\mu`$ is the step size, which controls how quickly the filter adapts. Dividing by $`\lVert \mathbf{x}[n] \rVert^2`$, the recent motion energy, keeps the step size consistent whether the user is still or moving vigorously, and $`\epsilon`$ is a small constant that prevents division by zero.
