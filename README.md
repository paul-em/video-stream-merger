# video-stream-merger

[![Standard - JavaScript Style Guide](https://img.shields.io/badge/code%20style-standard-brightgreen.svg)](http://standardjs.com/)

Merge the video of multiple MediaStreams. Also merges the audio via the WebAudio API.  

For sending composite videos across a single WebRTC MediaConnection, hot-swapping streams without stopping, or just editing live video streams.  

[Live Demo](https://rationalcoding.github.io/video-stream-merger/)

## install

```
npm install simple-peer
```

## usage

Let's first get two media streams. One from the webcam, and another a screen capture.
```javascript
var getusermedia = require('getusermedia')
var screenrecord = require('screen-record')

getusermedia({video: true, audio:true}, function (err, webcamStream) {
  screenRecord(window, function (err, sourceId, constraints) {
    getusermedia(constraints, function (err, screenStream) {
      // We now have 2 streams: webcamStream, screenStream
    })
  })
})
```

We want to overlay the webcam stream in the corner of the screen stream.  
```javascript
var VideoStreamMerger = require('video-stream-merger')

var merger = new VideoStreamMerger()

// Add the screen capture. Position it to fill the whole stream (the default)
merger.addStream(screenStream, {
  x: 0, // position of the topleft corner
  y: 0,
  width: merger.width,
  height: merger.height,
  mute: true // we don't want sound from the screen (if there is any)
})

// Add the webcam stream. Position it on the bottom left and resize it to 100x100.
merger.addStream(webcamStream, {
  x: 0,
  y: merger.height - 100,
  width: 100,
  height: 100,
  mute: false
})

// Start the merging. Calling this makes the result available to us
merger.start()

// We now have a merged MediaStream!
merger.result
```

## API

### `merger = new VideoStreamMerger([opts])`

Create a new video merger.

Optional `opts` defaults to the below:

```
{
  width: 400,   // Width of the output video
  height: 300,  // Height of the output video
  fps: 25       // Video capture frames per second
}
```

### `merger.addStream(mediaStream, [opts])`

Add a MediaStream to be merged.

The order that streams are added matters. Streams placed earlier will be behind later streams.

Optional `opts` defaults to the below:
```
{
  x: 0, // position of the top-left corner
  y: 0,
  width: '<width of output>',     // size to draw the stream
  height: '<height of output>',
  mute: false,  // if true, any audio tracks will not be merged
  draw: null    // A custom drawing function (see below)
}
```

### `merger.removeStream(mediaStream)`

Remove a MediaStream from the merging.

If you have added the same MediaStream multiple times, all instances will be removed.

### `merger.start()`

Start the merging and create `merger.result`.

You can call this any time, but you only need to call it once.  

You will still be able to add/remove streams and the result stream will automatically update.

### `merger.result`

The resulting merged MediaStream. Only available after calling `merger.start()`

Never has more than one Audio and one Video track.

### `merger.destroy()`

Clean up everything and destroy the result stream.  

## Hot-Swapping Streams

This library makes it easy to change streams in a WebRTC connection without needing to renegotiate.

The result MediaStream will appear to be constant and stable, no matter what streams you add/remove!

## Custom Draw Function

If sizing and poisitioning aren't enough, you can directly draw the video frames by passing a function to the `draw` option.

```javascript
merger.addStream(mediaStream, {
  draw: function (ctx, frame, done) {
    // You can do whatever you want with this canvas context
    ctx.drawImage(frame, 0, 0, merger.width, merger.height)
    done()
  })
})
```

See the bottom example of the [Live Demo](https://rationalcoding.github.io/video-stream-merger/) to see this in action.  
