# Implementation notes

These notes describe a static review of the source supplied for this repository. They are not results of Unity Play Mode or build testing. The runtime code has not been modified as part of repository preparation.

## Architecture

`MecanimControl.Awake` loads four `RuntimeAnimatorController` templates from `Resources`. Playback creates an `AnimatorOverrideController`, assigns the previous and target clips to `State1` and `State2`, then requests playback or a transition. `FixedUpdate` maintains a separate elapsed-time counter and implements wrap behavior and notifications.

Keep the controller names, placeholder animations, and their GUID references intact when moving the package. The generic Resources names can collide with assets in another package.

## Important caveats

- **Initialization:** assign a valid default animation and avoid empty clip entries. `FixedUpdate` dereferences `currentAnimationData` before checking its clip; no configured default or usable animation can cause a null-reference exception.
- **Startup ordering:** calling playback from another component's `Start` is order-dependent. Configure clips in the Inspector and wait until initialization completes before playing them.
- **Blending:** the implementation uses `-1` to fall back to configured transition values and `0` for immediate playback. Some historical manual prose describes the inverse. `Play(string)` explicitly requests zero blending; `Play(AnimationData)` uses that data's transition setting. The supplied value ultimately goes to `Animator.CrossFade` after division by speed; do not assume it is a strict wall-clock-seconds contract without testing.
- **Stop versus pause:** `Stop()` plays the default animation. `Pause()` sets Animator speed to zero; parameterless `Play()` resumes it.
- **Reverse playback:** `SetSpeed(float)` applies `Mathf.Abs`. `Rewind()` calls that overload, so it does not by itself establish reverse playback. Other playback paths select reverse-oriented controller templates. Treat reverse and PingPong behavior as requiring validation, not as verified features.
- **Static events:** `OnAnimationBegin`, `OnAnimationEnd`, and `OnAnimationLoop` are shared across instances and carry `AnimationData`, not the emitting component. Filter events where needed and unsubscribe when listeners are disabled or destroyed.
- **Timing:** elapsed time is tracked in `FixedUpdate`, separately from Animator evaluation. Loop notifications occur before the playback count is incremented. Validate synchronization for the intended use case; this is not a deterministic combat clock.
- **Clip registration:** `AddClip` clones the supplied clip. Looking up the original clip reference afterward may not return the registered entry; prefer the registered name or returned data lookup by name. Duplicate names produce a warning but are still added.
- **Default clip signature:** the supplied source requires `SetDefaultClip(clip, name, speed, wrapMode, mirror)`. Its `mirror` argument is currently unused. Some old manual examples omit it.
- **Root motion:** per-clip `applyRootMotion` is applied when the component's `overrideRootMotion` option is enabled.
- **Legacy materials:** historical demo materials may need conversion for the project's render pipeline. A configured Unity version is not evidence of shader or animation compatibility.

## Suggested validation before a release

Record the exact editor version, render pipeline, rig type, and platform used.

- [ ] Import with no compile errors; verify all four controller assets load.
- [ ] Default animation starts without null-reference errors.
- [ ] Named playback, clip registration, and clip removal behave as expected.
- [ ] Blends work at normal and changed speeds without a first-frame pop.
- [ ] Pause, resume, seeking, and return-to-default work.
- [ ] Mirroring and per-clip root motion work on a compatible rig.
- [ ] Once, Loop, and PingPong behavior is checked individually.
- [ ] Reverse behavior is either corrected and tested or explicitly unsupported.
- [ ] Multiple characters do not mishandle shared event notifications.
- [ ] A standalone build is tested, not only Editor Play Mode.

No checklist item is marked complete by the initial documentation pass.
