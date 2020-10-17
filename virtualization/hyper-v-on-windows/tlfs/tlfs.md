---
title: ハイパーバイザーの仕様
description: ハイパーバイザーの仕様
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 97615dc78f31a3a619130abb5a4d3786fdd412b2
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120896"
---
# <a name="hypervisor-top-level-functional-specification"></a>ハイパーバイザーのトップレベル機能仕様

Hyper-v ハイパーバイザー Top-Level 機能仕様 (TLFS) では、他のオペレーティングシステムコンポーネントに対するハイパーバイザーのゲストに見える動作を記述します。 この仕様は、ゲスト オペレーティング システムの開発者にとって役立ちます。

> この仕様は、Microsoft Open Specification Promise の下で提供されます。  [Microsoft Open Specification Promise](https://docs.microsoft.com/openspecs/dev_center/ms-devcentlp/51a0d3ff-9f77-464c-b83f-2de08ed28134) に関する詳細については、次をお読みください。

マイクロソフトは、これらの資料に記載されている内容に関し、特許、特許申請、商標、著作権、またはその他の無体財産権を有する場合があります。 Microsoft Open Specification Promise で明示的に指定されている場合を除き、これらの資料は、これらの特許、商標、著作権、またはその他の無体財産権に関する権利をお客様に許諾するものではありません。

## <a name="glossary"></a>用語集

- **パーティション** – hyper-v は、パーティションに関して分離をサポートします。 パーティションは、ハイパーバイザーでサポートされている、オペレーティング システムが実行される論理的な分離の単位です。
- **ルートパーティション** –ルートパーティション ("親" または "ホスト") は、特権管理パーティションです。 ルートパーティションは、デバイスドライバー、電源管理、デバイスの追加/削除などのコンピューターレベルの機能を管理します。 仮想化スタックは親パーティションで実行され、ハードウェアデバイスに直接アクセスできます。 ルート パーティションによって、ゲスト オペレーティング システムをホストする子パーティションが作成されます。
- **子パーティション** –子パーティション (別名 "ゲスト") は、ゲストオペレーティングシステムをホストします。 子パーティションによる物理メモリとデバイスへのすべてのアクセスは、仮想マシンバス (VMBus) またはハイパーバイザーを介して提供されます。
- **ハイパーコール** –ハイパーバイザーは、ハイパーバイザーと通信するためのインターフェイスです。

## <a name="specification-style"></a>仕様のスタイル

このドキュメントでは、高レベルのハイパーバイザーアーキテクチャについて理解していることを前提としています。

この仕様は非公式です。つまり、インターフェイスは、正式な言語では指定されていません。 ただし、これは正確な目標です。 また、アーキテクチャと実装固有の動作を指定することもできます。 呼び出し元は、後者のカテゴリに分類される動作に依存しないようにする必要があります。これは、将来の実装で変更される可能性があるためです。

## <a name="previous-versions"></a>以前のバージョン

Release | ドキュメント
--- | ---
Windows Server 2019 (リビジョン B) | [ハイパーバイザーのトップレベル機能仕様 v6.0b.pdf](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v6.0b.pdf)
Windows Server 2016 (リビジョン C) | [Hypervisor Top Level Functional Specification v5.0c.pdf](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/live/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v5.0C.pdf)
Windows Server 2012 R2 (リビジョン B) | [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 | [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## <a name="requirements-for-implementing-the-microsoft-hypervisor-interface"></a>Microsoft ハイパーバイザー インターフェイスを実装するための要件

TLFS には、Microsoft 固有のハイパーバイザー アーキテクチャのあらゆる側面を網羅した説明が記載されています。このアーキテクチャはゲスト仮想マシンに対して "HV#1" として宣言されます。  ただし、サード パーティ製ハイパーバイザーでは、Microsoft HV#1 ハイパーバイザー仕様への準拠を宣言するために、TLFS に記載されているインターフェイスをすべて実装する必要はありません。 「Requirements for Implementing the Microsoft Hypervisor Interface」 (Microsoft ハイパーバイザー インターフェイスを実装するための要件) には、Microsoft HV#1 との互換性が求められるすべてのハイパーバイザーに最小限実装する必要があるハイパーバイザー インターフェイスが記載されています。

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)