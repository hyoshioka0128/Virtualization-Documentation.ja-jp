---
title: HV_MESSAGE
description: HV_MESSAGE
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: af5a6a314f51178021ffaa7b3cc07c8baa016990
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121158"
---
# <a name="hv_message"></a>HV_MESSAGE

SynIC メッセージは、メッセージヘッダー (メッセージの種類とメッセージの送信元に関する情報を含む) で構成される固定サイズで、その後にペイロードが続きます。 [HvCallPostMessage](../hypercalls/HvCallPostMessage.md)への応答として送信されるメッセージには、ポート ID が含まれます。 インターセプトメッセージには、仮想プロセッサがインターセプトを生成したパーティションのパーティション ID が格納されます。 タイマーインターセプトに発信元 ID がありません (つまり、指定された ID が0です)。

MessagePending フラグは、合成割り込みソースのメッセージキューに保留中のメッセージがあるかどうかを示します。 存在する場合は、メッセージスロットを空にした後、ゲストが "メッセージの終わり" を実行する必要があります。 これにより、EOM MSR への便宜的な書き込みが可能になります (必要な場合のみ)。 このフラグは、ハイパーバイザーによってメッセージ配信時またはその後いつでも設定できることに注意してください。 フラグは、メッセージスロットが空になった後にテストする必要があります。設定されている場合は、保留中のメッセージが1つ以上あり、"メッセージの終わり" が実行されます。

## <a name="syntax"></a>構文

 ```c
#define HV_MESSAGE_SIZE 256
#define HV_MESSAGE_MAX_PAYLOAD_BYTE_COUNT 240
#define HV_MESSAGE_MAX_PAYLOAD_QWORD_COUNT 30

typedef struct
{
    UINT8 MessagePending:1;
    UINT8 Reserved:7;
} HV_MESSAGE_FLAGS;

typedef struct
{
    HV_MESSAGE_TYPE MessageType;
    UINT16 Reserved;
    HV_MESSAGE_FLAGS MessageFlags;
    UINT8 PayloadSize;
    union
    {
        UINT64 OriginationId;
        HV_PARTITION_ID Sender;
        HV_PORT_ID Port;
    };
} HV_MESSAGE_HEADER;

typedef struct
{
    HV_MESSAGE_HEADER Header;
    UINT64 Payload[HV_MESSAGE_MAX_PAYLOAD_QWORD_COUNT];
} HV_MESSAGE;
 ```
