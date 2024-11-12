## 공통점

- 둘 다 스크립트에서 최초 1번만 실행된다

## 차이점

- Start는 객체가 Enable 상태일 때만 불려오지만 Awake는 비활성 상태에서도 불려온다.
- Start는 최초의 Update 함수가 실행되기 직전에 불려오고 Awake는 스크립트 오브젝트가 불려오자마자 (좀 더 정확히는 게임이 실행되기 전에) 실행된다.
- Scene 안에 Start 함수와 Awake 함수가 혼재되어 있다면, Awake 함수는 어떠한 Start 함수보다도 먼저 불려오는 것이 보장된다. (단, 플레이 도중에 동적으로 만든 오브젝트에서는 당연히 위 순서 보장이 지켜지지 않는다)
- Awake 함수가 불려올 때는 Scene에 존재하는 모든 오브젝트가 이미 생성되었음이 보장되므로 해당 오브젝트를 참조하는 것이 가능하다

**[출처]** [[Unity] Awake() vs Start()](https://blog.naver.com/blastingzone/220406033699)|**작성자** [고기](https://blog.naver.com/blastingzone)