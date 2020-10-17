---
title: HvCallStartVirtualProcessor
description: HvCallStartVirtualProcessor ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 262ba48156d7b73d5a7aa3f7b72a12aacb09a2a0
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120996"
---
# <a name="hvcallstartvirtualprocessor"></a>HvCallStartVirtualProcessor

HvCallStartVirtualProcessor は、仮想プロセッサを起動するための対応メソッドです。 これは、従来の INIT ベースのメソッドと機能的には同じですが、VP は望ましいレジスタ状態で開始できます。

これは、ゼロ以外の VTL で VP を開始する唯一の方法です。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallStartVirtualProcessor(
    _In_ HV_PARTITION_ID PartitionId,
    _In_ HV_VP_INDEX VpIndex,
    _In_ HV_VTL TargetVtl,
    _In_ HV_INITIAL_VP_CONTEXT VpContext
    );
 ```

## <a name="call-code"></a>呼び出しコード

`0x0099` 簡潔

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `PartitionId`           | 0          | 8        | Partition                                 |
| `VpIndex`               | 8          | 4        | VP インデックスを開始します。 APIC ID から VP インデックスを取得するには、HvGetVpIndexFromApicId を使用します。 |
| `TargetVtl`             | 12         | 1        | ターゲット VTL                                |
| `VpContext`             | 16         | 224      | VP を開始する最初のコンテキストを指定します。 |

## <a name="return-values"></a>戻り値

| status code                         | エラー条件                                       |
|-------------------------------------|-------------------------------------------------------|
| `HV_STATUS_ACCESS_DENIED`           | アクセスが拒否されました                                         |
| `HV_STATUS_INVALID_PARTITION_ID`    | 指定されたパーティション ID は無効です。                |
| `HV_STATUS_INVALID_VP_INDEX`        | HV_VP_INDEX によって指定された仮想プロセッサが無効です。 |
| `HV_STATUS_INVALID_REGISTER_VALUE`  | 指定されたレジスタ値が無効です。               |
| `HV_STATUS_INVALID_VP_STATE`        | バーチャルプロセッサは、指定された操作のパフォーマンスに対して正しい状態ではありません。 |
| `HV_STATUS_INVALID_PARTITION_STATE` | 指定されたパーティションは "アクティブ" 状態ではありません。 |
| `HV_STATUS_INVALID_VTL_STATE`       | 要求された操作と、VTL の状態が競合しています。 |

## <a name="see-also"></a>関連項目

[HV_INITIAL_VP_CONTEXT](../datatypes/HV_INITIAL_VP_CONTEXT.md)