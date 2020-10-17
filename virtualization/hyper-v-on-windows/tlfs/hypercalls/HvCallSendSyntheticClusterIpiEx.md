---
title: HvCallSendSyntheticClusterIpiEx
description: HvCallSendSyntheticClusterIpiEx ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: e6f2f46892a1a56bf13520ed94b81f6fce0a217b
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121006"
---
# <a name="hvcallsendsyntheticclusteripiex"></a>HvCallSendSyntheticClusterIpiEx

このハイパーコールは、指定された仮想プロセッサセットに仮想固定割り込みを送信します。 NMIs はサポートされていません。 このバージョンは、可変サイズの VP セットを指定できるという点で、 [Hvcallsendsyntheの Clusteripi](HvCallSendSyntheticClusterIpi.md) とは異なります。

このハイパーコールの可用性を推定するには、次のチェックを使用する必要があります。

- ExProcessorMasks は、CPUID リーフ0x40000004 によって示される必要があります。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallSendSyntheticClusterIpiEx(
    _In_ UINT32 Vector,
    _In_ HV_INPUT_VTL TargetVtl,
    _In_ HV_VP_SET ProcessorSet
    );
 ```

## <a name="call-code"></a>呼び出しコード
`0x0015` 簡潔

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `Vector`                | 0          | 4        | アサートされたベクターを指定しました。 >= 0x10 と <= 0xFF の間である必要があります。  |
| `TargetVtl`             | 4          | 1        | ターゲット VTL                                |
| パディング                 | 5          | 3        |                                           |
| `ProcessorSet`          | 8          | 変数 | ターゲットにする VPs を表すプロセッサセットを指定します|

## <a name="see-also"></a>関連項目

[HV_VP_SET](../datatypes/HV_VP_SET.md)
