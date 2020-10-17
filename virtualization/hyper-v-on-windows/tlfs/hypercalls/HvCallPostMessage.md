---
title: HvCallPostMessage
description: HvCallPostMessage ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 7fc0693341316966f986bece60d007b651ce6870
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120754"
---
# <a name="hvcallpostmessage"></a>HvCallPostMessage

HvCallPostMessage ハイパーコールは、関連付けられた宛先ポートを持つ、指定された接続へのメッセージの送信 (非同期送信) を試みます。 メッセージが正常に送信されると、そのポートに関連付けられているパーティション内の仮想プロセッサに配信するためにキューに登録されます。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallPostMessage(
    _In_ HV_CONNECTION_ID ConnectionId,
    _In_ HV_MESSAGE_TYPE MessageType,
    _In_ UINT32 PayloadSize,
    _In_reads_bytes_(PayloadSize) PCVOID Message
    );
 ```

## <a name="call-code"></a>呼び出しコード

`0x005C` 簡潔

## <a name="input-parameters"></a>入力パラメーター

| 名前                    | Offset     | サイズ     | 情報提供済み                      |
|-------------------------|------------|----------|-------------------------------------------|
| `ConnectionId`          | 0          | 4        | 接続の ID を指定します。       |
| RsvdZ                   | 4          | 4        |                                           |
| `MessageType`           | 8          | 4        | メッセージヘッダー内に表示されるメッセージの種類を指定します。 呼び出し元は、最上位ビットがクリアされている任意の32ビットメッセージ型を指定できます。ただし、ゼロは例外です。 |
| `PayloadSize`           | 12         | 4        | メッセージに含まれるバイト数を指定します。 |
| `Message`               | 16         | 240      | 指定は、メッセージのペイロード (最大240バイト) を表示します。 実際には、最初の n バイトだけが対象パーティションに送信されます。ここで、n は PayloadSize パラメーターに指定されています。 |

## <a name="return-values"></a>戻り値

| status code                         | エラー条件                                       |
|-------------------------------------|-------------------------------------------------------|
| `HV_STATUS_ACCESS_DENIED`           | 呼び出し元のパーティションに PostMessages 特権がありません。 |
| `HV_STATUS_INVALID_CONNECTION_ID`   | 指定された接続 ID は無効です。               |
| `HV_STATUS_INVALID_PORT_ID`         | 指定された接続に関連付けられているポートが削除されました。 |
|                                     | 指定された接続に関連付けられているポートは、"アクティブ" 状態ではないパーティションに属しています。 |
|                                     | 指定された接続に関連付けられているポートは、"メッセージ" の種類のポートではありません。 |
| `HV_STATUS_INVALID_PARAMETER`       | 指定したメッセージの種類の最上位ビットが設定されます。 |
|                                     | MessageType パラメーターには0の値を指定します。  |
|                                     | 指定されたペイロードサイズが240バイトを超えています。         |
| `HV_STATUS_INSUFFICIENT_BUFFERS`    | ポートには、使用可能なゲストメッセージバッファーがありません。      |
| `HV_STATUS_INVALID_VP_INDEX`        | ターゲット VP が存在しないか、メッセージの投稿先として使用できる VPs がありません。 |
| `HV_STATUS_INVALID_SYNIC_STATE`     | ターゲット VP の SynIC は無効になっており、ポストされたメッセージを受け入れることができません。 |
|                                     | ターゲット VP の SIM ページが無効になっています。                 |