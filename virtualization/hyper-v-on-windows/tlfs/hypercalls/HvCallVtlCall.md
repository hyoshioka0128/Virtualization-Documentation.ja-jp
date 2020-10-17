---
title: HvCallVtlCall
description: HvCallVtlCall ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: c2481cc0da0b185c45a0285f2cd0394dff7af9c5
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120733"
---
# <a name="hvcallvtlcall"></a>HvCallVtlCall

HvCallVtlCall は "[VTL 呼び出し](../vsm.md#vtl-call)" を開始し、VP で有効になっている次の最高の vtl に切り替えます。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallVtlCall(
    VOID
    );
 ```

## <a name="call-code"></a>呼び出しコード

`0x0011` 簡潔