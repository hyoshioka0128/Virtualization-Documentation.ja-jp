---
title: HvCallFlushGuestPhysicalAddressSpace
description: HvCallFlushGuestPhysicalAddressSpace ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 7c37a08bea56cce7460c89b2625ca9cc3e4239aa
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120829"
---
# <a name="hvcallflushguestphysicaladdressspace"></a>HvCallFlushGuestPhysicalAddressSpace

HvCallFlushGuestPhysicalAddressSpace ハイパーコールは、第2レベルのアドレス空間内の GPA マッピングに対して、キャッシュされた L2 GPA を無効にします。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallFlushGuestPhysicalAddressSpace(
    _In_ HV_SPA AddressSpace,
    _In_ UINT64 Flags
    );
 ```

このハイパーコールは、入れ子になった仮想化がアクティブである場合にのみ使用できます。 仮想 TLB 無効化操作は、すべてのプロセッサに対して機能します。

Intel プラットフォームでは、HvCallFlushGuestPhysicalAddressSpace ハイパーコールは、すべてのプロセッサに対して "単一コンテキスト" 型の INVEPT 命令を実行するのと似ています。

この呼び出しにより、時間制御が呼び出し元に戻ったときに、すべてのフラッシュの観測可能な効果が発生したことが保証されます。
TLB が現在 "ロック" されている場合、呼び出し元の仮想プロセッサは中断されます。

## <a name="call-code"></a>呼び出しコード

`0x00AF` 簡潔

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `AddressSpace`          | 0          | 8        | アドレス空間 ID (EPT PML4 table ポインター) を指定します。 |
| `Flags`                 | 8          | 8        | RsvdZ                                     |