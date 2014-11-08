# cljs-audio


ClojureScript core.async interface for capturing audio in modern browsers


## Installation

cljs-audio is available in Maven Central. Add it to your `:dependencies `
in your Leiningen `project.clj`:


```clojure
[cljs-audiocapture "0.1.0"]
```

## Compatibility

Tested against lastest stable versions of Chromium-based browsers,
Firefox. With Flash fallback cljs-audiocapture works in IE11 (and maybe below)
and Safari.


## Usage

cljs-audiocapture namespace has one public funtion `capture-audio`. It returns
channel that blocks and waits while user allow or deny access to their microphone.
The only value in this channel is map with keys `:audio-chan` and `:error`.

If user denies access or other error occurs `:error` value will be filled with
browser-specific [error][error]. Otherwise you get bidirectional channel in `:audio-chan`.

To start or resume audio capture you should put `:start` to this channel. After
you should read frames from this channel. Each frame is ArrayBuffer of raw PCM data.
You can record it and wrap to RIFF WAV or process other way. To pause just put
`:pause` to channel.

To use flash fallback you should put `audiocapture.swf` to place your script
has access to.


```clojure
(ns cljs-audiocapture-demo
  (:require
    [cljs.core.async :as async]
    [cljs-audiocapture :refer [audio]])
  (:require-macros
    [cljs.core.async.macros :refer [go]]))

(go
  (let [{:keys [audio-chan error]} (async/<! (capture-audio))]
    (if error
      (js/console.error error)
      (do
        (async/put! audio-chan :start)
        ; Print first 5 frames to console
        (loop [counter 4]
          (.log js/console (async/<! audio-chan))
          (if (zero? counter)
            (async/put! audio-chan :pause)
            (recur (dec counter))))))))
```


## Examples

```clojure
(go
  ; This will fail after queue overflow
  (let [{:keys [audio-chan error]} (async/<! (capture-audio))]
    (if error
      (js/console.error error)
      (async/put! audio-chan :start))))

(go
  ; This will work while browser runs
  (let [{:keys [audio-chan error]} (async/<! (capture-audio (async/sliding-buffer 10)))]
    (if error
      (js/console.error error)
      (async/put! audio-chan :start)))))

(go
  ; This will stop after printing 6 frames and pause capture
  (let [{:keys [audio-chan error]} (async/<! (capture-audio))]
    (if error
      (js/console.error error)
      (do
        (async/put! audio-chan :start)
        (loop [counter 5]
          (.log js/console (async/<! audio-chan))
          (if (zero? counter)
            (async/put! audio-chan :pause)
            (recur (dec counter))))))))
```

[error]: http://webrtchacks.com/getusermedia/