---
title: HV_SYNIC_EVENT_FLAGS
description: HV_SYNIC_EVENT_FLAGS
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 741c01718890d453017fc22470b6b0e96298b05a
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121089"
---
# <a name="hv_synic_event_flags"></a>HV_SYNIC_EVENT_FLAGS

SynIC イベントフラグは、固定サイズのビットごとの配列です。 配列の最初のバイトがフラグ0から 7 (最下位ビット) であり、配列の2番目のバイトにフラグ 8 ~ 15 (8 番目の最下位ビット) が格納されていることを示す番号が付けられています。

## <a name="syntax"></a>構文

```c
#define HV_EVENT_FLAGS_COUNT (256 * 8)
#define HV_EVENT_FLAGS_BYTE_COUNT 256

typedef struct
{
    UINT8 Flags[HV_EVENT_FLAGS_BYTE_COUNT];
} HV_SYNIC_EVENT_FLAGS;
 ```
