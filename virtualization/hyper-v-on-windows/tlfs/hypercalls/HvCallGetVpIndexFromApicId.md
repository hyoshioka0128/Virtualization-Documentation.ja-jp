---
title: HvCallGetVpIndexFromApicId
description: HvCallGetVpIndexFromApicId ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: d7dc0d9314e1a51a693ca041039e0d6386f24c0f
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120749"
---
# <a name="hvcallgetvpindexfromapicid"></a>HvCallGetVpIndexFromApicId

HvCallGetVpIndexFromApicId を指定すると、呼び出し元は、指定された APID ID を持つ VP の VP インデックスを取得できます。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallGetVpIndexFromApicId(
    _In_ HV_PARTITION_ID PartitionId,
    _In_ HV_VTL TargetVtl,
    _Inout_ PUINT32 ApicIdCoount,
    _In_reads_(ApicIdCount) PHV_APIC_ID ApicIdList,
    _Out_writes(ApicIdCount) PHV_VP_INDEX VpIndexList
    );

 ```

## <a name="call-code"></a>呼び出しコード

`0x009A` 民主

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `PartitionId`           | 0          | 8        | Partition                                 |
| `TargetVtl`             | 8          | 1        | ターゲット VTL                                |
| パディング                 | 9          | 7        |                                           |

## <a name="input-list-element"></a>入力リスト要素

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `ApicId`                | 0          | 4        | VP の APIC ID                         |
| パディング                 | 4          | 4        |                                           |

## <a name="output-list-element"></a>出力リスト要素

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `VpIndex`               | 0          | 4        | 指定された APIC ID を持つ VP のインデックス|
| パディング                 | 4          | 4        |                                           |

## <a name="return-values"></a>戻り値

| status code                         | エラー条件                                       |
|-------------------------------------|-------------------------------------------------------|
| `HV_STATUS_ACCESS_DENIED`           | アクセスが拒否されました                                         |
| `HV_STATUS_INVALID_PARAMETER`       | 無効なパラメーターが指定されました                    |
| `HV_STATUS_INVALID_PARTITION_ID`    | 指定されたパーティション ID は無効です。                |
| `HV_STATUS_INVALID_REGISTER_VALUE`  | 指定されたレジスタ値が無効です。               |
| `HV_STATUS_INVALID_VP_STATE`        | バーチャルプロセッサは、指定された操作のパフォーマンスに対して正しい状態ではありません。 |
| `HV_STATUS_INVALID_PARTITION_STATE` | 指定されたパーティションは "アクティブ" 状態ではありません。 |
| `HV_STATUS_INVALID_VTL_STATE`       | 要求された操作と、VTL の状態が競合しています。 |