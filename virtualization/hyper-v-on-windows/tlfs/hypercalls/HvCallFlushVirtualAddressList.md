---
title: HvFlushVirtualAddressList
description: HvFlushVirtualAddressList ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: af8f736dbe6926c5712a26cf6978ae0fe020b89d
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120821"
---
# <a name="hvflushvirtualaddresslist"></a>HvFlushVirtualAddressList

HvCallFlushVirtualAddressList ハイパーコールは、指定されたアドレス空間に属する仮想 TLB の一部を無効にします。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallFlushVirtualAddressList(
    _In_ HV_ADDRESS_SPACE_ID AddressSpace,
    _In_ HV_FLUSH_FLAGS Flags,
    _In_ UINT64 ProcessorMask,
    _Inout_ PUINT32 GvaCount,
    _In_reads_(GvaCount) PCHV_GVA GvaRangeList
    );
 ```

仮想 TLB 無効化操作は、1つ以上のプロセッサに対して動作します。

フラッシュする必要があるプロセッサに関する知識がゲストにある場合は、プロセッサマスクを指定できます。 マスク内の各ビットは、仮想プロセッサインデックスに対応します。 たとえば、0x0000000000000051 というマスクは、ハイパーバイザーが仮想プロセッサ0、4、および6の TLB のみをフラッシュする必要があることを示します。

次のフラグを使用して、フラッシュの動作を変更できます。

- HV_FLUSH_ALL_PROCESSORS は、操作をパーティション内のすべての仮想プロセッサに適用する必要があることを示します。 このフラグが設定されている場合、ProcessorMask パラメーターは無視されます。
- HV_FLUSH_ALL_VIRTUAL_ADDRESS_SPACES は、すべての仮想アドレス空間に操作を適用する必要があることを示します。 このフラグが設定されている場合、AddressSpace パラメーターは無視されます。
- HV_FLUSH_NON_GLOBAL_MAPPINGS_ONLY はこの呼び出しには意味がなく、無効なオプションとして扱われます。

他のすべてのフラグは予約されており、0に設定する必要があります。

この呼び出しは、GVA 範囲のリストを受け取ります。 各範囲には基本 GVA があります。 フラッシュはページの粒度を使用して実行されるため、GVA の下位12ビットを使用して範囲の長さを定義できます。 これらのビットは、範囲内の追加ページ数 (最初のページを超えて) をエンコードします。 これにより、各エントリで 1 ~ 4096 ページの範囲をエンコードできます。

"大きいページ" のマッピング (2 MB または 4 mb) 内にある GVA は、仮想 TLB から大きなページ全体をフラッシュします。

この呼び出しにより、時間制御が呼び出し元に戻ったときに、指定された仮想プロセッサ上のすべてのフラッシュの観察可能な効果が発生したことが保証されます。

無効な GVAs (パーティションの GVAS 領域の末尾を越えたアドレスを指定するもの) は無視されます。

ターゲット仮想プロセッサの TLB にフラッシュが必要で、その仮想プロセッサが損なわれる TLB にフラッシュされる場合、呼び出し元の仮想プロセッサは中断されます。 TLB フラッシュが禁止されている場合、仮想プロセッサは "unsuspended" で、ハイパーコールは再発行されます。

## <a name="call-code"></a>呼び出しコード
`0x0003` 民主

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `AddressSpace`          | 0          | 8        | アドレス空間 ID (CR3 値) を指定します。 |
| `Flags`                 | 8          | 8        | フラッシュの操作を変更するフラグビットのセット。 |
| `ProcessorMask`         | 16         | 8        | フラッシュ操作によって影響を受けるプロセッサを示すプロセッサマスク。 |

## <a name="input-list-element"></a>入力リスト要素

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `GvaRange`              | 0          | 8        | GVA の範囲                                 |
