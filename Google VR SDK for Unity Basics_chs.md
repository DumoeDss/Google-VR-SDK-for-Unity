# Google VR SDK for Unity Basics

本篇文章描述了如何通过插件中的脚本和预设体使现有的Unity项目支持VR效果。

## 在Unity编辑器中模拟VR显示

VR的控制方式和普通的手机app不同。在一个VR app中，你通过摄像机来让用户在一个真实的世界中运动。用户在VR模式下无法点击屏幕，只能通过使用Cardboard上的开关或者Daydream的控制器来进行操作。最后，用户通过靠近屏幕的镜头看到的所有东西都是立体效果的。

我们在插件中添加了以下几种额外的输入方式，使它可以更容易的为你模拟虚拟现实的经验：

- **基于鼠标的头部跟踪**: 在播放模式下，如果你按住Alt然后移动鼠标，你可以In Play mode, if you press Alt and move the mouse around, you can pan horizontally and tilt vertically around your scene as if your head is moving a VR viewer around. Use Ctrl with the mouse to simulate tilting your head from side to side.

- **Simulating trigger pulls**: In Play mode, use a mouse click to simulate the action of a user using a trigger in a Cardboard viewer. You can also attach a phone by USB with the Daydream [controller emulator](https://developers.google.com/vr/concepts/dev-kit-setup) in Play mode.

- **Simulating distortion correction**: In Play mode, the cameras will render with an image effect that simulates the same distortion correction for the Cardboard lenses as occurs on the phone. You can edit the *GvrViewer*properties to select a phone model and viewer model to emulate, under the **Unity Editor Emulation Settings**heading in the Inspector.

  **Note:** This emulation setting applies only when playing the scene in the editor. It has no effect on an actual phone.

## 开启立体渲染模式(stereo rendering)

在Scene中创建一个Empty物体，并添加 `GvrViewer` 脚本。也可以从Prefabs文件夹中将名为`GvrViewerMain` 的预设体添加到Scene中。 点击Play之后，就可以在Game视图中看到立体渲染了。

## 与app的交互

通过立体摄像机与应用程序交互会有一些不寻常的麻烦。当用户使用VR viewer时，不能像平常那样直接点击屏幕。所有的UI元素都必须为左右眼各制作一份，否则的话不能正常的显示（左右眼的UI会相对有些偏移，以此显示3D效果）。所有的UI元素都必须放置到虚拟空间中，这样用户才能够看的到它们（uGUI的默认Render Mode是Screen Space，因此需要更改成World Space才能够在VR模式下显示出来）。并且随着头部追踪的实现，放置UI的方式也增加了：例如，把UI放置到用户的一侧，或者高于正常的视线。

### 凝视（Gaze）

与VR应用交互的一个常用的方法是通过**gaze（凝视）**，or where the user is looking。Google VR SDK为Unity提供了两个主要的方法来实现凝视:  `GvrGaze` 脚本或者Unity's Event System的 `GazeInputModule` 脚本。

#### 使用 GvrGaze

 `GvrGaze` 脚本允许你使用凝视与游戏物体进行简单的交互。但是不适用与uGUI的元素。通过把 `Assets/GoogleVR/Scripts/UI/GvrGaze` 脚本拖拽到camera上就可以使用它了。它会把事件发送给实现了IGvrGazePointer和IGvrGazeResponder的物体。

#### 使用 GazeInputModule

 `GazeInputModule` 脚本是Unity Event System的一个输入模块，用户可以通过凝视来选择物体。使用方法是把 `Assets/GoogleVR/Scripts/GazeInputModule` 脚本添加到Scene中的 *EventSystem* 物体上。

**注意:** 输入模块添加到 *EventSystem*的顺序决定了他们的优先级, 因此如果`StandaloneInputModule` 的优先级更高, 当鼠标移动的时候他就优先会处理事件。在编辑器中运行的时候，这可能会影响到使用鼠标来模拟凝视，因此你可能想要禁用 `StandaloneInputModule`, 或者把组件列表中的`GazeInputModule` 移动到 `StandaloneInputModule`的上面。

要想实现与UI元素的交互，需要把Canvas的*Render Mode* 设置成 **World Space**。然后通过 `StereoController` 脚本设置一个 *Event Camera* (直接给摄像机添加脚本组件，或者把脚本添加到父物体上)。通常情况下选择的摄像机都是Main Camera。

要想实现在场景中与3D物体交互，需要给 *Event Camera* 添加一个*PhysicsRaycaster* 组件。你还需要给所有可交互的物体添加一个脚本来响应所生成的事件。你可以使用一个 [EventTrigger](https://docs.unity3d.com/Manual/script-EventTrigger.html) 或者在你的脚本中实现Unity的标准事件接口。还有，想要交互的物体需要有一个collider组件。

#### 凝视标记（Gaze reticles）

如果你想添加一个可视化的提示，来让用户可以看到他们的视线，你可以使用*GvrReticle* 脚本。一个简单的做法是把`GvrReticle` 预设体拖拽到Main Camera下，作为Main Camera的子物体。每当你注视某个物体来响应触发器的时候，它会以视线焦点为中心画一个圆。It draws a dot which expands to a circle whenever you are gazing at something responsive to the trigger.如果Event Camera 有i*PhysicsRaycaster*, this includes 3D objects with collider components.

如果使用 `GvrGaze` 脚本，你需要设置准星来瞄准物体。如果使用`GazeInputModule`, 则不需要对标记做额外的设置。

**Note**: Be sure to set any raycasters to ignore the reticle's *layer*. In particular, the `Canvas`' *Graphics Raycaster*should be set to not be blocked by the reticle.

### Daydream 控制器

If your application is targeting the Daydream platform, you can also use the Daydream controller as an input device. Add the `GvrControllerMain` prefab to your scene and implement the [Controller API](https://developers.google.com/vr/unity/controller-basics) to use the controller.

## Deferred Rendering and Image Effects

These effects generally do not work correctly unless the camera using them is drawing to the full screen or target texture. But in side-by-side stereo, this is never the case: each eye only draws to one half of the screen. Trying to use these effects in this situation usually results in one of the two eyes' image being stretched across the entire screen.

If you need to use Deferred Rendering or Image Effects, the problem is solved by rendering each eye's view first to a temporary texture (during which Unity sees it as "full screen") and afterwards blitting that texture into the window. This incurs a frame-rate penalty, so it is not enabled by default.

The `StereoController`'s **Direct Render** checkbox determines whether to draw directly into the window (faster but no effects) or into a temporary texture followed by a blit (slower but allows effects). Also, for Image Effects, be sure to replicate the main camera's Image Effect components on each eye camera. To reduce the frame rate cost, you can choose to leave out some of them.

