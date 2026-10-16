# Datamosh Transition

Installed and verified in Motion and Final Cut Pro on 16 October 2026.

## Use in Final Cut Pro

Open **Transitions → 1V2T Glitch → Datamosh Transition** and apply it between two clips with sufficient handles. The template animates Progress internally: clean outgoing A at the start, motion-driven block breakup during the overlap, and clean incoming B at the end.

Start with Amount **0.85**, Block Size **32**, Motion **0.5**, Seed **1**, Mix **1**. Amount or Mix zero gives a clean cross-dissolve. Motion zero retains block replacement without displacement.

- **Amount:** strength of block replacement over the underlying dissolve.
- **Block Size:** 8–128 render pixels.
- **Motion:** exaggeration of coarse motion estimated from incoming B and its preceding frame.
- **Seed:** deterministic block reveal pattern.
- **Mix:** blends the block treatment with the underlying dissolve.

## Installed files and editable source

- Plugin application: `/Users/clementcombier/Applications/1V2T_Glitch.app`.
- Installed template: `/Users/clementcombier/Movies/Motion Templates.localized/Transitions.localized/1V2T Glitch/Datamosh Transition/Datamosh Transition.motr`.
- Editable source package in this repository: `Motion Templates/Transitions.localized/1V2T Glitch/Datamosh Transition/`, including browser thumbnails.
- Test event/project: **Datamosh Transition Test**, in the **1V2T** Final Cut library. It contains generated moving blue/orange test scenes and a two-second transition. Original test media are retained in the library.

Open the source `.motr` in Motion to edit the template. After edits, save/publish the template and update the repository copy. Build/run the Wrapper Application scheme after changing Swift or Metal code; reopen Motion/Final Cut to refresh plugin registration.

## Template wiring

A fixed-resolution, project-sized **Datamosh Output** group contains both standard Final Cut transition placeholders and the Datamosh Transition filter. Both placeholders span the entire two-second template; A and B feed the filter's image wells using their scene-object references. The parent filter composites the group, so neither input needs a separate visibility toggle or recursive reference.

Progress has linear keyframes at the first and last template frames. Only Amount, Block Size, Motion, Seed, and Mix are published. Final Cut maps this animation to the applied transition duration. The Motion template defines the transition role; the underlying FxPlug renderer intentionally remains registered as an FxFilter.

`DatamoshTransitionPlugIn` lives in `DatamoshPlugIn.swift` alongside the original temporal filter. It requests A/B at the current time and an earlier B frame during the active effect, identifying images by parameter ID and media time. Missing historical B falls back to zero motion. Missing scene inputs produce a connection error. `Datamosh.metal` uses `random.y == 1` for transition compositing and `controls.w` for Progress. The original Datamosh filter remains unchanged in behavior.

## Verification and limits

- Signed and unsigned Debug Xcode builds succeeded.
- 288 existing Datamosh GPU/reference cases, 20 transition cases, and 48 timing scenarios passed, plus boundary/error/replay checks.
- Motion loaded and saved the wired template; clean A, mixed/block midpoint, and clean B were visually verified.
- Final Cut discovered the installed template in its Transitions browser, displayed the five published controls, and rendered a two-second transition between generated moving 720p30 clips. First, midpoint, and last transition frames were visually verified.

This is a decoded-frame approximation, not codec bitstream corruption or accumulated optical-flow feedback. Inputs must have matching canvases. Full input frames are requested for reliable sampling; long-form performance, other frame rates/resolutions, retimed clips, and HDR export have not been validated in the host. No export or real-time performance benchmark was performed.
