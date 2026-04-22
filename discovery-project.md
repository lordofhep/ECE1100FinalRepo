---
title: Discovery Project
---

# Acoustic State Machine and Real-Time Analyzer
**ECE 1100 Discovery Project Showcase**

**Developer:** Harshit Singhal  
**Major:** Computer Engineering  
**Institution:** Georgia Institute of Technology  

---

## 1. Project Genesis & Core Objective

In the study of autonomous and agentic systems, the boundary between physical environments and digital logic is a critical frontier. The objective of this Discovery Project was to engage in hands-on exploration of that boundary by building a bridge between unseen analog data (ambient room acoustics) and discrete software architecture. 

Rather than building a passive monitoring tool, my goal was to design an active system capable of capturing noisy real-world inputs, processing them in real-time via digital signal processing (DSP), and utilizing a finite state machine to make deterministic evaluations based on that data.

> *"The core engineering challenge was not merely capturing sound, but imposing strict, deterministic logic onto an inherently chaotic analog signal."*

## 2. System Architecture & Development Progress

The project evolved from a rudimentary data visualizer into a robust, multi-stage signal processing pipeline. The architecture was developed iteratively through the following technical milestones:

* **Signal Acquisition:** Engineered a continuous data pipeline to capture live audio buffers directly from the system microphone using low-level hardware drivers.
* **Spectral Transformation:** Implemented a mathematical windowing function (Hanning window) to mitigate spectral leakage before passing the time-domain signal through a Fast Fourier Transform (FFT) algorithm.
* **Data Persistence:** Developed a decaying "Peak-Hold" memory array to track transient sound spikes, allowing for the observation of temporary acoustic anomalies over time.
* **Logic Integration:** Designed a continuous evaluation loop utilizing NumPy matrix operations to actively isolate and track the dominant frequency within the environment.

> **Live System Demonstration** > *[Insert your 30-second screen recording here: `![Demo Video](./demo_video.mp4)` or link to YouTube]*

## 3. Engineering Roadblocks & Iterative Solutions

The development lifecycle presented several excellent technical roadblocks that required deep system-level troubleshooting and algorithmic balancing:

* **The Hardware Abstraction Barrier:** Navigating macOS’s strict privacy architecture and low-level hardware access protocols proved challenging. Successfully binding the Python `sounddevice` library to the underlying PortAudio C-libraries required establishing isolated virtual environments to prevent dependency conflicts with the operating system.
* **Algorithmic Trade-offs:** Tuning the FFT `BLOCK_SIZE` provided a practical lesson in fundamental DSP trade-offs. Expanding the block size yielded a highly detailed frequency resolution but introduced unacceptable latency into the visualization. Conversely, minimizing the block size achieved instantaneous updates at the severe cost of data fidelity.
* **Taming Analog Noise:** The initial iterations of the state machine were overly sensitive to ambient room noise. I resolved this by mathematically masking the FFT array to create a specific "Detection Zone" (200Hz – 2kHz), ensuring the logic only evaluated relevant data within the human vocal range.

## 4. Fundamental ECE Skills Acquired

This project served as a practical synthesis of software engineering and foundational digital design principles. The primary competencies developed include:

* **Applied Digital Signal Processing:** Transitioning from theoretical mathematics to the practical application of Fourier transforms for real-time frequency domain analysis.
* **Hardware/Software Interfacing:** Establishing reliable, low-latency communication channels between physical sensors (microphones) and high-level software environments.
* **Finite State Machine (FSM) Design:** Translating core hardware logic concepts into software. I implemented a strict FSM architecture that categorizes dynamic decibel thresholds into immutable `OK`, `WARNING`, and `ERROR` states, mirroring the safety control logic utilized in complex physical systems.

## 5. Final Thoughts & Forward-Looking Applications

Completing this project was a highly rewarding experience that solidified my interest in the intersection of hardware logic, feedback systems, and software control. Observing the immediate mathematical execution of my code interacting with the physical world provided a level of engagement that purely theoretical coursework rarely achieves. 

Moving forward, I am interested in exploring how this type of deterministic acoustic logic can be applied to edge-computing environments. Potential next steps include migrating this Python architecture onto a dedicated hardware microcontroller, or expanding the software to incorporate a rolling 2D "Waterfall" spectrogram for extended time-history analysis.

---

## Appendix: Core Source Code

Below is the Python snippet driving the continuous evaluation loop, the FFT transformation, and the active state machine logic.

{% highlight python %}
# Core logic processing the FFT and State Machine
def update_plot(frame):
    global audio_data, peak_data
    
    # 1. Mathematical Transformation: Windowing and FFT
    yf = np.fft.rfft(audio_data * WINDOW)
    magnitude = np.abs(yf) * 2 / BLOCK_SIZE
    magnitude_db = 20 * np.log10(magnitude + 1e-9) 
    
    # 2. Transient Memory: Peak Hold Decay
    peak_data = np.maximum(peak_data, magnitude_db)
    peak_data -= 1.0 # Applied decay rate
    
    # 3. DETERMINISTIC STATE MACHINE (Target: 200Hz - 2kHz)
    band_mask = (xf >= TARGET_FREQ_MIN) & (xf <= TARGET_FREQ_MAX)
    target_max_db = np.max(magnitude_db[band_mask]) if np.any(band_mask) else -60

    # Evaluate dynamic thresholds into strict states
    if target_max_db > -20:       # Breach threshold (Clap/Snap)
        current_state = "ERROR"
    elif target_max_db > -40:     # Elevated threshold (Talking)
        current_state = "WARNING"
    else:                         # Baseline threshold (Background)
        current_state = "OK"
        
    return live_line, peak_line, freq_text, state_text
{% endhighlight %}
