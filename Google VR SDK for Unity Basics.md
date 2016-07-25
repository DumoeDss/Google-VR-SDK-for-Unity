# Google VR SDK for Unity Basics

This document describes how to convert an existing Unity project to support VR using the scripts and prefabs in this plugin.

## VR viewer simulation in the Unity editor

VR controls are very different than for normal mobile apps. In a VR app, you match a user's movement in the real world with your camera. Users can't tap the screen with their phone in a VR viewer, instead using the trigger on a Cardboard viewer or a Daydream controller. Finally, the user views everything in stereo, through lenses close to the screen.

We've added the following extra inputs to our Unity plugins to make it easier for you to simulate the VR experience:

- **Mouse-based head tracking**: In Play mode, if you press Alt and move the mouse around, you can pan horizontally and tilt vertically around your scene as if your head is moving a VR viewer around. Use Ctrl with the mouse to simulate tilting your head from side to side.

- **Simulating trigger pulls**: In Play mode, use a mouse click to simulate the action of a user using a trigger in a Cardboard viewer. You can also attach a phone by USB with the Daydream [controller emulator](https://developers.google.com/vr/concepts/dev-kit-setup) in Play mode.

- **Simulating distortion correction**: In Play mode, the cameras will render with an image effect that simulates the same distortion correction for the Cardboard lenses as occurs on the phone. You can edit the *GvrViewer*properties to select a phone model and viewer model to emulate, under the **Unity Editor Emulation Settings**heading in the Inspector.

  **Note:** This emulation setting applies only when playing the scene in the editor. It has no effect on an actual phone.

## Enable stereo rendering

Add VR support to a scene by attaching a `GvrViewer` script to an object in your scene. You can do this simply by adding the `GvrViewerMain` prefab to the scene. Hit *Play* and you should see a stereo view.

## Interact with the app

There are many unique challenges when interacting with an application through stereo cameras. Users generally cannot tap on specific parts of the screen while using a VR viewer. UI elements must all be replicated in each eye or the user cannot see them properly. They also must be placed in virtual space so that the user can focus on them. And with head tracking available, placement options are increased: The UI can be placed to the user's side, or above the normal line of sight, for instance.

### Gaze

A common way to interact with VR applications is by **gaze**, or where the user is looking. The Google VR SDK for Unity provides two main ways for implementing gaze: the `GvrGaze` script or the `GazeInputModule` script with Unity's Event System.

#### Using GvrGaze

The `GvrGaze` script allows you to easily interact with game objects using gaze. It does not work with uGUI elements. Drag the script from `Assets/GoogleVR/Scripts/UI/GvrGaze` onto to the camera to use it. It will send events to objects that implement IGvrGazePointer and IGvrGazeResponder.

#### Using GazeInputModule

The `GazeInputModule` script is an Input Module for Unity's [Event System](https://docs.unity3d.com/Manual/EventSystem.html) which selects GameObjects based on the user's gaze. To use it, add the script from `Assets/GoogleVR/Scripts/GazeInputModule` to the scene's*EventSystem* object.

**Note:** The order of input modules attached to *EventSystem* determines their priority, so if the`StandaloneInputModule` is higher, then it will handle events when the mouse moves. When playing in the Editor, this may interfere with using the mouse to simulate gaze, so you may wish to disable `StandaloneInputModule`, or move the `GazeInputModule` up above it in the component list.

To interact with UI elements, set the Canvas *Render Mode* to **World Space**. Set the *Event Camera* to a camera controlled by a `StereoController` script (either directly or as a parent). In most cases, this will be the Main Camera.

To interact with 3D objects in the scene, add a *PhysicsRaycaster* component to the Event Camera. You must also have a script on each interactive object to respond to the generated events. You could use an [EventTrigger](https://docs.unity3d.com/Manual/script-EventTrigger.html) or implement some of the standard Unity Event interfaces in your scripts. The object needs a collider component as well.

#### Gaze reticles

If you want to add a visual cue to let the user see the point of their gaze, you can use the *GvrReticle* script. A simple way to do this is to drag and drop the `GvrReticle` prefab as a child of the Main Camera. It draws a dot which expands to a circle whenever you are gazing at something responsive to the trigger. If the Event Camera has a *PhysicsRaycaster*, this includes 3D objects with collider components.

If using the `GvrGaze` script, you'll need to set the reticle as its Pointer Object. If using the `GazeInputModule`, the reticle shouldn't require any additional setup.

**Note**: Be sure to set any raycasters to ignore the reticle's *layer*. In particular, the `Canvas`' *Graphics Raycaster*should be set to not be blocked by the reticle.

### Daydream controller

If your application is targeting the Daydream platform, you can also use the Daydream controller as an input device. Add the `GvrControllerMain` prefab to your scene and implement the [Controller API](https://developers.google.com/vr/unity/controller-basics) to use the controller.

## Deferred Rendering and Image Effects

These effects generally do not work correctly unless the camera using them is drawing to the full screen or target texture. But in side-by-side stereo, this is never the case: each eye only draws to one half of the screen. Trying to use these effects in this situation usually results in one of the two eyes' image being stretched across the entire screen.

If you need to use Deferred Rendering or Image Effects, the problem is solved by rendering each eye's view first to a temporary texture (during which Unity sees it as "full screen") and afterwards blitting that texture into the window. This incurs a frame-rate penalty, so it is not enabled by default.

The `StereoController`'s **Direct Render** checkbox determines whether to draw directly into the window (faster but no effects) or into a temporary texture followed by a blit (slower but allows effects). Also, for Image Effects, be sure to replicate the main camera's Image Effect components on each eye camera. To reduce the frame rate cost, you can choose to leave out some of them.

