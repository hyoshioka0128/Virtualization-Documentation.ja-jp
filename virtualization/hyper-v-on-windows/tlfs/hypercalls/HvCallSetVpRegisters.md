---
title: HvCallSetVpRegisters
description: HvCallSetVpRegisters ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 82b5aa39c1cf2d9e1372e9fba6c86579b11e15bf
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121206"
---
# <a name="hvcallsetvpregisters"></a>HvCallSetVpRegisters

HvCallSetVpRegisters ハイパーコールは、仮想プロセッサの状態を書き込みます。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallSetVpRegisters(
    _In_ HV_PARTITION_ID PartitionId,
    _In_ HV_VP_INDEX VpIndex,
    _In_ HV_INPUT_VTL InputVtl,
    _Inout_ PUINT32 RegisterCount,
    _In_reads(RegisterCount) PCHV_REGISTER_NAME RegisterNameList,
    _In_reads(RegisterCount) PCHV_REGISTER_VALUE RegisterValueList
    );
 ```

状態は一連のレジスタ値として書き込まれ、それぞれが入力として指定されたレジスタ名に対応します。

レジスタ値が変更されると、最小限のエラーチェックが実行されます。 特に、ハイパーバイザーは、レジスタの予約ビットがゼロに設定されていることを検証します。このビットは、常にゼロを含むように定義されている、または1つのビットが適切に設定されており、レジスタのアーキテクチャサイズを超えて指定されたビットがゼロに設定されます。

この呼び出しを使用して、読み取り専用レジスタの値を変更することはできません。

レジスタの変更による副作用は実行されません。 これには、例外の生成、パイプラインの同期、TLB のフラッシュなどが含まれます。

### <a name="restrictions"></a>制限

- 呼び出し元は、PartitionId で指定されたパーティションの親であるか、または指定されたパーティションが "self" で、パーティションに AccessVpRegisters 特権が必要です。

## <a name="call-code"></a>呼び出しコード

`0x0051` 民主

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
| `RegisterName`          | 0          | 4        | 変更するレジスタの名前を指定します。 |
| RsvdZ                   | 4          | 12       |                                           |
| `RegisterValue`         | 16         | 16       | 指定したレジスタの新しい値を指定します。 |

## <a name="see-also"></a>関連項目

[HV_REGISTER_NAME](../datatypes/HV_REGISTER_NAME.md)

[HV_REGISTER_VALUE](../datatypes/HV_REGISTER_VALUE.md)
