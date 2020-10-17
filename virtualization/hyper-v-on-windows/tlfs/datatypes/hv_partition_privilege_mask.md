---
title: HV_PARTITION_PRIVILEGE_MASK
description: HV_PARTITION_PRIVILEGE_MASK
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 2efabf3eedba77d9b3a51e8ee130ba3a0b975ccd
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121134"
---
# <a name="hv_partition_privilege_mask"></a>HV_PARTITION_PRIVILEGE_MASK

パーティションは、"ハイパーバイザー機能識別" CPUID リーフ (0x40000003) を使用して、その特権マスクを照会できます。

## <a name="syntax"></a>構文

```c
typedef struct
{
    // Access to virtual MSRs
    UINT64 AccessVpRunTimeReg:1;
    UINT64 AccessPartitionReferenceCounter:1;
    UINT64 AccessSynicRegs:1;
    UINT64 AccessSyntheticTimerRegs:1;
    UINT64 AccessIntrCtrlRegs:1;
    UINT64 AccessHypercallMsrs:1;
    UINT64 AccessVpIndex:1;
    UINT64 AccessResetReg:1;
    UINT64 AccessStatsReg:1;
    UINT64 AccessPartitionReferenceTsc:1;
    UINT64 AccessGuestIdleReg:1;
    UINT64 AccessFrequencyRegs:1;
    UINT64 AccessDebugRegs:1;
    UINT64 AccessReenlightenmentControls:1
    UINT64 Reserved1:18;

    // Access to hypercalls
    UINT64 CreatePartitions:1;
    UINT64 AccessPartitionId:1;
    UINT64 AccessMemoryPool:1;
    UINT64 Reserved:1;
    UINT64 PostMessages:1;
    UINT64 SignalEvents:1;
    UINT64 CreatePort:1;
    UINT64 ConnectPort:1;
    UINT64 AccessStats:1;
    UINT64 Reserved2:2;
    UINT64 Debugging:1;
    UINT64 CpuManagement:1;
    UINT64 Reserved:1
    UINT64 Reserved:1;
    UINT64 Reserved:1;
    UINT64 AccessVSM:1;
    UINT64 AccessVpRegisters:1;
    UINT64 Reserved:1;
    UINT64 Reserved:1;
    UINT64 EnableExtendedHypercalls:1;
    UINT64 StartVirtualProcessor:1;
    UINT64 Reserved3:10;
} HV_PARTITION_PRIVILEGE_MASK;
 ```

各特権は、一連の統合 MSRs またはハイパーコールコールへのアクセスを制御します。

| 特権フラグ                        | 説明                                       |
|---------------------------------------|-----------------------------------------------|
|`AccessVpRunTimeReg`                   | このパーティションは、統合 MSR HV_X64_MSR_VP_RUNTIME にアクセスできます。 |
|`AccessPartitionReferenceCounter`      | パーティションには、パーティション全体の参照カウント MSR、HV_X64_MSR_TIME_REF_COUNT へのアクセス権があります。 |
|`AccessSynicRegs`                      | このパーティションは、Synic に関連付けられている統合 MSRs にアクセスできます (HV_X64_MSR_SCONTROL から HV_X64_MSR_EOM と HV_X64_MSR_SINT0 HV_X64_MSR_SINT15)。|
|`AccessSyntheticTimerMsrs`             | このパーティションは、Synic (HV_X64_MSR_STIMER0_CONFIG HV_X64_MSR_STIMER3_COUNT) に関連付けられている統合 MSRs にアクセスできます。 |
|`AccessIntrCtrlRegs`                   | このパーティションは、APIC (HV_X64_MSR_EOI、HV_X64_MSR_ICR および HV_X64_MSR_TPR) に関連付けられている統合 MSRs にアクセスできます。 |
|`AccessHypercallMsrs`                  | パーティションは、ハイパーコールインターフェイス (HV_X64_MSR_GUEST_OS_ID および HV_X64_MSR_HYPERCALL) に関連する統合 MSRs にアクセスできます。 |
|`AccessVpIndex`                        | パーティションは、仮想プロセッサインデックスを返す合成 MSR にアクセスできます。 |
|`AccessResetReg`                       | このパーティションは、システムをリセットする合成 MSR にアクセスできます。 |
|`AccessStatsReg`                       | このパーティションは、ゲストが独自の統計ページをマップおよびマップ解除できる合成 MSRs にアクセスできます。 |
|`AccessPartitionReferenceTsc`          | パーティションは、参照 TSC にアクセスできます。 |
|`AccessGuestIdleReg`                   | パーティションは、ゲストがゲストのアイドル状態に入ることを許可する合成 MSR にアクセスできます。 |
|`AccessFrequencyRegs`                  | パーティションは、サポートされている場合、TSC と APIC の周波数を指定する合成 MSRs にアクセスできます。 |
|`AccessDebugRegs`                      | パーティションは、何らかの形式のゲストデバッグに使用される統合 MSRs にアクセスできます。 |
|`AccessReenlightenmentControls`        | パーティションは、再利用可能なコントロールにアクセスできます。 |
|`CreatePartitions`                     | パーティションは、ハイパーコール HvCallCreatePartition を呼び出すことができます。 また、パーティションは、子での操作に制限されている他のハイパーコールを作成することもできます。 |
|`AccessPartitionId`                    | パーティションは、ハイパーコール HvCallGetPartitionId を呼び出して、独自のパーティション ID を取得できます。 |
|`AccessMemoryPool`                     | パーティションは、ハイパーコールの HvCallDepositMemory、HvCallWithdrawMemory、HvCallGetMemoryBalance を呼び出すことができます。 |
|`PostMessages`                         | パーティションは、ハイパーコール [HvCallPostMessage](../hypercalls/HvCallPostMessage.md)を呼び出すことができます。 |
|`SignalEvents`                         | パーティションは、ハイパーコール [Hvcallsignalevent](../hypercalls/HvCallSignalEvent.md)を呼び出すことができます。 |
|`CreatePort`                           | パーティションは、ハイパーコール HvCallCreatePort を呼び出すことができます。  |
|`PostMessages`                         | パーティションは、ハイパーコール HvCallPostMessage を呼び出すことができます。 |
|`ConnectPort`                          | パーティションは、ハイパーコール HvCallConnectPort を呼び出すことができます。 |
|`AccessStats`                          | このパーティションは、ハイパーコール HvCallMapStatsPage と HvCallUnmapStatsPage を呼び出すことができます。 |
|`Debugging`                            | パーティションは、ハイパーコールの HvCallPostDebugData、HvCallRetrieveDebugData、Hvcallpostdebugdata を呼び出すことができます。 |
|`CpuManagement`                        | パーティションは、CPU 管理のためにさまざまなハイパーコールを呼び出すことができます。 |
|`AccessVSM`                            | このパーティションでは、 [VSM](../vsm.md)を使用できます。 |
|`AccessVpRegisters`                    | このパーティションは、ハイパーコール HvCallSetVpRegisters と HvCallGetVpRegisters を呼び出すことができます。 |
|`EnableExtendedHypercalls`             | パーティションでは、拡張された [ハイパーコールインターフェイス](../hypercall-interface.md#extended-hypercall-interface)を使用できます。 |
|`StartVirtualProcessor`                | パーティションでは、 [Hvcallstartvirtualprocessor](../hypercalls/HvCallStartVirtualProcessor.md) を使用して仮想プロセッサを初期化できます。 |
