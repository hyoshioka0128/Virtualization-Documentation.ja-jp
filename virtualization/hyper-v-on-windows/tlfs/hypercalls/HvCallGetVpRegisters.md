---
title: HvCallGetVpRegisters
description: HvCallGetVpRegisters ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: fbddc59feefb9025a4abf27f8ae55524f0adb30a
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120741"
---
# <a name="hvcallgetvpregisters"></a>HvCallGetVpRegisters

HvCallGetVpRegisters ハイパーコールは、仮想プロセッサの状態を読み取ります。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallGetVpRegisters(
    _In_ HV_PARTITION_ID PartitionId,
    _In_ HV_VP_INDEX VpIndex,
    _In_ HV_INPUT_VTL InputVtl,
    _Inout_ PUINT32 RegisterCount,
    _In_reads(RegisterCount) PCHV_REGISTER_NAME RegisterNameList,
    _In_writes(RegisterCount) PHV_REGISTER_VALUE RegisterValueList
    );
 ```

状態は一連のレジスタ値として返されます。各レジスタ値は、入力として指定されたレジスタ名に対応しています。

### <a name="restrictions"></a>制限

- 呼び出し元は、PartitionId で指定されたパーティションの親であるか、または指定されたパーティションが "self" で、パーティションに AccessVpRegisters 特権が必要です。

## <a name="call-code"></a>呼び出しコード

`0x0050` 民主

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `PartitionId`           | 0          | 8        | パーティション Id を指定します。               |
| `VpIndex`               | 8          | 4        | 仮想プロセッサのインデックスを指定します。 |
| `TargetVtl`             | 12         | 1        | ターゲットの VTL を指定します。                 |
| RsvdZ                   | 13         | 3        |                                           |

## <a name="input-list-element"></a>入力リスト要素

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `RegisterName`          | 0          | 4        | 読み取るレジスタの名前を指定します。 |

## <a name="output-list-element"></a>出力リスト要素

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `RegisterValue`         | 0          | 16       | 指定されたレジスタの値を返します。 |

## <a name="see-also"></a>関連項目

[HV_REGISTER_NAME](../datatypes/HV_REGISTER_NAME.md)

[HV_REGISTER_VALUE](../datatypes/HV_REGISTER_VALUE.md)
