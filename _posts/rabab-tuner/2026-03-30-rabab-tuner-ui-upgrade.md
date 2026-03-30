---
title: "Upgrading the UI"
date: 2026-03-30
project: rabab-tuner
---
Date: 2026-03-30

## The problem

I am building a tuner, yet the UI is bloated with colours and fancy live waveform. It looks modern, but it doesn't capture the essence of what I am trying to create. A simple tuner, that can tune a rabab to the bilawal scale.

## Making the UI more approachable

The landing page shouldn't take you to a screen that says "Let's get your rabab tuned", this doesn't make sense at all. The user is already here to get their rabab tuned, adding a button that takes them to the tuner afterwards is just redundant.

Adding a tuning 'gauge' with a needle, and an image with the rabab makes more sense. After playing around with the css, this is what I have reached:

<img src="/images/ui-upgrade-test.png" alt="Rabab tuner UI sketch" style="max-width: 400px; width: 100%; height: auto;">

Now, the notes that are labelled on the rabab on the circles are not accurate. They shouldn't be labelled at all as they are the sympathetic strings. I got AI to generate the wireframe rabab image with 8 sympathetic string knobs so I could label them with 8 notes that are being tuned. This needs to be changed, there are 3 or 4 main strings that need tuning which are the larger knobs at the top of the rabab.

Apart from the UI, the tuning response from the website is quite slow. I still need to fully implement YIN into the project rather than relying on the error-prone FFT.
