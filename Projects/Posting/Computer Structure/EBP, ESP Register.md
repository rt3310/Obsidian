
## ESP - Stack Pointer

현재 스택의 가장 위(최신주소)에 들어있는 데이터를 가리키는 포인터이다.

ESP는 Push, Pop을 할 때 기준이 되는 포인터이다. (ESP를 기준으로 Push/Pop 된다) 데이터가 추가될 때마다 ESP가 내려가며(x86) 함수의 경계를 가리키게 된다.

## EBP - Base Pointer

현재 스택의 가장 바닥(처음주소)을 가리키고 있는 포인터이다.

새로운 함수가 호출될 시에 EBP가 push되어 이전 EBP 값을 저장해 놓는다.
(EBP는 호출되기 전의 값을 스택에 넣어 반환 시 호출되기 직전(caller)으로 이동한다)

## SFP - Stack Frame Pointer

함수 에필로그에서 EBP가 Push할 때 이전 EBP 값을 저장해놓는 장소다. (caller 주소가 저장된다)

## EIP - Instruction Pointer

다음에 실행할 명령어의 주소를 가지고 있는 PC다. 현재 실행하고 있는 명령어가 종료되면 EIP에 있는 주소를 불러오게 된다.

## RET

함수 호출 다음에 실행할 코드의 주소를 담고 있는 값이다.
RET는 popeip, jmpeip로 구성되며 에필로그에서 EIP 값으로 전달되고 다음 코드를 실행한다.

## 스택 프레임 동작 순서

```asm
func() 함수 스택 프레임 :
에필로그 {
PUSH EIP - 다음에 실행할 코드를 스택에 미리 저장한다.
PUSH EBP - 함수를 호출하고(스택 프레임 생성), EBP를 사용하기 전에 기존 값을 스택에 저장한다
MOVE EBP, ESP - 현재 ESP 값(push 후 주소)을 EBP에 옮긴다. EBP = ESP
}

SUB ESP, 8 - 8만큼 ESP를 내린다. (2개의 지역변수 만큼)
MOV DWORD [EBP-4], 1
MOV DWORD [EBP-2], 2 // 변수 2개에 각각 1과 2를 대입한다. (값 할당)
MOV EAX, [value] // 리턴 값이 있을 경우 EAX에 저장된다

프롤로그 {
MOV ESP, EBP - 내려간 ESP를 다시 EBP가 있는 곳으로 올려줘서 변수 공간을 무효화한다.
POP EBP - pop한 값을 EBP에 전달하여 EBP가 호출 전(caller) 주소에 복원되도록 한다.
RETN - RET 명령어는 두 개의 명령어로 구성된다.
	POP EIP : pop한 값을 EIP에 저장하여 다음에 실행될 코드 주소를 저장한다.
	JMP EIP : 새로 저장된 주소로 EIP가 점프하여 다음 코드를 실행하게 한다.
}

main() :
ADD ESP, 8 // _cdcel 규약에 의해 스택 정리를 한다. (함수 호출에 사용한 인수를 정리함)
```
- 프롤로그 : 함수 내에서 사용할 스택 프레임을 설정하는 구간
- 에필로그 : 함수 처리를 완료하고 처음 호출한 지점으로 돌아가기 위해 스택을 복원하는 구간

## 함수 호출 규약

함수 호출 규약이란 함수가 호출될 때 **인자의** 전달방법(스택 or 레지스터), 전달 순서(오른쪽 -> 왼쪽 or 왼쪽 -> 오른쪽), 인자 전달에 이용된 스택의 해제 위치(호출한 함수 or 호출된 함수)를 정해놓은 규약이다.

**스택 정리는** 함수를 실행하고 **반환할 때** 인자를 정리하는데 이 때 호출 규약 별로 caller/callee에서 인자를 정리한다.

visual studio는 _cdcel 규약을 사용하여 함수를 호출할 쪽(caller)에서 스택을 정리한다.  
장점으론 caller가 전달되는 인자의 개수를 알고 있기 때문에 가변 인자를 사용할 수 있다.

반면 _stdcall은 callee에서 인자를 정리하는데 callee의 RET 문에서 정리한다.  
_fastcall은 처음 2개의 인자는 ecx, edx에 넣되 나머지 인자부턴 스택에 넣는다. (빠르다)

## 함수 Arguments 전달 과정

main()에서 인자를 2개 받는 func(int , int) 함수를 호출한다고 가정할 때 스택 프레임에서  
어떻게 callee의 매개변수에 인수를 전달하는지 그 과정을 알아본다.

```null
main() :
push b // 인수 a,b를 push로 생성한다.
push a // <- ESP location
call func00040167 // func(int f, int g) 함수를 호출
add esp, 8 // 함수가 종료되면 생성한 인수 a,b를 정리한다.

func(a,b) :
′′′ 에필로그 생략 ′′′
SUB ESP, 8 // 매개변수 수만큼 다시 push
mov EAX, DWORD [EBP+8] // callee의 아규먼트 [ebp+8]를
mov DWORD [EBP-8], EAX // caller의 매개변수 [ebp-8]에 전달
mov ECX, DWORD [EBP+C] // [ebp+c]를
mov DWORD [EBP-4], ECX // [ebp-4]에 전달

※ 매개변수와 인수는 push를 통해 구현된다.
```

실제 인수의 전달은 **스택 프레임을 넘나들어** caller의 스택 프레임에 접근해 [ebp+8]의 값을  
callee의 매개변수인 [ebp-8]에 복사하는 방식으로 이뤄진다.

[ebp+8]은 caller 인수 a를 가리키고, [ebp-8]은 callee 매개변수 f를 뜻한다.

그리고 _cdcel은 caller에서 인자를 정리하기에 함수가 종료되면 caller에서 ESP 값을 올려준다.

```null
add esp, 8 // 인수 스택 정리
```