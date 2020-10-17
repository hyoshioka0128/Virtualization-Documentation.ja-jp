---
title: HvCallFlushVirtualAddressSpace
description: HvCallFlushVirtualAddressSpace ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: ce0c0a4f0f5879fb43cf173c51882c63e733c747
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120805"
---
# <a name="hvcallflushvirtualaddressspace"></a>HvCallFlushVirtualAddressSpace

HvCallFlushVirtualAddressSpace ハイパーコールは、指定されたアドレス空間に属するすべての仮想 TLB エントリを無効にします。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallFlushVirtualAddressSpace(
    _In_ HV_ADDRESS_SPACE_ID AddressSpace,
    _In_ HV_FLUSH_FLAGS Flags,
    _In_ UINT64 ProcessorMask
    );
 ```

仮想 TLB 無効化操作は、1つ以上のプロセッサに対して動作します。

フラッシュする必要があるプロセッサに関する知識がゲストにある場合は、プロセッサマスクを指定できます。 マスク内の各ビットは、仮想プロセッサインデックスに対応します。 たとえば、0x0000000000000051 というマスクは、ハイパーバイザーが仮想プロセッサ0、4、および6の TLB のみをフラッシュする必要があることを示します。 仮想プロセッサは、MSR HV_X64_MSR_VP_INDEX から読み取ることによってインデックスを決定できます。

次のフラグを使用して、フラッシュの動作を変更できます。

- HV_FLUSH_ALL_PROCESSORS は、操作をパーティション内のすべての仮想プロセッサに適用する必要があることを示します。 このフラグが設定されている場合、ProcessorMask パラメーターは無視されます。
- HV_FLUSH_ALL_VIRTUAL_ADDRESS_SPACES は、すべての仮想アドレス空間に操作を適用する必要があることを示します。 このフラグが設定されている場合、AddressSpace パラメーターは無視されます。
- HV_FLUSH_NON_GLOBAL_MAPPINGS_ONLY は、ハイパーバイザーが "グローバル" としてマップされていないページマッピングをフラッシュするためにのみ必要であることを示します (つまり、ページテーブルエントリで x64 "G" ビットが設定されています)。 グローバルエントリは、ハイパーバイザーによってフラッシュ解除される (ただし、必須ではありません) 場合があります。

他のすべてのフラグは予約されており、0に設定する必要があります。

この呼び出しにより、時間制御が呼び出し元に戻ったときに、指定された仮想プロセッサ上のすべてのフラッシュの観察可能な効果が発生したことが保証されます。

ターゲット仮想プロセッサの TLB にフラッシュが必要で、仮想プロセッサの TLB が現在 "ロック" されている場合、呼び出し元の仮想プロセッサは中断されます。 呼び出し元の仮想プロセッサが "unsuspended" の場合、ハイパーコールは再発行されます。

## <a name="call-code"></a>呼び出しコード
`0x0002` 簡潔

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `AddressSpace`          | 0          | 8        | アドレス空間 ID (CR3 値) を指定します。 |
| `Flags`                 | 8          | 8        | フラッシュの操作を変更するフラグビットのセット。 |
| `ProcessorMask`         | 16         | 8        | フラッシュ操作によって影響を受けるプロセッサを示すプロセッサマスク。 |
