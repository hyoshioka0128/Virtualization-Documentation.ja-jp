---
title: HvCallSignalEvent
description: HvCallSignalEvent ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 7d64700cbb2589e9d9b7780ea5d1acb2f6b8cfe3
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120886"
---
# <a name="hvcallsignalevent"></a>HvCallSignalEvent

HvCallSignalEvent ハイパーコールは、指定された接続に関連付けられているポートを所有するパーティションのイベントを通知します。

イベントは、受信パーティションの仮想プロセッサの1つである SIEF ページ内でビットを設定することによって通知されます。 呼び出し元は、相対フラグ番号を指定します。 実際の SIEF ビット番号は、ポートに関連付けられた基本フラグ番号に指定されたフラグ番号を追加することによって、ハイパーバイザーによって計算されます。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallSignalEvent(
    _In_ HV_CONNECTION_ID ConnectionId,
    _In_ UINT16 FlagNumber
    );
 ```

## <a name="call-code"></a>呼び出しコード

`0x005D` 簡潔

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `ConnectionId`          | 0          | 4        | 接続の ID を指定します。       |
| `FlagNumber`            | 4          | 2        | 呼び出し元がターゲット SIEF 領域内で設定するイベントフラグの相対インデックスを指定します。 この数値は、ポートに関連付けられた基本フラグ番号に対する相対値です。 |
| RsvdZ                   | 6          | 2        |                                           |

## <a name="return-values"></a>戻り値

| status code                         | エラー条件                                       |
|-------------------------------------|-------------------------------------------------------|
| `HV_STATUS_ACCESS_DENIED`           | 呼び出し元のパーティションには、SignalEvents 特権がありません。 |
| `HV_STATUS_INVALID_CONNECTION_ID`   | 指定された接続 ID は無効です。               |
| `HV_STATUS_INVALID_PORT_ID`         | 指定された接続に関連付けられているポートが削除されました。 |
|                                     | 指定された接続に関連付けられているポートは、"アクティブ" 状態ではないパーティションに属しています。 |
|                                     | 指定された接続に関連付けられているポートは、"イベント" の種類のポートではありません。 |
| `HV_STATUS_INVALID_PARAMETER`       | 指定されたフラグ番号が、ポートのフラグ数以上です。 |
| `HV_STATUS_INVALID_VP_INDEX`        | ターゲット VP が存在しないか、メッセージの投稿先として使用できる VPs がありません。 |
| `HV_STATUS_INVALID_SYNIC_STATE`     | ターゲット VP の SynIC は無効になっており、シグナル状態のイベントを受け入れることができません。 |
|                                     | ターゲット VP の SIEF ページが無効になっています。                 |