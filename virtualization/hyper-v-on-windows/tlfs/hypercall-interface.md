---
title: Hypercall 인터페이스
description: 하이퍼바이저 Hypercall 인터페이스
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 88014624350a13b9f2672b3a08ac4161463fa602
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120838"
---
# <a name="hypercall-interface"></a>Hypercall 인터페이스

하이퍼바이저는 게스트에 대 한 호출 메커니즘을 제공 합니다. 이러한 호출을 hypercall 라고 합니다. 각 hypercall는 입력 및/또는 출력 매개 변수 집합을 정의 합니다. 이러한 매개 변수는 메모리 기반 데이터 구조 측면에서 지정 됩니다. 입력 및 출력 데이터 구조의 모든 요소는 자연 경계에 최대 8 바이트까지 채워집니다. 즉, 2 바이트 요소가 2 바이트 경계에 있어야 합니다.

두 번째 hypercall 호출 규칙은 필요에 따라 hypercall의 하위 집합에 사용할 수 있습니다. 특히 입력 매개 변수가 두 개 이하인 경우와 출력 매개 변수는 없습니다. 이 호출 규칙을 사용 하는 경우 입력 매개 변수는 일반 용도의 레지스터에 전달 됩니다.

세 번째 hypercall 호출 규칙은 필요에 따라 입력 매개 변수 블록이 최대 112 바이트인 hypercall의 하위 집합에 사용할 수 있습니다. 이 호출 규칙을 사용 하는 경우 휘발성 XMM 레지스터를 포함 하 여 입력 매개 변수가 레지스터에 전달 됩니다.

입력 및 출력 데이터 구조는 모두 8 바이트 경계의 메모리에 배치 되 고 8 바이트의 배수로 채워집니다. 패딩 영역 내의 값은 하이퍼바이저에서 무시 됩니다.

출력의 경우 하이퍼바이저는 패딩 영역을 덮어쓸 수 있습니다 (보장 되지는 않음). 패딩 영역을 덮어쓰는 경우 0을 씁니다.

## <a name="hypercall-classes"></a>Hypercall 클래스

두 클래스의 hypercall: simple 및 rep ("repeat"의 경우 short)가 있습니다. 간단한 hypercall는 단일 작업을 수행 하 고 입력 및 출력 매개 변수의 고정 크기 집합을 가집니다. Rep hypercall는 일련의 간단한 hypercall 처럼 작동 합니다. 입력 및 출력 매개 변수의 고정 크기 집합 외에도, rep hypercall에는 고정 크기의 입력 및/또는 출력 요소 목록이 포함 됩니다.

호출자가 처음에 hypercall를 호출 하면 입력 및/또는 출력 매개 변수 목록의 요소 수를 나타내는 rep 수를 지정 합니다. 또한 호출자는 사용 해야 하는 다음 입력 및/또는 출력 요소를 나타내는 rep 시작 인덱스를 지정 합니다. 하이퍼바이저는 목록 순서에서 (즉, 요소 인덱스를 늘려서) rep 매개 변수를 처리 합니다.

Hypercall에 대 한 후속 호출의 경우 담당자 시작 인덱스는 완료 된 요소 수를 나타내고,는 표시 개수 값 – 남은 요소 수와 함께 표시 됩니다. 예를 들어 호출자가 지정 개수 25를 지정 하 고 시간 제약 조건 내에서 20 개의 반복만 완료 된 경우 hypercall는 rep 시작 인덱스를 20으로 업데이트 한 후 호출 하는 가상 프로세서로 제어를 다시 반환 합니다. Hypercall이 다시 실행 되 면 하이퍼바이저가 요소 20에서 다시 시작 되 고 나머지 5 개 요소를 완료 합니다.

요소를 처리할 때 오류가 발생 하는 경우에는 오류가 발생 하기 전에 성공적으로 처리 된 요소의 수를 나타내는 적절 한 상태 코드가 담당자 완료 횟수와 함께 제공 됩니다. 지정 된 hypercall 제어 단어가 유효함 (다음 참조) 하 고 입/출력 매개 변수 목록에 액세스할 수 있다고 가정 하면 하이퍼바이저는 하나 이상의 표현을 시도 하지만, 호출자에 게 제어를 다시 반환 하기 전에 전체 목록을 처리 하지 않아도 됩니다.

## <a name="hypercall-continuation"></a>Hypercall 연속

Hypercall은 많은 주기를 사용 하는 복잡 한 명령으로 간주할 수 있습니다. 하이퍼바이저는 hypercall를 호출한 가상 프로세서에 제어를 반환 하기 전에 hypercall 실행을 50 μs 이하로 제한 하려고 시도 합니다. 일부 hypercall 작업은 50 μs 보장을 수행 하기 어렵기 때문에 충분히 복잡 합니다. 따라서 하이퍼바이저는 모든 rep hypercall forms를 포함 하 여 일부 hypercall에 대해 hypercall 연속 메커니즘을 사용 합니다.

Hypercall 연속 메커니즘은 대부분 호출자에 게 투명 합니다. 지정 된 시간 제한 내에 hypercall를 완료할 수 없는 경우 제어가 다시 호출자에 게 반환 되지만 명령 포인터는 hypercall를 호출한 명령을 벗어나 고급이 아닙니다. 이렇게 하면 보류 중인 인터럽트가 처리 되 고 다른 가상 프로세서를 예약할 수 있습니다. 원래 호출 스레드가 실행을 다시 시작 하면 hypercall 명령을 다시 실행 하 고 작업을 완료 하기 위한 작업을 진행 합니다.

가장 간단한 hypercall는 지정 된 시간 제한 내에 완료 되도록 보장 됩니다. 그러나 간단한 hypercall 수가 적으면 시간이 더 걸릴 수 있습니다. 이러한 hypercall는 hypercall 연속을 rep hypercall와 비슷한 방식으로 사용 합니다. 이 경우 작업에는 둘 이상의 내부 상태가 포함 됩니다. 첫 번째 호출은 개체 (예: 파티션 또는 가상 프로세서)를 하나의 상태로 전환 하 고, 반복 되는 호출 후 상태를 마지막으로 터미널 상태로 전환 합니다. 이 패턴을 따르는 각 hypercall에 대해 중간 내부 상태의 눈에 띄는 부작용이 설명 되어 있습니다.

## <a name="hypercall-atomicity-and-ordering"></a>Hypercall 원자성 및 순서 지정

명시 된 경우를 제외 하 고 hypercall에서 수행 하는 작업은 다른 모든 게스트 작업 (예: 게스트 내에서 실행 된 명령)과 시스템에서 실행 되는 다른 모든 hypercall에 대해 원자 단위입니다. 간단한 hypercall는 단일 원자성 작업을 수행 합니다. hypercall는 여러 독립적인 원자성 작업을 수행 합니다.

Hypercall 연속을 사용 하는 단순 hypercall에는 외부에서 볼 수 있는 여러 내부 상태가 포함 될 수 있습니다. 이러한 호출은 여러 원자성 작업으로 구성 됩니다.

각 hypercall 작업은 입력 매개 변수 및/또는 쓰기 결과를 읽을 수 있습니다. 각 작업에 대 한 입력은 모든 세분성에서 읽을 수 있으며, hypercall가 실행 된 후에 언제 든 지 작업이 실행 되기 전에 읽을 수 있습니다. 각 작업과 관련 된 결과 (즉, 출력 매개 변수)는 모든 세분성으로 작성 될 수 있으며 작업이 실행 된 후 hypercall가 반환 되기 전에 언제 든 지 기록 될 수 있습니다.

게스트는 실행 중인 hypercall 관련 된 입력 또는 출력 매개 변수를 검사 하 고 조작 하는 것을 방지 해야 합니다. Hypercall를 실행 하는 가상 프로세서는이 작업을 수행할 수 없지만 (hypercall가 반환 될 때까지 게스트 실행이 일시 중단 될 때) 다른 가상 프로세서에서이 작업을 수행할 수 없습니다. 이런 방식으로 작동 하는 게스트는 충돌 하거나 파티션 내에서 손상을 일으킬 수 있습니다.

## <a name="legal-hypercall-environments"></a>법적 Hypercall 환경

Hypercall는 가장 권한 있는 게스트 프로세서 모드 에서만 호출할 수 있습니다. X64 platfoms에서이는 현재 사용 권한 수준 (CPL)이 0 인 보호 모드를 의미 합니다. 실제 모드 코드는 유효 CPL 0으로 실행 되지만 실제 모드에서는 hypercall를 사용할 수 없습니다. 잘못 된 프로세서 모드에서 hypercall을 호출 하려고 하면 #UD (정의 되지 않은 작업) 예외가 생성 됩니다.

모든 hypercall는 아키텍처에서 정의 된 hypercall 인터페이스를 통해 호출 되어야 합니다 (아래 참조). 다른 방법 (예: hypercall 코드 페이지의 코드를 대체 위치로 복사 하 고 해당 위치에서 실행)으로 hypercall을 호출 하려고 하면 정의 되지 않은 작업 (#UD) 예외가 발생할 수 있습니다. 하이퍼바이저는이 예외를 제공 하지 않을 수 있습니다.

## <a name="alignment-requirements"></a>맞춤 요구 사항

호출자는 입력 및/또는 출력 매개 변수의 64 비트 게스트 실제 주소 (GPA)를 지정 해야 합니다. GPA 포인터는 8 바이트로 정렬 되어야 합니다. Hypercall에 입력 또는 출력 매개 변수가 포함 되지 않은 경우 하이퍼바이저는 해당 GPA 포인터를 무시 합니다.

입력 및 출력 매개 변수 목록은 겹칠 수 없습니다. Hypercall 입력 및 출력 페이지는 GPA 페이지로 예상 되며 "오버레이" 페이지가 아닙니다. 가상 프로세서가 입력 매개 변수를 오버레이 페이지에 쓰고이 페이지 내에서 GPA를 지정 하는 경우 입력 매개 변수 목록에 대 한 하이퍼바이저 액세스는 정의 되지 않습니다.

하이퍼바이저는 요청 된 hypercall를 실행 하기 전에 호출 파티션이 입력 페이지에서 읽을 수 있는지 확인 합니다. 이 유효성 검사는 지정 된 GPA가 매핑되고 GPA를 읽을 수 있는 것으로 표시 하는 두 가지 검사로 구성 됩니다. 이러한 테스트 중 하나가 실패 하면 하이퍼바이저는 메모리 가로채기 메시지를 생성 합니다. 출력 매개 변수가 있는 hypercall의 경우 하이퍼바이저는 파티션을 출력 페이지에 쓸 수 있는지 확인 합니다. 이 유효성 검사는 지정 된 GPA 매핑되고 GPA는 쓰기 가능으로 표시 되는 두 가지 검사로 구성 됩니다.

## <a name="hypercall-inputs"></a>Hypercall 입력

호출자는 hypercall 입력 값 이라고 하는 64 비트 값으로 hypercall을 지정 합니다. 다음과 같이 형식이 지정됩니다.

| 필드                   | 비트    | 제공된 정보                                        |
|-------------------------|---------|-------------------------------------------------------------|
| 호출 코드               | 15-0    | 요청 된 hypercall를 지정 합니다.                      |
| 빠름                    | 16      | Hypercall에서 레지스터 기반 호출 규칙을 사용 하는지 여부를 지정 합니다. 0 = 메모리 기반, 1 = 레지스터 기반 |
| 변수 헤더 크기    | 26-17   | 변수 헤더의 크기 (QWORDS)입니다.                   |
| RsvdZ                   | 31-27   | 0 이어야 합니다.                                                |
| 담당자 수               | 43-32   | 담당자의 총 수 (담당자 호출의 경우 0 이어야 합니다. |
| RsvdZ                   | 47-44   | 0 이어야 합니다.                                                |
| 담당자 시작 인덱스         | 59-48   | 시작 인덱스 (rep 호출의 경우 0 이어야 합니다.       |
| RsvdZ                   | 64-60   | 0 이어야 합니다.                                                |

Rep hypercall의 경우 담당자 수 필드는 총 담당자 수를 나타냅니다. Rep start index는 목록의 시작 부분을 기준으로 특정 반복을 나타냅니다 (0은 목록의 첫 번째 요소가 처리 됨을 나타냄). 따라서 rep count 값은 항상 rep 시작 인덱스 보다 커야 합니다.

Fast 플래그가 0 인 경우 hypercall 입력에 대 한 매핑을 등록 합니다.

| X64     | x86     | 제공된 정보                                        |
|---------|---------|-------------------------------------------------------------|
| RCX     | EDX: EAX | Hypercall 입력 값                                       |
| RDX     | EBX: ECX | 입력 매개 변수 GPA                                        |
| R8      | EDI: ESI | 출력 매개 변수 GPA                                       |

Hypercall 입력 값은 입력 및 출력 매개 변수를 가리키는 GPA 함께 레지스터에 전달 됩니다.

X64에서 등록 매핑은 호출자가 32 비트 (x86) 또는 64 비트 (x64) 모드에서 실행 되는지에 따라 달라 집니다. 하이퍼바이저는 EFER의 값을 기준으로 호출자의 모드를 결정 합니다. LMA 및 .CS. L. 이러한 플래그가 모두 설정 되 면 호출자는 64 비트 호출자로 간주 됩니다.

Fast 플래그가 1 인 경우 hypercall 입력에 대 한 매핑을 등록 합니다.

| X64     | x86     | 제공된 정보                                        |
|---------|---------|-------------------------------------------------------------|
| RCX     | EDX: EAX | Hypercall 입력 값                                       |
| RDX     | EBX: ECX | 입력 매개 변수                                             |
| R8      | EDI: ESI | 출력 매개 변수                                            |

Hypercall 입력 값은 입력 매개 변수와 함께 레지스터에 전달 됩니다.

### <a name="variable-sized-hypercall-input-headers"></a>가변 크기 Hypercall 입력 헤더

대부분의 hypercall 입력 헤더의 크기가 고정 되어 있습니다. 따라서 게스트에서 하이퍼바이저로 전달 되는 헤더 데이터의 양은 hypercall 코드로 암시적으로 지정 되므로 별도로 지정할 필요가 없습니다. 그러나 일부 hypercall에는 가변 크기의 헤더 데이터가 필요 합니다. 이러한 hypercall에는 일반적으로 고정 크기 입력 헤더와 가변 크기의 추가 헤더 입력이 있습니다.

가변 크기의 헤더는 고정 된 hypercall 입력과 유사 하며 8 바이트에 맞추고 크기는 8 바이트의 배수로 정렬 됩니다. 호출자는 입력 헤더로 제공 되는 데이터 양을 지정 해야 합니다. 이 크기는 hypercall 입력 값의 일부로 제공 됩니다 (위의 표에서 "변수 헤더 크기" 참조).

고정 헤더 크기가 암시적 이므로 전체 헤더 크기를 제공 하는 대신 입력 컨트롤에는 변수 부분만 제공 됩니다.

```
Variable Header Bytes = {Total Header Bytes - sizeof(Fixed Header)} rounded up to nearest multiple of 8

Variable HeaderSize = Variable Header Bytes / 8
```

가변 크기 입력 헤더를 허용 하는 것으로 명시적으로 문서화 되지 않은 hypercall에 대해 0이 아닌 변수 헤더 크기를 지정할 수 없습니다. 이 경우 hypercall는의 반환 코드를 생성 합니다 `HV_STATUS_INVALID_HYPERCALL_INPUT` .

모든 헤더 입력이 고정 크기 헤더에 완전히 부합 하는 가변 크기의 입력 헤더를 허용 하는 hypercall의 지정 된 호출을 수행할 수 있습니다. 이러한 경우 가변 크기의 입력 헤더는 크기가 0이 고 hypercall 입력의 해당 비트는 0으로 설정 되어야 합니다.

다른 모든 경우에는 변수 크기 입력 헤더를 수락 하는 hypercall는 호출 규칙에 대 한 고정 크기 입력 헤더 hypercall와 비슷합니다. 또한 변수 크기가 지정 된 헤더 hypercall에서 제공 의미 체계를 추가로 지원할 수 있습니다. 이러한 경우에는 헤더의 전체 크기에 고정 및 변수 부분이 모두 포함 된다는 점을 제외 하 고,이 경우에는 일반적으로 rep 요소가 헤더 뒤에 있습니다. 다른 모든 규칙은 동일 하 게 유지 됩니다. 예를 들어 첫 번째 rep 요소는 8 바이트로 정렬 되어야 합니다.

### <a name="xmm-fast-hypercall-input"></a>XMM Fast Hypercall 입력

X64 플랫폼에서 하이퍼바이저는 XMM fast hypercall 사용을 지원 합니다 .이를 통해 일부 hypercall는 입력 매개 변수가 두 개 이상 필요한 경우에도 fast hypercall 인터페이스의 향상 된 성능을 활용할 수 있습니다. XMM fast hypercall 인터페이스는 6 개의 XMM 레지스터를 사용 하 여 호출자가 최대 112 바이트 크기의 입력 매개 변수 블록을 전달할 수 있도록 합니다.

XMM fast hypercall 인터페이스의 가용성은 "하이퍼바이저 기능 식별" CPUID 리프 (0x40000003)을 통해 표시 됩니다.

- 비트 4: XMM 레지스터를 통해 hypercall 입력을 전달 하는 기능을 사용할 수 있습니다.

XMM fast 출력에 대 한 지원을 나타내는 별도의 플래그가 있습니다. 하이퍼바이저가 가용성을 나타내지 않는 경우이 인터페이스를 사용 하려고 하면 #UD 오류가 발생 합니다.

#### <a name="register-mapping-input-only"></a>매핑 등록 (입력 전용)

| X64     | x86     | 제공된 정보                                        |
|---------|---------|-------------------------------------------------------------|
| RCX     | EDX: EAX | Hypercall 입력 값                                       |
| RDX     | EBX: ECX | 입력 매개 변수 블록                                       |
| R8      | EDI: ESI | 입력 매개 변수 블록                                       |
| XMM0    | XMM0    | 입력 매개 변수 블록                                       |
| XMM1    | XMM1    | 입력 매개 변수 블록                                       |
| XMM2    | XMM2    | 입력 매개 변수 블록                                       |
| XMM3    | XMM3    | 입력 매개 변수 블록                                       |
| XMM4    | XMM4    | 입력 매개 변수 블록                                       |
| ~ XMM5    | ~ XMM5    | 입력 매개 변수 블록                                       |

Hypercall 입력 값은 입력 매개 변수와 함께 레지스터에 전달 됩니다. 등록 매핑은 호출자가 32 비트 (x86) 또는 64 비트 (x64) 모드에서 실행 되는지에 따라 달라 집니다. 하이퍼바이저는 EFER의 값을 기준으로 호출자의 모드를 결정 합니다. LMA 및 .CS. L. 이러한 플래그가 모두 설정 되 면 호출자는 64 비트 호출자로 간주 됩니다. 입력 매개 변수 블록이 112 바이트 보다 작은 경우 레지스터의 추가 바이트는 무시 됩니다.

## <a name="hypercall-outputs"></a>Hypercall 출력

모든 hypercall는 hypercall 결과 값 이라고 하는 64 비트 값을 반환 합니다. 다음과 같이 형식이 지정됩니다.

| 필드           | 비트  | 의견                                                                               |
|-----------------|-------|---------------------------------------------------------------------------------------|
| 결과          | 15-0  | `HV_STATUS` 성공 또는 실패를 나타내는 코드                                        |
| Rsvd            | 31-16 | 호출자는 이러한 비트의 값을 무시 해야 합니다.                                         |
| 완료 된 담당자  | 43-32 | 성공적으로 완료 된 담당자 수                                                 |
| RsvdZ           | 63-40 | 호출자는 이러한 비트의 값을 무시 해야 합니다.                                         |

Rep hypercall의 경우 담당자 완료 필드는 담당자 시작 인덱스를 기준으로 하지 않고 완료 된 총 담당자 수입니다. 예를 들어 호출자가 담당자를 시작 인덱스 5로 지정 하 고 rep 수가 10 인 경우 성공적으로 완료 되 면 대표 담당자 필드는 10을 표시 합니다.

Hypercall 결과 값은 레지스터에 다시 전달 됩니다. 등록 매핑은 호출자가 32 비트 (x86) 또는 64 비트 (x64) 모드에서 실행 되 고 있는지 여부에 따라 달라 집니다 (위 참조). Hypercall 출력에 대 한 등록 매핑은 다음과 같습니다.

| X64     | x86     | 제공된 정보                                        |
|---------|---------|-------------------------------------------------------------|
| RAX     | EDX: EAX | Hypercall 결과 값                                      |

### <a name="xmm-fast-hypercall-output"></a>XMM Fast Hypercall 출력

하이퍼바이저에서 XMM fast hypercall 입력을 지 원하는 방법과 유사 하 게 동일한 레지스터를 공유 하 여 출력을 반환할 수 있습니다. X64 플랫폼 에서만 지원 됩니다.

XMM 레지스터를 통해 출력을 반환 하는 기능은 "하이퍼바이저 기능 식별" CPUID 리프 (0x40000003)을 통해 표시 됩니다.

- 비트 15: XMM 레지스터를 통한 hypercall 출력 반환에 대 한 지원을 사용할 수 있습니다.

XMM 빠른 입력 지원을 나타내는 별도의 플래그가 있습니다. 하이퍼바이저가 가용성을 나타내지 않는 경우이 인터페이스를 사용 하려고 하면 #UD 오류가 발생 합니다.

#### <a name="register-mapping-input-and-output"></a>매핑 등록 (입력 및 출력)

입력 매개 변수를 전달 하는 데 사용 되지 않는 레지스터를 사용 하 여 출력을 반환할 수 있습니다. 즉, 입력 매개 변수 블록이 112 바이트 보다 작은 경우 (가장 가까운 16 바이트 맞춤 청크로 반올림) 나머지 레지스터는 hypercall output을 반환 합니다.

| X64     | 제공된 정보                                        |
|---------|-------------------------------------------------------------|
| RDX     | 입력 또는 출력 블록                                       |
| R8      | 입력 또는 출력 블록                                       |
| XMM0    | 입력 또는 출력 블록                                       |
| XMM1    | 입력 또는 출력 블록                                       |
| XMM2    | 입력 또는 출력 블록                                       |
| XMM3    | 입력 또는 출력 블록                                       |
| XMM4    | 입력 또는 출력 블록                                       |
| ~ XMM5    | 입력 또는 출력 블록                                       |

예를 들어 입력 매개 변수 블록의 크기가 20 바이트인 경우 하이퍼바이저는 다음 12 바이트를 무시 합니다. 나머지 80 바이트는 hypercall output (해당 하는 경우)을 포함 합니다.

## <a name="volatile-registers"></a>Volatile 레지스터

Hypercall는 다음과 같은 경우에만 지정 된 레지스터 값을 수정 합니다.

1. RAX (x64) 및 EDX: EAX (x86)는 항상 hypercall result 값 및 output 매개 변수 (있는 경우)로 덮어쓰여집니다.
1. Rep hypercall는 새 rep 시작 인덱스를 사용 하 여 RCX (x64) 및 EDX: EAX (x86)를 수정 합니다.
1. [HvCallSetVpRegisters](./hypercalls/HvCallSetVpRegisters.md) 는 해당 hypercall에서 지원 되는 모든 레지스터를 수정할 수 있습니다.
1. Fast hypercall 입력에 사용 되는 경우 RDX, R8 및 XMM0를 사용 하면 수정 되지 않은 상태로 유지 됩니다. 그러나 fast hypercall 출력에 사용 되는 레지스터는 RDX, R8 및 XMM0를 통한 ~ XMM5를 포함 하 여 수정할 수 있습니다. Hyper-v는 fast hypercall 출력에 대해서만 이러한 레지스터를 수정 합니다 .이는 x 64로 제한 됩니다.

## <a name="hypercall-restrictions"></a>Hypercall 제한 사항

사용자가 의도 한 기능을 수행 하기 위해 hypercall에 연결 된 제한이 있을 수 있습니다. 모든 제한이 충족 되지 않으면 hypercall는 적절 한 오류와 함께 종료 됩니다. 적용 되는 경우 다음 제한이 나열 됩니다.

- 호출 파티션에는 특정 권한이 있어야 합니다.
- 처리 중인 파티션이 특정 상태 (예: "활성") 여야 합니다.

## <a name="hypercall-status-codes"></a>Hypercall 상태 코드

각 hypercall는 여러 필드를 포함 하는 출력 값을 반환 하는 것으로 설명 됩니다. 상태 값 필드 (형식)는 `HV_STATUS` 호출의 성공 또는 실패 여부를 나타내는 데 사용 됩니다.

### <a name="output-parameter-validity-on-failed-hypercalls"></a>실패 한 Hypercall의 출력 매개 변수 유효성

명시적으로 지정 되지 않은 경우 hypercall가 실패 하면 (즉, hypercall 결과 값의 결과 필드에가 아닌 값이 포함 된 경우 `HV_STATUS_SUCCESS` ) 모든 출력 매개 변수의 콘텐츠는 비활성화 되며 호출자가 검사 하지 않아야 합니다. Hypercall가 성공 하는 경우에만 적절 한 모든 출력 매개 변수에 올바른 예상 결과가 포함 됩니다.

### <a name="ordering-of-error-conditions"></a>오류 조건 순서 지정

하이퍼바이저에서 오류 조건을 검색 하 여 보고 하는 순서는 정의 되지 않습니다. 즉, 여러 오류가 있는 경우 하이퍼바이저는 보고할 오류 조건을 선택 해야 합니다. 우선 순위는 더 높은 보안을 제공 하는 오류 코드, 즉 하이퍼바이저가 충분 한 권한이 없는 호출자에 게 정보를 노출 하지 않도록 하기 위한 것입니다. 예를 들어 상태 코드는 `HV_STATUS_ACCESS_DENIED` 권한에 따라 전적으로 일부 컨텍스트 또는 상태 정보를 표시 하는 기본 상태 코드입니다.

### <a name="common-hypercall-status-codes"></a>일반적인 Hypercall 상태 코드

일부 결과 코드는 모든 hypercall에 공통적 이므로 각 hypercall에 대해 개별적으로 문서화 되지 않습니다. 여기에는 다음이 포함됩니다.

| 상태 코드                        | 오류 조건                                                                       |
|------------------------------------|---------------------------------------------------------------------------------------|
| `HV_STATUS_SUCCESS`                | 호출에 성공 했습니다.                                                                   |
| `HV_STATUS_INVALID_HYPERCALL_CODE` | Hypercall 코드를 인식할 수 없습니다.                                                 |
| `HV_STATUS_INVALID_HYPERCALL_INPUT`| Rep 개수가 올바르지 않습니다. 예를 들어 0이 아닌 담당자 수가 비 담당자 호출에 전달 되거나 0 인 경우 담당자 개수가 rep 호출에 전달 됩니다.
|                                    | Rep 시작 인덱스는 rep count 보다 작지 않습니다.                                   |
|                                    | 지정 된 hypercall 입력 값의 예약 비트가 0이 아닙니다.                    |
| `HV_STATUS_INVALID_ALIGNMENT`      | 지정한 입력 또는 출력 GPA 포인터가 8 바이트로 정렬 되지 않았습니다.                  |
|                                    | 지정 된 입력 또는 출력 매개 변수는 범위 페이지를 나열 합니다.                            |
|                                    | 입력 또는 출력 GPA 포인터가 GPA 공간 범위 내에 있지 않습니다.            |

반환 코드는 `HV_STATUS_SUCCESS` 오류 조건이 검색 되지 않았음을 나타냅니다.

## <a name="reporting-the-guest-os-identity"></a>게스트 OS Id 보고

파티션에서 실행 되는 게스트 OS는 `HV_X64_MSR_GUEST_OS_ID` hypercall를 호출 하기 전에 해당 시그니처와 버전을 MSR ()에 기록 하 여 하이퍼바이저에서 자신을 식별 해야 합니다. 이 MSR은 파티션 전체로, 모든 가상 프로세서에서 공유 됩니다.

이 레지스터의 값은 처음에 0입니다. Hypercall 코드 페이지를 사용할 수 있으려면 먼저 게스트 OS ID MSR에 0이 아닌 값을 기록해 야 합니다 ( [Hypercall 인터페이스 설정](#establishing-the-hypercall-interface)참조). 이 레지스터가 이후에 0이 되 면 hypercall 코드 페이지를 사용할 수 없게 됩니다.

 ```c
#define HV_X64_MSR_GUEST_OS_ID 0x40000000
 ```

### <a name="guest-os-identity-for-proprietary-operating-systems"></a>독점 운영 체제에 대 한 게스트 OS Id

다음은이 MSR의 권장 인코딩입니다. 일부 필드는 일부 게스트 OSs에 적용 되지 않을 수 있습니다.

| 비트      | 필드           | Description                                                                 |
|-----------|-----------------|-----------------------------------------------------------------------------|
| 15:0      | 빌드 번호    | OS의 빌드 번호를 나타냅니다.                                        |
| 23:16     | 서비스 버전 | 서비스 버전을 나타냅니다 (예: "Service Pack" 번호).          |
| 31:24     | 부 버전   | 운영 체제의 부 버전을 나타냅니다.                                       |
| 39:32     | 주 버전   | OS의 주 버전을 나타냅니다.                                       |
| 47:40     | OS ID           | OS 변형을 나타냅니다. 인코딩은 공급 업체 마다 고유 합니다. Microsoft 운영 체제는 0 = Undefined, 1 = MS-DOS®, 2 = Windows® 3(sp3), 3 = Windows® 9x, 4 = Windows® NT (및 파생물), 5 = Windows® CE와 같이 인코딩됩니다.
| 62:48     | 공급업체 ID       | 게스트 OS 공급 업체를 나타냅니다. 값 0은 예약 되어 있습니다. 아래 공급 업체의 목록을 참조 하세요.
| 63        | OS 유형         | OS 유형을 나타냅니다. 값이 0 이면 소유 된 비공개 원본 OS를 나타냅니다. 값 1은 오픈 소스 OS를 나타냅니다.

공급 업체 값은 Microsoft에서 할당 합니다. 새 공급 업체를 요청 하려면 GitHub 가상화 문서 리포지토리 ()에서 문제를 해결 하세요 <https://aka.ms/VirtualizationDocumentationIssuesTLFS> .

| Vendor    | 값                                                                                |
|-----------|--------------------------------------------------------------------------------------|
| Microsoft | 0x0001                                                                               |
| HPE       | 0x0002                                                                               |
| LANCOM    | 0x0200                                                                               |

### <a name="guest-os-identity-msr-for-open-source-operating-systems"></a>오픈 소스 운영 체제에 대 한 게스트 OS Id MSR

다음 인코딩은이 사양에 맞는 오픈 소스 운영 체제 공급 업체에 대 한 지침으로 제공 됩니다. 오픈 소스 운영 체제는 다음 규칙을 적용 하는 것이 좋습니다.

| 비트      | 필드           | Description                                                                 |
|-----------|-----------------|-----------------------------------------------------------------------------|
| 15:0      | 빌드 번호    | 추가 정보                                                      |
| 47:16     | 버전         | 업스트림 커널 버전 정보입니다.                                        |
| 55:48     | OS ID           | 추가 공급 업체 정보                                               |
| 62:56     | OS 유형         | OS 유형 (예: Linux, FreeBSD 등). 아래 알려진 OS 유형 목록 참조      |
| 63        | 오픈 소스     | 값 1은 오픈 소스 OS를 나타냅니다.                                   |

OS 유형 값은 Microsoft에서 할당 합니다. 새 OS 유형을 요청 하려면 GitHub 가상화 문서 리포지토리 ()에서 문제를 해결 하세요 <https://aka.ms/VirtualizationDocumentationIssuesTLFS> .

| OS 유형   | 값                                                                                |
|-----------|--------------------------------------------------------------------------------------|
| Linux     | 0x1                                                                                  |
| FreeBSD   | 0x2                                                                                  |
| Xen       | 0x3                                                                                  |
| Illumos   | 0x4                                                                                  |

## <a name="establishing-the-hypercall-interface"></a>Hypercall 인터페이스 설정

Hypercall는 특수 opcode를 사용 하 여 호출 됩니다. 이 opcode는 가상화 구현 마다 다르므로 하이퍼바이저가 이러한 차이를 추상화 하는 데 필요 합니다. 특수 한 hypercall 페이지를 통해 수행 됩니다. 이 페이지는 하이퍼바이저에서 제공 되며 게스트의 GPA 공간 내에 표시 됩니다. 게스트 Hypercall MSR을 프로그래밍 하 여 페이지의 위치를 지정 해야 합니다.

 ```c
#define HV_X64_MSR_HYPERCALL 0x40000001
 ```

| 비트    | Description                                                                        | 특성      |
|---------|------------------------------------------------------------------------------------|-----------------|
| 63:12   | Hypercall GPFN-Hypercall 페이지의 게스트 실제 페이지 번호를 나타냅니다.    | 읽기/쓰기    |
| 11:2    | RsvdP. 비트는 읽기에서 무시 되 고 쓰기 시 유지 되어야 합니다.                    | 예약됨        |
| 1       | 처리용. MSR을 변경할 수 있는지 여부를 나타냅니다. 설정 되 면이 MSR이 잠기므로 hypercall 페이지의 재배치를 방지 합니다. 설정 되 면 시스템 다시 설정만 비트를 지울 수 있습니다.                                                    | 읽기/쓰기    |
| 0       | Hypercall 사용 페이지                                                              | 읽기/쓰기    |

Hypercall 페이지는 게스트의 GPA 공간 내의 어느 위치에 나 배치할 수 있지만 페이지에 맞춰야 합니다. 게스트가 hypercall 페이지를 GPA 공간 범위 밖으로 이동 하려고 하면 MSR이 작성 될 때 #GP 오류가 발생 합니다.

이 MSR은 파티션 전체 MSR입니다. 즉, 파티션의 모든 가상 프로세서에서 공유 됩니다. 한 가상 프로세서가 MSR에 성공적으로 작성 되 면 다른 가상 프로세서는 동일한 값을 읽습니다.

Hypercall 페이지를 사용 하도록 설정 하기 전에 게스트 OS는 개별 MSR (HV_X64_MSR_GUEST_OS_ID)에 버전 서명을 작성 하 여 해당 id를 보고 해야 합니다. 게스트 OS id가 지정 되지 않은 경우 hypercall를 사용 하도록 설정 하려고 하면 실패 합니다. 사용 비트는 값이 기록 된 경우에도 0으로 유지 됩니다. 또한 hypercall 페이지가 활성화 된 후 게스트 OS id가 0으로 선택 취소 되 면 사용할 수 없게 됩니다.

Hypercall 페이지는 GPA 공간에 "오버레이"로 표시 됩니다. 즉, GPA 범위에 매핑되는 다른 모든 항목을 다룹니다. 게스트는 해당 콘텐츠를 읽고 실행할 수 있습니다. Hypercall 페이지에 쓰려고 하면 보호 (#GP) 예외가 발생 합니다. Hypercall 페이지를 사용 하도록 설정한 후에는 hypercall를 호출 하면 페이지 시작에 대 한 호출이 포함 됩니다.

다음은 hypercall 페이지를 설정 하는 것과 관련 된 단계에 대 한 자세한 목록입니다.

1. 게스트는 CPUID 리프 1을 읽고 레지스터 ECX의 비트 31을 확인 하 여 하이퍼바이저가 있는지 여부를 확인 합니다.
2. 게스트는 CPUID 리프 0x40000000을 읽어 최대 하이퍼바이저 CPUID 리프 (등록 EAX에서 반환 됨) 및 CPUID 리프 0x40000001을 확인 하 여 인터페이스 서명 (등록 EAX에서 반환 됨)을 확인 합니다. 최대 리프 값이 0x40000005 이상 인지와 인터페이스 서명이 "Hv # 1"과 같은지 확인 합니다. 이 서명은, 및가 구현 됨을 의미 `HV_X64_MSR_GUEST_OS_ID` `HV_X64_MSR_HYPERCALL` `HV_X64_MSR_VP_INDEX` 합니다.
3. 게스트가 해당 레지스터가 0 인 경우 해당 OS id를 MSR에 기록 합니다 `HV_X64_MSR_GUEST_OS_ID` .
4. 게스트가 Hypercall MSR ()을 읽습니다 `HV_X64_MSR_HYPERCALL` .
5. 게스트가 Hypercall 사용 페이지 비트를 확인 합니다. 설정 되어 있으면 인터페이스가 이미 활성화 되어 있고 6 단계와 7 단계를 생략 해야 합니다.
6. 게스트가 GPA 공간 내에서 페이지를 찾습니다. 특히 RAM, MMIO 등에서 차지 하지 않는 페이지를 찾습니다. 페이지가 사용 중인 경우 게스트는 다른 용도로 기본 페이지를 사용 하지 않도록 해야 합니다.
7. 게스트가 6 단계의 GPA를 포함 하는 Hypercall MSR ()에 새 값을 쓰고 `HV_X64_MSR_HYPERCALL` 인터페이스를 사용 하도록 설정 Hypercall 페이지 비트를 설정 합니다.
8. 게스트는 hypercall 페이지 GPA에 대 한 실행 파일 VA 매핑을 만듭니다.
9. 게스트는 사용 가능한 하이퍼바이저 기능을 결정 하는 데 사용 되는 0000003입니다.
인터페이스를 설정 하 고 나면 게스트가 hypercall을 시작할 수 있습니다. 이렇게 하려면 hypercall 프로토콜 별로 레지스터를 채우고 hypercall 페이지의 시작 부분에 대 한 호출을 실행 합니다. 게스트는 hypercall 페이지에서 호출자에 게 반환 하는 데 가까운 반환 (0xC3)에 해당 하는 것으로 가정 합니다. 따라서 hypercall를 올바른 스택으로 호출 해야 합니다.

## <a name="extended-hypercall-interface"></a>확장 된 Hypercall 인터페이스

0x8000 위의 호출 코드를 사용 하는 hypercall를 확장 된 hypercall 라고 합니다. 확장 된 hypercall는 일반적인 hypercall와 동일한 호출 규칙을 사용 하 고 게스트 VM의 관점에서 동일 하 게 표시 됩니다. 확장 된 hypercall는 내부적으로 Hyper-v 하이퍼바이저 내에서 다르게 처리 됩니다.

확장 된 hypercall 기능은 [Hvextcallquerycapabilities](hypercalls/HvExtCallQueryCapabilities.md)를 사용 하 여 쿼리할 수 있습니다.