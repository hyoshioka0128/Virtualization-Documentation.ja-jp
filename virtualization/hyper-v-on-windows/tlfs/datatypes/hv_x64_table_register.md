---
title: HV_X64_TABLE_REGISTER
description: HV_X64_TABLE_REGISTER
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 1fddba94270b7c1591ac6f7aa92d3ce5e926fe1e
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121014"
---
# <a name="hv_x64_table_register"></a>HV_X64_TABLE_REGISTER

テーブルレジスタはセグメントレジスタに似ていますが、セレクターまたは属性がないため、制限は16ビットに制限されています。

## <a name="syntax"></a>構文

```c
typedef struct
{
    UINT16 Pad[3];
    UINT16 Limit;
    UINT64 Base;
} HV_X64_TABLE_REGISTER;
 ```
