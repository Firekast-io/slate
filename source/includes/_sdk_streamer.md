# SDK - Streamer

<blockquote class="lang-specific javascript shell">
<p class="lang-specific shell">Streams can be created using mobile SDKs.</p>
<p class="lang-specific javascript">The javascript SDK currently only supports <a href="#watch-live-or-replay-as-vod">live and vod</a> content playback, not publishing.</p>
</blockquote>

The streamer is responsible for creating streams and actually sends frames and audio for live streaming.

## Create Streams

```swift
streamer.createStream { (stream, error) in
  //...
}
```

```objective_c
[_streamer createStreamWithOutputs:nil completion:^(FKStream * stream, NSError * error) {
  //...     
}];
```

```java
mStreamer.createStream(new FKStreamer.CreateStreamCallback() { ... });
```

First step to do live broadcasting is to create a stream.

This call provisions server resources to handle live streaming, create and returns a [stream](#sdk-stream) with state `waiting`. Waiting for frames and audio to be sent.

This newly created stream is immediatly visible in your dashboard.

## Go Live

<blockquote class="lang-specific swift objective_c java">
<p>Start streaming:</p>
</blockquote>

```swift
streamer.startStreaming(on: stream, delegate: self)
```

```objective_c
[_streamer startStreamingOn:stream delegate:self];
```

```java
mStreamer.startStreaming(stream, new FKStreamer.StreamingCallback() { ... });
```
### Start streaming

Once you have created a stream, you can start streaming whenever your User is ready.
<aside class="notice">You must start streaming before <a href="#timeout">timeout</a>.</aside>

<blockquote class=
"lang-specific swift objective_c java shell">
<p>Stop streaming:</p>
</blockquote>

```shell
curl -X POST \
    https://api.firekast.io/v2/streams/%STREAM-ID%/stop \
    -H 'Authorization: SDK %YOUR-APP-PRIVATE-KEY%' 
```

```swift
streamer.stopStreaming()
```

```objective_c
[_streamer stopStreaming];
```

```java
mStreamer.stopStreaming()
```
### Stop streaming

Then, stop streaming whenever your User is done.

<aside class="notice">You should call <code>stopStreaming()</code> when User leaves the dedicated streaming screen.</aside>

## Streamer Events

```swift
func streamer(_ streamer: FKStreamer, willStart stream: FKStream, unless error: NSError?) {}
func streamer(_ streamer: FKStreamer, didBecomeLive stream: FKStream) {}
func streamer(_ streamer: FKStreamer, didStop stream: FKStream, error: NSError?) {}
func streamer(_ streamer: FKStreamer, didUpdateStreamHealth health: Float) {}
```

```objective_c
- (void)streamer:(FKStreamer *)streamer willStart:(FKStream *)stream unless:(NSError *)error {}
- (void)streamer:(FKStreamer *)streamer didBecomeLive:(FKStream *)stream {}
- (void)streamer:(FKStreamer *)streamer didStop:(FKStream *)stream error:(NSError *)error {}
- (void)streamer:(FKStreamer *)streamer didUpdateStreamHealth:(float)health {}
```

```java
private class AppStreamingCallback implements FKStreamer.StreamingCallback {
  @Override
  public void onSteamWillStartUnless(@NonNullable FKStream stream, @Nullable FKError error) {}
  
  @Override
  public void onStreamDidBecomeLive(@NonNullable FKStream stream) {}
  
  @Override
  public void onStreamDidStop(@NonNullable FKStream stream, FKError error) {}
  
  @Override
  public void onStreamingUpdateAvailable(boolean lag) {}
}
```

Your app can rely on the streamer events to adapt its UI. Events notify whether the live streaming has started properly or failed, stopped normally or prematurely, and how the live stream is performing.

<aside class="notice">Once <code>startStreaming</code> is called, frames and audio are being sent to our server. However we guarantee that frames and audio are recorded (VOD) and are live as soon as stream's state is <code>live</code>. The SDK provides <code>didBecomeLive</code> callback for that.</aside>

<aside class="notice">
Since a stream can be stopped in many ways (SDK, dashboard, server), it is important to rely on <code>didStop</code> callback to update your UI.
</aside>

## Restream

```swift
streamer.createStream(outputs: listOfRtmpLink) { (stream, error) in 
  // ...
}
```

```objective_c
[_streamer createStreamWithOutputs:listOfRtmpLink completion:^(FKStream * stream, NSError * error) {
  // 
}];
```

```java
mStreamer.createStream(mListOfRtmpLink, new FKStreamer.CreateStreamCallback() { ... });
```

Firekast allows to push your live stream simultaneously to other live streaming platforms and social medias, such as Facebook, Youtube, Twitch, Periscope etc...

Refer to the targeted platform API docs to find out how to generate a live stream and get its RTMP link.

<aside class="notice">
Note that the stream remains pushed to Firekast so it's still accessible on your mobile or web app.
</aside>

<aside class="warning">
For the moment, Firekast allows <strong>3 restreams max</strong> per stream. Please <a href="https://firekast.zendesk.com/hc/en-gb/requests/new">contact us</a> if you need more.
</aside>

## Test Bandwidth

<blockquote class="lang-specific swift objective_c java">
<p>Testing bandwidth for 15 seconds</p>
</blockquote>

```swift
let testDuration: TimeInterval = 15
streamer.startStreaming(on: FKStream.bandwidthTest, delegate: self)
DispatchQueue.main.asyncAfter(deadline: .now() + testDuration) { [weak self] in
  guard let this = self else { return }
  this.streamer.stopStreaming()
}
```

```objective_c
double testDuration = 15.0;
[_streamer startStreamingOn:[FKStream bandwidthTest] delegate:self];
dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(testDuration * NSEC_PER_SEC));
dispatch_after(popTime, dispatch_get_main_queue(), ^(void){
  [self->_streamer stopStreaming];
});
```

```java
long durationMs = 15000;
mStreamer.startStreaming(FKStream.bandwidthTest, new FKStreamer.StreamingCallback() { ... });
mHandler.sendEmptyMessageDelayed(MSG_TEST_BANDWIDTH_STOP_STREAMING, durationMs)
```

<blockquote class="lang-specific objective_c swift java">
<p>Observe stream health</p>
</blockquote>

```swift
func streamer(_ streamer: FKStreamer, didUpdateStreamHealth health: Float) {
  // stream health between 70-100% is good.
}
```

```objective_c
- (void)streamer:(FKStreamer *)streamer didUpdateStreamHealth:(float)health {
  // stream health between 70-100% is good.
}
```

```java
@Override
public void onStreamHealthDidUpdate(boolean congestion, float health) {
    // congestion means frames and audio is waiting for network to be sent. Note that, while congested, the camera preview stucks on the frame and will resume as soon as data is sent.
    // stream health between 70-100% is good.
}
```

Run a test bandwidth to know whether the network is good enough to provide a healthy live stream.

Running a test bandwidth consists of streaming on a dummy stream `FKStream.testBandwidth`, that our server recognizes and whose frames and audio are not recorded. Then, while streaming, observe `onStreamDidUpdate`'s streamer callback to estimate the global streaming performance of the test.

The stream health indicates how the live stream is performing. A stream health around 70-100% indicates that the live stream is good.

If the stream health falls below this range for a consistent period, it suggests the network may not be capable of providing a healthy live stream.

<aside class="notice">
We recommand test duration to be between 5 and 30 seconds. The longer the more accurate.
</aside>

## Quality

```swift
streamer.quality = .standard
```

```objective_c
[_streamer setQuality:FKQualityStandard];
```

```java
mStreamer.setQuality(FKQuality.STANDARD);
```

Mobile SDKs stream at the highest possible resolution allowed by your plan.

In case your internet connection does not provide sufficient bandwidth, SDK will degrade encoding quality to keep up with available bandwidth at the target resolution. 

However, we still encourage to run a [test bandwidth](#test-bandwidth) before creating a stream. Depending on the outcome of that test, you may decide to start streaming at a lower resolution in order to provide your Users a smooth streaming experience and consistent image quality.

<aside class="notice">Once streamer starts encoding, it is not possible to change the resolution on the fly.</aside>

<!-- <aside class="warning">You are responsible of targetting a resolution inferior or equal that is handled by your plan, otherwise <code>createStream</code> will fail.</aside> -->
