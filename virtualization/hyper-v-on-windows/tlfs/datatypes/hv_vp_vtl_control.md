---
title: HV_VP_VTL_CONTROL
description: HV_VP_VTL_CONTROL
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 2d307b85e073cc0d8e29d45ef8fdf3f5371de05c
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121065"
---
# <a name="hv_vp_vtl_control"></a>HV_VP_VTL_CONTROL

ハイパーバイザーは、VP サポートページの一部を使用して、VTL0 より高い VTL で実行されるコードとの通信を容易にします。 各 VTL には独自の制御構造があります (VTL0 を除く)。

次の情報は、コントロールページを使用して伝達されます。

1. VTL エントリの理由。
2. VINA がアサートされていることを示すフラグ。
3. VTL に読み込まれるレジスタの値はを返します。

## <a name="syntax"></a>構文

```c
typedef enum
{
    // This reason is reserved and is not used.
    HvVtlEntryReserved = 0,

    // Indicates entry due to a VTL call from a lower VTL.
    HvVtlEntryVtlCall = 1,

    // Indicates entry due to an interrupt targeted to the VTL.
    HvVtlEntryInterrupt = 2
} HV_VTL_ENTRY_REASON;

typedef struct
{
    // The hypervisor updates the entry reason with an indication as to why
    // the VTL was entered on the virtual processor.
    HV_VTL_ENTRY_REASON EntryReason;

    // This flag determines whether the VINA interrupt line is asserted.
    union
    {
        UINT8 AsUINT8;
        struct
        {
            UINT8 VinaAsserted :1;
            UINT8 VinaReservedZ :7;
        };
    } VinaStatus;

    UINT8 ReservedZ00;
    UINT16 ReservedZ01;

    // A guest updates the VtlReturn* fields to provide the register values
    // to restore on VTL return. The specific register values that are
    // restored will vary based on whether the VTL is 32-bit or 64-bit.
    union
    {
        struct
        {
            UINT64 VtlReturnX64Rax;
            UINT64 VtlReturnX64Rcx;
        };

        struct
        {
            UINT32 VtlReturnX86Eax;
            UINT32 VtlReturnX86Ecx;
            UINT32 VtlReturnX86Edx;
            UINT32 ReservedZ1;
        };
    };
} HV_VP_VTL_CONTROL;
 ```

## <a name="see-also"></a>関連項目

[HV_VP_ASSIST_PAGE](HV_VP_ASSIST_PAGE.md)
