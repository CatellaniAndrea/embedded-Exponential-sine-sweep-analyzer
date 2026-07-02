# Exponential Sine Sweep (ESS) Method

## Overview

The Exponential Sine Sweep (ESS) is an acoustic measurement technique that uses a chirp signal to characterize the frequency response of audio systems.

This document describes the ESS method, its implementation on the STM32L476RG, and practical measurement considerations.

## Theory

### Chirp Signal Generation

An exponential sine sweep is a sinusoidal signal whose frequency increases exponentially with time:

```
f(t) = f_start * exp(t * ln(f_end/f_start) / T)
```

Where:
- **f_start**: Initial frequency (Hz)
- **f_end**: Final frequency (Hz)
- **T**: Total sweep duration (seconds)
- **t**: Time from 0 to T (seconds)

### Phase Function

The instantaneous phase is calculated as:

```
φ(t) = 2π * f_start * T / ln(f_end/f_start) * (exp(t * ln(f_end/f_start) / T) - 1)
```

The output signal is:

```
s(t) = A * sin(φ(t))
```

Where **A** is the amplitude.

## Advantages of ESS

1. **Frequency Resolution**: Better resolution at lower frequencies
   - Linear frequency sweep has uniform resolution
   - Exponential sweep concentrates resolution where most needed

2. **Signal-to-Noise Ratio**: Improved SNR across frequency range
   - Energy distributed proportionally to frequency bandwidth
   - Constant SNR vs. frequency (ideal case)

3. **Measurement Speed**: Shorter measurement time
   - Full frequency sweep in single pass
   - Compared to stepped sine (N frequency points needed)

4. **System Stress**: Lower peak amplitude required
   - Constant amplitude throughout sweep
   - No amplitude spikes

## Deconvolution Process

### Recording Setup

```
Input Signal → DUT (Device Under Test) → Output Signal
          ↓                                ↓
       SAI2_SD_A                        SAI2_SD_B
           ↓                                ↓
        DMA1_CH6 ← ← ← DMA2_CH4
```

### Time-Domain Deconvolution

The impulse response h(t) is extracted by deconvolving the output y(t) with the input x(t):

```
h(t) = deconvolve(y(t), x(t))
```

In frequency domain:

```
H(f) = Y(f) / X(f)
```

Where H(f) is the frequency response (magnitude and phase).

## Implementation on STM32

### Signal Generation

#### Lookup Table Approach
- Pre-compute ESS chirp samples offline
- Store in FLASH as lookup table
- Read at sample rate during audio output
- Advantages: Fast, deterministic timing
- Disadvantages: Limited sweep duration in flash

#### Real-Time Generation
- Calculate samples on-the-fly
- Use floating-point arithmetic for phase
- Advantages: Flexible sweep parameters
- Disadvantages: Higher CPU load (~20-30% @ 80 MHz)

### Example Code Structure

```c
// Configuration
#define SAMPLE_RATE         48000   // Hz
#define SWEEP_DURATION      10      // seconds
#define FREQ_START          20      // Hz
#define FREQ_END            20000   // Hz
#define AMPLITUDE           32767   // Max 16-bit

// Sweep parameters
typedef struct {
    float f_start;
    float f_end;
    float duration;
    float log_ratio;     // ln(f_end/f_start)
    uint32_t sample_rate;
    uint32_t total_samples;
} ESS_Config_t;

// Generate ESS sample
int16_t ESS_GenerateSample(uint32_t sample_idx, ESS_Config_t *cfg)
{
    float t = (float)sample_idx / cfg->sample_rate;
    
    // Phase calculation
    float exp_factor = expf(t * cfg->log_ratio / cfg->duration);
    float phase = 2.0f * PI * cfg->f_start * cfg->duration / cfg->log_ratio 
                  * (exp_factor - 1.0f);
    
    // Wrap phase to [0, 2π)
    while (phase > 2.0f * PI) phase -= 2.0f * PI;
    
    // Sine and amplitude scaling
    int16_t sample = (int16_t)(cfg->amplitude * sinf(phase));
    return sample;
}
```

### DMA Circular Buffering

```
SAI2_TX Buffer Layout (Double Buffering):
┌─────────────────┐
│  Samples 0-1023 │ ← DMA reads from here
├─────────────────┤ ← DMA half-complete (process 0-1023)
│  Samples 1024-  │ ← CPU refills 0-1023
│   2047          │
└─────────────────┘ ← DMA complete (process 1024-2047)
↑
DMA pointer continuously cycles
```

### Interrupt-Driven Processing

```c
void DMA1_Channel6_IRQHandler(void)
{
    if (DMA1->ISR & DMA_ISR_HTIF6) {  // Half transfer
        DMA1->IFCR |= DMA_IFCR_CHTIF6;
        
        // Process first half (0 to BUFFER_SIZE/2)
        ProcessAudioBuffer(&ess_buf[0], BUFFER_SIZE/2);
    }
    
    if (DMA1->ISR & DMA_ISR_TCIF6) {  // Transfer complete
        DMA1->IFCR |= DMA_IFCR_CTCIF6;
        
        // Process second half
        ProcessAudioBuffer(&ess_buf[BUFFER_SIZE/2], BUFFER_SIZE/2);
    }
}
```

## Frequency Response Analysis

### Magnitude Response

The magnitude of the frequency response:

```
|H(f)| = |Y(f)| / |X(f)| (in frequency domain)
```

Expressed in dB:

```
|H(f)|_dB = 20 * log10(|H(f)|)
```

### Phase Response

```
∠H(f) = ∠Y(f) - ∠X(f)
```

Wrapped to [-π, π] range.

### Resolution

Frequency resolution in ESS:

```
Δf ≈ f_start / (T * ln(f_end/f_start)) * f  (varies with frequency)
```

For logarithmic resolution:

```
Δf_log ≈ constant (octaves)
```

## Performance Metrics

### Recommended Measurement Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| Sample Rate | 48 kHz | Nyquist limit 24 kHz |
| Sweep Duration | 5-10 s | Longer = better SNR |
| Frequency Range | 20-20k Hz | Human hearing range |
| Sweep Type | Log | Exponential recommended |
| Amplitude | 0.8 FS | Avoid clipping |
| Averaging | 3-5 sweeps | Reduce noise |

### SNR Improvement

SNR improves with sweep duration:

```
SNR_improvement = 10 * log10(T_sweep * sample_rate / 2)
```

Example:
- 1 second sweep: ~35 dB improvement
- 10 second sweep: ~45 dB improvement

## Practical Considerations

### Calibration

1. **Level Calibration**
   - Measure reference tone (1 kHz, 0 dBFS)
   - Adjust input/output gain accordingly

2. **Phase Calibration**
   - Account for system latency
   - Align input and output time axes
   - Use cross-correlation peak

### Windowing

For non-continuous signals, apply window function:

```
s_windowed(t) = s(t) * w(t)
```

Common windows:
- **Rectangular**: No windowing (default for ESS)
- **Hann**: Smoother frequency response
- **Hamming**: Better sidelobe rejection

### FFT Parameters

For frequency analysis:

```c
#define FFT_SIZE 2048        // 2048-point FFT
#define HOP_SIZE 512         // 75% overlap

// Frequency resolution
df = SAMPLE_RATE / FFT_SIZE  // 48000 / 2048 ≈ 23.4 Hz
```

## Measurement Examples

### Audio System Characterization

```
1. Generate ESS chirp (20 Hz - 20 kHz, 10 seconds)
2. Output through audio codec (DAC)
3. System under test (amplifier, speaker, room)
4. Capture through audio codec (ADC)
5. Deconvolve and compute frequency response
6. Display magnitude and phase plots
```

### Room Acoustic Measurement

```
1. Generate ESS in speaker
2. Microphone captures room response
3. Deconvolve to get room impulse response
4. Analyze reflections and absorption
5. Calculate room modes and resonances
```

## Computational Requirements

### Memory
- **ESS Buffer (10s @ 48kHz)**: 480 kB (within 512 kB FLASH)
- **Audio Input Buffer**: 4 kB (84 samples @ 16-bit)
- **FFT Buffer**: 8-16 kB (2048-4096 samples)
- **Total**: ~100 kB working memory

### Processing Time
- **Sample generation**: ~5 µs per sample (real-time)
- **FFT (2048 points)**: ~10 ms (ARM CMSIS-DSP)
- **Deconvolution**: ~50 ms per 1-second window

## References

1. Farina, A. (2007). "Advancements in the impulse response measurements by sine sweeps"
2. ISO 18233:2011 "Acoustics - Measurement of noise emitted by appliances and equipment"
3. ARM CMSIS-DSP Documentation
4. STM32L4 SAI Interface Reference Manual
