# Google VR SDK for Unity Basics

本篇文章描述了如何通过插件中的脚本和预设体使现有的Unity项目支持VR效果。

## 在Unity编辑器中模拟VR显示

VR的控制方式和普通的手机app不同。在一个VR app中，你通过摄像机来让用户在一个真实的世界中运动。用户在VR模式下无法点击屏幕，只能通过使用Cardboard上的开关或者Daydream的控制器来进行操作。最后，用户通过靠近屏幕的镜头看到的所有东西都是立体效果的。

我们在插件中添加了以下几种额外的输入方式，使它可以更容易的为你模拟虚拟现实的经验：

- **基于鼠标的头部跟踪**: 在播放模式下，按住Alt并移动鼠标，你可以在场景中转动视角，就好像是你的头在VR视角下转动一样。按住Ctrl并移动鼠标可以模拟晃动头部。

- **模拟触发按钮**: 在播放模式下，点击鼠标来模拟用户在Cardboard中使用触发按钮。你也可以在播放模式下，通过USB连接手机，在手机上使用Daydream [控制模拟器](https://developers.google.com/vr/concepts/dev-kit-setup)。

- **模拟失真校正**: 在播放模式下，摄像机会渲染一种特殊的图像效果，来模拟Cardboard镜头在手机上的失真校正。你可以在Inspector面板中通过编辑 *GvrViewer* 脚本中 **Unity Editor Emulation Settings** 下方的属性来选择模拟不同手机型号和Viewer类型。

  **注意:** 此模拟设置仅适用于在编辑器播放场景时使用，对实际的手机中没有作用。

## 开启立体渲染模式(stereo rendering)

在Scene中创建一个Empty物体，并添加 `GvrViewer` 脚本。也可以从Prefabs文件夹中将名为`GvrViewerMain` 的预设体添加到Scene中。 点击Play之后，就可以在Game视图中看到立体渲染了。

## 与App的交互

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

如果你想添加一个可视化的提示，来让用户可以看到他们的视线，你可以使用*GvrReticle* 脚本。一个简单的做法是把`GvrReticle` 预设体拖拽到Main Camera下，作为Main Camera的子物体。如果Event Camera 有i*PhysicsRaycaster* ，每当你注视某个带collider组件的3D物体来响应触发器的时候，它会以视线焦点为中心画一个圆。

如果使用 `GvrGaze` 脚本，你需要设置准星来瞄准物体。如果使用`GazeInputModule`, 则不需要对标记做额外的设置。

**注意**: 记得设置某些raycasters忽略reticle的层。尤其是 `Canvas` 的 *Graphics Raycaster* 需要被设置成不被reticle阻断。 

### Daydream 控制器

如果你的应用程序是Daydream 平台，你也可以使用Daydream 控制器作为输入设备。添加 `GvrControllerMain` 预设体到游戏场景中，并通过 [Controller API](https://developers.google.com/vr/unity/controller-basics) 来使用控制器。

## 延迟渲染与图像特效

这些效果通常不能正确的工作，除非摄像机使用这些效果来绘制到完整的屏幕或目标纹理。但是在side-by-side stereo中，不会出现这样的情况：每个眼睛只绘制屏幕的一半。试图在这种情况下使用这些效果通常会导致两个眼睛的图像中的一个在整个屏幕上被拉伸。

如果你需要使用延迟渲染或图像特效，问题可以通过xx首先绘制每个眼睛的视野到一个临时的纹理中（在Unity中是“full Screen”）和事后切割纹理到窗口中来解决。这会导致帧速率的惩罚，所以它默认是不启用的。

 `StereoController` 脚本的 **Direct Render** 复选框决定了是直接绘制到窗口（速度较快，但是没有特效）还是通过一个blit传送到临时的纹理中（速度慢，但是可以使用特效）。此外，对于图像效果，一定要把主摄像机的图像效果组件复制到每个眼睛相机上。为了降低帧率成本，你可以选择省略其中的一些。

