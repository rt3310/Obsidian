**새롭게 생성된 객체가 할당(Allocation)되는 영역**
- 대부분의 객체가 금방 Unreachable 상태가 되기 때문에, 많은 객체가 Young 영역에 생성되었다가 사라진다.
- Young 영역에 대한 가비지 콜렉션(Garbage Collection)을 **Minor GC**라고 부른다.
- 더욱 효율적인 GC를 위해 Young 영역을 3가지 영역(Eden, survivor 0, survivor 1)으로 나눈다.