---
title: HV_CONNECTION_ID
description: HV_CONNECTION_ID
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: a3b1238c23c017dfb2fe756269cc2d59de1af5ff
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120926"
---
# <a name="hv_connection_id"></a>HV_CONNECTION_ID

接続は、32ビットの Id によって識別されます。 上位8ビットは予約されており、0である必要があります。 すべての接続 Id は、パーティション内で一意です。

## <a name="syntax"></a>構文

 ```c
typedef union
{
    UINT32 AsUInt32;
    struct
    {
        UINT32 Id:24;
        UINT32 Reserved:8;
    };
} HV_CONNECTION_ID;
 ```
