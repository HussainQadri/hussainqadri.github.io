---
title: Building a lightweight business chart editor
summary: I built Plotlyst to make presentation-ready business charts quickly, while testing agentic coding for the first time.
image: /images/plotlyst.png
topic: tools
date: 2026-06-20
pinned: true
---

My father makes a lot of PowerPoint presentations for work and often needs business charts. Excel can make the underlying chart, but getting it presentation-ready means adjusting labels, colours, and spacing before repeatedly copying it back into PowerPoint. Think-cell is designed for this, but it can feel clunky when all you need is a small, lightweight chart tool.

That was the reason I built [Plotlyst](https://plotlyst-eight.vercel.app/).

## A vibe-coded experiment

Plotlyst was also my first proper test of agentic coding. The entire project was vibe coded.

I wanted to try building a complete project this way rather than using AI for autocomplete or isolated functions. The agent worked across the React interface, SVG renderer, chart maths, tests, persistence, API routes, and export flow. My role became specifying behaviour, catching problems, and deciding what should be built next.

This was also my first time I hit my rate limit on Codex 5.5 high :) . 

## What Plotlyst does

I kept the scope narrow. Plotlyst supports three charts commonly used in presentations:

- Pie charts for simple composition;
- Marimekko charts for market size and composition; and
- Waterfall charts for showing how a starting value becomes a final value.

The chart is rendered as SVG, keeping it sharp when resized in PowerPoint. The same SVG powers the editor and export, with selection controls and editing handles removed from the downloaded file. More charts will be added soon.

There is also a spreadsheet-like datasheet for pasting data directly from Excel. Projects save in the browser by default, so there is no account or setup step before making a chart.

## What I learned

Agentic coding made implementation much faster, especially when a feature had clear acceptance criteria and tests. It also made reviewing behaviour and keeping the architecture coherent more important. An agent can generate a lot of code quickly; knowing what the product should do is still the part I had to provide.

Plotlyst does not need to replace Excel, PowerPoint, or think-cell. It only needs to make one common workflow shorter:

```text
choose a chart -> paste data -> make adjustments -> export
```

## Things I will add

At the moment, I am experimenting with a simple payment model. Draft exports will remain free with a watermark, while removing the watermark will cost $1. I do not want to turn a small utility into another subscription or force users to create an account just to export one chart. 

Adding more charts is also in motion.

The next step is getting my father to use Plotlyst for real presentations and seeing where it still slows him down. That should make it clearer which charts and controls are actually worth adding instead of expanding the tool for the sake of it. PDF export would be useful, and editable PowerPoint charts would be an interesting but much harder problem for later.

The aim is still the same as when I started: open the website, paste some data, make a few adjustments, and have a chart ready for a slide within a couple of minutes.
