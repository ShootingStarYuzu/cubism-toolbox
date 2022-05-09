# ![Yuzu's Logo](./logo/yuzu-logo.png)[1] Yuzu's Toolkit for Cubism

> Disclaimer: These tools are all experimental and should not be used for productive client works.
> Make sure to always save the model beforehand and check thoroughly if it is still intact afterwards.

## What is this and how to use it?

This is a collection of tools for Live2D Cubism to automate repetitive tasks.
They are primarily of experimental nature and not meant to be used in a day to day workflow for now - probably until they are stable enough.
Currently, the following tools are included in this collection:

- [Python Scripting (ToDo)](./documentation/scripting.md)
- [Frame-by-Frame Animations (ToDo)](./documentation/image-sequence.md)

## How do I continue from here?

- [Installation (for Users)](#installation-for-users)
- [Build (for Developers)](#build-for-developers)

### Installation (for Users)

Download the release from the [Releases](https://github.com/ShootingStarYuzu/cubism-toolbox/releases) page.
Extract the ZIP-archive into the installation folder of Cubism.
Now start Cubism with the lauchner named `CubismEditor4_yuzu.exe` or `CubismEditor4_yuzu.bat`.

### Build (for Developers)

Let `CUBISM` be the installation folder of Live2D Cubism.

1) Import the source code into your IDE of choice
2) Set the execution environment to use the JRE from `CUBISM/app/jre`
3) Set the Java VM Arguments to the following:
   - `-Djava.library.path=CUBISM/natives;CUBISM/natives/windows-amd64`
   - `-Djava.security.manager`
   - `-Djava.security.policy=CUBISM/app/res/live2d.policy`
   - `-Djogamp.gluegen.UseTempJarCache=false`
   - `-Dsun.java2d.d3d=false`
   - `-Duser.language=en`
4) Set the main class to `me.yuzu.cubism.Launcher`
5) Update the Cubism installation folder in `build.gradle`
6) Update the configuration for your IDE (e.g. `gradle eclipse`; Make sure to use the Cubism JRE)
7) Build the source code with your IDE or Gradle

Now you should be fully prepared and able to launch Cubism with all the extensions.

---

[1] Logo (c) Shooting Star Yuzu (Drawn by [EmeraldGreenCat](https://emeraldgreencat.carrd.co/)) excluded from MIT license
