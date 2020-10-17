---
title: HV_VIRTUALIZATION_FAULT_INFORMATION
description: HV_VIRTUALIZATION_FAULT_INFORMATION
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 503b11383bf02f541e253f673a412254c9cbbf72
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121078"
---
# <a name="hv_virtualization_fault_information"></a>HV_VIRTUALIZATION_FAULT_INFORMATION

[仮想化エラー情報] 領域には、VP の現在のエラーコードとエラーパラメーターが表示されます。 16バイトでアラインされます。

## <a name="syntax"></a>構文

```c
typedef union
{
    struct
    {
        UINT16 Parameter0;
        UINT16 Reserved0;
        UINT32 Code;
        UINT64 Parameter1;
    };
} HV_VIRTUALIZATION_FAULT_INFORMATION, *PHV_VIRTUALIZATION_FAULT_INFORMATION;
 ```

## <a name="see-also"></a>関連項目

 [HV_VP_ASSIST_PAGE](HV_VP_ASSIST_PAGE.md)