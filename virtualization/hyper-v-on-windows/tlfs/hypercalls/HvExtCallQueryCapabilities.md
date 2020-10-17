---
title: HvExtCallQueryCapabilities
description: HvExtCallQueryCapabilities ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 5e1a896b22c5087d3cd9041224846b94557f5c10
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120986"
---
# <a name="hvextcallquerycapabilities"></a>HvExtCallQueryCapabilities

このハイパーコールは、拡張ハイパーコールの可用性を報告します。

このハイパーコールの可用性は、EnableExtendedHypercalls コールのパーティション権限フラグを使用して照会する必要があります。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvExtCallQueryCapabilities(
    _Out_ UINT64 Capabilities
    );
 ```

## <a name="call-code"></a>呼び出しコード

`0x8001` 簡潔

## <a name="output-parameters"></a>出力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `Capabilities`          | 0          | 8        | サポートされている拡張ハイパーコールのビットマスク                          |

サポートされている拡張ハイパーコールのビットマスクの形式は次のとおりです。

| ビット     | 拡張ハイパーコール                                          |
|---------|-------------------------------------------------------------|
| 0       | Hvextcallgetbootゼロ Edmemory                                |
| 1       | Hvextcallmemoryheの Int                                     |
| 2       | HvExtCallEpfSetup                                           |
| 3       | HvExtCallSchedulerAssistSetup                               |
| 4       | Hvextcallmemoryheheinender                                |
| 63-5    | 予約済み                                                    |
