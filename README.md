# fog

A bathroom mirror that has steamed up, in your browser.

Your webcam is the mirror. Breathe at your laptop and fog blooms where your breath lands. Pinch your thumb and index finger together and draw on the fog; wherever you draw, the fog wipes away and your face shows through. Let go of the pinch to stop drawing. Make a fist and swipe to clear a wide area, like using your sleeve.

Everything runs locally. No video, audio or data ever leaves your machine.

## Credit

This project was inspired by [Lilian Chiu's fog mirror tutorial](https://lilian.world/share/fogmirror-tutorial/). I built it after following her tutorial; go and see the original.

## Run it

Browsers only allow the camera on `https` or `localhost`, so double-clicking the file will not work. Serve the folder instead:

```
python -m http.server
```

Then open <http://localhost:8000/> and press the button. Allow the camera and the microphone when asked.

There is no build step and nothing to install. The whole app is one file, `index.html`.

You need a recent Chrome, Edge or Firefox, a webcam, and an internet connection on first load (the hand-tracking model is downloaded from jsDelivr; nothing is sent back). A microphone is optional: without one, hold Space to fog.

## Controls

| Action | What it does |
| --- | --- |
| Breathe at the laptop | Fog blooms around your mouth, and drifts the way your head is turned |
| Pinch thumb and index finger together | Draw on the fog (either hand, or both at once) |
| Release the pinch | Lift the pen, so you can move without drawing |
| Fist, then swipe | Wipe a wide area clear, leaving sleeve streaks |
| Hold `Space` | Fog up without breathing |
| `C` | Re-fog the mirror. Finger marks stay on the glass |
| `Shift`+`C` | Clean the glass completely |
| `S` or the shutter button | Save the current frame as a PNG |
| `R` or the replay button | Replay everything drawn so far as a time-lapse |
| `V` or the video button | Replay and save it as a video (mp4 or webm) |
| `M` or the speaker button | Sound on or off |
| `1` to `4` or the mood buttons | mirror, window, shower, neon |
| `W` or the snow / rain buttons | Weather outside the window (window mood only) |
| `Esc` | Stop a replay, or go back to the start screen with the camera off |
| `brush` slider | Brush radius, 7 to 38 px |

## What the glass does

- **Fog creeps back.** Once a second every pixel moves a little towards the mood's resting fog level, so drawn lines mist over in under a minute and heavy breath thins out. It pauses while a hand is drawing or wiping.
- **Hidden messages.** A finger leaves an invisible mark on the glass. Breath condenses less on those marks, so writing that has fogged over reappears when you breathe on it. A fist wipe cleans the marks off; `C` does not.
- **Droplets.** In the shower mood only, beads form on their own and where you wipe or breathe heavily. They stick, let go, and run down the glass, leaving thin clear trails.
- **Sleeve streaks.** A fist wipe leaves faint parallel smears along the direction of travel.
- **Sound.** A low rub while wiping, generated with the Web Audio API. Drawing is silent. The microphone ignores the app's own sound so it does not read as breath.
- **Moods.** `mirror` is the default. `window` is colder and bluer, with a snowy day (falling snow, frost ferns at the edges) or a rainy day (falling rain, beads on the outside of the pane) behind the glass. `shower` is heavier and warmer, re-fogs fast and drips on its own. `neon` darkens the room and tints the fog pink to cyan.
- **Replay.** Every change to the fog is recorded as an operation. Replay re-applies the list from the start with idle time removed, in 3 to 15 seconds. Resizing the window restarts the recording from the glass as it is at that moment.

## How it works

Three layers are drawn every frame:

1. **Camera**, mirrored horizontally so it behaves like a real mirror.
2. **Fog**, a hidden offscreen canvas that stores only how much fog is at each pixel. It is never drawn to the screen.
3. **Frost**, the camera frame again, blurred, brightened and tinted, then masked by the fog canvas with `destination-in`. This is what you see as fog.

Drawing and wiping never paint anything. They subtract from the fog canvas with `destination-out`, so the sharp camera layer underneath shows through.

Every change to the fog goes through one function, `rec()`, which logs the operation and applies it. That log is what replay and video export play back.

Hand and face tracking use MediaPipe, pinned to exact versions:

- `@mediapipe/hands@0.4.1675469240`
- `@mediapipe/face_detection@0.4.1646425229`

Frames are fed to the trackers by the app's own animation loop. MediaPipe's `camera_utils` helper is deliberately not used, because it reopens the camera at its own resolution and the picture goes soft.

## Things that are easy to break

- **Microphone processing must be off.** Echo cancellation, noise suppression and auto gain control are all disabled, because noise suppression is designed to remove the sound of breathing.
- **The breath threshold floats on the room's noise floor.** The floor drops quickly towards quiet and rises very slowly towards noise. A fixed threshold works in one room and fails in the next.
- **Typing is ignored.** Sound in the 300 ms after any key press does not count as breath.
- **Pinching has two thresholds so it does not chatter.** The pen goes down when the thumb and index tips are within a quarter of a hand width of each other. It lifts when the gap opens past 0.42 hand widths, and holds its state in between. The pen sits midway between the two tips, so it does not jump when you let go.
- **Every pixel size is multiplied by `devicePixelRatio`**, capped at 2.
- **Errors are not caught.** They go to the console and to a red banner on screen.

All the tuned values (fog opacity, blur, blob count, smoothing, thresholds) are named constants at the top of the script in `index.html`.

## Troubleshooting

| Symptom | Cause |
| --- | --- |
| The start screen says the camera only works on https or localhost | The page was opened as a file. Serve it as shown above. |
| "Camera access was refused" | Allow the camera from the icon in the address bar, then press the button again. |
| Breathing does nothing | The microphone was refused or is missing (the page says so at the top). Hold `Space`, or allow the microphone and reload. |
| Hands are not picked up | Wait for "loading hand tracking…" to disappear, and keep your hand in good light, fully in frame. |
| The mirror fogs by itself | The room is loud or the level is changing a lot. Give the noise floor a few seconds to settle. If it happens while wiping with the sound on, mute with `M` or use headphones. |
| It runs slowly | Tracking two hands is heavier than one. Make the browser window smaller. |
