## 테스트 전략

기본적인 수준에서 테스트는 성공 경로, 오류 경로, 경계 사례라는 세 가지의 시나리오로 구분할 수 있다.

- **성공 경로:** 긍정적인 흐름의 기능 테스트에 집중한다. 앱의 의도된 동작에 초점을 맞추기 때문에
예외나 오류 조건을 포함하지 않는다. 
- **오류 경로:** 부정적인 흐름의 기능 테스트에 집중한다. 즉 앱이 오류 조건 또는 잘못된 입력에 어떻게
응답하는지 확인하는 데 초점을 맞춘다. 의도된 동작이 실행되지 않을 때 발생하는 결과를 테스트하는 것이기에 
테스트 목록 구성이 어렵다.
- **경계 사례:** 앱의 경계 조건을 테스트하는 데 초점을 맞춘다.

>[!NOTE]
> 가능한 모든 오류 경로를 나열하고 이에 관한 테스트를 작성하여 다양한 시나리오를 발견하면서 단위 테스트를 계속
개선하는 것이 바람직하다.

## 테스트 만들기

좋은 단위 테스트에는 다음 4가지 특성이 있다.

1. **집중:** 코드 조각과 같은 단위를 테스트해야 한다. 코드 조각이란 클래스 또는 메서드를 하나만 포함하는 것이다. 
테스트 범위를 좁히고 개별 코드의 정확성을 검증하는 데 집중해야 한다.
2. **이해 가능:** 코드를 읽을 때 간단하고 이해하기 쉬워야 한다. 개발자는 테스트의 의도를 한눈에 파악할 수 있어야 한다.
3. **확장성:** 일관되게 통과하거나 실패해야 한다. 즉 코드 수정이 없었는 데도 어떤 때는 실패하고 어떤 때는 
통과하는 것처럼 불안정해서는 안된다.
4. **독립형:** 사용자와 상호작용할 필요없이 개별적으로 실행된다.

## 테스트 예제

지금부터 나오는 코드들은 알파벳이 뒤섞인 단어를 맞추는 'Unscramble' 애플리케이션에 대한 단위 테스트 예제이다. 
위에서 다룬 세 가지 시나리오에 대한 케이스를 포함한다.

### 1. 성공 경로

>Unscramble 앱의 성공 경로 예제는 뒤섞인 단어에 대해 올바른 답이 제시되었을 때
정수형의 점수 변수와 단어가 맞는지를 표현하는 부울 변수의 값이 올바르게 업데이트되는지 확인하는 것입니다.

```
private val viewModel = GameViewModel()

@Test
fun gameViewModel_CorrectWordGuessed_ScoreUpdatedAndErrorFlagUnset() {
    var currentGameUiState = viewModel.uiState.value
    val correctPlayerWord = getUnscrambledWord(currentGameUiState.currentScrambledWord)
    viewModel.updateUserGuess(correctPlayerWord)
    viewModel.checkUserGuess()

    currentGameUiState = viewModel.uiState.value
    // Assert that checkUserGuess() method updates isGuessedWordWrong is updated correctly.
    assertFalse(currentGameUiState.isGuessedWordWrong)
    // Assert that score is updated correctly.
    assertEquals(20, currentGameUiState.score)
}
```
위 코드가 하는 일은 다음과 같다:
1. `GameViewModel()` 객체를 만든다.
2. `GameViewModel()` 객체의 상태를 변수에 저장한다.
3. 생성된 뒤섞인 단어의 정답을 변수에 저장한다.
4. 정답 값을 객체의 사용자 추측 변수에 넣는다.
5. 사용자 추측와 정답을 비교하는 함수를 호출한다.
6. 상태 변수에 지금까지 수정된 객체의 상태를 재할당한다.
7. `Assert`를 통해 확인하고자 하는 두 변수를 확인한다.

### 2. 오류 경로
> Unscramble 앱에서 오류 경로의 예는 사용자가 잘못된 단어를 입력하고 Submit 버튼을 클릭하면 
> 오류 메시지가 표시되고 점수와 단어 개수가 업데이트되지 않는 것입니다.

```
@Test
fun gameViewModel_IncorrectGuess_ErrorFlagSet() {
    // Given an incorrect word as input
    val incorrectPlayerWord = "and"

    viewModel.updateUserGuess(incorrectPlayerWord)
    viewModel.checkUserGuess()

    val currentGameUiState = viewModel.uiState.value
    // Assert that score is unchanged
    assertEquals(0, currentGameUiState.score)
    // Assert that checkUserGuess() method updates isGuessedWordWrong correctly
    assertTrue(currentGameUiState.isGuessedWordWrong)
}
```

위 코드가 하는 일은 다음과 같다:
1. 임의의 단어 하나를 뷰모델 객체의 사용자-추측 변수에 넣는다.
2. 사용자 추측과 정답을 비교하는 함수를 호출한다.
3. 상태 변수에 지금까지 수정된 객체의 상태를 재할당한다.
4. `Assert`를 통해 변수 값들이 올바른지 확인한다.

### 3. 경계 사례
> Unscramble 앱에서 경계는 앱이 로드될 때의 UI 상태와 사용자가 최대 단어 수를 
> 재생한 후의 UI 상태를 확인하는 것이다.

- 다음 코드는 초기화가 잘 작동하는지 확인하는 테스트이다.
    
  ```
    @Test
    fun gameViewModel_Initialization_FirstWordLoaded() {
        val gameUiState = viewModel.uiState.value
        val unScrambledWord = getUnscrambledWord(gameUiState.currentScrambledWord)

        // Assert that current word is scrambled.
        assertNotEquals(unScrambledWord, gameUiState.currentScrambledWord)
        // Assert that current word count is set to 1.
        assertTrue(gameUiState.currentWordCount == 1)
        // Assert that initially the score is 0.
        assertTrue(gameUiState.score == 0)
        // Assert that the wrong word guessed is false.
        assertFalse(gameUiState.isGuessedWordWrong)
        // Assert that game is not over.
        assertFalse(gameUiState.isGameOver)
    }
    ```


- 다음 코드는 모든 라운드가 끝났을 때 변수 값이 제대로 설정되는지 확인하는 테스트이다.
        
    ```
      @Test
      fun gameViewModel_AllWordsGuessed_UiStateUpdatedCorrectly() {
        var expectedScore = 0
        var currentGameUiState = viewModel.uiState.value
        var correctPlayerWord = getUnscrambledWord(currentGameUiState.currentScrambledWord)
        repeat(MAX_NO_OF_WORDS) {
            expectedScore += SCORE_INCREASE
            viewModel.updateUserGuess(correctPlayerWord)
            viewModel.checkUserGuess()
            currentGameUiState = viewModel.uiState.value
            correctPlayerWord = getUnscrambledWord(currentGameUiState.currentScrambledWord)
            // Assert that after each correct answer, score is updated correctly.
            assertEquals(expectedScore, currentGameUiState.score)
        }
        // Assert that after all questions are answered, the current word count is up-to-date.
        assertEquals(MAX_NO_OF_WORDS, currentGameUiState.currentWordCount)
        // Assert that after 10 questions are answered, the game is over.
        assertTrue(currentGameUiState.isGameOver)
      }
    ```

> [!IMPORTANT]
> 모든 테스트는 독립적으로 실행되며 전역변수가 다른 메서드에서 초기화되어도 전혀 영향을 받지
> 않는다. 따라서 테스트 메서드의 순서는 어떻게 해도 상관이 없다.


## 코드 적용 범위에 대하여...

`Android Studio`는 로컬 단위 테스트를 위한 테스트 적용 범위 도구를 제공하여 단위 테스트에서 
다루는 앱 코드의 비율과 영역을 추적한다.

1. 테스트 파일을 마우스 우측 버튼으로 클릭하고 `Run 'TestName.kt' with Coverage`를 선택한다.
2. 테스트 실행이 완료되면 우측에 나타나는 적용 범위 패널에서 `Flatten Packages` 옵션을 선택한다.
3. 테스트 대상이였던 패키지를 확인한다. 더블 클릭하면 테스트 대상 파일들의 적용 범위가 표시된다.

위 단계를 통해 나타나는 보고서는 두 가지 측면으로 분류된다:
1. **단위 테스트가 적용된 메서드 비율**
2. **단위 테스트가 적용된 줄 비율**

위 보고서를 보고 누락된 부분에 대한 테스트 케이스를 추가하여 보완이 가능하다.    

허나 적용 범위가 높다고 테스트가 올바르게 설계된 것은 아니다. 작성된 테스트에 클래스 동작을 확인하는 
어셜션이 반드시 있어야 한다.

또한 테스트 적용 비율을 100%로 맞추기 위해 노력할 필요는 없다. UI 테스트를 사용하여 활동과 같은 
앱 코드의 일부를 테스트할 수도 있다.
