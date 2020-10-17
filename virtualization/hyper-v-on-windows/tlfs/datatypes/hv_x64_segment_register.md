---
title: HV_X64_SEGMENT_REGISTER
description: HV_X64_SEGMENT_REGISTER
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 6c661e3ce8314d6aeaf94286b0bba190a78f629a
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121017"
---
# <a name="hv_x64_segment_register"></a>HV_X64_SEGMENT_REGISTER

セグメント登録状態は次のようにエンコードされます。

## <a name="syntax"></a>構文

```c
typedef struct
{
    UINT64 Base;
    UINT32 Limit;
    UINT16 Selector;
    union
    {
        struct
        {
            UINT16 SegmentType:4;
            UINT16 NonSystemSegment:1;
            UINT16 DescriptorPrivilegeLevel:2;
            UINT16 Present:1;
            UINT16 Reserved:4;
            UINT16 Available:1;
            UINT16 Long:1;
            UINT16 Default:1;
            UINT16 Granularity:1;
        };

        UINT16 Attributes;
    };
} HV_X64_SEGMENT_REGISTER;
 ```

この制限は、32ビット値としてエンコードされます。 X64 の長いモードセグメントの場合、制限は無視されます。 レガシ x86 セグメントの場合、この制限は、x64 プロセッサアーキテクチャの境界内で表現できる必要があります。 たとえば、コードまたはデータセグメントの属性内に "G" (粒度) ビットが設定されている場合、制限の下位12ビットは1である必要があります。

"Present" ビットは、セグメントが null セグメントとして機能するかどうかを制御します (つまり、そのセグメントを通じて実行されるメモリアクセスが #GP エラーを生成するかどうか)。

MSRs IA32_FS_BASE と IA32_GS_BASE は、セグメントレジスタ構造の基本要素のエイリアスであるため、レジスタの一覧で定義されていません。 代わりに、HvX64RegisterFs および HvX64RegisterGs と、上記の構造体を使用してください。