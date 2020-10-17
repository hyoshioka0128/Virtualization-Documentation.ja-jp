---
title: HvCallNotifyLongSpinWait
description: HvCallNotifyLongSpinWait ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 468a1c941d547d3a1243ac67e6d63812341fe589
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120757"
---
# <a name="hvcallnotifylongspinwait"></a>HvCallNotifyLongSpinWait

HvCallNotifyLongSpinWait ハイパーコールは、ゲスト OS によって使用され、呼び出し元の仮想プロセッサが、同じパーティション内の別の仮想プロセッサによって保持されている可能性のあるリソースを取得しようとしていることをハイパーバイザーに通知します。 このスケジュール設定ヒントを使用すると、複数の仮想プロセッサを持つパーティションのスケーラビリティが向上します。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvNotifyLongSpinWait(
    _In_ UINT64 SpinCount
    );
 ```

## <a name="call-code"></a>呼び出しコード

`0x0008` 簡潔

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `SpinCount`             | 0          | 4        | ゲストがスピンした累積カウントを指定します。 |
| RsvdZ                   | 4          | 4        |                                           |

## <a name="return-values"></a>戻り値

このハイパーコールにはエラー状態がありません。これは勧告ハイパーコールであるため、HV_STATUS_SUCCESS のみが返されます。