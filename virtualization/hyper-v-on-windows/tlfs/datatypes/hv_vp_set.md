---
title: HV_VP_SET
description: HV_VP_SET
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: af327161049650a95e39e9a658896d8e718cbd84
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121070"
---
# <a name="hv_vp_set"></a>HV_VP_SET

仮想プロセッサセットは、仮想プロセッサのコレクションを表し、一部のハイパーコールの入力として使用できます。

## <a name="syntax"></a>構文

```c
typedef struct
{
    UINT64 Format;
    UINT64 ValidBanksMask;
    UINT64 BankContents[];
} HV_VP_SET;
 ```

プロセッサセットには2つのモードがあります。これらは、format フィールドによって指定されます。 "1" という形式のプロセッサセットは、指定されたパーティションのすべての仮想プロセッサを表します。 "0" という形式のプロセッサセットは、仮想プロセッサのスパースセットを表します。

| 値の書式設定  | 動作の設定                                                |
|---------------|-------------------------------------------------------------|
| 0             | VPs のスパースサブセット                                      |
| 1             | すべての VPs (パーティションに属しています)                          |

### <a name="sparse-virtual-processor-set"></a>スパース仮想プロセッサセット

次のセクションでは、仮想プロセッサのスパースセットを作成する方法について説明します。

仮想プロセッサの合計セットは、"bank" と呼ばれる64のチャンクに分割されます。 たとえば、プロセッサ0-63 はバンク0、64-127 はバンク1にあります。

個々のプロセッサを記述するには、その銀行を ValidBanksMask で指定します。 ValidBanksMask の各ビットは、特定の銀行を表します。

```
bank = VPindex / 64
```
ValidBanksMask で設定されているビットごとに、の内容配列に要素が含まれている必要があります。 この要素は、バンク自体を記述するマスクです。

ValidBankMask のビットが0の場合、この要素には、[no] の内容に対応する要素はありません。 さらに、ValidBankMask のビット1については、これは、の対応する要素の有効な状態であることができます。これは、この銀行でプロセッサが指定されていないことを意味します。

#### <a name="processor-set-example"></a>プロセッサセットの例

パーティションに 200 VPs があり、次のセットを指定したいとします: {0, 5130}

最初の形式は、スパースセットであるため、0です。 次に、対応する銀行 (つまり、ValidBanksMask のセットビット) は {0, 0, 2} です。 したがって、ValidBanksMask は0x05 です。

銀行0は、その銀行内の VPs を指定するためにビット0と5を設定します。 したがって、BankContents mask の対応する要素は0x21 です。

ビット1は ValidBanksMask に設定されていないため、BankContents には対応する要素がありません。 銀行2は VP インデックス128-191 を表します。 インデックス130を記述するには、対応するマスクのビット2を設定します。 したがって、BankContents は {0x21, 0x04} です。