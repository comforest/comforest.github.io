---
layout: post
title: "LibGdx Tutorial - Drop 01"
categories:
  - Game Dev 
tags:
  - libGdx
  - game
---

[공식 홈페이지의 Sample Game](https://libgdx.com/dev/simple-game/)을 따라하면서 libGdx의 기초적인 기능을 공부했습니다.
프로젝트를 생성할 때 언어를 Kotlin을 선택하였는데 Html은 지원하지 않는다 하여 생성이 안되었는데 코드는 Java로 생성되어 Kotlin으로 직접 변경 하였다.


## 프로젝트 구조

![libgdx_project_structure](/assets/images/libgdx/project_structure.png)

프로젝트는 다음과 같은 구조를 띄고 있었습니다. core에서 게임의 로직을 구현하고 각 플랫폼에 맞춰 Launcher가 존재하는 형태 입니다.
공부는 Desktop Launcher로만 진행 하였고 코드 자체는 특별할게 없었습니다.

```kotlin
val config = LwjglApplicationConfiguration()
config.title = "Drop"
config.width = 800
config.height = 480
LwjglApplication(TutorialGame(), config)
```

## 게임 개요

문서에서 소개하는 게임은 간단한 게임입니다.

![game_image](/assets/images/libgdx/tutorial_01_drop_bucket.png)

하단에 있는 바구니가 좌우로 움직이며 위에서 랜덤하게 생성된 물방울을 받으면 되는 게임입니다.
바구니는 화살표를 이용하거나 마우스 클릭으로 이동이 가능합니다.

## 특징적인 Code

튜토리얼의 로직 자체는 특별할 것은 없었습니다.
단지 따로 살펴볼만한 객체가 있어서 이를 정리하겠습니다.

### ApplicationAdapter, ApplicationListener

[Application의 수명 주기](https://github.com/libgdx/libgdx/wiki/The-life-cycle)와 관련되어 있는 인터페이스입니다.
Adapter 클래스는 Listener 인터페이스를 상속하여 빈 메소드로 구현해두었다.
인터페이스를 직접 사용해도 될 것 같은데 빈 메소드 밖에 없는 Adapter 클래스를 따로 만든 이유를 잘 모르겠습니다.

![LifeCycle](/assets/images/libgdx/application_lifecycle.png)

수명주기는 안드로이드의 수명주기랑 비슷하였는데 Unity의 Update 메소드와 같은 역할을 하는 메소드가 render()입니다.
[위키](https://github.com/libgdx/libgdx/wiki/The-life-cycle#where-is-the-main-loop)에서  render 메소드가 메인 Loop가 **명백(explicit)**하게 있진 않지만 render 메소드가 관련 있다고 합니다.)

(이 글은 강의보다 일지에 가까우니 지극히 필자의 배경지식으로 작성합니다.)

### OrthographicCamera

OpenGL이나 Unity의 Camera랑 같은 역할입니다.
Orthographic(직각)의 카메라라고만 설명이 적혀 있는데 2D Sprite에 특화된 카메라라고 생각 됩니다.  
또한 PerspectiveCamera 카메라도 있던데 Unity 카메라가 많이 생각 나는 부분입니다.
하지만 지금은 2D를 생각하고 있으니 OrthographicCamera로도 충분 할 것 같습니다.

### SpriteBatch

이미지를 그리는 클래스입니다.
중요하다고 생각하는 것은 Sprite와의 차이점 인데 [이 링크](https://gamedev.stackexchange.com/questions/121316/what-is-the-difference-between-sprite-and-spritebatch-specifically-in-the-conte/121340)에서 설명이 잘 되어있습니다.
정리하면 Sprite는 이미지의 정보를 저장하는 Data Structure 이고, SpriteBatch는 여러 Sprite를 그리는 클래스입니다.
SpriteBatch를 사용하는 주된 이유는 최적화 때문입니다. OpenGL의 Quad라는 개념이 나타나는데 여기까지 공부하지는 못하였습니다.
대략적인건 이미지 중첩 등으로 사실 모든 이미지를 화면에 그릴 필요가 없는데 이러한걸 정리해주는 객체라고 이해하고 있습니다.

이러한 특징 때문에 사용법이 독특한데 begin 메소드로 시작해서 end 메소드로 끝내야 한다는 점입니다.
```kotlin
batch.begin()
batch.draw(bucketImage, bucket.x, bucket.y)
for (raindrop in raindrops) {
  batch.draw(dropImage, raindrop.x, raindrop.y)
}
batch.end()
```

### Gdx

[Gdx 클래스](https://libgdx.badlogicgames.com/ci/nightlies/docs/api/)는 여러 static 필드로 이루어져 있습니다.
Application, Graphics, Auido, Files, Input에 어디서든 접근 가능할 수 있게 해주는 클래스입니다.
단 Graphics는 Rendering Thread에서만 사용해야한다고 되어 있습니다.

웃긴 이야기로 이 클래스는 실수(normally a design faux pas)라고 문서에 언급되어 있습니다. 하지만 최선이라고도 적혀있죠. ㅎㅎ

### com.badlogic.gdx.math 패키지

마지막으로 마음에 들었던 패키지 입니다. 이번 예제에서는 Vector3와 Rectangle인데 [Rectangle.overlaps](https://libgdx.badlogicgames.com/ci/nightlies/docs/api/)라는 메소드가 인상 깊었습니다.
이 외에도 자주 사용하는 수학적인 연산들을 메소드로 만들어 둔 것 같습니다.
추후에 이러한 연산을 할 일이 있을 때 잊지 말고 한번쯤은 찾아봐야 할 것 같습니다.
