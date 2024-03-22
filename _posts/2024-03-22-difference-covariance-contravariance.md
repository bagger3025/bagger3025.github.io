---
title: 
tags: 
post_time: 2024-03-22T15:23:57 +09:00
edit_time: 2024-03-22T17:53:32+09:00
---
# 동기

한 [유튜브 영상](https://www.youtube.com/watch?v=vfMpIsJwpjU)에서 Rust언어가 안전하지 않다는 이야기를 봤다. 이 내용의 후반에 나오는 Covariance와 ContraVariance가 잘 이해가 가지 않아 알아보고 정리한 글이다.

# Covariance

한국어로 '공변성'이라고도 하는 Covariance는 원래 지정된 것보다 더 많이 파생된 형식을 사용할 수 있다는 것이다.

`A`와 `B`가 타입이라 하고 `I<U>`는 type constructor `I`를 type argument `U`로 적용한 것이라 하자. type constructor `I`의 typing rule은 다음과 같다:

- [type의 순서](https://en.wikipedia.org/wiki/Subtyping)를 보존하면 covariant하다. 즉, 구체적인 것에서 일반적인 것으로의 타입 순서를 보존한다. 만약 `A <= B`이면, `I<A> <= I<B>`이다.

## 개념 이해

`Animal`의 subtype `Cat`(`Cat` <= `Animal`)을 생각하자. 이때, `Cat`객체는 `Animal` 객체라고 할 수 있다.

`Cat`은 `Animal`의 subtype이므로 `Animal`이 가지는 모든 것을 다 가지고 있다. 따라서 `Cat`타입을 `Animal`타입으로 대입할 수 있다.

`Cat`을 반환하는 함수 타입 `R<Cat>`과 `Animal`을 반환하는 함수 타입 `R<Animal>`이 있다 하자.

- 타입 `R<Cat>`를 타입 `R<Animal>`에 대입하는 경우
	- `R<Animal>`을 호출하면 반환 타입이 `Animal`이다.
	- 사실 이 함수는 `Cat`을 반환하는 함수였으므로 실제로는 `Cat`을 반환하게 된다.
	- 하지만 `Cat`은 `Animal`로부터 상속받았으므로, 실제와 달리 `Animal`을 반환받았다고 생각하고 사용하여도 문제가 없다. 위에서 "`Cat` 타입을 `Animal` 타입으로 대입할 수 있다"는 것과 같은 상황이기 때문이다.
- 타입 `R<Animal>`을 타입 `R<Cat>`에 대입하는 경우
	- `R<Cat>`을 호출하면 반환 타입이 `Cat`이다.
	- 사실 이 함수 타입은 `R<Animal>`이었으므로 반환 타입이 실제로는 `Animal`이다.
	- 실제로는 타입이 `Animal`인 변수를 `Cat`처럼 사용한다면 `Cat`에는 있지만 `Animal`에는 없는 특성을 불렀을 때 문제가 된다.
	- 따라서 이 경우는 가능하지 않다.

위 사실로 `R<Cat>`를 타입 `R<Animal>`에 대입할 수 있다. 즉, `R<Cat> <= R<Animal>`이다.

`Cat <= Animal`일 때 `R<Cat> <= R<Animal>`이므로 함수 반환값은 Covariance하다.

# Contravariance

한국어로 '반공변성'이라고도 하는 Contravariance는 원래 지정된 것보다 더 일반적인, 덜 파생적인 형식을 사용할 수 있다는 것이다.

[Covariance](#Covariance)에서 정의한 type constructor `I`의 typing rule은 다음과 같다.

- type의 순서를 역전하면 covariant하다. 만약 `A <= B`이면, `I<B> <= I<A>`이다.

## 개념 이해

앞선 Covariance에서와 같이 `Animal`의 subtype `Cat` (`Cat` <= `Animal`)을 생각하자.

`Cat`을 받는 함수 타입 `F<Cat>`과 `Animal`을 받는 타입 `F<Animal>`이 있다 하자.

- `F<Cat>`에 `F<Animal>`을 대입하는 경우를 살펴보자.
	- `F<Cat>`에 `Cat` 인자를 주어 함수를 호출해보자.
	- 이는 사실 인자 타입이 `Animal`이므로 가능한 호출 방법이다(왜냐하면 `Animal`에 `Cat`을 대입할 수 있기 때문에).
	- 따라서 `F<Cat>`에 `F<Animal>`을 대입할 수 있다.
- `F<Animal>`에 `F<Cat>`을 대입하는 경우를 살펴보자.
	- `F<Animal>`에 `Animal` 인자를 주어 함수를 호출하자.
	- 사실 인자 타입이 `Cat`인데 `Animal` 인자를 주었으므로 이는 가능하지 않다.
	- 따라서 `F<Animal>`에 `F<Cat>`을 대입할 수 없다.

위 사실로 `F<Animal>`를 `F<Cat>`에 대입할 수 있다. 즉, `F<Animal> <= F<Cat>`이다.

`Cat <= Animal`일 때 `R<Cat> <= R<Animal>`이므로 함수 인자는 Contravariance하다.

# 참고자료

- [Wikipedia Covariance and contravariance (computer science)](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science))
- [Microsoft 제네릭의 공변성(Covariance) 및 반공변성(Contravariance)](https://learn.microsoft.com/ko-kr/dotnet/standard/generics/covariance-and-contravariance)
