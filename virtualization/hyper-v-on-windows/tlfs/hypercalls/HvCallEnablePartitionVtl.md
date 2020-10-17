---
title: HvCallEnablePartitionVtl
description: HvCallEnablePartitionVtl ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: fde108fff5a7b7ca9867234b8e8fcceb5a98722f
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120826"
---
# <a name="hvcallenablepartitionvtl"></a>HvCallEnablePartitionVtl

HvCallEnablePartitionVtl ハイパーコールは、指定されたパーティションに対して仮想信頼レベルを有効にします。 新しい VTL を開始して使用するには、HvCallEnableVpVtl と組み合わせて使用する必要があります。

## <a name="interface"></a>インターフェイス

 ```c

typedef union
{
    UINT8 AsUINT8;
    struct {
        UINT8 EnableMbec:1;
        UINT8 Reserved:7;
    };
} HV_ENABLE_PARTITION_VTL_FLAGS;

HV_STATUS
HvCallEnablePartitionVtl(
    _In_ HV_PARTITION_ID TargetPartitionId,
    _In_ HV_VTL TargetVtl,
    _In_ HV_ENABLE_PARTITION_VTL_FLAGS Flags
    );
 ```

### <a name="restrictions"></a>制限

- ターゲットの vtl が起動している VTL よりも低い場合は、起動中の VTL がターゲットの VTL を常に有効にすることができます。
- 起動中の vtl が、ターゲットの VTL よりも低いパーティションに対して有効な最大の VTL である場合は、より高いターゲットの VTL を実現できます。

## <a name="call-code"></a>呼び出しコード

`0x000D` 簡潔

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `TargetPartitionId`     | 0          | 8        | この要求の対象となるパーティションのパーティション ID を指定します。 |
| `TargetVtl`             | 8          | 1        | このハイパーコールによって有効になる VTL を指定します。 |
| `Flags`                 | 9          | 1        | VSM に関連する機能を有効にするマスクを指定します。|
| RsvdZ                   | 10         | 6        |                                           |