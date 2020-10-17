---
title: HvCallEnableVpVtl
description: HvCallEnableVpVtl ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 0be265f26c7c0e3ee844505eee4e8743ec01faaf
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121209"
---
# <a name="hvcallenablevpvtl"></a>HvCallEnableVpVtl

HvCallEnableVpVtl を使用すると、1つの VP で VTL を実行できます。 このハイパーコールは、HvCallEnablePartitionVtl と共に使用して、VTL を有効にし、使用する必要があります。 VP で VTL を有効にするには、まずそのパーティションに対して有効にする必要があります。 この呼び出しでは、アクティブな VTL は変更されません。

## <a name="interface"></a>インターフェイス

 ```c

HV_STATUS
HvEnableVpVtl(
    _In_ HV_PARTITION_ID TargetPartitionId,
    _In_ HV_VP_INDEX VpIndex,
    _In_ HV_VTL TargetVtl,
    _In_ HV_INITIAL_VP_CONTEXT VpVtlContext
    );
 ```

### <a name="restrictions"></a>制限

一般に、VTL は、より高い VTL によってのみ有効にすることができます。 このルールには1つの例外があります。パーティションに対して有効な最大の VTL では、より高いターゲットの VTL を実現できます。

副社長でターゲットの VTL が有効になると、その他のすべての呼び出しで、VTL を有効にする必要があります。
このハイパーコールは、VP に対して既に有効になっている VTL を有効にするために呼び出されると失敗します。

## <a name="call-code"></a>呼び出しコード
`0x000F` 簡潔

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `TargetPartitionId`     | 0          | 8        | この要求の対象となるパーティションのパーティション ID を指定します。 |
| `VpIndex`               | 8          | 4        | VTL を有効にする仮想プロセッサのインデックスを指定します。 |
| `TargetVtl`             | 12         | 1        | このハイパーコールによって有効になる VTL を指定します。 |
| RsvdZ                   | 13         | 3        |                                           |
| `VpVtlContext`          | 16         | 224      | ターゲット VTL への最初のエントリで VP を開始する最初のコンテキストを指定します。 |

## <a name="see-also"></a>関連項目

[HV_INITIAL_VP_CONTEXT](../datatypes/HV_INITIAL_VP_CONTEXT.md)