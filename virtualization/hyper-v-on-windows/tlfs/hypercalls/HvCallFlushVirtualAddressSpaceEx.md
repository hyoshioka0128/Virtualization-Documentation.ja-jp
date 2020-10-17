---
title: Hvcallflushvirtualaddressspace Ex
description: Hvcallflushvirtualaddressspace Ex ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: cb2fed3873df1a768a2098e7f69a3896a33e004c
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120730"
---
# <a name="hvcallflushvirtualaddressspaceex"></a>Hvcallflushvirtualaddressspace Ex

Hvcallflushvirtualaddressspace Ex ハイパーコールは [HvCallFlushVirtualAddressSpace](HvCallFlushVirtualAddressSpace.md)に似ていますが、可変サイズのスパース VP を入力として設定できます。

このハイパーコールの可用性を推定するには、次のチェックを使用する必要があります。

- ExProcessorMasks は、CPUID リーフ0x40000004 によって示される必要があります。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallFlushVirtualAddressSpaceEx(
    _In_ HV_ADDRESS_SPACE_ID AddressSpace,
    _In_ HV_FLUSH_FLAGS Flags,
    _In_ HV_VP_SET ProcessorSet
    );
 ```

## <a name="call-code"></a>呼び出しコード

`0x0013` 簡潔

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `AddressSpace`          | 0          | 8        | アドレス空間 ID (CR3 値) を指定します。 |
| `Flags`                 | 8          | 8        | フラッシュの操作を変更するフラグビットのセット。 |
| `ProcessorSet`          | 16         | 変数 | フラッシュ操作によって影響を受けるプロセッサを示すプロセッサセット。 |

## <a name="see-also"></a>関連項目

[HV_VP_SET](../datatypes/HV_VP_SET.md)
