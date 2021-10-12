---
layout: post
title: "LibGdx Tutorial - Viewport"
categories:
  - Game Dev 
tags:
  - libGdx
  - game
  - Graphics
---

다른 연습용 게임을 만들기에 앞서 기본적인 개념을 공부하기 위해 [위키](https://github.com/libgdx/libgdx/wiki)를 읽어보았습니다.
여기서 저를 너무 어렵게 만든 개념이 있었는데 바로 [Viewport](https://github.com/libgdx/libgdx/wiki/Viewports) 입니다.
(아직 배우지 않은) Stage와 Camera와는 다른 strategies를 가지고 있다고 합니다만, 이런저런 공부를 해보니 Viewport를 쓰게 되면 카메라에 내장 되어 있는 viewport는 다 무시된다는 것이 결론이였습니다.

# Viewport 정의

Viewport는 화면 크기와 이를 어떻게 보여주느냐를 결정하는 개념입니다. 여기서 화면 크기는 (데스크탑 기준) 윈도우 크기와는 다릅니다.

![Viewport](/assets/images/libgdx/FitViewport.jpg)

이미지에서 보다시피 게임화면(이미지부분)과 전체적인 윈도우 창의 크기는 다릅니다.
아래에서 소개할 Viewport 종류에 따라 윈도우 크기에 따른 그려지는 게임 화면이 달라집니다.

# Viewport Size

Viewport를 사용하게 되면, Game World Size가 변동 됩니다. 
게임 화면 크기는 Viewport에 정해진 수치만큼이 되고, 단위 역시 그에 따라 변경 됩니다.  
예를 들어 Viewport width가 100이고 이미지의 width가 1이라면 화면에서는 1/100의 크기로 보여지게 됩니다.  
(Camera Viewport의 기본 값은 윈도우 크기와 동일합니다.)

# Viewport 사용 방법

```kotlin
private val camera by lazy { OrthographicCamera() }
private val viewport by lazy { FitViewport(100f, 100f, camera) }

override fun resize(width: Int, height: Int) {
  viewport.update(width, height)
}

override fun render() {
  super.render()
  viewport.apply()
  batch.projectionMatrix = camera.combined
  
  batch.run {
    begin()
    draw(img, 0f, 0f, 100f, 100f)
    end()
  }
}
```

1. Viewport를 생성할 때 Camera 인자를 넘겨 줍니다. 이는 생략 가능하며 생략할시 Viewport 내부에서 OrthographicCamera 생성하여 사용합니다.

2. Viewport를 사용한다면 resize Event에서 update 메소드를 사용하여 윈도우 창 크기가 변경 되었다는걸 알려주어야 합니다.
만약 이를 하지 않는다면 윈도우 크기에 맞춰 Viewport가 변경되지 않습니다.

3. render에서는 camera를 update 하듯이 viewport.apply()를 해야합니다.
이 메소드에서 camera.update를 진행하고 있고, camera의 viewport를 덮어씌워버립니다.
(처음 이야기한 Camera Viewport가 무시된다는 이야기!)
아래는 이를 행하는 apply 내부 코드입니다.

```java
public void apply () {
  apply(false);
}

public void apply (boolean centerCamera) {
  HdpiUtils.glViewport(screenX, screenY, screenWidth, screenHeight);
  camera.viewportWidth = worldWidth;
  camera.viewportHeight = worldHeight;
  if (centerCamera) camera.position.set(worldWidth / 2, worldHeight / 2, 0);
  camera.update();
}
```

# Viewport 종류

Viewport 종류에 따라 화면이 다르게 그려집니다. 특히 이 중 Stretch, Fit, Fill은 [ScalingViewport](https://libgdx.badlogicgames.com/ci/nightlies/docs/api/com/badlogic/gdx/utils/viewport/ScalingViewport.html)를 사용합니다.  
[위키](https://github.com/libgdx/libgdx/wiki/Viewports#stretchviewport)에 더 자세한 설명이 적혀 있습니다.

## StretchViewport

![StretchViewport](/assets/images/libgdx/StretchViewport.jpg)

게임 화면 종회비를 무시하고 윈도우 크기 그대로 보여줍니다.

## FillViewport

![FillViewport](/assets/images/libgdx/FillViewport.jpg)

게임 화면 종횡비가 개지지 않습니다. 대신 이미지가 짤릴 수 있습니다.

## FitViewport

![FitViewport](/assets/images/libgdx/FitViewport.jpg)

게임 화면 종회비가 깨지지 않는 대신 해당 크기가 부족한 곳에는 Black Bar가 생깁니다.

## ScreenViewport

![ScreenViewport](/assets/images/libgdx/ScreenViewport.jpg)

화면 크기 그대로 보여줍니다. 유일하게 크기 설정이 존재 하지 않습니다.

## ExtendViewport

![ExtendViewport](/assets/images/libgdx/ExtendViewport.jpg)

Scale 정책은 FitViewport와 똑같습니다. 하지만 부족한 곳에 Black Bar 대신 게임 화면이 그려집니다.
(위 이미지에선 같은 이미지가 반복 되는 것으로 보이지만 3개의 이미지를 각각 따로 그린 것입니다.)

## CustomViewport

직접 클래스를 생성하여 나만의 Viewport를 만들 수 도 있습니다.
