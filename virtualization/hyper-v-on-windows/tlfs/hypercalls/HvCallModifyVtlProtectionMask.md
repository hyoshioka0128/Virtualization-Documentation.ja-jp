---
title: HvCallModifyVtlProtectionMask
description: HvCallModifyVtlProtectionMask ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: a88dacf32a57eeb98c7971b5fd752dfe6562e1c8
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120738"
---
# <a name="hvcallmodifyvtlprotectionmask"></a>HvCallModifyVtlProtectionMask

HvCallModifyVtlProtectionMask ハイパーコールは、既存の GPA ページのセットに適用される VTL 保護を変更します。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvModifyVtlProtectionMask(
    _In_ HV_PARTITION_ID TargetPartitionId,
    _In_ HV_MAP_GPA_FLAGS MapFlags,
    _In_ HV_INPUT_VTL TargetVtl,
    _In_reads(PageCount) HV_GPA_PAGE_NUMBER GpaPageList
    );
 ```

VTL は、低レベルの VTL にのみ保護を配置できます。

RAM 以外の範囲での VTL 保護を適用しようとすると、HV_STATUS_INVALID_PARAMETER で失敗します。

## <a name="call-code"></a>呼び出しコード

`0x000C` 民主

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `TargetPartitionId`     | 0          | 8        | この要求の対象となるパーティションのパーティション ID を指定します。 |
| `MapFlags`              | 8          | 4        | 適用する新しいマッピングフラグを指定します。 |
| `TargetVtl`             | 12         | 1        | ターゲットの VTL が指定されました。                 |
| RsvdZ                   | 13         | 3        |                                           |

## <a name="input-list-element"></a>入力リスト要素

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `GpaPageList`           | 0          | 8        | 保護するページを提供します。       |