---
title: "Rabab Tuner: Change of Frequency Detection Algorithm"
date: 2025-12-23
project: rabab-tuner
---
Date: 2025-12-23

My initial implementation of the Rabab tuner used FFT, and it felt like the obvious starting point. Now that I have it somewhat functioning, I notice how inaccurate FFT is for real, imperfect audio samples.

My main misunderstanding was trying to find the pitch using FFT. That is not its function; it presents frequencies present within a signal. I would take these frequencies and accept the strongest peak as pitch, the fundamental frequency (`f0`). However, `f0` does not have to be the strongest component of the signal. Harmonics can dominate the spectrum, and noise and string resonances can introduce unwanted peaks. The lesson learned is that the greatest-magnitude FFT peak cannot reliably determine `f0`.

There are other issues as well, like frequency bins, which are discrete frequency intervals produced by FFT. Larger bin spacing leads to less accurate pitch estimates, and this depends on window length. If I need faster analysis, `f0` detection can become inaccurate. If I prioritize accuracy, I need a longer window, which increases analysis time.

This is where the YIN algorithm comes in. I would describe it as based on autocorrelation with additional optimization. The error rate produced by this algorithm is far lower than traditional FFT methods for pitch tracking.

FFT asks which frequencies are present within a signal, but YIN measures the time it takes for the signal to repeat itself.
