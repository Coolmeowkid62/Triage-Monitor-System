# Triage-Monitor-System

The goal is to design, build and calibrate a low-cost, non-invasive triage monitor that measures heart rate (HR), blood oxygen saturation (SpO₂), and skin temperature from a fingertip. Two optical sensing channels, one red and one infrared (IR), and a thermistor (a temperature-sensitive resistor) are read by an ESP32 microcontroller and processed in python. Triage, by definition, is the quick sorting of patients by how urgently they need care. Heart rate and SpO₂ are two of the fastest signs of cardiac or respiratory distress, so a device that shows both within seconds helps staff decide who is seen first.

The system measures SpO₂, the percentage of hemoglobin in the blood that is carrying oxygen: SpO2 = ( [HbO2] / [HbO2] + [Hb] ) x 100% where [HbO2] is the concentration of oxygenated hemoglobin and $[Hb]$ is the concentration of deoxygenated hemoglobin.

The system uses a red LED (~660 nm), an infrared (IR) LED (~940 nm) and a photodiode. The two LEDs are switched on one at a time and shine light through the fingertip. The photodiode, on the opposite side, captures the light that is not absorbed by the finger and converts it into an electrical signal. The amount of light absorbed is described by the Beer-Lambert law: I = I0^e-(lambda)cd 

The finger is made of multiple layers. Skin, tissue and bone absorb the same amount of light at all times, so their absorption does not change as the heart beats. Venous blood (blood in the veins) also changes very little, because veins are under low pressure. Arterial blood (blood in the arteries), on the other hand, is pushed into and out of the fingertip with each heartbeat. This briefly increases the path length $d$ and absorbs more light. As a result, the photodiode signal has two components: 
      - DC component: the large, steady voltage level set by skin, tissue, bone, venous blood and non-pulsing arterial blood.
      - AC component: the small fluctuation around the DC level that rises and falls with each heartbeat, caused only by the pulsing arterial blood.
      

Each peak of the AC waveform is one heartbeat, so the heart rate is: HR = 60 / T (bpm), where T is the time in seconds between two neighbouring peaks.

Both LEDs are needed because oxygenated and deoxygenated hemoglobin absorb the two wavelengths differently. At red, deoxygenated hemoglobin absorbs much more strongly than oxygenated hemoglobin, so the red signal is very sensitive to oxygen level. At IR, the two types absorb by similar amounts, with oxygenated hemoglobin absorbing slightly more. So the IR signal mainly shows how much blood is pulsing and acts as a reference.

