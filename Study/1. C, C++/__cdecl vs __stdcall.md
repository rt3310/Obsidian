cdecl(Caller Cleanup)과 stdcall(Standard Call)은 함수 호출 규약의 주요 유형이고 이외에도 여러 가지 유형이 있는데 일반적으로 cdecl과 stdcall이 자주 스인다..
cdecl은 C Declaration의 약자이다.
## C의 `__stdcall` 사용

```c
int __stdcall MyFunc(int a, int b)
{
	return a > b;
}
```
우선 cdecl은 C언어의 기본이기 때문에 명시적으로 사용할 필요가 없고 stdcall은 함수 앞에 `__stdcall`이라는 지정자를 붙여야 한다.

## Windows API의 `__stdcall` 사용

```c
DWORD WINAPI MyThreadFunction(LPVOID lpParam)
{
	while(1) {
		WaitForSingleObject(hExitEvent, 100);
	}
	return 0;
}
```
Windows API 같은 환경에서는 `__stdcall`보다는 WINAPI라는 매크로를 붙인다.
`WINAPI`라는 매크로 정의에 대해서 들여다보면 `#define WINAPI __stdcall`이라고 되어있다.

## `__cdecl` vs `__stdcall`
cdecl은 호출자(caller)가 스택을 정리하는 규약이고, stdcall은 피호출자(callee)가 스택을 정리하는 규약이다.

### 1. `__cdecl` (C Declaration)
- **기본 사용처**: C/C++ 언어의 기본(Default) 호출규약이다.
- **매개변수 전달**: 오른쪽에서 왼쪽 순서(`Right-to-Left`)로 스택에 푸시한다.
- **스택 정리 주체**: 호출자(Caller)
- **핵심 특징**: 
	- **가변 인자(Variable Arguments)** 함수를 구현할 수 있는 유일한 방식이다.
	- `printf("%d %d", a, b)`처럼 인자의 개수가 가변적일 때, 인자를 몇 개나 넣었는지 알고 있는 것은 함수를 호출한 쪽(Caller)뿐이기 때문에 호출자가 실행 끝에 스택을 직접 비워줘야 한다.
- **이름 장식 (x86 C 기준)**: 함수 이름 앞에 언더바(`_`)가 붙습니다. (예: `_add`)
### 2. `__stdcall` (Standard Call)
- **기본 사용처**: Windows API(Win32 API) 함수들의 표준 호출규약이다.
- **매개변수 전달**: 오른쪽에서 왼쪽 순서(`Right-to-Left`)로 스택에 푸시한다.
- **스택 정리 주체**: **피호출자(Callee)**
- **핵심 특징**
	- 함수 내부가 끝나는 시점에 `RET N` 명령어를 사용해 스스로 스택을 정리한다.
	- 함수를 호출하는 곳마다 스택 정리 명령어를 붙일 필요가 없어 **전체 바이너리(실행 파일) 크기를 줄일 수 있다는 장점**이 있다. 단, 인자의 개수가 고정되어 있어야 하므로 가변 인자 함수에는 사용할 수 없다.
- **이름 장식 (x86 C 기준)**: 함수 이름 앞에 `_`, 뒤에 `@인자바이트크기`가 붙는다. (예: `_add@8`)

## 어셈블리 코드로 보는 직관적 비교

두 인자를 더하는 `int add(int a, int b)` 함수를 호출할 때 생성되는 x86 어셈블리 코드 차이이다.

```asm
push 2          ; 인자 b
push 1          ; 인자 a
call _add       ; 함수 호출
add esp, 8      ; [호출자가 직접 스택 정리] (4바이트 * 2개 = 8바이트)
```

### `__stdcall`
```asm
push 2          ; 인자 b
push 1          ; 인자 a
call _add@8     ; 함수 호출 (스택 정리는 _add@8 내부 끝의 'ret 8' 명령어가 처리)
```

> [!info] **참고 (64비트 x64 환경)** 
> 32비트(x86)에서는 `__cdecl`, `__stdcall`, `__fastcall` 등의 구분이 명확하게 쓰였지만, **64비트(x64) 환경에서는 architecture 차원에서 호출규약이 하나로 통일**되었다. (Windows x64 ABI / System V AMD64 ABI)
> 64비트에서는 스택 대신 레지스터(`RCX`, `RDX`, `R8`, `R9` 등)를 통해 매개변수를 우선 전달하므로 속도가 더 빠르다.


