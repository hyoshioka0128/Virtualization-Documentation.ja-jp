---
title: HV_NESTED_ENLIGHTENMENTS_CONTROL
description: HV_NESTED_ENLIGHTENMENTS_CONTROL
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 71e4871aaa3cd50fac9e4b5e212ca029b180f157
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121142"
---
# <a name="hv_nested_enlightenments_control"></a>HV_NESTED_ENLIGHTENMENTS_CONTROL

ハイパーバイザーが親ハイパーバイザーに対して、入れ子になった啓蒙特権を現在の入れ子になったゲストコンテキストに付与することを許可する制御構造。

## <a name="syntax"></a>構文

```c
typedef struct
{
    union {
        UINT32 AsUINT32;
        struct
        {
            UINT32 DirectHypercall:1;
            UINT32 VirtualizationException:1;
            UINT32 Reserved:30;
        };
    } Features;

    union
    {
        UINT32 AsUINT32;
        struct
        {
            UINT32 InterPartitionCommunication:1;
            UINT32 Reserved:31;
        };
     } HypercallControls;
} HV_NESTED_ENLIGHTENMENTS_CONTROL, *PHV_NESTED_ENLIGHTENMENTS_CONTROL;
 ```

## <a name="see-also"></a>関連項目

[HV_VP_ASSIST_PAGE](HV_VP_ASSIST_PAGE.md)