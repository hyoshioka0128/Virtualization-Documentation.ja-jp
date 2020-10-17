---
title: Hvcallsendsyntheの Clusteripi
description: Hvcallsendsyntheハイパーコール Clusteripi
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 21d7a7f5f43123b984e64d76008978e152e4a8a6
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120746"
---
# <a name="hvcallsendsyntheticclusteripi"></a>Hvcallsendsyntheの Clusteripi

このハイパーコールは、指定された仮想プロセッサセットに仮想固定割り込みを送信します。 NMIs はサポートされていません。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallSendSyntheticClusterIpi(
    _In_ UINT32 Vector,
    _In_ HV_INPUT_VTL TargetVtl,
    _In_ UINT64 ProcessorMask
    );
 ```

## <a name="call-code"></a>呼び出しコード
`0x000b` 簡潔

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `Vector`                | 0          | 4        | アサートされたベクターを指定しました。 >= 0x10 と <= 0xFF の間である必要があります。  |
| `TargetVtl`             | 4          | 1        | ターゲット VTL                                |
| パディング                 | 5          | 3        |                                           |
| `ProcessorMask`         | 8          | 1        | ターゲットにする VPs を表すマスクを指定します|
