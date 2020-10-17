---
title: 入れ子になった仮想化
description: 入れ子になった仮想化
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 36fb58759f7236f5fac0a66a2b3621d35b382c28
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121214"
---
# <a name="nested-virtualization"></a>入れ子になった仮想化

入れ子になった仮想化とは、Hyper-v ハイパーバイザーエミュレーションハードウェア仮想化拡張機能を指します。 これらのエミュレートされた拡張機能は、他の仮想化ソフトウェア (入れ子になったハイパーバイザーなど) が Hyper-v プラットフォームで実行できるようにするために使用できます。

この機能は、ゲストパーティションでのみ使用できます。 仮想マシンごとに有効にする必要があります。 入れ子になった仮想化は、Windows ルートパーティションではサポートされていません。

入れ子になった仮想化のさまざまなレベルを定義するには、次の用語を使用します。

| 用語                   | 定義                                                              |
|------------------------|-------------------------------------------------------------------------|
| **L0 ハイパーバイザー**      | 物理ハードウェア上で実行されている Hyper-v ハイパーバイザー。                    |
| **L1 ルート**            | Windows ルートオペレーティングシステム。                                      |
| **L1 ゲスト**           | 入れ子になったハイパーバイザーのない Hyper-v 仮想マシン。                  |
| **L1 ハイパーバイザー**      | Hyper-v 仮想マシン内で実行されている入れ子になったハイパーバイザー。           |
| **L2 ルート**            | Hyper-v 仮想マシンのコンテキスト内で実行されるルート Windows オペレーティングシステム。|
| **L2 ゲスト**           | 入れ子になった仮想マシン。 Hyper-v 仮想マシンのコンテキスト内で実行されます。 |

ベアメタルと比較すると、ハイパーバイザーでは、VM で実行するとパフォーマンスが大幅に低下する可能性があります。 L1 ハイパーバイザーは、L0 ハイパーバイザーによって提供される対応インターフェイスを使用して、Hyper-v VM で実行するように最適化できます。

## <a name="enlightened-vmcs"></a>対応 VMCS

Intel プラットフォームでは、仮想化ソフトウェアは仮想マシン制御データ構造 (VMCSs) を使用して、仮想化に関連するプロセッサの動作を構成します。 VMCSs は、VMPTRLD 命令を使用してアクティブにし、VMCSS および VMCSS 命令を使用して変更する必要があります。 これらの手順は、エミュレートされる必要があるため、入れ子になった仮想化では重大なボトルネックになることがよくあります。

ハイパーバイザーは、"対応 VMCS" 機能を公開します。これを使用すると、ゲスト物理メモリ内のデータ構造を使用して、仮想化関連のプロセッサ動作を制御できます。 このデータ構造は通常のメモリアクセス命令を使用して変更できます。そのため、L1 ハイパーバイザーで VMREAD または VMREAD または VMPTRLD 命令を実行する必要はありません。

L1 ハイパーバイザーでは、 [仮想プロセッサのサポートページ](vp-properties.md#virtual-processor-assist-page)の対応するフィールドに1を書き込むことで、対応 vmcss の使用を選択できます。 VP アシストページの別のフィールドは、現在アクティブな対応 VMCS を制御します。 各対応 VMCS は、サイズが1ページ (4 KB) であるため、最初はゼロにする必要があります。 対応 VMCS をアクティブまたは現在の状態にするために、VMPTRLD 命令を実行する必要はありません。

L1 ハイパーバイザーが対応 VMCS で VM エントリを実行すると、VMCS はプロセッサ上でアクティブと見なされます。 対応 VMCS は、一度に1つのプロセッサでのみアクティブにできます。 L1 ハイパーバイザーは、VMCLEAR 命令を実行して、対応 VMCLEAR をアクティブから非アクティブ状態に遷移させることができます。 対応 VMREAD をアクティブにしている間の VMREAD または VMREAD 命令はサポートされていないため、予期しない動作が発生する可能性があります。

[HV_VMX_ENLIGHTENED_VMCS](datatypes/HV_VMX_ENLIGHTENED_VMCS.md)構造体は、対応 vmcs のレイアウトを定義します。 すべての非合成フィールドは、Intel の物理 VMCS エンコードにマップされます。

### <a name="feature-discovery"></a>機能の検出

対応 VMCS インターフェイスのサポートは、CPUID リーフ0x40000004 で報告されます。

対応 VMCS 構造は、今後の変更に対応するためにバージョン管理されます。 各対応 VMCS 構造体には、L0 ハイパーバイザーによって報告されるバージョンフィールドが含まれています。

現在サポートされている VMCS バージョンは1のみです。

### <a name="clean-fields"></a>フィールドのクリーンアップ

L0 ハイパーバイザーでは、対応 VMCS の一部をキャッシュすることができます。 対応 VMCS clean フィールドは、入れ子になった VM エントリのゲストメモリから対応 VMCS のどの部分を再読み込みするかを制御します。 L1 ハイパーバイザーは、対応 VMCS を変更するたびに、対応する VMCS clean フィールドをクリアする必要があります。そうしないと、L0 ハイパーバイザーは古いバージョンを使用する可能性があります。

すっきりしたフィールドは、対応 VMCS の合成 "CleanFields" フィールドを介して制御されます。 既定では、すべてのビットが、L0 ハイパーバイザーが入れ子になった VM エントリごとに対応する VMCS フィールドを再読み込みするように設定されます。

## <a name="enlightened-msr-bitmap"></a>対応 MSR ビットマップ

Intel プラットフォームでは、L0 ハイパーバイザーは VMX の "MSR-ビットマップアドレス" コントロールをエミュレートします。これにより、仮想化ソフトウェアは、インターセプトを生成する MSR アクセスを制御できます。

L1 ハイパーバイザーは、L0 ハイパーバイザーを使用して、MSR アクセスの効率を高めることができます。 対応 VMCS の対応するフィールドを1に設定することによって、対応 MSR ビットマップを有効にすることができます。 有効にした場合、L0 ハイパーバイザーは、MSR ビットマップの変更を監視しません。 代わりに、L1 ハイパーバイザーは、いずれかの MSR ビットマップを変更した後、対応する clean フィールドを無効にする必要があります。

## <a name="compatibility-with-live-migration"></a>ライブマイグレーションとの互換性

Hyper-v では、あるホストから別のホストに子パーティションをライブマイグレーションすることができます。 ライブマイグレーションは、通常、子パーティションに対して透過的です。 ただし、入れ子になった仮想化の場合は、L1 ハイパーバイザーが移行を認識している必要があります。

### <a name="live-migration-notification"></a>ライブマイグレーション通知

L1 ハイパーバイザーは、パーティションが移行されたときに通知を受け取るように要求できます。 この機能は、CPUID で "AccessReenlightenmentControls" 特権として列挙されます。 L0 ハイパーバイザーでは、合成 MSR (HV_X64_MSR_REENLIGHTENMENT_CONTROL) が公開されています。これは、L1 ハイパーバイザーで割り込みベクターとターゲットプロセッサを構成するために使用される可能性があります。 L0 ハイパーバイザーは、移行のたびに、指定されたベクターを使用して割り込みを挿入します。

```c
#define HV_X64_MSR_REENLIGHTENMENT_CONTROL (0x40000106)

typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 Vector :8;
        UINT64 RsvdZ1 :8;
        UINT64 Enabled :1;
        UINT64 RsvdZ2 :15;
        UINT64 TargetVp :32;
    };
} HV_REENLIGHTENMENT_CONTROL;
```

指定されたベクターは、固定された APIC 割り込みに対応している必要があります。 TargetVp は、仮想プロセッサのインデックスを指定します。

### <a name="tsc-emulation"></a>TSC エミュレーション

TSC 周波数が異なる2台のコンピューター間では、ゲストパーティションをライブマイグレーションすることができます。 そのような場合は、 [参照 TSC ページ](timers.md#partition-reference-time-enlightenment) の TscScale 値を再計算する必要がある場合があります。

L0 ハイパーバイザーは、必要に応じて、TscScale 値を再計算する機会を持つようになるまで、移行後のすべての TSC アクセスをエミュレートします。 L1 ハイパーバイザーは HV_X64_MSR_TSC_EMULATION_CONTROL MSR に書き込むことで、TSC エミュレーションを選択できます。 オプトインされている場合、L0 ハイパーバイザーは移行の実行後に TSC アクセスをエミュレートします。

L1 ハイパーバイザーは、HV_X64_MSR_TSC_EMULATION_STATUS MSR を使用して TSC アクセスが現在エミュレートされているかどうかを照会できます。 たとえば、L1 ハイパーバイザーは [ライブマイグレーション通知](#live-migration-notification) をサブスクライブし、移行の割り込みを受信した後に TSC の状態を照会することができます。 また、この MSR を使用して TSC エミュレーション (TscScale 値を更新した後) を無効にすることもできます。

```c
#define HV_X64_MSR_TSC_EMULATION_CONTROL (0x40000107)
#define HV_X64_MSR_TSC_EMULATION_STATUS (0x40000108)

typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 Enabled :1;
        UINT64 RsvdZ :63;
    };
 } HV_TSC_EMULATION_CONTROL;

typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 InProgress : 1;
        UINT64 RsvdP1 : 63;
    };
} HV_TSC_EMULATION_STATUS;
```

## <a name="virtual-tlb"></a>仮想 TLB

ハイパーバイザーによって公開される仮想 TLB は、L2 gpas GPAs への変換をキャッシュするように拡張できます。 論理プロセッサ上の TLB と同様、仮想 TLB は整合性のないキャッシュであり、このような非一貫性はゲストに表示されます。 ハイパーバイザーは、TLB を管理するための操作を公開します。
Intel プラットフォームでは、L0 ハイパーバイザーは次の追加の方法で TLB を管理します。

- INVVPID 命令を使用して、キャッシュされた GVA を GPA または SPA マッピングに無効にすることができます。
- INVEPT 命令を使用して、キャッシュされた L2 GPA から GPA へのマッピングを無効にすることができます。

### <a name="direct-virtual-flush"></a>直接仮想フラッシュ

ハイパーバイザーは、オペレーティングシステムが仮想 TLB をより効率的に管理できるハイパーコール ([HvCallFlushVirtualAddressSpace](hypercalls/HvCallFlushVirtualAddressSpace.md)、 [hvcallflushvirtualaddressspace ex](hypercalls/HvCallFlushVirtualAddressSpaceEx.md)、 [hvcallflushvirtualaddresslist](hypercalls/HvCallFlushVirtualAddressList.md)、および [HvCallFlushVirtualAddressListEx](hypercalls/HvCallFlushVirtualAddressListEx.md)) を公開します。 L1 ハイパーバイザーでは、ゲストがこれらのハイパーコールを使用できるようにし、それを L0 ハイパーバイザーに処理する責任を委任することができます。 そのためには、対応 VMCSs と partition assist のサポートページを使用する必要があります。

対応 VMCSs が使用されている場合、仮想 TLB は、キャッシュされたすべてのマッピングを対応 VMCSS の識別子を使用してタグ付けします。 L2 ゲストからの直接の仮想フラッシュハイパーコールへの応答として、L0 ハイパーバイザーは、対応 VMCSs によって作成されたすべてのキャッシュされたマッピングを無効にします。

- VmId は呼び出し元の VmId と同じです
- 指定された ProcessorMask に VpId が含まれているか、HV_FLUSH_ALL_PROCESSORS が指定されています。

#### <a name="configuration"></a>構成

仮想フラッシュハイパーコールの直接処理は、次の方法で有効になります。

1. [仮想プロセッサのサポートページ](vp-properties.md#virtual-processor-assist-page)の DirectHypercall フィールドを1に設定しています (& s)。
2. 対応 VMCS の NestedFlushVirtualHypercall フィールドを1に設定します。

L1 ハイパーバイザーでは、有効にする前に、対応 VMCS の次の追加フィールドを構成する必要があります。

- VpId: 対応 VMCS によって制御される仮想プロセッサの ID。
- VmId: 対応 VMCS が属している仮想マシンの ID。
- PartitionAssistPage: partition assist ページのゲスト物理アドレス。

L1 ハイパーバイザーは、次の機能も、CPUID 経由でゲストに公開する必要があります。

- UseHypercallForLocalFlush
- UseHypercallForRemoteFlush

#### <a name="partition-assist-page"></a>パーティションのサポートページ

Partition assist ページは、L1 ハイパーバイザーが割り当てる必要があるメモリのページサイズの固定ページサイズの領域であり、direct flush ハイパーコールを使用できるようになる前に0になります。 GPA は、対応 VMCS 内の対応するフィールドに書き込まれる必要があります。

```c
struct
{
    UINT32 TlbLockCount;
} VM_PARTITION_ASSIST_PAGE;
```

#### <a name="synthetic-vm-exit"></a>合成 VM-Exit

呼び出し元のパーティションアシスタンスページの TlbLockCount が0以外の場合、L0 ハイパーバイザーは、直接の仮想フラッシュハイパーコールを処理した後、合成終了の理由がある VM-Exit を L1 ハイパーバイザーに配信します。

```c
#define HV_VMX_SYNTHETIC_EXIT_REASON_TRAP_AFTER_FLUSH 0x10000031
```

### <a name="second-level-address-translation"></a>Second Level Address Translation

入れ子になった仮想化がゲストパーティションに対して有効になっている場合、パーティションによって公開されているメモリ管理ユニット (MMU) には、第2レベルのアドレス変換のサポートが含まれます。 第2レベルのアドレス変換は、L1 ハイパーバイザーで物理メモリを仮想化するために使用できる機能です。 使用中の場合、ゲスト物理アドレス (GPAs) として扱われる特定のアドレスは、L2 ゲスト物理アドレス (L2 GPAs) として扱われ、一連のページング構造を走査することによって GPAs 変換されます。

L1 ハイパーバイザーでは、第2レベルのアドレス空間を使用する方法と場所を決定できます。 各第2レベルのアドレス空間は、ゲストが定義した64ビット ID 値によって識別されます。 Intel プラットフォームでは、この値は EPT PML4 テーブルのアドレスと同じです。

#### <a name="compatibility"></a>互換性

Intel プラットフォームでは、ハイパーバイザーによって公開される第2レベルのアドレス変換機能は、一般にアドレス変換の VMX サポートと互換性があります。 ただし、次のようなゲストに見える相違点があります。

- 内部的には、ハイパーバイザーは L2 GPAs SPAs として変換するシャドウページテーブルを使用する場合があります。 このような実装では、これらのシャドウページテーブルはソフトウェアに large TLBs として表示されます。 ただし、いくつかの違いが監視可能である場合があります。 まず、シャドウページテーブルは2つの仮想プロセッサ間で共有できます。一方、従来の TLBs はプロセッサごとの構造であり、独立しています。 この共有は表示されることがあります。1つの仮想プロセッサによるページアクセスで、後で別の仮想プロセッサによって使用されるシャドウページテーブルエントリがいっぱいになる可能性があるためです。
- 一部のハイパーバイザー実装では、ゲストページテーブルの内部書き込み保護を使用して、内部データ構造 (たとえば、シャドウページテーブル) からの MMU マッピングを遅延させることができます。 これらのテーブルへの書き込みは、ハイパーバイザーによって透過的に処理されるため、このようなアーキテクチャはゲストには見えません。 ただし、基になる GPA ページへの書き込みが他のパーティションまたはデバイスによって行われた場合、適切な TLB フラッシュがトリガーされないことがあります。
- 一部のハイパーバイザー実装では、第2レベルのページフォールト ("EPT 違反") によってキャッシュされたマッピングが無効にならないことがあります。

#### <a name="enlightened-second-level-tlb-flushes"></a>対応第2レベルの TLB フラッシュ

ハイパーバイザーでは、ゲストが第2レベルの TLB をより効率的に管理できるようにする拡張機能のセットもサポートしています。 これらの拡張操作は、従来の TLB 管理操作と同じ意味で使用できます。

ハイパーバイザーは、次のハイパーバイザーをサポートして TLBs を無効にします。

| ハイパーコール                                                                           | 説明                                     |
|-------------------------------------------------------------------------------------|-------------------------------------------------|
| [HvCallFlushGuestPhysicalAddressSpace](hypercalls/HvCallFlushGuestPhysicalAddressSpace.md)      | 2番目のレベルのアドレス空間内の GPA マッピングに対して、キャッシュされた L2 GPA を無効にします。   |
| [HvCallFlushGuestPhysicalAddressList](hypercalls/HvCallFlushGuestPhysicalAddressList.md)  | キャッシュされた GVA/L2 GPA を無効にします。    |

## <a name="nested-virtualization-exceptions"></a>入れ子になった仮想化の例外

L1 ハイパーバイザーは、ページフォールト例外クラスで仮想化例外を組み合わせることを選択できます。 L0 ハイパーバイザーは、ハイパーバイザーの入れ子になった仮想化機能の CPUID リーフで、この enlightment のサポートをアドバタイズします。

この啓蒙は、仮想プロセッサのサポートページの [HV_NESTED_ENLIGHTENMENTS_CONTROL](datatypes/HV_NESTED_ENLIGHTENMENTS_CONTROL.md) データ構造で VirtualizationException を "1" に設定することによって有効にすることができます。

## <a name="nested-virtualization-msrs"></a>入れ子になった仮想化 MSRs

### <a name="nested-vp-index-register"></a>入れ子になった VP インデックスレジスタ

L1 ハイパーバイザーは、現在のプロセッサの基になる VP インデックスを報告する MSR を公開します。

| MSR アドレス      | レジスタ名              | 説明                                                                 |
|------------------|----------------------------|-----------------------------------------------------------------------------|
| 0x40001002       | HV_X64_MSR_NESTED_VP_INDEX | 入れ子になったルートパーティションで、は現在のプロセッサの VP インデックスを報告します。 |

### <a name="nested-synic-msrs"></a>入れ子になった SynIC MSRs

入れ子になったルートパーティションでは、次の MSRs は、ベースハイパーバイザーの対応する [Synic MSRs](inter-partition-communication.md#synic-msrs) にアクセスできるようにします。

基になるプロセッサのインデックスを検索するには、呼び出し元はまず HV_X64_MSR_NESTED_VP_INDEX を使用する必要があります。

| MSR アドレス      | レジスタ名                 | 基になる MSR                                              |
|------------------|-------------------------------|--------------------------------------------------------------|
| 0x40001080       | HV_X64_MSR_NESTED_SCONTROL    | HV_X64_MSR_SCONTROL                                          |
| 0x40001081       | HV_X64_MSR_NESTED_SVERSION    | HV_X64_MSR_SVERSION                                          |
| 0x40001082       | HV_X64_MSR_NESTED_SIEFP       | HV_X64_MSR_SIEFP                                             |
| 0x40001083       | HV_X64_MSR_NESTED_SIMP        | HV_X64_MSR_SIMP                                              |
| 0x40001084       | HV_X64_MSR_NESTED_EOM         | HV_X64_MSR_EOM                                               |
| 0x40001090       | HV_X64_MSR_NESTED_SINT0       | HV_X64_MSR_SINT0                                             |
| 0x40001091       | HV_X64_MSR_NESTED_SINT1       | HV_X64_MSR_SINT1                                             |
| 0x40001092       | HV_X64_MSR_NESTED_SINT2       | HV_X64_MSR_SINT2                                             |
| 0x40001093       | HV_X64_MSR_NESTED_SINT3       | HV_X64_MSR_SINT3                                             |
| 0x40001094       | HV_X64_MSR_NESTED_SINT4       | HV_X64_MSR_SINT4                                             |
| 0x40001095       | HV_X64_MSR_NESTED_SINT5       | HV_X64_MSR_SINT5                                             |
| 0x40001096       | HV_X64_MSR_NESTED_SINT6       | HV_X64_MSR_SINT6                                             |
| 0x40001097       | HV_X64_MSR_NESTED_SINT7       | HV_X64_MSR_SINT7                                             |
| 0x40001098       | HV_X64_MSR_NESTED_SINT8       | HV_X64_MSR_SINT8                                             |
| 0x40001099       | HV_X64_MSR_NESTED_SINT9       | HV_X64_MSR_SINT9                                             |
| 0x4000109A       | HV_X64_MSR_NESTED_SINT10      | HV_X64_MSR_SINT10                                            |
| 0x4000109B       | HV_X64_MSR_NESTED_SINT11      | HV_X64_MSR_SINT11                                            |
| 0x4000109C       | HV_X64_MSR_NESTED_SINT12      | HV_X64_MSR_SINT12                                            |
| 0x4000109D       | HV_X64_MSR_NESTED_SINT13      | HV_X64_MSR_SINT13                                            |
| 0x4000109E       | HV_X64_MSR_NESTED_SINT14      | HV_X64_MSR_SINT14                                            |
| 0x4000109F       | HV_X64_MSR_NESTED_SINT15      | HV_X64_MSR_SINT15                                            |
