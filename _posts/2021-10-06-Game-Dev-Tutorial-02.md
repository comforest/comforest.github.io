---
layout: post
title: "LibGdx Tutorial - Drop 02"
categories:
  - Game Dev 
tags:
  - libGdx
  - game
---

이전 포스트에 이어 [공식 홈페이지의 Extending the Simple Game](https://libgdx.com/dev/simple-game-extended/)을 마저 따라해보았습니다
이번 예제에서는 여러 화면을 만드는 것을 목표로 하고 있습니다. 

## Game class

ApplicationListener을 상속 받고 있는 클래스입니다. 다시 말해 각 플랫폼의 Launcher 클래스에서 ApplicationAdapter 대신 사용할 수 있다는 이야기이기도 합니다
```kotlin
class Drop : Game()

LwjglApplication(Drop(), config)
```
Adpater 클래스와 달리 각 Lifecycle 메소드가 비어있지 않은데 이는 아래 서술할 Screen의 Lifecycle를 관리하고 있습니다.
```kotlin
class Drop : Game() {
    val batch by lazy{ SpriteBatch() }
    val font by lazy{ BitmapFont() }

    override fun create() {
        setScreen(MainMenuScreen(this))
    }

    override fun render(){
        super.render()
    }

    override fun dispose() {
        batch.dispose()
        font.dispose()
    }
}
```
그래서 위 코드의 render을 보시면 super.render을 호출해야하는 것을 볼 수 있습니다. 또한 예제에서 나오진 않았는데 공부해보니 dispose 메소드도 super을 호출 해야 할 것 같습니다.

또한 batch/font 필드가 **public**되어 있습니다. 이는 Screen에서 사용하기 위함인데 이는 Screen 코드에서 더 자세히 이야기하겠습니다. 
## Screen interface

Screen interface는 ApplicationListener와 비슷한 메소드들을 포함하고 있습니다. 
대신 create가 없고, show와 hide 메소드가 추가되었습니다. create의 역할은 생성자가 대신 하고 있는 것 같으며, show/hide는 화면이 전환 될 때도 호출됩니다.

```kotlin

class MainMenuScreen(val game: Drop) : Screen {
    private val camera by lazy { OrthographicCamera() }

    init {
        camera.setToOrtho(false, 800f, 480f)
    }

    override fun render(delta: Float) {
        ScreenUtils.clear(0f, 0f, 0.2f, 1f)
        camera.update()
        game.batch.projectionMatrix = camera.combined
        game.batch.begin()
        game.font.draw(game.batch, "Welcome to Drop!!! ", 100f, 150f)
        game.font.draw(game.batch, "Tap anywhere to begin!", 100f, 100f)
        game.batch.end()
        if (Gdx.input.isTouched) {
            game.screen = GameScreen(game)
            dispose()
        }
    }
    
    // ... 나머지 inteface 메소드들
}
```

코드를 살펴보면 생성자에서 카메라 세팅을 하고 있습니다. (Adpater에서는 create에서 했었습니다.)

render 메소드를 살펴보면 game 클래스에서 만들어두었던 batch와 font 클래스를 쓰고 있습니다. 문서에서도  
It is a bad practice to create multiple objects that can be shared instead (see [DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself)).  
라고 언급을 하고 있습니다. 단지 이러한 구조를 다른 프로젝트들도 따라하는지 추후 더 찾아보면 좋을 것 같습니다.

---
실제 게임 로직이 들어가 있는 코드는 저번 예제와 별반 다르지 않습니다. create 메소드가 없어 생성자와 show 메소드에 나눠 처리하고 있다는 점 정도가 특별한 것 같습니다.

```kotlin
class GameScreen(val game: Drop) : Screen {
  ...
  
  init {
      camera.setToOrtho(false, 800f, 480f)
      rainMusic.isLooping = true

      spawnRaindrop()
  }

  override fun show() {
      rainMusic.play()
  }
  
  ...  
}
```
