# Mecanim Control

**Control Mecanim animations through code, with a familiar, legacy-style API.**

Created by [Daniel Lema / Mind Studios](https://mindstudios.dev/) during development of [Universal Fighting Engine](https://www.ufe3d.com/), Mecanim Control wraps Unity's Animator with named clip playback, runtime clip registration, blending, mirroring, and playback inspection.

This repository preserves the original tool and documents its implementation. It is not a new animation engine or a claim of production validation on every modern Unity version.

[Source code](Assets/MecanimControl/Scripts/MecanimControl.cs) · [Asset Store / complete demo](https://assetstore.unity.com/packages/tools/animation/mecanim-control-15156) · [Online documentation](https://www.ufe3d.com/ufe2/docs/#/more:mecanim-control) · [Portfolio](https://mindstudios.dev/)

## Why it exists

Fighting-game characters need many animations, with playback decisions driven by gameplay code. Mecanim Control provides a small, code-facing abstraction so developers can select and manage clips without authoring a separate state for every move.

Internally, it selects between four bundled controller templates and uses `AnimatorOverrideController` to substitute clips into their states. The templates support normal, mirrored, and reverse-oriented playback configurations. The public interface exposes `AnimationData` rather than requiring callers to manage those controller swaps.

## Capabilities

- Register and remove named clips at runtime with `AddClip` and `RemoveClip`.
- Play clips by name, clip reference, or `AnimationData`; request blended transitions with `CrossFade`.
- Pause, resume, seek to a normalized position, and control playback speed.
- Request mirrored playback and configure per-clip root-motion behavior.
- Inspect current clip name, position, elapsed time, length, and playback count.
- Subscribe to begin, end, and loop notifications.
- Inspect animation data using the optional runtime debug display.

These describe the included API, not a modern compatibility certification. See [implementation caveats](docs/IMPLEMENTATION-NOTES.md) before integrating.

## Getting started

The project records **Unity 6000.5.2f1** in `ProjectSettings/ProjectVersion.txt`. That records the local project version; it does not establish that the historical tool has passed a Unity 6 runtime test.

For integration into an existing project:

1. Copy `Assets/MecanimControl/Scripts`, `Resources`, and `MecanimAssets`, preserving their `.meta` files.
2. Keep the four `controller1` through `controller4` assets under a `Resources` folder. The script loads these exact names.
3. Add `MecanimControl` to a character with an `Animator` and a suitable Avatar for its animation rig.
4. Assign a valid **Default Animation** clip in the Inspector. Add named clips to **Animations**, with no empty clip entries.
5. Begin playback after component initialization. Do not call playback methods on an uninitialized component.

The runtime script has no UFE dependency. Importing it into a project that already includes UFE's version may create duplicate `MecanimControl` or `AnimationData` classes; use one implementation.

### Example: trigger a registered attack

Configure the default clip and an animation named `Attack` in the Inspector, then attach this script to the same character. The coroutine waits a frame so Mecanim Control's `Start` initialization has completed.

```csharp
using System.Collections;
using UnityEngine;

[RequireComponent(typeof(MecanimControl))]
public class AttackExample : MonoBehaviour
{
    private IEnumerator Start()
    {
        yield return null;

        MecanimControl control = GetComponent<MecanimControl>();
        if (control.GetAnimationData("Attack") == null)
        {
            Debug.LogWarning("Register an Attack clip in the Inspector first.");
            yield break;
        }

        control.Play("Attack", 0.15f, 0f, false);
    }
}
```

### Useful calls

```csharp
// Call after initialization, with valid registered clips.
control.AddClip(walkClip, "Walk", 1f, WrapMode.Loop);
control.CrossFade("Walk", 0.15f);
control.Pause();
control.Play();                       // Resume the current clip.
control.SetCurrentClipPosition(0.5f, true); // Seek and pause.
control.SetMirror(true);
control.SetSpeed(0.5f);
control.RestoreSpeed();
control.Stop();                       // Return to the default animation.
```

`walkClip` above is an assigned `AnimationClip`; `control` is the initialized component. These are API examples, not an additional standalone script.

## Documentation and history

- [Implementation notes and testing checklist](docs/IMPLEMENTATION-NOTES.md)
- [Original online manual](https://www.ufe3d.com/ufe2/docs/#/more:mecanim-control)
- [Unity community announcement and historical discussion](https://discussions.unity.com/t/mecanim-control/526650)
- [Contribution guidance](CONTRIBUTING.md)

The included local package identifies itself as **v1.1**. The original community announcement dates to 2014, and the local PDF is a March 2015 documentation snapshot. Historical documentation may differ from this source; use the implementation notes for important discrepancies.

## Licensing and demo assets

Daniel Lema's Mecanim Control source code is released under the [MIT License](LICENSE). This permission does not relicense Unity-provided assets or third-party content in the surrounding Unity project.

The entire historical `Demo` folder is intentionally excluded from GitHub, including its Unity-provided Constructor character, textures, animations, and scenes. For the complete demo package, get [Mecanim Control on the Unity Asset Store](https://assetstore.unity.com/packages/tools/animation/mecanim-control-15156), subject to that distribution's applicable terms. Free availability does not imply permission to redistribute Unity's model as standalone source assets.

The original PDF and package notes are retained as historical documentation. Their Asset Store references describe the original distribution; the author's source-code grant for this GitHub release is MIT.

## Author

**Daniel Lema — Mind Studios**

[Portfolio](https://mindstudios.dev/) · [Universal Fighting Engine](https://www.ufe3d.com/) · [GitHub](https://github.com/MindStudios)
