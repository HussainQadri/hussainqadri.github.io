---
title: "Rabab Tuner: Understanding the YIN Algorithm"
date: 2026-01-28
project: rabab-tuner
---
Date: 2026-01-28

I have started implementing YIN in `tuner_core.py`, but before doing so in the main project I needed a prototype that worked with artificial noisy signals and WAV audio files.

YIN fundamentally builds on ACF and adapts it into a difference function. When expanded and simplified, the difference function can be written using ACF terms, which makes implementation easier. But why switch to the difference function at all? ACF can prefer a lag that is too low, because small lags can maximize similarity between lagged and original signals.

Instead, the difference function starts with the idea that the difference between the original signal and a signal shifted by the true period `T` should be zero. Squaring and summing over the signal preserves this condition. In practice, we look for minima in this function because exact zero is unrealistic for real signals.

The next stage in the paper is still something I am building intuition for.

The difference function is zero at zero-lag and near zero at the true period, because real signals are not perfectly periodic. Resonance can create minima lower than the true-period minima, so picking the global minimum is not always correct.

The proposed fix is the cumulative mean normalized difference function (CMNDF). I implemented it in testing, but still want a deeper understanding of why it suppresses those misleading minima.

There are additional optimizations in the paper that I plan to implement. I also realized microphone audio is often stereo, while this signal-processing pipeline expects mono, so I need a preprocessing step before feeding audio to YIN.
