---
title: HV_VP_SET
description: HV_VP_SET
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: af327161049650a95e39e9a658896d8e718cbd84
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121071"
---
# <a name="hv_vp_set"></a>HV_VP_SET

가상 프로세서 집합은 가상 프로세서의 컬렉션을 나타내며 일부 hypercall의 입력으로 사용할 수 있습니다.

## <a name="syntax"></a>구문

```c
typedef struct
{
    UINT64 Format;
    UINT64 ValidBanksMask;
    UINT64 BankContents[];
} HV_VP_SET;
 ```

프로세서 집합에는 형식 필드에 지정 된 두 가지 모드가 있습니다. "1" 형식의 프로세서 집합은 지정 된 파티션에 대 한 모든 가상 프로세서를 나타냅니다. "0" 형식의 프로세서 집합은 가상 프로세서의 스파스 집합을 설명 합니다.

| 서식 값  | 동작 설정                                                |
|---------------|-------------------------------------------------------------|
| 0             | VPs의 스파스 하위 집합                                      |
| 1             | 모든 VPs (파티션에 속함)                          |

### <a name="sparse-virtual-processor-set"></a>스파스 가상 프로세서 집합

다음 섹션에서는 가상 프로세서의 스파스 집합을 생성 하는 방법을 설명 합니다.

총 가상 프로세서 집합은 "bank" 라는 64 청크로 분할 됩니다. 예를 들어 프로세서 0-63는 뱅크 0, 64-127은 bank 1 등에 있습니다.

개별 프로세서를 설명 하기 위해 해당 뱅크는 ValidBanksMask로 지정 됩니다. ValidBanksMask의 각 비트는 특정 뱅크를 나타냅니다.

```
bank = VPindex / 64
```
ValidBanksMask로 설정 된 모든 비트의 경우 BanksContents 배열에 요소가 있어야 합니다. 이 요소는 은행 자체를 설명 하는 마스크입니다.

ValidBankMask의 비트가 0 이면 BanksContents에 해당 하는 요소가 없습니다. 또한 ValidBankMask의 비트 1의 경우 BanksContents의 해당 요소에 대 한 유효한 상태는 모두 0이 될 수 있습니다. 즉,이 은행에 지정 된 프로세서가 없음을 의미 합니다.

#### <a name="processor-set-example"></a>프로세서 집합 예제

파티션에 200 VPs가 있고 {0, 5130} 집합을 지정 하려고 한다고 가정 합니다.

첫째,이 형식은 스파스 집합 이므로 0입니다. 그런 다음 해당 뱅크 (그리고 따라서 ValidBanksMask의 설정 비트)는 {0, 0, 2}입니다. 따라서 ValidBanksMask는 0x05입니다.

Bank 0은 비트 0과 5를 설정 하 여 해당 은행 내의 VPs를 지정 합니다. 따라서 BankContents mask의 해당 요소는 0x21입니다.

ValidBanksMask에 비트 1이 설정 되어 있지 않으므로 BankContents에 해당 하는 요소가 없습니다. Bank 2는 VP 인덱스 128-191를 나타냅니다. 인덱스 130을 설명 하기 위해 해당 마스크의 비트 2가 설정 됩니다. 따라서 BankContents은 {0x21, 0x04}입니다.