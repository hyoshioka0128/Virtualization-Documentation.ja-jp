---
title: HV_DEVICE_INTERRUPT_TARGET
description: HV_DEVICE_INTERRUPT_TARGET
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 44d31155b667a1e1f0114720150717318ab52451
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120911"
---
# <a name="hv_device_interrupt_target"></a>HV_DEVICE_INTERRUPT_TARGET

## <a name="syntax"></a>構文

 ```c

#define HV_DEVICE_INTERRUPT_TARGET_MULTICAST 1
#define HV_DEVICE_INTERRUPT_TARGET_PROCESSOR_SET 2

typedef struct
{
    HV_INTERRUPT_VECTOR Vector;
    UINT32 Flags;
    union
    {
        UINT64 ProcessorMask;
        UINT64 ProcessorSet[];
    };
} HV_DEVICE_INTERRUPT_TARGET;
 ```

"Flags" は、割り込みターゲットに対して省略可能なフラグを提供します。 "マルチキャスト" は、割り込みがターゲットセット内のすべてのプロセッサに送信されることを示します。 既定では、割り込みはターゲットセット内の任意の1つのプロセッサに送信されます。
