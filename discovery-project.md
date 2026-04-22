# Acoustic State Machine & Real-Time Analyzer
**ECE 1100 Discovery Project Showcase**

**Developer:** Harshit Singhal  
**Major:** Computer Engineering  
**Institution:** Georgia Institute of Technology  

---

## Discovery Project Idea
The initial goal of this project was to engage in hands-on exploration of digital signal processing within a low-stakes environment. I wanted to build a bridge between the physical world of unseen analog data (ambient room audio) and the digital world by creating a tool that visualizes a real-time frequency spectrum using Python. This aligns with my broader interests in autonomous systems and how hardware interfaces with environmental sensors.

## Project Progress
The project evolved significantly over the course of its development. What started as a basic, single-line data visualizer was iteratively upgraded into a robust piece of analysis software. Key milestones included:

* Capturing live audio buffers directly from the system microphone.
* Implementing a mathematical windowing function (Hanning window) to prevent spectral leakage before executing the Fast Fourier Transform (FFT).
* Adding a decaying "Peak-Hold" line to track transient sound spikes.
* Developing a dominant frequency tracking algorithm using NumPy matrix operations.

> **Live Demonstration** > *[Insert your 30-second screen recording here: `![Demo Video](./demo_video.mp4)` or link to YouTube]*

## Project Successes and Failures
The development process presented several excellent engineering roadblocks that required debugging and system-level troubleshooting:

* **Failure/Roadblock:** Navigating strict macOS privacy architecture and hardware access protocols. Setting up isolated Python virtual environments to correctly bind the `sounddevice` library to the low-level PortAudio drivers was initially a major hurdle.
* **Roadblock:** Balancing the FFT `BLOCK_SIZE`. I had to experiment to find the exact mathematical trade-off: too large of a block size resulted in a highly detailed but visually lagging graph, while too small of a block updated instantly but suffered from poor frequency resolution.
* **Success:** Successfully integrating a logical State Machine overlay that parses noisy analog inputs into clean, binary system states without failing or crashing.

## ECE Skills Gained
This project bridged the gap between software engineering and fundamental digital logic design. The core ECE skills developed include:

* **Digital Signal Processing (DSP):** Practical application of the Fast Fourier Transform to convert time-domain signals into the frequency domain.
* **Hardware/Software Integration:** Pulling raw sensor data (microphone inputs) into a software environment in real-time.
* **Finite State Machine Logic:** Establishing strict `OK`, `WARNING`, and `ERROR` states based on real-time decibel thresholds within specific acoustic frequency bands (200Hz - 2kHz). This mimics the logic used in hardware safety controls.

## Final Thoughts & Next Steps
Completing this project was a highly rewarding experience that solidified my interest in the intersection of hardware logic and software control. Being able to visualize the immediate mathematical results of the code in the physical world is much more engaging than purely theoretical work. Moving forward, I am interested in potentially migrating this Python logic onto a dedicated hardware microcontroller, or expanding the software to include a rolling 2D "Waterfall" spectrogram for time-history analysis.

---

## Core Source Code
Below is the Python snippet driving the active state machine and FFT visualization.

```python
# A snippet of the core logic processing the FFT and State Machine
def update_plot(frame):
    global audio_data, peak_data
    
    # 1. Math: Windowing and FFT
    yf = np.fft.rfft(audio_data * WINDOW)
    magnitude = np.abs(yf) * 2 / BLOCK_SIZE
    magnitude_db = 20 * np.log10(magnitude + 1e-9) 
    
    # 2. Peak Hold Decay
    peak_data = np.maximum(peak_data, magnitude_db)
    peak_data -= 1.0 # Decay rate
    
    # 3. STATE MACHINE LOGIC (200Hz - 2kHz)
    band_mask = (xf >= TARGET_FREQ_MIN) & (xf <= TARGET_FREQ_MAX)
    target_max_db = np.max(magnitude_db[band_mask]) if np.any(band_mask) else -60

    if target_max_db > -20:       # Loud sound (Clap)
        current_state = "ERROR"
    elif target_max_db > -40:     # Medium sound (Talking)
        current_state = "WARNING"
    else:                         # Quiet (Background)
        current_state = "OK"
        
    return live_line, peak_line, freq_text, state_text
