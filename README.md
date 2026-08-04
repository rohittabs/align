<div align="center">

<img src="Logo.png" alt="Align" width="200">

# Align

</div>

---

<div align="center">

**A free guitar and ukulele tuner that runs in your browser.** No app to install, no account, no ads. Just open it and play.

### [Open Align](https://rohittabs.github.io/align/)

![license MIT](https://img.shields.io/badge/license-MIT-555)
![price free forever](https://img.shields.io/badge/price-free%20forever-2EE6A8?labelColor=555)
![whole app 1 file](https://img.shields.io/badge/whole%20app-1%20file-EC008C?labelColor=555)
![account not needed](https://img.shields.io/badge/account-not%20needed-7C3AED?labelColor=555)
![tracking none](https://img.shields.io/badge/tracking-none-555)
![accuracy 0.04 cents](https://img.shields.io/badge/accuracy-0.04%20cents-3B82F6?labelColor=555)

### [Take the one minute tour](#the-one-minute-tour)

</div>

---

## What it is

Align is a chromatic tuner and a preset tuner in one page. Point your phone at your instrument, pluck a string, and the needle tells you which way to turn the peg. When the string lands, it turns green and plays a short chime so you can keep your eyes on your hands.

It is a single HTML file. There is no backend, no build step, and nothing is ever uploaded. The audio from your microphone is analysed on your own device and thrown away frame by frame.

## The one minute tour

**Tap Start tuning.** Your browser will ask for microphone permission. Align needs it to hear the string, and that is the only permission it ever asks for.

**Pick a mode.** *Chromatic* names whatever note it hears, which is what you want for odd tunings, other instruments, or just checking a pitch. *Preset* locks to a tuning and tracks the six strings individually.

**Play a string.** The big note tells you what Align heard. The needle and the cents readout tell you how far off you are. Flat means tune up, sharp means tune down.

**Watch it lock.** Inside three cents the needle turns green, the gauge fills green, and you get a chime. Strings you have already finished stay marked so you can see your progress along the row.

**Tap a string to pin it.** In preset mode, tapping a string in the row tells Align to track only that string. This is worth doing when other strings are still ringing, because it removes any ambiguity about which note you meant.

## Features

**Two modes.** Chromatic for anything, preset for the tuning you are actually in.

**Twelve built-in tunings.**

| Guitar | Ukulele |
| --- | --- |
| Standard E | Standard C (High G) |
| Half Step Down | Low G |
| Full Step Down | Baritone |
| Drop D | D Tuning |
| Drop C | |
| Open G | |
| Open D | |
| DADGAD | |

**Your own tunings.** Build any tuning from 1 to 12 strings, name it, and it is saved on your device. Edit or delete it later. Half Step Down displays as E♭ A♭ D♭ G♭ B♭ E♭ rather than D♯ G♯ and so on, the way guitarists actually write it.

**Adjustable reference pitch.** A4 moves from 415 to 466 Hz in one hertz steps, for baroque pitch, playing along with a recording that is slightly off, or matching an old piano.

**In-tune chime.** A short two note blip when a string locks, with a toggle in the header that remembers your choice. Detection mutes itself while the chime sounds so your own speaker cannot be mistaken for a string.

**Built for a phone.** Large touch targets, a gauge sized to the screen, safe area insets for notches and home bars, and no pinch zoom needed.

**Remembers what matters.** Your custom tunings, your selected preset, your reference pitch, and your sound preference all persist. Nothing else is stored, and none of it leaves your device.

## How it hears the string

Most browser tuners run a plain autocorrelation over the raw buffer. That is accurate on a clean sine wave and unusable on a real guitar in a real room. Align does several things differently, and the numbers below are measured, not estimated.

**A decimated, lag-limited, normalised autocorrelation.** The buffer is downsampled four times and the lag search is restricted to the musical range, then the winning lag is refined at the full sample rate with parabolic interpolation. A naive full-length autocorrelation over 4096 samples costs about 47 ms per frame, which is three times the entire frame budget at 60 fps. Align's costs **0.33 ms**, roughly 140 times cheaper, which is what keeps the interface smooth on a mid-range phone.

**First peak, not tallest peak.** Picking the strongest correlation peak is what makes cheap tuners jump an octave down on a plucked string. Align takes the first peak that clears 90 percent of the maximum instead. Across 150 random pitches from 62 to 500 Hz: **zero octave errors**.

**A confidence gate.** The peak correlation value is a good predictor of whether a reading is trustworthy. Frames below 0.80 are discarded rather than displayed.

**A four stage stabiliser.** Octave snapping against the string you are tuning, rejection of single wild frames, a five frame median guard, and finally a One Euro adaptive filter that smooths hard while the pitch is steady and opens up the instant you turn a peg.

The effect on how the needle actually behaves:

| Condition | Needle movement per frame, before | after |
| --- | --- | --- |
| Clean, loud | 0.02 cents | 0.05 cents |
| Quiet room | 2.1 cents | 0.27 cents |
| Noisy room | 10.1 cents | 0.23 cents |
| Loud room or traffic | 8.0 cents | 0.34 cents |
| Playing quietly, phone across the room | 12.3 cents | 0.66 cents |

Steady-state accuracy is **0.04 cents** across seventeen notes from C2 to E5. A cent is a hundredth of a semitone, and the human ear can just about pick out five cents on a sustained note, so the display is far finer than anyone needs. It also reads correctly when the fundamental is missing entirely, which is common when a phone microphone is asked to hear a low E.

**It does not fake it.** When Align cannot get a trustworthy reading it shows nothing rather than guessing. Room noise, a dying decay tail, or two strings ringing at an awkward interval will all produce a blank display instead of a confident lie.

## Known limitation

If two strings ring loudly together at certain intervals, the combined waveform genuinely repeats at a lower pitch than either string. B3 and E4 sounded together, for example, are the third and fourth harmonics of a low E, so the waveform really does repeat at 82 Hz. No time-domain tuner can tell which of the two strings you meant, and Align will either show nothing or name the lower note.

The fix takes a second: damp the strings you are not tuning, or tap the string you want in the strings row to pin it.

## Browser support

Works in any current browser with Web Audio and `getUserMedia`, which covers Chrome, Safari, Firefox and Edge on desktop, Android and iOS.

Two requirements worth knowing. The page must be served over **HTTPS**, or from `localhost`, because browsers refuse microphone access otherwise. And on **iOS Safari** the audio engine only unlocks inside a real tap, which is why the Start tuning button exists rather than the tuner simply beginning on load.

## Running it yourself

Download `index.html`. That is the whole application, images included as embedded data. Open it and it works.

To host it, drop the file anywhere that serves over HTTPS. GitHub Pages, Netlify, Cloudflare Pages and Vercel will all serve it for nothing:

```bash
git clone https://github.com/rohittabs/align.git
cd align
python3 -m http.server 8000
# then open http://localhost:8000
```

The typefaces load from Google Fonts. If that request fails, the app falls back to your system fonts and everything still works.

## Privacy

No analytics. No cookies. No third party scripts. No network requests other than the fonts.

Audio never leaves your device. It is read from the microphone, analysed, and discarded frame by frame. Nothing is recorded and nothing is uploaded. Your saved tunings and settings live in your browser's local storage and are yours to clear at any time.

## Support

Align is free and always will be. If it has helped your practice and you feel like it, there is a UPI QR code and a payment link under the Support tab. It is completely optional and nothing in the app is gated behind it.

## Contributing

Bug reports and feature requests are welcome in the issues tab. If you are opening a report about a tuning problem, it helps a great deal to include your instrument, your browser and device, the tuning you were in, and whether other strings were ringing at the time.

## License

MIT. See [LICENSE](LICENSE). Use it, change it, ship it, sell it. Attribution appreciated but not required.

---

<div align="center">

Built by [rohittabs](https://github.com/rohittabs)

</div>
