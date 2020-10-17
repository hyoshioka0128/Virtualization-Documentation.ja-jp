---
title: HV_MESSAGE_TYPE
description: HV_MESSAGE_TYPE
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 0d55012789000828292fb9e88995cc70d1bb13d2
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121153"
---
# <a name="hv_message_type"></a>HV_MESSAGE_TYPE

SynIC メッセージは、メッセージの種類を32ビットの数値としてエンコードします。 上位ビットが設定されているメッセージ型は、ハイパーバイザーで使用するために予約されています。 ゲストによって開始されたメッセージは、ハイパーバイザーメッセージ型のメッセージを送信できません。

## <a name="syntax"></a>構文

 ```c
#define HV_MESSAGE_TYPE_HYPERVISOR_MASK 0x80000000

typedef enum
{
    HvMessageTypeNone = 0x00000000,

    // Memory access messages
    HvMessageTypeUnmappedGpa = 0x80000000,
    HvMessageTypeGpaIntercept = 0x80000001,

    // Timer notifications
    HvMessageTimerExpired = 0x80000010,

    // Error messages
    HvMessageTypeInvalidVpRegisterValue = 0x80000020,
    HvMessageTypeUnrecoverableException = 0x80000021,
    HvMessageTypeUnsupportedFeature = 0x80000022,
    HvMessageTypeTlbPageSizeMismatch = 0x80000023,

    // Hypercall intercept
    HvMessageTypeHypercallIntercept = 0x80000050,

    // Platform-specific processor intercept messages
    HvMessageTypeX64IoPortIntercept = 0x80010000,
    HvMessageTypeMsrIntercept = 0x80010001,
    HvMessageTypeX64CpuidIntercept = 0x80010002,
    HvMessageTypeExceptionIntercept = 0x80010003,
    HvMessageTypeX64ApicEoi = 0x80010004,
    HvMessageTypeX64LegacyFpError = 0x80010005,
    HvMessageTypeRegisterIntercept = 0x80010006,
} HV_MESSAGE_TYPE;
 ```
