# Frame2AnimatorPro

Frame2AnimatorPro is a complete 2D animation workflow toolkit for Unity. It lets you import sprites, build animation clips, generate directional blend-tree animator controllers, and wire everything up to a character with ready-to-use runtime components. It’s designed to be fast for prototyping and robust enough for production and Asset Store use.

## Highlights

- Editor Window to manage 2D animation sets by type and direction
- Auto generation of AnimationClips and AnimatorControllers with 2D blend trees
- Supports 4-way, 8-way, and single-direction workflows
- Batch grid slicer for sprite sheets (Editor)
- Folder-based auto-builder (Editor): generate clips/controller from a folder structure
- Runtime driver updates Animator parameters from input/AI
- Keyboard and New Input System input sources
- Animation event receiver, Attack trigger helper, controller validator
- Prefab wizard for quick character setup
- Mini Timeline Editor (Editor): build clips with per-frame durations and events
- Preview Exporter (Editor): export animations to PNG sequences for documentation/tests

## Requirements

- Unity 2020.3+ (tested with newer LTS versions)
- 2D Sprite workflow
- Optional: Unity Input System package (for InputSystemInputSource). If not installed, a stub is used.

## Installation

1. Import the Frame2AnimatorPro folder into your Unity project (keep the folder structure under Assets).
2. Ensure your sprites are imported as Sprite (2D and UI).
3. Optional: Install “Input System” package if you want to use InputSystemInputSource.

Folder overview:
- Assets/Frame2AnimatorPro/Editor: Editor tooling and windows
- Assets/Frame2AnimatorPro/Scripts/Runtime: Runtime components
- Assets/Frame2AnimatorPro/Animations: Default output folder (you can change this)
- Assets/Frame2AnimatorPro/Prefabs: Created by the Prefab Wizard

## Quick Start (Editor Window)

1. Open the main window: Menu > Frame2Animator > 2D Animation Editor
2. Character Setup:
   - Assign your character GameObject (should have SpriteRenderer and Animator; buttons provided to add if missing).
   - Choose Direction Type (FourDirectional, EightDirectional, or SingleDirection).
3. Import Sprites:
   - Add frames manually or use “Import from Sprite Sheet” to slice frames.
   - Set frame size, start frame, count, frames per row, and pixels per unit, then Extract.
4. Animation Settings:
   - Configure Speed and Loop per animation type (Idle/Walk/Run/Jump/Attack/Custom).
5. Preview:
   - Play/Pause, step frames; verify your frames look right per direction.
6. Export:
   - Set Save Path and Controller Name.
   - Click “Create Animation Clips” to generate .anim files.
   - Click “Create Animator Controller” to build a controller with 2D blend trees.
   - Click “Create & Assign Everything” to assign the controller to your character Animator.

Tip: Default parameters used by generated controllers are listed under “Animator Parameters” below.

## Quick Start (Runtime)

Attach these components to your character:
- Animator (with the generated controller assigned)
- SpriteRenderer
- Frame2AnimatorDriver (updates parameters on the Animator)
- A CharacterInputSource:
  - KeyboardInputSource (WASD/Arrows + Space/Mouse0)
  - InputSystemInputSource (if using the new Input System)
- Optional: TopDownMover2D (Rigidbody2D-based simple mover)

At runtime, the Frame2AnimatorDriver sets:
- Horizontal/Vertical based on input
- Speed and IsMoving
- LastHorizontal/LastVertical for idle facing and 2D blend trees
- Triggers Attack on input when present

## Auto Builder (Improved)

Use Frame2Animator/Auto Build (Improved). It ensures:
- All selected parameters are present (Horizontal/Vertical, Last*, Speed, IsMoving, optional legacy MoveX/MoveY).
- Blend trees are created for Idle/Move with proper 2D directional mapping.
- Attack states (per direction or single) are created with AnyState transitions using a Trigger ("Attack") if attack clips are found.
- Folder scanning by type and direction:
  - Root/Idle/[N|S|E|W|NE|NW|SE|SW] or Root/Idle
  - Root/Move/[...] or Root/Move
  - Root/Attack/[...] or Root/Attack

Troubleshooting:
- Missing states/parameters previously: this builder adds/updates them. Check console warnings for missing clip folders or sprites.
- If transitions don’t work, verify Speed is updated by your driver and adjust Idle->Move threshold in the window.

## Animation Timeline Editor (Mini)

Menu: Frame2Animator/Animation Timeline Editor (Mini)
- Drag & drop sprites or add selected sprites.
- Set per-frame durations, add optional event names (e.g., Footstep).
- Build an AnimationClip asset, with loop toggle and FPS helper.

## Preview Exporter

Menu: Frame2Animator/Preview Exporter
- Export an AnimationClip or a selected sprite list to PNG frames.
- Configure output size, FPS, and background color.
- For GIF/MP4, consider integrating Unity Recorder or a GIF encoder plugin. This tool focuses on PNG sequences for reliability.

## Sprite Sheet Slicing (Editor)

Menu > Frame2Animator > Slice SpriteSheet (Grid)

- Assign a Texture2D sprite sheet
- Configure frame width/height, pixels per unit, and pivot (default bottom center)
- Click Slice to generate multiple sprites from the grid

## Additional Tools (Editor)

- Validate Animator Parameters:
  - Menu > Frame2Animator > Validate Animator Parameters
  - Checks an AnimatorController for required parameters and can auto-fix missing ones.
- Create Clip From Selected Sprites:
  - Select Sprites in the Project window, then Menu > Frame2Animator > Create Clip From Selected Sprites
  - Quickly builds a clip from selected sprites.
- Character Prefab Wizard:
  - Menu > Frame2Animator > Create Character Prefab Wizard
  - Creates a prefab with SpriteRenderer, Animator, Frame2AnimatorDriver, input source, and TopDownMover2D wired up.

## Animation Events (Runtime)

Add AnimationEventReceiver to your character to receive animation events:
- AttackStart(), AttackHit(), AttackEnd(), Footstep()
- CustomEvent(string payload)

You can bind these to UnityEvents in the inspector for sound/FX/gameplay.

For Attack trigger cleanup, you can attach AttackResetOnExitSMB (StateMachineBehaviour) to attack states.

## Animator Parameters

Created by default in generated controllers:
- Float: Horizontal, Vertical
- Float: LastHorizontal, LastVertical
- Float: Speed
- Bool: IsMoving
- Trigger: Attack (only if attack animations found)
- Legacy Floats (for compatibility): MoveX, MoveY, LastMoveX, LastMoveY

Centralized constant names are defined in:
- Scripts/Runtime/AnimatorParameterNames.cs

## Directions and Blend Trees

- 4-way: Down, Up, Left, Right
- 8-way: adds DownLeft, DownRight, UpLeft, UpRight
- SingleDirection: Default

Blend tree mapping (SimpleDirectional2D):
- Down: (0, -1)
- Up: (0, 1)
- Left: (-1, 0)
- Right: (1, 0)
- Diagonals: ~0.707 normalized values

Idle uses LastHorizontal/LastVertical, Movement uses Horizontal/Vertical.

## Best Practices

- Use consistent pixels per unit across your project (e.g., 16, 32, 64).
- Set pivot to bottom-center for characters to avoid foot sliding.
- Keep import Filter Mode = Point for pixel art.
- Order your frames consistently by name when using folder-based workflows.
- Keep Save Path inside the Assets/ folder (Unity requirement).

## API Reference (Key Runtime Classes)

- Frame2AnimatorDriver:
  - SetMovementInput(Vector2), SetLastFacing(Vector2), TriggerAttack()
  - Reads from CharacterInputSource if present
- CharacterInputSource (abstract):
  - Implement Move (Vector2) and AttackPressed (bool)
- KeyboardInputSource:
  - WASD/Arrows + Space/Mouse0 (or Input Manager axes)
- InputSystemInputSource:
  - Uses Unity Input System actions (conditionally compiled)
- TopDownMover2D:
  - Rigidbody2D-based simple mover (feeds Frame2AnimatorDriver)
- AnimationEventReceiver:
  - Exposes common animation events as UnityEvents

## Troubleshooting

- No sprites in preview:
  - Ensure the sprites exist and are assigned for the selected animation and direction.
  - If slicing, verify frame size and rows/columns are correct.
- Animator controller created but animations not playing:
  - Ensure the generated controller is assigned to your character’s Animator.
  - Verify parameters are updating (use the Animator window during Play mode).
- Attack trigger not doing anything:
  - Ensure Attack parameter exists on the controller and transitions to Attack_* states are set.
  - Check your input source and Frame2AnimatorDriver.TriggerAttack().
- Movement blend tree not switching:
  - Speed must rise above 0.1 to transition from Idle to Movement; adjust threshold if needed.
- Input System compile errors:
  - Either install the “Input System” package or ignore the InputSystemInputSource (a stub keeps compilation working).

## Menu Map

- Frame2Animator/2D Animation Editor
- Frame2Animator/Auto Build from Folder
- Frame2Animator/Slice SpriteSheet (Grid)
- Frame2Animator/Validate Animator Parameters
- Frame2Animator/Create Clip From Selected Sprites
- Frame2Animator/Create Character Prefab Wizard

## License and Support

- Publisher website: https://vectorstudios.kesug.com/assets
- Documentation: See the Docs section on our website.
- Support: Please use the contact/support page on our website.
- License: Include your license here (MIT/Custom).
