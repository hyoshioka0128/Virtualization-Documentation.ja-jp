---
title: HvCallVtlReturn
description: HvCallVtlReturn ハイパーコール
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 7070443a4e41c7059980196f47fc8d31d6f5ced3
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120991"
---
# <a name="hvcallvtlreturn"></a>HvCallVtlReturn

HvCallVtlReturn は、"[VTL return](../vsm.md#vtl-return)" を開始し、VP で有効になっている次の最低の vtl に切り替えます。

## <a name="interface"></a>インターフェイス

 ```c
HV_STATUS
HvCallVtlReturn(
    VOID
    );
 ```

## <a name="call-code"></a>呼び出しコード

`0x0012` 簡潔