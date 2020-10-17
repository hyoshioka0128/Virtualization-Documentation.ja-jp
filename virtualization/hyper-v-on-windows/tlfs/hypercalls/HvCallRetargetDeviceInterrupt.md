---
title: HvCallRetargetDeviceInterrupt
description: HvCallRetargetDeviceInterrupt ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: df12f74a0eb34947e20afb61264c2434689b052f
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120725"
---
# <a name="hvcallretargetdeviceinterrupt"></a>HvCallRetargetDeviceInterrupt

このハイパーコールは、デバイスの割り込みを対象します。これは、ゲスト内の Irq を再調整する場合に便利です。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvRetargetDeviceInterrupt(
    _In_ HV_PARTITION_ID PartitionId,
    _In_ UINT64 DeviceId,
    _In_ HV_INTERRUPT_ENTRY InterruptEntry,
    _In_ UINT64 Reserved,
    _In_ HV_DEVICE_INTERRUPT_TARGET InterruptTarget
    );
 ```

## <a name="call-code"></a>呼び出しコード
`0x007e` 簡潔

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `PartitionId`           | 0          | 8        | パーティション Id (HV_PARTITION_SELF 可能)   |
| `DeviceId`              | 8          | 6        | ホストによって割り当てられる一意の (ゲスト内の) 論理デバイス ID を提供します。   |
| RsvdZ                   | 32         | 8        | 予約済み                                  |
| `InterruptEntry`        | 16         | 16       | は、割り込みを識別する MSI アドレスとデータを提供します。 |
| `InterruptTarget`       | 40         | 16       | ターゲットにする VPs を表すマスクを指定します|

## <a name="see-also"></a>関連項目

[HV_INTERRUPT_ENTRY](../datatypes/HV_INTERRUPT_ENTRY.md)

[HV_DEVICE_INTERRUPT_TARGET](../datatypes/HV_DEVICE_INTERRUPT_TARGET.md)
