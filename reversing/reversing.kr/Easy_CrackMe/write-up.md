# [Reverse Engineering] Easy CrackMe Write-up

## 1. 개요

이번 문제는 Windows 실행 파일(`.exe`)을 역공학하여 프로그램 내부에 구현된 비밀번호 검증 로직을 분석하고, 올바른 비밀번호를 찾아내는 CrackMe 문제이다.

Ghidra를 사용하여 실행 파일의 기계어를 C 형태의 의사코드(Pseudocode)로 디컴파일하고, 문자열 비교 함수와 조건 분기문을 분석하여 정답을 도출하였다.

### 분석 환경

| 항목 | 내용 |
|---|---|
| 분석 도구 | Ghidra 12.1.4 |
| 운영체제 | Windows 11 |
| 파일 형식 | Windows PE (.exe) |
| 아키텍처 | x86 (32-bit) |
| 문제 이름 | Easy CrackMe |
| 비밀번호 검증 함수 | `FUN_00401080` |
| 함수 시작 주소 | `0x00401080` |

## 2. 프로그램 분석

실행 파일을 Ghidra로 불러온 후 자동 분석(Auto Analysis)을 진행하였다.

프로그램의 진입점인 `entry()` 함수를 확인한 결과, Windows API를 이용한 실행 환경 초기화 및 C 런타임 초기화 코드가 존재하였다.

```c
pHVar3 = GetModuleHandleA((LPCSTR)0x0);
UVar4 = FUN_00401000(pHVar3);
FUN_004012f4(UVar4);
```

여기서 `FUN_00401000()`이 호출되는 것을 확인할 수 있다.

해당 함수는 프로그램의 실제 동작과 연결되는 진입 함수로 판단하였으며, 관련 함수들을 추적하여 비밀번호를 검사하는 `FUN_00401080()`을 확인하였다.

## 3. 비밀번호 검증 함수 분석

**분석 주소: `0x00401080`**

```c
void __cdecl FUN_00401080(HWND param_1)
```

해당 함수는 사용자로부터 입력받은 문자열을 조건에 따라 검사하고, 결과에 따라 성공 또는 실패 메시지를 출력한다.

### 3.1 사용자 입력 처리

```c
CHAR local_64;
char local_63;
char local_62;
char acStack_61[97];
```

디컴파일된 코드에서는 입력 문자열이 여러 지역 변수로 나뉘어 표시된다.

하지만 실제로는 연속된 스택 메모리 영역을 사용하고 있으며, 각 변수는 입력 문자열의 특정 위치를 가리킨다.

```c
GetDlgItemTextA(param_1, 1000, &local_64, 100);
```

`GetDlgItemTextA()`는 Windows API 함수로, 다이얼로그의 입력 컨트롤에서 문자열을 가져온다.

- `param_1`: 다이얼로그 윈도우 핸들
- `1000`: 입력 컨트롤 ID
- `&local_64`: 입력 문자열이 저장될 버퍼
- `100`: 버퍼 크기(문자 수)

따라서 최대 99개의 문자와 NULL 종료 문자를 저장할 수 있다.

### 3.2 두 번째 문자 검증

```c
if (local_63 == 'a') {
```

`local_63`은 입력 문자열의 두 번째 문자에 해당한다.

따라서 두 번째 문자가 `a`인지 검사한다.

```text
? a ? ? ? ? ? ? ? ? ? ? ?
  ^
  두 번째 문자
```

조건에 일치하지 않으면 이후 비교를 진행하지 않고 실패 메시지를 출력한다.

### 3.3 세 번째와 네 번째 문자 검증

```c
iVar3 = _strncmp(&local_62, &DAT_00406078, 2);

if (iVar3 == 0) {
```

`strncmp()`는 두 문자열을 지정된 길이만큼 비교하는 함수이다.

```c
strncmp(str1, str2, 2);
```

반환값이 `0`이면 비교한 두 문자가 일치한다는 뜻이다.

분석 결과 `DAT_00406078`에 저장된 비교 대상 문자열은 `5y`이다.

따라서 세 번째와 네 번째 문자는 다음과 같다.

```text
? a 5 y ? ? ? ? ? ? ? ? ?
    ^ ^
    비교 대상
```

### 3.4 나머지 문자열 검증

다음 코드는 나머지 비밀번호를 비교하는 부분이다.

```c
pcVar4 = s_AGR3versing_0040606a;
pbVar2 = (byte *)(acStack_61 + 1);

do {
    pcVar4 = pcVar4 + 2;
    bVar1 = *pbVar2;
    bVar5 = bVar1 < (byte)*pcVar4;

    if (bVar1 != *pcVar4) {
        goto LAB_00401102;
    }

    if (bVar1 == 0)
        break;

    bVar1 = pbVar2[1];
    bVar5 = bVar1 < (byte)pcVar4[1];

    if (bVar1 != pcVar4[1])
        goto LAB_00401102;

    pbVar2 = pbVar2 + 2;

} while (bVar1 != 0);
```

여기서 `s_AGR3versing_0040606a`는 비교에 사용되는 문자열 데이터이다.

코드를 분석하면 다음과 같은 특징을 확인할 수 있다.

1. `pcVar4`를 2바이트 이동시키며 비교 대상 위치를 조정한다.
2. `pbVar2`는 사용자가 입력한 문자열의 다섯 번째 문자부터 가리킨다.
3. 반복문에서 두 문자를 순서대로 비교한다.
4. 문자가 다르면 실패 처리 경로로 이동한다.
5. 문자열 종료 문자(`\0`)까지 일치하면 비교가 성공한다.

분석 및 실제 입력 검증 결과, 이 부분에서 요구하는 문자열은 다음과 같다.

```text
R3versing
```

따라서 현재까지 확인한 비밀번호는 다음과 같다.

```text
? a 5 y R 3 v e r s i n g
```

### 3.5 첫 번째 문자 검증

나머지 문자열 비교가 끝난 후 다음 조건문이 실행된다.

```c
if ((iVar3 == 0) && (local_64 == 'E')) {
```

두 가지 조건을 동시에 확인한다.

- `iVar3 == 0`: 이전 문자열 비교 성공
- `local_64 == 'E'`: 첫 번째 문자가 `E`

따라서 첫 번째 문자는 `E`이다.

모든 조건을 결합하면 다음 비밀번호가 완성된다.

```text
Ea5yR3versing
```

## 4. 성공 및 실패 처리

### 4.1 비밀번호 일치

```c
MessageBoxA(
    param_1,
    s_Congratulation_!!_00406044,
    s_EasyCrackMe_00406058,
    0x40
);

EndDialog(param_1, 0);
```

비밀번호가 일치하면 `MessageBoxA()`를 호출하여 성공 메시지를 출력한다.

```text
Congratulation !!
```

이후 `EndDialog()`를 호출하여 다이얼로그를 종료한다.

### 4.2 비밀번호 불일치

```c
MessageBoxA(
    param_1,
    s_Incorrect_Password_00406030,
    s_EasyCrackMe_00406058,
    0x10
);
```

조건 중 하나라도 만족하지 않으면 다음 오류 메시지가 출력된다.

```text
Incorrect Password
```

## 5. 검증 로직 재구성

Ghidra가 생성한 의사코드는 컴파일 과정에서 최적화된 명령어를 기반으로 하기 때문에 복잡하게 보인다.

하지만 실제 비밀번호 검증 로직을 단순화하면 다음과 같이 표현할 수 있다.

```c
#include <windows.h>
#include <string.h>

void CheckPassword(HWND hwnd)
{
    char password[100] = {0};

    GetDlgItemTextA(
        hwnd,
        1000,
        password,
        sizeof(password)
    );

    if (
        password[0] == 'E' &&
        password[1] == 'a' &&
        strncmp(password + 2, "5y", 2) == 0 &&
        strcmp(password + 4, "R3versing") == 0
    ) {
        MessageBoxA(
            hwnd,
            "Congratulation !!",
            "EasyCrackMe",
            MB_ICONINFORMATION
        );

        EndDialog(hwnd, 0);
    }
    else {
        MessageBoxA(
            hwnd,
            "Incorrect Password",
            "EasyCrackMe",
            MB_ICONERROR
        );
    }
}
```

위 코드는 원본 소스코드가 아니라 분석 결과를 바탕으로 기능을 이해하기 쉽게 재구성한 코드이다.

## 6. 최종 정답

**Answer: `Ea5yR3versing`**

| 검사 단계 | 검사 내용 | 결과 |
|---|---|---|
| 1 | 첫 번째 문자 | `E` |
| 2 | 두 번째 문자 | `a` |
| 3 | 세 번째~네 번째 문자 | `5y` |
| 4 | 다섯 번째 문자 이후 | `R3versing` |
| 5 | 전체 문자열 검증 | `Ea5yR3versing` |

실제 프로그램에 해당 문자열을 입력하여 성공 메시지 `Congratulation !!`이 출력되는 것을 확인하였다.

## 7. 결론

이번 Easy CrackMe 문제에서는 Ghidra를 이용해 Windows EXE 파일을 분석하고, 내부 비밀번호 검증 함수의 동작을 파악하여 정답을 찾아냈다.

비밀번호를 단일 문자열 비교 함수로 검증하는 것이 아니라 개별 문자 비교, `strncmp()`, 반복문을 조합하여 나누어 검사한다는 점이 특징이었다.

이번 분석을 통해 다음 내용을 학습할 수 있었다.

- Windows PE 실행 파일의 기본 구조
- Ghidra의 Decompiler를 이용한 C 의사코드 분석
- 스택에 저장된 문자열과 지역 변수의 관계
- `GetDlgItemTextA()`와 `MessageBoxA()` 등 Windows API의 동작
- `strncmp()` 및 문자 단위 비교를 활용한 비밀번호 검증
- 조건 분기를 추적하여 프로그램의 검증 로직을 복원하는 방법

**최종적으로 비밀번호 `Ea5yR3versing`을 도출하였으며, 실제 프로그램에서 정답임을 확인하였다.**