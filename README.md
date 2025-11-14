# Thalamocortical Neural Network Model for Sleep Simulation

## Overview

This project implements a biologically-inspired thalamocortical neural network computational model using the Brian2 simulator to study neural activity patterns and EEG signal characteristics during different sleep stages. The model integrates multi-layered cortical structures with thalamic networks to reproduce neural oscillations characteristic of various sleep states.

## Computational Models Used

### 1. **Izhikevich Neuron Model**
The cortical and thalamic neurons are implemented using the Izhikevich spiking neuron model, which provides computational efficiency while capturing rich dynamics:

```
dv/dt = (0.04*v² + 5*v + 140 - u + I)/ms
du/dt = a*(b*v - u)/ms
```

With spike-triggered reset: when v ≥ 30 mV, then v ← c and u ← u + d

Different neuron types are configured with specific parameters:
- **Regular Spiking (RS)**: Excitatory cortical neurons (a=0.02, b=0.2, c=-65, d=2)
- **Fast Spiking (FS)**: Parvalbumin (PV) interneurons in L4 and L6 (a=0.1, b=0.2, c=-65, d=2)
- **Somatostatin (SOM)**: Slower interneurons in L2/3 and L5 (a=0.1, b=0.2, c=-65, d=2)

### 2. **T-type Calcium Channel Model**
Thalamic relay neurons incorporate T-type calcium channels to generate sleep-related oscillations:

```
I_T_Ca = gT * m³ * h * (v - E_Ca)
dm/dt = (m_∞ - m)/τ_m
dh/dt = (h_∞ - h)/τ_h
```

Where:
- m_∞ = 1/(1 + exp(-(v+50)/7))
- h_∞ = 1/(1 + exp((v+70)/5.5))
- τ_m = 5 ms, τ_h = 15 ms

This mechanism is crucial for generating spindle oscillations and slow-wave activity during sleep.

### 3. **Synaptic Current Model**
Synaptic interactions are modeled using exponential decay dynamics:

```
dI_exc/dt = -I_exc/τ_exc
dI_inh/dt = -I_inh/τ_inh
```

With τ_exc = 5 ms for excitatory (AMPA-like) and τ_inh = 10 ms for inhibitory (GABA-like) synapses.

### 4. **Potassium Leak Conductance**
Sleep depth is modulated through adjustable potassium leak conductance:

```
I_KL = g_KL * (v - E_KL)
```

This parameter varies across sleep stages to control neuronal excitability.

### 5. **Welch Power Spectral Density Analysis**
LFP-like signals are analyzed using Welch's method for PSD estimation with:
- Sampling frequency: 10 kHz
- Window size: 4096 or 2048 samples
- 50% overlap between segments

## Key Features

### Network Architecture

**Cortical Layers:**
- **L2/3**: 80 excitatory (RS) + 20 inhibitory (SOM) neurons
- **L4**: 80 excitatory (RS) + 20 inhibitory (PV) neurons  
- **L5**: 80 excitatory (RS) + 20 inhibitory (SOM) neurons
- **L6**: 80 excitatory (RS) + 20 inhibitory (PV) neurons

**Thalamic Structures:**
- **Relay neurons**: 100 neurons with T-type Ca²⁺ channels
- **Reticular nucleus (nRt)**: 100 inhibitory neurons
- **Local inhibitory neurons**: Supporting population

### Sleep Stage Simulation

The model supports five sleep/wake states with distinct parameters:

| Stage | Description | Key Parameters |
|-------|-------------|----------------|
| **Wake** | Alert wakefulness | Low K⁺ leak, high noise |
| **N1** | Light sleep | Moderate parameters |
| **N2** | Intermediate sleep | Spindle activity |
| **N3** | Deep slow-wave sleep | High K⁺ leak, low noise |
| **REM** | Rapid eye movement | Similar to wake, desynchronized |

Each stage is characterized by specific patterns of:
- Potassium leak conductance (g_KL)
- Noise intensity (σ_noise)
- Dominant frequency bands

### Synaptic Connectivity

The network implements biologically-inspired connectivity patterns:
- **Intra-layer**: Local excitation and inhibition within each cortical layer
- **Inter-layer**: Feedforward (L4→L2/3→L5→L6) and feedback connections
- **Thalamocortical**: Bidirectional L4↔Relay and L6↔Relay loops
- **Thalamic**: Reciprocal Relay↔nRt connections
- **Transmission delays**: 15 ms for thalamocortical, 3 ms for intracortical

### Signal Analysis & Visualization

1. **Raster Plots**: Spike timing visualization for all neuronal populations
2. **Local Field Potential (LFP)**: Simulated by averaging membrane potentials
3. **Power Spectral Density**: Frequency domain analysis identifying:
   - Delta (0.5-4 Hz): Deep sleep
   - Theta (4-8 Hz): Light sleep, drowsiness
   - Alpha (8-12 Hz): Relaxed wakefulness
   - Beta (12-30 Hz): Active thinking
   - Gamma (30-60 Hz): Cognitive processing

## Technical Stack

- **Python 3.x**
- **Brian2**: Spiking neural network simulator
- **NumPy**: Numerical computations
- **Matplotlib**: Data visualization
- **SciPy**: Signal processing and spectral analysis

## Installation & Usage

### Environment Setup
```bash
pip install brian2 numpy matplotlib scipy jupyter
```

### Running the Simulation

Open the Jupyter notebook and execute cells sequentially:

**Cell 1: Network Construction and Simulation**
- Configure network parameters
- Create neuron groups with Izhikevich dynamics
- Establish synaptic connections
- Set up monitors (spike and state monitors)
- Run simulation with `run(duration)`

**Cell 2: Power Spectral Density Analysis**
- Compute LFP proxy signals from state monitors
- Apply Welch's method for PSD estimation
- Generate normalized spectrograms with frequency band annotations

### Parameter Customization

Explore different simulation scenarios by modifying:

```python
# Select sleep stage
sleep_stage = 'N3'  # Options: 'Wake', 'N1', 'N2', 'N3', 'REM'

# Simulation duration
duration = 10*second

# Layer-specific noise intensity
sigma_noise_L23_exc = 1.0
sigma_noise_L4_exc = 1.0
sigma_noise_L5_exc = 1.0
sigma_noise_L6_exc = 0.3  # Lower for deeper layers

# Potassium leak conductance (sleep depth)
g_KL_value = 0.001  # Increase for deeper sleep
```

## Output Files

The simulation generates:

1. **`raster_plots.png`**: Six-panel figure showing spike patterns across neuronal populations
2. **`lfp_psd_linear_normalized_*.png`**: Layer-specific LFP power spectral density plots with frequency band markers

## Scientific Applications

This model is suitable for investigating:

- **Sleep neuroscience**: Mechanisms underlying sleep stages
- **Slow-wave oscillations**: Generation of delta rhythms in N3 sleep
- **Spindle activity**: Thalamic contribution to sleep spindles in N2
- **Thalamocortical dynamics**: Circuit-level interactions during sleep
- **Computational psychiatry**: Sleep disorder modeling
- **EEG interpretation**: Theoretical basis of clinical EEG features

## Model Limitations

- Simplified neuronal models (no detailed dendritic computations)
- Relatively small network size (400 cortical + 200 thalamic neurons)
- Idealized synaptic connectivity patterns
- Lacks neuromodulatory systems (acetylcholine, norepinephrine, etc.)
- No spatial structure or topographic organization

## Future Improvements

- [ ] Scale up network size for improved biological realism
- [ ] Integrate additional cortical layers (L1)
- [ ] Implement neuromodulatory dynamics (ACh, NE, 5-HT)
- [ ] Add adaptive mechanisms for spontaneous sleep stage transitions
- [ ] Incorporate sensory input pathways
- [ ] Include spatial structure and columnar organization
- [ ] Validate against experimental sleep EEG data

## References

This model builds upon foundational work in computational sleep neuroscience:

1. **Izhikevich, E. M. (2003).** "Simple model of spiking neurons." *IEEE Transactions on Neural Networks*, 14(6), 1569-1572.
2. **Destexhe, A., Contreras, D., & Steriade, M. (1998).** "Mechanisms underlying the synchronizing action of corticothalamic feedback through inhibition of thalamic relay cells." *Journal of Neurophysiology*, 79(2), 999-1016.
3. **Bazhenov, M., Timofeev, I., Steriade, M., & Sejnowski, T. J. (2002).** "Model of thalamocortical slow-wave sleep oscillations and transitions to activated states." *Journal of Neuroscience*, 22(19), 8691-8704.
4. **Hill, S., & Tononi, G. (2005).** "Modeling sleep and wakefulness in the thalamocortical system." *Journal of Neurophysiology*, 93(3), 1671-1698.

## License

This project is intended for academic research and educational purposes only.

## Contact

For questions or suggestions:
- Submit an issue to this repository
- Contact the project maintainer via email

---

**Note**: This is a simplified research tool and should not be used for clinical diagnosis or treatment decisions.
