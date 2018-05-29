# Frame Accuracy in HTML5 Video

HTML5 `<video>` currently has no means of obtaining perfect frame accuracy via JS APIs. To begin with, there are no standard methods for getting information about the current frame in the VideoElement API. Now, this in itself wouldn't necessarily be a problem, as `video.currentTime` exists, and assuming you know the (constant) framerate of your video (or have frame timings), you should be able to calculate the number of the currently displayed frame.

Unfortunate, `video.currentTime` is not accurate for this purpose - a simple calculation will be slightly off most of the time in most browsers, and when it comes to making use of frame accuracy, in many cases 100% accuracy is very much a critical requirement. The current lack of frame accuracy effectively closes off entire fields of possibilities from the web, such as non-linear video editing, but it also has unfortunate effects on things as simple as subtitle rendering.

## Research

The purpose of this repository is to research ways to improve frame accuracy for browsers today. 100% accuracy is unlikely to be possible in any of them at the time of writing, but various per-browser tricks can at least improve the situation somewhat.

### [Check the browser test case here.](https://daiz.github.io/frame-accurate-ish/)

Here's how things look today:

### General observations

- The `timeupdate` event on VideoElement is completely useless for frame accuracy purposes.
- Browsers update `video.currentTime` at differing intervals. Details to follow under the specific browser headers.

### Firefox (Windows)

- Firefox has a non-standard VideoElement property `video.mozPaintedFrames`. This value increments every time a frame is painted on screen, so theoretically it should be possible to get to 100% accuracy... though in practice since you need to connect this to `video.currentTime` to get the actual frame number, perfect accuracy is yet to be reached.
- All in all, Firefox offers the best options for improving frame accuracy today. Having a way to react at the exact moment a frame is painted on screen is critical for frame accuracy purposes.
- `video.currentTime`, at least for 23.976 FPS video, seems to update approximately every 40 milliseconds. At 23.976 FPS, each frame duration is approximately 41.7 milliseconds. The actual time tends to differ from the displayed frame timestamps (it can for example be lower than the frame's defined starting time).

### Chrome (Windows)

- Chrome has a non-standard VideoElement property `video.webkitDecodedFrameCount`. As the name implies, it's not a counter for painted frames so theoretically the prospects for frame accuracy are worse than with Firefox, but in practice it often seems to increment on paint anyway during normal playback, which gives us decent possibilities for improving frame accuracy.
- `video.currentTime` updates on every display refresh. It also seems to have a slight amount of "drift" - at the start of video, when currentTime should be exactly 0, it's usually slightly above zero.

### Microsoft Edge (Windows)

- No non-standard VideoElement properties that could help with frame accuracy.
- The actual video painting seems rather janky and weird in general?
- `video.currentTime` updates on every display refresh. Has "drift" much like Chrome.

### Safari (macOS)

I don't have a Mac so I can't test this one: help accepted.

### Safari (iOS)

- iOS is quite lacking in terms of video capabilities so for production usage you'd most likely need to resort to native playback capabilities that you can't really interact with, making frame accuracy for JS largely a moot point.
