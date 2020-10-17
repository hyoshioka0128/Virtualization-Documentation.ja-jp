---
title: Partition
description: ハイパーバイザーパーティションインターフェイス
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: c2a040603a5b0f3d354731ed3c2cc95f4c3da49b
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120813"
---
# <a name="partitions"></a>メジャー グループ

ハイパーバイザーは、パーティションに関して分離をサポートします。 パーティションは、ハイパーバイザーでサポートされている、オペレーティング システムが実行される論理的な分離の単位です。

## <a name="partition-privilege-flags"></a>パーティション権限フラグ

各パーティションには、ハイパーバイザーによって割り当てられた一連の特権があります。 特権は、統合 MSRs またはハイパーコールコールへのアクセスを制御します。

パーティションは、"ハイパーバイザー機能識別" CPUID リーフ (0x40000003) を使用して、その特権を照会できます。 すべての特権の詳細については、「 [HV_PARTITION_PRIVILEGE_MASK](datatypes/HV_PARTITION_PRIVILEGE_MASK.md) 」を参照してください。

## <a name="partition-crash-enlightenment"></a>パーティションクラッシュ啓蒙

ハイパーバイザーは、クラッシュ・啓蒙施設でゲストパーティションを提供します。 このインターフェイスにより、ゲストパーティションで実行されているオペレーティングシステムは、クラッシュダンプ手順の一部として、致命的な OS の状態に関する法廷情報をハイパーバイザーに提供するオプションを使用できるようになります。 オプションには、ゲストクラッシュパラメーター MSRs の内容を保持し、クラッシュメッセージを指定するオプションがあります。 その後、ハイパーバイザーによって、この情報がログ用のルートパーティションで使用できるようになります。 このメカニズムにより、仮想化ホスト管理者は、クラッシュしたゲスト OS によって格納されている可能性があるクラッシュダンプまたはコアダンプ情報のゲストパーティションに接続されている永続的なストレージを調べる必要なく、ゲスト OS のクラッシュイベントに関する情報を収集できます。

このメカニズムの可用性は、GuestCrashMsrsAvailable フラグによって示されています。 `CPUID.0x400003.EDX:10` [機能の検出](feature-discovery.md)に関する参照を参照してください。

### <a name="guest-crash-enlightenment-interface"></a>ゲストクラッシュの啓蒙インターフェイス

次に定義されているように、ゲストクラッシュの啓蒙インターフェイスは6つの合成 MSRs を通じて提供されます。

 ```c
#define HV_X64_MSR_CRASH_P0 0x40000100
#define HV_X64_MSR_CRASH_P1 0x40000101
#define HV_X64_MSR_CRASH_P2 0x40000102
#define HV_X64_MSR_CRASH_P3 0x40000103
#define HV_X64_MSR_CRASH_P4 0x40000104
#define HV_X64_MSR_CRASH_CTL 0x40000105
 ```

#### <a name="guest-crash-control-msr"></a>ゲストクラッシュコントロール MSR

ゲストのクラッシュコントロール MSR HV_X64_MSR_CRASH_CTL は、ハイパーバイザーのゲストクラッシュ機能を特定し、指定されたアクションを呼び出すためにゲストパーティションによって使用される場合があります。 [HV_CRASH_CTL_REG_CONTENTS](datatypes/HV_CRASH_CTL_REG_CONTENTS.md)データ構造は、MSR の内容を定義します。

##### <a name="determining-guest-crash-capabilities"></a>ゲストクラッシュ機能の決定

ゲストのクラッシュ機能を確認するために、ゲストパーティションは HV_X64_MSR_CRASH_CTL 登録を読み取ることができます。 ハイパーバイザーでサポートされている、サポートされている一連のアクションと機能が報告されます。

##### <a name="invoking-guest-crash-capabilities"></a>ゲストクラッシュ機能の呼び出し

サポートされているハイパーバイザーのゲストクラッシュアクションを呼び出すために、ゲストパーティションは、目的のアクションを指定して HV_X64_MSR_CRASH_CTL レジスタに書き込みます。 2つのバリエーションがサポートされています。 CrashNotify では、CrashMessage は CrashNotify と組み合わせて使用します。 ゲストのクラッシュが発生するたびに、2つのバリエーションのいずれかを指定して、MSR HV_X64_MSR_CRASH_CTL に1つの書き込みを実行する必要があります。

| ゲストクラッシュアクション  | 説明                                                 |
|---------------------|-------------------------------------------------------------|
| CrashMessage        | このアクションは、ハイパーバイザーへのクラッシュメッセージを指定するために CrashNotify と組み合わせて使用されます。 選択すると、P3 と P4 の値がメッセージの場所とサイズとして扱われます。 HV_X64_MSR_CRASH_P3 はメッセージのゲスト物理アドレスで、HV_X64_MSR_CRASH_P4 はメッセージのバイト数 (最大4096バイト) です。 |
| CrashNotify         | このアクションは、ゲストパーティションが必要なデータをゲストクラッシュパラメーター MSRs (P0 から P4) に書き込む処理を完了したことをハイパーバイザーに示し、ハイパーバイザーはこれらの MSRs の内容のログ記録を続行する必要があります。 |