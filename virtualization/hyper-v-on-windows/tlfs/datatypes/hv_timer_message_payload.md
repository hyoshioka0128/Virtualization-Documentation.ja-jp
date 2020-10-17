---
title: HV_TIMER_MESSAGE_PAYLOAD
description: HV_TIMER_MESSAGE_PAYLOAD
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 48a66d841b20f575868a30f2f08d3377c724effd
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121081"
---
# <a name="hv_timer_message_payload"></a>HV_TIMER_MESSAGE_PAYLOAD

タイマーの有効期限メッセージは、タイマーイベントが発生したときに送信されます。 この構造体は、メッセージペイロードを定義します。

## <a name="syntax"></a>構文

```c
typedef struct
{
    UINT32          TimerIndex;
    UINT32          Reserved;
    HV_NANO100_TIME ExpirationTime;     // When the timer expired
    HV_NANO100_TIME DeliveryTime;       // When the message was delivered
} HV_TIMER_MESSAGE_PAYLOAD;
 ```

TimerIndex は、メッセージを生成した合成タイマー (0 ~ 3) のインデックスです。 これにより、クライアントは、同じ割り込みベクターを使用してメッセージを区別するように複数のタイマーを構成できます。

ExpirationTime は、パーティションの参照時間カウンターの時間ベースを使用して、100ナノ秒単位で計測されたタイマーの予想される有効期限です。 有効期限が切れると、有効期限切れのメッセージが到着する可能性があります。

DeliveryTime は、SIM ページの各メッセージスロットにメッセージが配置される時刻です。 時間は、パーティションの参照時間カウンターに基づいて100ナノ秒単位で計測されます。

## <a name="see-also"></a>関連項目

 [HV_MESSAGE](HV_MESSAGE.md)