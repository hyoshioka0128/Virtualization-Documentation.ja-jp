---
title: HvCallFlushGuestPhysicalAddressList
description: HvCallFlushGuestPhysicalAddressList ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 990b89e1ba422fa169c3334e69de0f55dd609061
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120818"
---
# <a name="hvcallflushguestphysicaladdresslist"></a>HvCallFlushGuestPhysicalAddressList

HvCallFlushGuestPhysicalAddressList ハイパーコールは、第2レベルのアドレス空間の一部内の GPA マッピングに対して、キャッシュされた GVA/L2 GPA を無効にします。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallFlushGuestPhysicalAddressList(
    _In_ HV_SPA AddressSpace,
    _In_ UINT64 Flags,
    _In_reads_(RangeCount) PHV_GPA_PAGE_RANGE GpaRangeList
    );
 ```

このハイパーコールは、入れ子になった仮想化がアクティブである場合にのみ使用できます。 仮想 TLB 無効化操作は、すべてのプロセッサに対して機能します。

この呼び出しにより、時間制御が呼び出し元に戻ったときに、すべてのフラッシュの観測可能な効果が発生したことが保証されます。
TLB が現在 "ロック" されている場合、呼び出し元の仮想プロセッサは中断されます。

この呼び出しでは、フラッシュする L2 GPA 範囲の一覧が取得されます。 各範囲には、第1の L2 GPA があります。 フラッシュはページの粒度を使用して実行されるため、L2 GPA の下位12ビットを使用して範囲の長さを定義できます。 これらのビットは、範囲内の追加ページ数 (最初のページを超えて) をエンコードします。 これにより、各エントリで 1 ~ 4096 ページの範囲をエンコードできます。

## <a name="call-code"></a>呼び出しコード

`0x00B0` 民主

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `AddressSpace`          | 0          | 8        | アドレス空間 ID (EPT PML4 table ポインター) を指定します。 |
| `Flags`                 | 8          | 8        | RsvdZ                                     |

## <a name="input-list-element"></a>入力リスト要素

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `GpaRange`              | 0          | 8        | GPA の範囲                                 |