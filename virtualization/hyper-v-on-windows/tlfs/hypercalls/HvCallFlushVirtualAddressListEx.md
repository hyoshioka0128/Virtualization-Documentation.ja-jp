---
title: HvFlushVirtualAddressListEx
description: HvFlushVirtualAddressListEx ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 60d56bdc4735c9b16534d3c7ad813e69734254af
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120810"
---
# <a name="hvflushvirtualaddresslistex"></a>HvFlushVirtualAddressListEx

HvFlushVirtualAddressListEx ハイパーコールは [Hvcallflushvirtualaddresslist](HvCallFlushVirtualAddressList.md)に似ていますが、可変サイズのスパース VP を入力として設定できます。
このハイパーコールの可用性を推定するには、次のチェックを使用する必要があります。

- ExProcessorMasks は、CPUID リーフ0x40000004 によって示される必要があります。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallFlushVirtualAddressListEx(
    _In_ HV_ADDRESS_SPACE_ID AddressSpace,
    _In_ HV_FLUSH_FLAGS Flags,
    _In_ HV_VP_SET ProcessorSet,,
    _Inout_ PUINT32 GvaCount,
    _In_reads_(GvaCount) PCHV_GVA GvaRangeList
    );
 ```

## <a name="call-code"></a>呼び出しコード
`0x0014` 民主

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `AddressSpace`          | 0          | 8        | アドレス空間 ID (CR3 値) を指定します。 |
| `Flags`                 | 8          | 8        | フラッシュの操作を変更するフラグビットのセット。 |
| `ProcessorSet`          | 16         | 変数 | フラッシュ操作によって影響を受けるプロセッサを示すプロセッサセット。 |

## <a name="input-list-element"></a>入力リスト要素

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `GvaRange`              | 0          | 8        | GVA の範囲                                 |

## <a name="see-also"></a>関連項目

[HV_VP_SET](../datatypes/HV_VP_SET.md)
