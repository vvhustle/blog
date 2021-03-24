---
title: 'Unity 정리'
date: 2020-07-04 12:21:13
category: 'unity'
draft: false
---
![](./images/unity.png)
<div style="text-align: center;">
    <span style="font-size:11px; color:grey">
        이 정적 페이지는 PC 버전에 최적화되어 있습니다.  
    </span>
</div>

---
이번에는 유니티의 코루틴에 대해서 알아보겠다.
## 코루틴 (Co - rotinue); 

멀티 쓰레드처럼 병렬적(Parallel)이진 않지만 협력적(Cooperative)이고 동시적(Concurrency)으로 동작하는 방식이다.

일반적으로 함수는 메인 루틴 + 서브루틴으로 구성되는 데 코루틴은 메인 루틴과 병렬적으로 동작하여 여러 entrypoint를 가지고 yield와 resume를 통해 루틴을 중단, 재시작 할 수 있다.

일반적으로 프로그램의 성능 향상을 위해 여러 쓰레드로 프로그램을 작성하여 병렬적으로 처리하지만, 컨텍스트 스위칭 비용이나 교착 등의 이유로 닷넷 프레임워크 기반의 유니티는 C#에서 Threading 클래스를 사용 못하고 monobehavior를 단일 스레드로 제공하여 대안으로 코루틴 사용을 권장한다. 


## Unity Corotine 특징

유니티에서 코루틴은 IEnumerator를 반환하고 yield가 한번 이상 사용되는 함수이다.

유니티 엔진과 스크립트의 상호작용하여
- 유니티 엔진의 Delayed Call Manager는 코루틴에서 yield return한 것들을 보관한다.
- 유니티 엔진은 매 프레임마다 D.C.M에게 yield 된 게 있나 확인하고 yield된 것이 있으면 
MoveNext()한다.
- 프로그래머가 임의로 시작하고 종료 가능하다.

## 장점

의도적으로 시간 혹은 흐름을 지연시켜 update()에서 매 프레임마다 오브젝트의 상태를 검사하지 않아도 된다. 
(Time.deltaTime의 대안)

파라미터를 넘기지 못하는 invoke의 단점을 극복한다.

메인쓰레드(유니티 엔진)는 코루틴 외부의 메인 루틴을 수행함으로 효율적이다.
(데이터 로딩이나 네트워크 통신은 유니티 엔진에게 넘기고 그 사이에 UI만 처리하는 식..)

멀티쓰레드 프로그래밍이 아니기에 자원 관리 및 ContextSwitching 등으로 부터 자유롭다.

## 동작 순서

1. 스크립트는 코루틴의 IEnumerator 객체를 생성한 다음 유니티 엔진에게 넘겨준다.
2. 엔진은 IEnumerator 객체를 엔진 스케줄러에 보관하고, IEnumerator 객체의 포인터(managed pointer)를 가지는 Coroutine이라는 객체를 생성해서 호출자에게 넘겨준다.
3. StopCoutine()을 호출하거나 MonoBehavior가 사라질때까지 Coroutine을 들고 있다가 사라진다.

아래는 sprit의 알파 값을 update에서 조절하는 코드 예시이다.
```csharp
public class Alpha : MonoBehaviour {

	private SpriteRenderer spriteRenderer;

	void Start () {
		spriteRenderer = GetComponent<SpriteRenderer> ();
	}

	void Update () { // NOTE : 매 프레임마다 계속 Call..
		Color color = spriteRenderer.color;
		if (color.a > 0.0f) {
			color.a -= 0.1f;
			spriteRenderer.color = color;
		}
	}
}
```

위의 코드를 코루틴을 사용하여 구현하면 아래와 같다.
```csharp
public class Alpha : MonoBehaviour {

	private SpriteRenderer spriteRenderer;

	void Start () {
		spriteRenderer = GetComponent<SpriteRenderer> ();
		StartCoroutine ("FadeIN");
	}

	IEnumerator FadeIN() {
		Color color = spriteRenderer.color;
		while (color.a > 0.0f) {
			color.a -= 0.1f;
			spriteRenderer.color = color;
			yield return new WaitForSeconds(0.1f);
		}
	}
}
```

유니티에서 제공하는 IEnumerator 인터페이스의 구조는 아래와 같다.
```csharp
public interface IEnumerator
{
	object Current { get; } // 코루틴 entry point를 관리하는 녀석 (like function pointer)
	
	bool MoveNext(); // Current를 움직인다.
	void Reset(); // Current를 초기화한다. 
}
```
아래의 코드는 IEnumerator 인터페이스를 사용한 예시이다. 
```csharp
IEnumerator enumerator = HelloWorld(); // 아무 일도 일어나지 않음;

while (enumerator.MoveNext()) { // yield를 만날때까지 수행, 값은 Current에 보관
		object res = enumerator.Current;
		Debug.Log(res);
}
IEnumerator HelloWorld() {
	yield return "hello";
	yield return "WaitForSeconds(1.0f)";
	yield return "world!";
}
/*
hello
1초 뒤
world!
*/
```

실질적인 게임 구현에서는 
위의 예제처럼 gui 구현 등에 사용되거나 네트워크 통신에서 사용된다.

## yield return의 사용

플래그 유형
: 렌더링 루프에서 업데이트하며 플래그 체크 후 실행
(WaitForSeconds, WaitForFixedUpdate, WaitForEndOfFrame, null)
콜백 유형
: 코루틴을 가지고 있다가 작업 끝나면 호출
(다른 코루틴, www, SyncOpertaion, webRequest.SendWebRequest)

자세한 내용은 
https://docs.unity3d.com/kr/530/Manual/ExecutionOrder.html

## yield의 한계

메서드 밖에서 yield를 쓰면 에러이다. 
익명 함수 혹은 람다에 쓰면 에러이다.  
try-catch-finally에서 yield break 빼고는 거의 대부분 에러이다.  

## 단점

병렬 프로그래밍이 아니기에 성능 향상에 대한 기대값이 작다.
게임 오브젝트가 disable될때 같이 종료시켜야 하므로 다시 시작하기 어렵다.
ref, out 등 레퍼런스형을 사용할 수 없다(비동기 동작)
실행 도중 취소 불가능하다.(yield할 때까지 묵묵히 수행)

## 기타

코루틴 entry point를 관리하는 녀석 (like function pointer)
- 코루틴도 중첩이 가능하다. 또한 문자열이나 배열 형식의 인자로 StartCorotine이 가능하다.
-> 이것을 이용해 인자를 여러개 넘길 수 있다. 하지만 코드 가독성에 주의한다.

- Yield때 매번 new해야하는가?
-> 코루틴을 래핑한 싱글턴한 매니저 클래스를 만들자

- 최적화
-> StartCorotine()도 하나의 인스턴스를 띄우는 것이므로 쓸데 없이 많이 만들지 말고 필요없으면 잘 정리하자.


## 참고 

[https://en.wikipedia.org/wiki/Coroutine#Implementations](https://en.wikipedia.org/wiki/Coroutine#Implementations)

[https://wooooooak.github.io/kotlin/2019/08/25/코틀린-코루틴-개념-익히기/](https://wooooooak.github.io/kotlin/2019/08/25/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%BD%94%EB%A3%A8%ED%8B%B4-%EA%B0%9C%EB%85%90-%EC%9D%B5%ED%9E%88%EA%B8%B0/)

[https://docs.unity3d.com/kr/530/Manual/Coroutines.html](https://docs.unity3d.com/kr/530/Manual/Coroutines.html)

[http://theeye.pe.kr/archives/2725](http://theeye.pe.kr/archives/2725)