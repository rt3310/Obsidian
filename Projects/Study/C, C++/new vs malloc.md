- malloc은 해당 포인터의 자료형을 모르기 때문에 void* 로 리턴한다. 때문에 malloc 사용시 type casting 을 해주어야 한다. 하지만 new의 경우 해당 객체에 맞는 포인터를 반환한다. 따라서 자료형을 따로 적어주지 않아도 된다.
- new를 이용해 객체를 생성하면 생성자를 자동으로 호출하여 초기 값을 줄 수 있다. 하지만 malloc의 경우 생성자 호출 기능이 없어 초기값을 줄 수 없다.
- free와 delete는 메모리를 해제하는 것은 동일하지만 delete는 소멸자를 호출한다.
- malloc의 경우 realloc을 이용해 메모리 크기를 조정할 수 있다. 하지만 new는 할당된 크기에 대한 메모리 재조정이 불가능하다. 재조정하기 위해서는 새로 할당, 복사, 해제 하는 과정을 통해야 한다. 따라서 객체가 아니고, 재할당이 빈번하게 일어나야 한다면 malloc이 좋은 선택이 될 수 있다.
- new는 연산자이고 malloc은 함수이다.
- new는 실패 시 bad_alloc exception을 던진다. 반면 malloc은 NULL을 반환한다.