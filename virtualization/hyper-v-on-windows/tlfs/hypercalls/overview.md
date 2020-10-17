---
title: ハイパーコールリファレンス
description: サポートされているハイパーコールの一覧
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: d40fc621374d7c3dacf616c01f1ea1efb3f08364
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120936"
---
# <a name="hypercall-reference"></a>ハイパーコールリファレンス

次の表に、サポートされているハイパーコールコードの一覧を示します。

| 呼び出しコード | Type    | ハイパーコール                                                                           |
|-----------|---------|-------------------------------------------------------------------------------------|
| 0x0001    | シンプル  | HvCallSwitchVirtualAddressSpace                                                     |
| 0x0002    | シンプル  | [HvCallFlushVirtualAddressSpace](HvCallFlushVirtualAddressSpace.md)                 |
| 0x0003    | 民主     | [HvCallFlushVirtualAddressList](HvCallFlushVirtualAddressList.md)                   |
| 0x0008    | シンプル  | [HvCallNotifyLongSpinWait](HvCallNotifyLongSpinWait.md)                             |
| 0x000b    | シンプル  | [Hvcallsendsyntheの Clusteripi](HvCallSendSyntheticClusterIpi.md)                   |
| 0x000c    | 民主     | [HvCallModifyVtlProtectionMask](HvCallModifyVtlProtectionMask.md)                   |
| 0x000d    | シンプル  | [HvCallEnablePartitionVtl](HvCallEnablePartitionVtl.md)                             |
| 0x000f    | シンプル  | [HvCallEnableVpVtl](HvCallEnableVpVtl.md)                                           |
| 0x0011    | シンプル  | [HvCallVtlCall](HvCallVtlCall.md)                                                   |
| 0x0012    | シンプル  | [HvCallVtlReturn](HvCallVtlReturn.md)                                               |
| 0x0013    | シンプル  | [Hvcallflushvirtualaddressspace Ex](HvCallFlushVirtualAddressSpaceEx.md)             |
| 0x0014    | 民主     | [HvCallFlushVirtualAddressListEx](HvCallFlushVirtualAddressListEx.md)               |
| 0x0015    | シンプル  | [HvCallSendSyntheticClusterIpiEx](HvCallSendSyntheticClusterIpiEx.md)               |
| 0x0050    | 民主     | [HvCallGetVpRegisters](HvCallGetVpRegisters.md)                                     |
| 0x0051    | 民主     | [HvCallSetVpRegisters](HvCallSetVpRegisters.md)                                     |
| 0x005C    | シンプル  | [HvCallPostMessage](HvCallPostMessage.md)                                           |
| 0x005D    | シンプル  | [HvCallSignalEvent](HvCallSignalEvent.md)                                           |
| 0x007e    | シンプル  | [HvCallRetargetDeviceInterrupt](HvCallRetargetDeviceInterrupt.md)                   |
| 0x0099    | シンプル  | [HvCallStartVirtualProcessor](HvCallStartVirtualProcessor.md)                       |
| 0x009A    | 民主     | [HvCallGetVpIndexFromApicId](HvCallGetVpIndexFromApicId.md)                         |
| 0x00AF    | シンプル  | [HvCallFlushGuestPhysicalAddressSpace](HvCallFlushGuestPhysicalAddressSpace.md)     |
| 0x00B0    | 民主     | [HvCallFlushGuestPhysicalAddressList](HvCallFlushGuestPhysicalAddressList.md)       |

次の表に、サポートされている拡張ハイパーコールを呼び出しコード別に示します。

| 呼び出しコード | Type    | ハイパーコール                                                                           |
|-----------|---------|-------------------------------------------------------------------------------------|
| 0x8001    | シンプル  | [HvExtCallQueryCapabilities](HvExtCallQueryCapabilities.md)                         |
| 0x8002    | シンプル  | Hvextcallgetbootゼロ Edmemory                                                        |
| 0x8003    | シンプル  | Hvextcallmemoryheの Int                                                             |
| 0x8004    | シンプル  | HvExtCallEpfSetup                                                                   |
| 0x8006    | シンプル  | Hvextcallmemoryheheinender                                                        |
