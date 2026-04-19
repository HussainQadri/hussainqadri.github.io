---
title: "Speeding things up"
date: 2026-04-19
project: rabab-tuner
---
### The problem

Tuning feedback was too slow. The old pipeline recorded a 1 second blob with `MediaRecorder`, posted it to `/analyze`, and only started the next recording after the response came back. The backend then had to decode that blob with pydub, which is then sent to ffmpeg to be converted to the raw PCM samples. This was adding too much overhead and noticeable lag.

### Streaming raw PCM over Socket.IO

The real fix was realising that I just need raw PCM samples, and I am able to do that directly with Socket.IO rather than relying on HTTPS.

HTTP forces a record-stop-upload-wait shape, and the next recording can't start until the previous request finishes. A persistent WebSocket handshakes once and then frames just flow in both directions.

`MediaRecorder` now does not make sense. It's designed to hand you a finalised encoded file, which is exactly the wrong thing when the server only wants floats. An `AudioWorkletProcessor` sits on the audio thread and gives direct access to the raw `Float32Array` samples.

```js
class PCMProcessor extends AudioWorkletProcessor {
  constructor() {
    super();
    this.bufferSize = Math.floor(sampleRate * 0.2);
    this.buffer = new Float32Array(this.bufferSize);
    this.writeIndex = 0;
  }

  process(inputs) {
    const input = inputs[0];
    if (input.length === 0) return true;
    const channelData = input[0];
    for (let i = 0; i < channelData.length; i++) {
      this.buffer[this.writeIndex++] = channelData[i];
      if (this.writeIndex >= this.bufferSize) {
        this.port.postMessage(this.buffer.slice());
        this.writeIndex = 0;
      }
    }
    return true;
  }
}
```

I'm buffering about 200ms before posting — small enough to feel smooth, big enough for YIN to have reliable samples to process.

On the server, the whole pydub/ffmpeg step disappears. Decoding looks like this now. 

```python
samples = np.frombuffer(audio_bytes, dtype=np.float32)
```
A single line of code, incredibly simple.

### Two things the stream exposed

If YIN ever runs longer than a chunk, frames start piling up. A flag per connection — drop new frames while one is still being processed — handles it. This **does** have consequences, in the worst case if YIN takes more than 200ms for every chunk, the server would drop every one. Our update rate would drop sharply and the tuner would feel choppy.

And because the connection is now bidirectional, results can come back in a different order than you sent the chunks (or arrive after you've already moved on to a different string). Tagging each chunk with an incrementing sequence number and ignoring any result that doesn't match the latest one fixes that.

Neither of these were problems before, because HTTP was already enforcing strict one-at-a-time by accident. Streaming makes you do it deliberately but stays true to the 'live tuning' behaviour that is expected.

### The big issue

The tuner is much more responsive. The problem is now the backend hosted on onrender, which for some reason terminates the worker thread. Because of this, any tuning currently happening halts and the tuner displays 'connecting'. I have no idea why this is happening - maybe the free tier gives too little RAM for the process and terminates it when the process asks for more? I am not sure.

