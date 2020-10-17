---
title: 仮想保護モード
description: 仮想保護モード
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: b858b24c8ddd672f8d109a1fbda4cb006437d68d
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120901"
---
# <a name="virtual-secure-mode"></a>仮想保護モード

仮想保護モード (VSM) は、ホストおよびゲストパーティションに提供されるハイパーバイザー機能と enlightenments のセットであり、オペレーティングシステムソフトウェア内で新しいセキュリティ境界を作成および管理することができます。 VSM は、Device Guard、Credential Guard、virtual Tpm、シールドされた Vm を含む Windows セキュリティ機能の基盤となるハイパーバイザー機能です。 これらのセキュリティ機能は、Windows 10 および Windows Server 2016 で導入されました。

VSM を使用すると、ルートパーティションおよびゲストパーティションのオペレーティングシステムソフトウェアで、システムセキュリティ資産の格納と処理のためにメモリの分離された領域を作成できます。 これらの分離されたリージョンへのアクセスは、ハイパーバイザーによってのみ制御および許可されます。これは、システムの信頼されたコンピューティングベース (TCB) の高度に信頼できる部分です。 ハイパーバイザーは、オペレーティングシステムソフトウェアよりも高い特権レベルで実行され、システムの初期化時に CPU MMU や IOMMU のメモリアクセス許可制御などの主要システムハードウェアリソースを排他的に制御しているため、ハイパーバイザーは、スーパーバイザーモードアクセス (CPL0 など) を使用して、これらの分離されたリージョンを不正アクセスから保護できます。、または "Ring 0") を呼び出します。

このアーキテクチャでは、スーパーバイザーモード (カーネルやドライバーなど) で実行されている通常のシステムレベルのソフトウェアが悪意のあるソフトウェアによって侵害された場合でも、ハイパーバイザーによって保護されている分離されたリージョンの資産をセキュリティで保護することができます。

## <a name="virtual-trust-level-vtl"></a>仮想信頼レベル (VTL)

VSM は、仮想信頼レベル (Vtl) を使用して分離を実現し、維持します。 Vtl は、パーティションごとと仮想ごとのプロセッサの両方で有効にして管理されます。

仮想信頼レベルは階層構造であり、より高いレベルの特権は下位レベルよりも高くなります。 VTL0 は最小限の特権レベルで、VTL1 は VTL0 よりも高い特権を持ち、VTL2 は VTL1 よりも高い特権を持ちます。

アーキテクチャ上、最大16レベルの Vtl がサポートされています。ただし、ハイパーバイザーは、16台未満の VTL を実装することを選択できます。 現時点では、2つの Vtl のみが実装されています。

 ```c
typedef UINT8 HV_VTL, *PHV_VTL;

#define HV_NUM_VTLS 2
#define HV_INVALID_VTL ((HV_VTL) -1)
#define HV_VTL_ALL 0xF
 ```

各 VTL には、独自のメモリアクセス保護のセットがあります。 これらのアクセス保護は、ハイパーバイザーによってパーティションの物理アドレス空間に管理されるため、パーティションで実行されているシステムレベルのソフトウェアでは変更できません。

特権を持つ Vtl の方が独自のメモリ保護を適用できるため、より大きな vtl では、下位の Vtl から効率的にメモリ領域を保護することができます。 実際には、これにより、より高い vtl で保護することで、分離されたメモリ領域を保護することができます。 たとえば、VTL0 は VTL1 にシークレットを格納することができ、その時点で VTL1 がアクセスできるようになります。 VTL0 が侵害された場合でも、シークレットは安全です。

### <a name="vtl-protections"></a>VTL による保護

Vtl 間の分離を実現するには、複数のファセットがあります。

- メモリアクセス保護: 各 VTL は、一連のゲスト物理メモリアクセス保護を保持します。 特定の VTL で実行されているソフトウェアは、これらの保護に従ってのみメモリにアクセスできます。
- 仮想プロセッサの状態: 仮想プロセッサは、VTL ごとに独立した状態を維持します。 たとえば、各 VTL は、プライベート VP レジスタのセットを定義します。 下位の VTL で実行されているソフトウェアは、より高い VTL のプライベート仮想プロセッサの登録状態にアクセスできません。
- 割り込み: 個別のプロセッサ状態と共に、各 VTL にも独自の割り込みサブシステム (ローカル APIC) があります。 これにより、より高い vtl では、より低い VTL からの干渉を受けることなく割り込みを処理できます。
- オーバーレイページ: 特定のオーバーレイページは、より高い Vtl が信頼性の高いアクセスを実現するように、VTL ごとに保持されます。 例: VTL ごとに個別のハイパーコールオーバーレイページがあります。

## <a name="vsm-detection-and-status"></a>VSM の検出と状態

VSM 機能は、AccessVsm パーティションの特権フラグを使用してパーティションに提供されます。 次のすべての特権を持つパーティションのみ、VSM: AccessVsm、AccessVpRegisters、および AccessSynicRegs を利用できます。

### <a name="vsm-capability-detection"></a>VSM 機能の検出

各ゲストは、次のモデル固有の登録を使用して、VSM の機能に関するレポートにアクセスする必要があります。

| MSR アドレス      | レジスタ名                   | 説明                                                    |
|------------------|---------------------------------|----------------------------------------------------------------|
| 0x000D0006       | HV_X64_REGISTER_VSM_CAPABILITIES| VSM の機能についてレポートします。                                    |

VSM の登録機能 MSR の形式は次のとおりです。

| Bits          | 説明                         | 属性                                                  |
|---------------|-------------------------------------|-------------------------------------------------------------|
| 63            | Dr6Shared                           | Read                                                        |
| 62:47         | Mv のマスク                         | Read                                                        |
| 46            | Denyvtlの起動                 | Read                                                        |
| 45:0          | RsvdZ                               | Read                                                        |

Dr6Shared は、Dr6 が Vtl 間の共有レジスタであるかどうかをゲストに示します。

MvecVtlMask は、Mbec を有効にできる Vtl をゲストに示します。

Denylower Vtlstartup は、Vtl が下位の VTL によってリセットされた VP を拒否できるかどうかをゲストに示します。

### <a name="vsm-status-register"></a>VSM ステータスレジスタ

パーティション特権フラグに加えて、2つの仮想レジスタを使用して、VSM ステータス (および) に関する追加情報を学習できます `HvRegisterVsmPartitionStatus` `HvRegisterVsmVpStatus` 。

#### <a name="hvregistervsmpartitionstatus"></a>HvRegisterVsmPartitionStatus

HvRegisterVsmPartitionStatus は、すべての Vtl 間で共有される、パーティションごとの読み取り専用レジスタです。 この登録により、どの vtl がパーティションに対して有効になっているかに関する情報が提供されます。これには、モードベースの実行制御が有効になっている vtl と、許可される最大の VTL が含まれます。

```c
typedef union
{
    UINT64 AsUINT64;
    struct {
        UINT64 EnabledVtlSet : 16;
        UINT64 MaximumVtl : 4;
        UINT64 MbecEnabledVtlSet: 16;
        UINT64 ReservedZ : 28;
    };
} HV_REGISTER_VSM_PARTITION_STATUS;
```

#### <a name="hvregistervsmvpstatus"></a>HvRegisterVsmVpStatus

HvRegisterVsmVpStatus は読み取り専用レジスタで、すべての Vtl 間で共有されます。 これは VP ごとの登録です。つまり、各仮想プロセッサは独自のインスタンスを保持します。 この登録では、有効になっている、アクティブな Vtl、および VP でアクティブになっている MBEC モードに関する情報を提供します。

```c
typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 ActiveVtl : 4;
        UINT64 ActiveMbecEnabled : 1;
        UINT64 ReservedZ0 : 11;
        UINT64 EnabledVtlSet : 16;
        UINT64 ReservedZ1 : 32;
    };
} HV_REGISTER_VSM_VP_STATUS;
```

ActiveVtl は、仮想プロセッサで現在アクティブになっている VTL コンテキストの ID です。

ActivemMBEC Enabled は、仮想プロセッサで現在アクティブになっていることを指定します。

EnabledVtlSet は、仮想プロセッサ上で有効になっている VTL のビットマップです。

### <a name="partition-vtl-initial-state"></a>パーティション VTL の初期状態

パーティションが開始またはリセットされると、VTL0 で実行が開始されます。 その他のすべての Vtl は、パーティションの作成時に無効になります。

## <a name="vtl-enablement"></a>VTL の有効化

VTL の使用を開始するには、以下を開始する必要があります。

1. パーティションのターゲット VTL を有効にします。 これにより、パーティションで VTL を一般に利用できるようになります。
2. 1つ以上の仮想プロセッサでターゲットの VTL を有効にします。 これにより、VP で VTL を使用できるようになり、初期コンテキストが設定されます。 すべての VPs で同じ有効な Vtl を使用することをお勧めします。 一部の VPs (すべてではない) で VTL を有効にすると、予期しない動作が発生する可能性があります。
3. 1つのパーティションと VP に対して VTL が有効になると、EnableVtlProtection フラグが設定された後、アクセス保護の設定を開始できます。

Vtl は連続している必要はないことに注意してください。

### <a name="enabling-a-target-vtl-for-a-partition"></a>パーティションのターゲットの VTL を有効にする

[Hvcallenablepartitionvtl](hypercalls/HvCallEnablePartitionVtl.md)ハイパーコールは、特定のパーティションに対して VTL を有効にするために使用されます。 ソフトウェアを実際に特定の VTL で実行するには、その前に、パーティション内の仮想プロセッサでその VTL を有効にする必要があることに注意してください。

### <a name="enabling-a-target-vtl-for-virtual-processors"></a>仮想プロセッサのターゲット VTL の有効化

パーティションに対して有効になった VTL は、パーティションの仮想プロセッサで有効にすることができます。 [HvCallEnableVpVtl](hypercalls/HvCallEnableVpVtl.md)ハイパーコールを使用すると、仮想プロセッサの vtl を有効にすることができます。これにより、初期のコンテキストが設定されます。

仮想プロセッサには、VTL ごとに1つの "コンテキスト" があります。 VTL が切り替わると、VTL の [プライベート状態](#private-state) も切り替わります。

## <a name="vtl-configuration"></a>VTL の構成

VTL が有効になったら、同等以上の VTL で実行されている VP によって、その構成を変更できます。

### <a name="partition-configuration"></a>パーティション構成

パーティション全体の属性は、HvRegisterVsmPartitionConfig レジスタを使用して構成できます。 各パーティションには、各 VTL (0 を超える) に対してこのレジスタのインスタンスが1つあります。

各 VTL は、HV_REGISTER_VSM_PARTITION_CONFIG の独自のインスタンスと、下位の Vtl のインスタンスを変更できます。 Vtl では、より高い Vtl に対してこのレジスタを変更することはできません。

```c
typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 EnableVtlProtection : 1;
        UINT64 DefaultVtlProtectionMask : 4;
        UINT64 ZeroMemoryOnReset : 1;
        UINT64 DenyLowerVtlStartup : 1;
        UINT64 ReservedZ : 2;
        UINT64 InterceptVpStartup : 1;
        UINT64 ReservedZ : 54; };
} HV_REGISTER_VSM_PARTITION_CONFIG;
```

このレジスタのフィールドについては、以下で説明します。

#### <a name="enable-vtl-protections"></a>VTL の保護を有効にする

VTL が有効になったら、メモリ保護の適用を開始する前に EnableVtlProtection フラグを設定する必要があります。
このフラグは1回だけ書き込まれます。つまり、いったん設定されると、変更することはできません。

#### <a name="default-protection-mask"></a>既定の保護マスク

既定では、システムは、現在マップされているすべてのページ、および今後の "ホット追加" ページに RWX 保護を適用します。 ホット追加ページは、サイズ変更操作中にパーティションに追加されたメモリを参照します。

より高い VTL では、HV_REGISTER_VSM_PARTITION_CONFIG で DefaultVtlProtectionMask を指定することによって、別の既定のメモリ保護ポリシーを設定できます。 このマスクは、VTL が有効になっているときに設定する必要があります。 いったん設定した後は変更できず、パーティションのリセットによってのみクリアされます。

| ビット       | 説明                                                                          |
|-----------|--------------------------------------------------------------------------------------|
| 0         | Read                                                                                 |
| 1         | Write                                                                                |
| 2         | ユーザーモードの実行 (UMX)                                                              |
| 3         | カーネルモードの実行 (KMX)                                                            |

#### <a name="zero-memory-on-reset"></a>リセット時のメモリのゼロ

ゼロ Memonreset は、パーティションをリセットする前にメモリをゼロにするかどうかを制御するビットです。 この構成は既定でオンになっています。 ビットが設定されている場合は、リセット時にパーティションのメモリがゼロになり、より低い vtl でより多くのメモリを損なうことがなくなります。
このビットがオフの場合、リセット時にパーティションのメモリがゼロになりません。

#### <a name="denylowervtlstartup"></a>Denyvtlの起動

Denylower Vtlstartup フラグは、仮想プロセッサが下位の Vtl によって起動またはリセットされるかどうかを制御します。 これには、仮想プロセッサ (X64 の SIPI など) および [Hvcallstartvirtualprocessor](hypercalls/HvCallStartVirtualProcessor.md) ハイパーコールをリセットするアーキテクチャ上の方法が含まれます。

#### <a name="interceptvpstartup"></a>InterceptVpStartup

InterceptVpStartup フラグが設定されている場合、仮想プロセッサを開始またはリセットすると、より高い VTL に対するインターセプトが生成されます。

### <a name="configuring-lower-vtls"></a>下位の Vtl の構成

次のレジスタを使用して、より低い vtl の動作を構成することができます。

```c
typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 MbecEnabled : 1;
        UINT64 TlbLocked : 1;
        UINT64 ReservedZ : 62;
    };
} HV_REGISTER_VSM_VP_SECURE_VTL_CONFIG;
```

各 VTL (0 を超える) は、それ自体より下位のすべての VTL に対してこのレジスタのインスタンスを持ちます。 たとえば、VTL2 には、このレジスタの2つのインスタンス (VTL1 用と VTL0 用の2つのインスタンス) があります。

このレジスタのフィールドについては、以下で説明します。

#### <a name="mbecenabled"></a>Mcfm が有効

このフィールドでは、下位の VTL に対して MBEC を有効にするかどうかを構成します。

#### <a name="tlblocked"></a>TlbLocked

このフィールドは、下位の VTL の TLB をロックします。 この機能を使用すると、より高い VTL に干渉する可能性がある TLB の無効化を防ぐことができます。 このビットが設定されている場合、下位の VTL からのすべてのアドレス空間フラッシュ要求は、ロックが解除されるまでブロックされます。

TLB のロックを解除するために、より高い VTL がこのビットをクリアできます。 また、VP が下位の VTL に戻ると、その時点で保持されているすべての TLB ロックが解放されます。

## <a name="vtl-entry"></a>VTL エントリ

VP が下位の VTL から高いものに切り替わると、VTL は "入力済み" になります。 これが発生する理由としては次のようなことが考えられます。

1. VTL 呼び出し: これは、ソフトウェアがより高い VTL でコードを呼び出すことを明示的に希望している場合です。
2. セキュリティで保護された割り込み: より高い VTL で割り込みを受け取った場合、VP はより高い VTL を入力します。
3. セキュリティで保護されたインターセプト: 特定のアクションによってセキュリティで保護された割り込みが発生します (特定の MSRs にアクセスするなど)。

VTL を入力したら、自発的に終了する必要があります。 下位の vtl では、より高い VTL を横取りすることはできません。

### <a name="identifying-vtl-entry-reason"></a>VTL エントリの理由を特定しています

エントリに適切に対応するために、より高い VTL では、入力された理由を知る必要があります。 エントリの理由を識別するために、VTL エントリは [HV_VP_VTL_CONTROL](datatypes/HV_VP_VTL_CONTROL.md) 構造に含まれています。

### <a name="vtl-call"></a>VTL 呼び出し

"VTL 呼び出し" とは、下位の vtl が [Hvcallvtlcall](hypercalls/HvCallVtlCall.md) ハイパーコールを通じてより高い vtl にエントリを (たとえば、より高い量の vtl で保護するために) 開始する場合です。

VTL 呼び出しでは、VTL スイッチ全体で共有レジスタの状態が保持されます。 プライベートレジスタは、各 VTL レベルで保持されます。 これらの制限の例外は、VTL 呼び出しシーケンスで必要とされるレジスタです。 次のレジスタは、VTL 呼び出しに必要です。

| X64     | x86     | 説明                                                 |
|---------|---------|-------------------------------------------------------------|
| RCX     | EDX: EAX | ハイパーバイザーへの VTL 呼び出し制御入力を指定します        |
| RAX     | ECX     | 予約済み                                                    |

現在、VTL 呼び出し制御入力のすべてのビットが予約されています。

#### <a name="vtl-call-restrictions"></a>VTL 呼び出しの制限

VTL 呼び出しは、最も特権の高いプロセッサモードからのみ開始できます。 たとえば、x64 システムでは、VTL 呼び出しは CPL0 からのみ取得できます。 プロセッサモードから開始された VTL 呼び出しは、システム上で最も権限の高いものですが、ハイパーバイザーは #UD 例外を仮想プロセッサに挿入します。

VTL 呼び出しは、次の最高の VTL にのみ切り替えることができます。 つまり、複数の Vtl が有効になっている場合、1回の呼び出しでは、VTL を "スキップ" できません。
次のアクションを実行すると #UD 例外が発生します。

- プロセッサモードから開始された VTL 呼び出し (システム上で最も特権が高いが、アーキテクチャ固有のもの)。
- Real モードからの VTL 呼び出し (x86/x64)
- ターゲットの VTL が無効になっている (または既に有効になっていない) 仮想プロセッサでの VTL 呼び出し。
- 無効な制御入力値を持つ VTL 呼び出し

## <a name="vtl-exit"></a>VTL の終了

下位の VTL への切り替えは "return" と呼ばれます。 VTL が処理を完了すると、下位の VTL に切り替えるために、VTL の戻り値を開始できます。 VTL が返される唯一の方法は、より高い VTL が自発的に開始する場合です。 低い VTL では、より高い1つを横取りすることはできません。

### <a name="vtl-return"></a>VTL の戻り値

"VTL の戻り値" とは、 [Hvcallvtlreturn](hypercalls/HvCallVtlReturn.md) ハイパーコールを通じて、より上位の vtl へのスイッチの起動が行われる場合です。 VTL 呼び出しと同様に、プライベートプロセッサの状態は切り替えられ、共有状態はそのまま維持されます。 下位の VTL が明示的に上位の VTL に呼び出された場合、ハイパーバイザーは、戻りが完了する前に、より高い VTL の命令ポインターをインクリメントして、VTL 呼び出しの後も続行できるようにします。

VTL のリターンコードシーケンスでは、次のレジスタを使用する必要があります。

| X64     | x86     | 説明                                                 |
|---------|---------|-------------------------------------------------------------|
| RCX     | EDX: EAX | VTL によるハイパーバイザーへのリターン制御入力を指定します。      |
| RAX     | ECX     | 予約済み                                                    |

VTL の戻り制御入力の形式は次のとおりです。

| Bits      | フィールド           | 説明                                                                 |
|-----------|-----------------|-----------------------------------------------------------------------------|
| 63:1      | RsvdZ           |                                                                             |
| 0         | 高速復帰     | レジスタは復元されません                                                  |

次のアクションを実行すると #UD 例外が生成されます。

- 最低の VTL が現在アクティブになっているときに、VTL を返します。
- 無効な制御入力値を使用して、VTL を返しようとしています
- システム上で最も特権が高い (アーキテクチャ固有の) プロセッサモードから、VTL を返します。

#### <a name="fast-return"></a>高速復帰

ハイパーバイザーでは、戻り値の処理の一環として、 [HV_VP_VTL_CONTROL](datatypes/HV_VP_VTL_CONTROL.md) 構造から、より低い VTL の登録状態を復元できます。 たとえば、セキュリティで保護された割り込みを処理した後は、より低い vtl の状態を損なうことなく、より高い VTL で戻ることができます。 そのため、ハイパーバイザーは、より低い VTL のレジスタを、VTL 制御構造に格納されている事前呼び出し値に単に復元するメカニズムを提供します。

この動作が必要でない場合は、より高い VTL で "高速リターン" を使用できます。 高速な戻りは、ハイパーバイザーが制御構造からレジスタ状態を復元しない場合です。 これは、不要な処理を回避するために、可能な限り利用する必要があります。

このフィールドは、VTL が返す入力のビット0を使用して設定できます。 0に設定されている場合、レジスタは HV_VP_VTL_CONTROL 構造から復元されます。 このビットが1に設定されている場合、レジスタは復元されません (高速に戻ります)。

## <a name="hypercall-page-assist"></a>ハイパーコール Page Assist

ハイパーバイザーは、 [ハイパーコールページ](hypercall-interface.md#establishing-the-hypercall-interface)を介して、VTL の呼び出しと戻りを支援するメカニズムを提供します。 このページでは、Vtl を切り替えるために必要な特定のコードシーケンスを抽象化します。

ハイパーコールページで特定の命令を実行すると、VTL の呼び出しと戻り値を実行するコードシーケンスにアクセスできます。 呼び出し/リターンチャンクは、HvRegisterVsmCodePageOffset 仮想レジスタによって決定されるハイパーコールページのオフセット位置にあります。 これは読み取り専用で、パーティション全体のレジスタであり、各 VTL に個別のインスタンスがあります。

VTL は、呼び出し命令を使用して、VTL 呼び出し/復帰を実行できます。 ハイパーコールページ内の正しい場所を呼び出すと、VTL 呼び出し/戻りが開始されます。

```c
typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 VtlCallOffset : 12;
        UINT64 VtlReturnOffset : 12;
        UINT64 ReservedZ : 40;
    };
} HV_REGISTER_VSM_CODE_PAGE_OFFSETS;
```

要約すると、ハイパーコールページを使用してコードシーケンスを呼び出す手順は次のとおりです。

1. ハイパーコールページを VTL の GPA space にマップする
2. コードシーケンス (VTL 呼び出しまたはリターン) の正しいオフセットを決定します。
3. 呼び出しを使用してコードシーケンスを実行します。

## <a name="memory-access-protections"></a>メモリアクセス保護

VSM によって提供される保護の1つに、メモリアクセスを分離する機能があります。

より高い Vtl では、下位の Vtl で許容されるメモリアクセスの種類を高いレベルで制御できます。 特定の GPA ページ (読み取り、書き込み、実行) に対してより高い VTL で指定できる保護には、3つの基本的な種類があります。 これらは、次の表で定義されています。

| 名前                       | 説明                                                         |
|----------------------------|---------------------------------------------------------------------|
| Read                       | メモリページへの読み取りアクセスを許可するかどうかを制御します            |
| Write                      | メモリページへの書き込みアクセスを許可するかどうかを制御します              |
| 実行                    | メモリページで命令フェッチが許可されるかどうかを制御します。 |

これら3つの組み合わせにより、次の種類のメモリ保護が組み合わされます。

1. アクセス権なし
2. 読み取り専用、実行なし
3. 読み取り専用、実行
4. 読み取り/書き込み、実行なし
5. 読み取り/書き込み、実行

"Mode based execution control (MBEC))" が enabed の場合、ユーザーとカーネルモードの実行保護を個別に設定できます。

より高い Vtl では、 [Hvcallmodifyvtlprotectionmask](hypercalls/HvCallModifyVtlProtectionMask.md) ハイパーコールを介して GPA のメモリ保護を設定できます。

### <a name="memory-protection-hierarchy"></a>メモリ保護階層

メモリアクセスのアクセス許可は、特定の VTL のさまざまなソースによって設定できます。 各 VTL のアクセス許可は、ホストパーティションだけでなく、他の多くの Vtl によって制限される可能性があります。 保護を適用する順序は次のとおりです。

1. ホストによって設定されたメモリ保護
2. より高い Vtl によって設定されるメモリ保護

つまり、VTL による保護はホストの保護を優先します。 上位レベルの Vtl は下位レベルの Vtl を優先します。 VTL では、それ自体にメモリアクセス許可を設定することはできないことに注意してください。

準拠しているインターフェイスは、ram 以外の種類の RAM をオーバーレイしないことが想定されています。

### <a name="memory-access-violations"></a>メモリアクセス違反

低い VTL で実行されている VP が、より高い VTL によって設定されたメモリ保護に違反しようとすると、インターセプトが生成されます。 このインターセプトは、保護を設定する上位の VTL によって受信されます。 これにより、大規模な Vtl はケースバイケースで違反に対処できます。 たとえば、より高い VTL では、エラーを返すか、またはアクセスをエミュレートすることができます。

### <a name="mode-based-execute-control-mbec"></a>モードベースの実行制御 (MBEC)

VTL が下位の VTL にメモリ制限を設定する場合、"実行" 特権を付与するときに、ユーザーモードとカーネルモードを区別することが必要になる場合があります。 たとえば、コードの整合性チェックをより高い VTL で実行する場合、ユーザーモードとカーネルモードを区別できることは、カーネルモードアプリケーションに対してのみコードの整合性を適用できることを意味します。

従来の3つのメモリ保護 (読み取り、書き込み、実行) とは別に、MBEC では、ユーザーモードとカーネルモードを区別して実行保護を行います。 したがって、MBEC が有効になっている場合、VTL は次の4種類のメモリ保護を設定できます。

| 名前                       | 説明                                                         |
|----------------------------|---------------------------------------------------------------------|
| Read                       | メモリページへの読み取りアクセスを許可するかどうかを制御します            |
| Write                      | メモリページへの書き込みアクセスを許可するかどうかを制御します              |
| ユーザーモードの実行 (UMX)    | ユーザーモードで生成された命令フェッチをメモリページで許可するかどうかを制御します。 注: MBEC が無効になっている場合、この設定は無視されます。 |
| カーネルモードの実行 (UMX)  | メモリページに対してカーネルモードで生成された命令フェッチを許可するかどうかを制御します。 注: MBEC が無効になっている場合、この設定により、ユーザーモードとカーネルモードの両方の実行アクセスが制御されます。 |

"ユーザーモードで実行" 保護でマークされたメモリは、仮想プロセッサがユーザーモードで実行されている場合にのみ実行可能です。 同様に、"カーネルモードの実行" メモリは、仮想プロセッサがカーネルモードで実行されている場合にのみ実行可能です。

KMX と UMX は、ユーザーモードとカーネルモードで異なる実行アクセス許可が適用されるように、個別に設定できます。 KMX = 1、UMX = 0 を除き、UMX と KMX のすべての組み合わせがサポートされます。 この組み合わせの動作は定義されていません。

MBEC は、すべての Vtl および仮想プロセッサに対して既定で無効になっています。 MBEC を無効にすると、カーネルモードの execute ビットによってメモリアクセスの制限が決まります。 したがって、MBEC が無効になっている場合、KMX = 1 コードは、カーネルモードとユーザーモードの両方で実行できます。

#### <a name="descriptor-tables"></a>記述子テーブル

記述子テーブルにアクセスするすべてのユーザーモードコードは、KMX = UMX = 1 とマークされている GPA ページ内にある必要があります。 KMX = 0 とマークされた GPA ページから、記述子テーブルにアクセスするユーザーモードのソフトウェアはサポートされていないため、一般的な保護エラーが発生します。

#### <a name="mbec-configuration"></a>MBEC の構成

モードベースの実行制御を使用するには、次の2つのレベルで有効にする必要があります。

1. パーティションに対して VTL が有効になっている場合は、HvCallEnablePartitionVtl を使用して MBEC を有効にする必要があります。
2. MBEC は、HvRegisterVsmVpSecureVtlConfig を使用して、VP 単位および VTL 単位で構成する必要があります。

#### <a name="mbec-interaction-with-supervisor-mode-execution-prevention-smep"></a>MBEC とスーパーバイザーモードの実行防止 (SMEP)

Supervisor-Mode 実行防止 (SMEP) は、一部のプラットフォームでサポートされているプロセッサ機能です。 SMEP は、メモリページへのスーパーバイザーアクセスが制限されているため、MBEC の操作に影響を与える可能性があります。 ハイパーバイザーは、SMEP に関連する次のポリシーに準拠しています。

- ゲスト OS で SMEP を使用できない場合 (ハードウェア機能またはプロセッサ互換性モードのいずれが原因であるかにかかわらず)、MBEC は影響を受けません。
- SMEP が使用可能で、が有効になっている場合、MBEC は影響を受けません。
- SMEP が使用可能で、が無効になっている場合、すべての実行制限は KMX コントロールによって管理されます。 したがって、KMX = 1 とマークされたコードのみ実行が許可されます。

## <a name="virtual-processor-state-isolation"></a>仮想プロセッサの状態の分離

仮想プロセッサは、アクティブな各 VTL に対して別々の状態を保持します。 ただし、この状態の一部は特定の VTL に対してプライベートであり、残りの状態はすべての Vtl 間で共有されます。

VTL ごとに保持される状態 ( プライベート状態) は、VTL 遷移間でハイパーバイザーによって保存されます。 VTL スイッチが開始されると、ハイパーバイザーはアクティブな VTL の現在のプライベート状態を保存し、ターゲットの VTL のプライベート状態に切り替えます。 共有状態は、VTL スイッチに関係なくアクティブなままです。

### <a name="private-state"></a>プライベート状態

一般に、各 VTL には、独自のコントロールレジスタ、RIP レジスタ、RSP レジスタ、および MSRs があります。 次に示すのは、各 VTL に対してプライベートな特定のレジスタと MSRs の一覧です。

プライベート MSRs:

- SYSENTER_CS、SYSENTER_ESP、SYSENTER_EIP、STAR、LSTAR、CSTAR、SFMASK、EFER、PAT、KERNEL_GSBASE、FS。BASE、GS。BASE、TSC_AUX
- HV_X64_MSR_HYPERCALL
- HV_X64_MSR_GUEST_OS_ID
- HV_X64_MSR_REFERENCE_TSC
- HV_X64_MSR_APIC_FREQUENCY
- HV_X64_MSR_EOI
- HV_X64_MSR_ICR
- HV_X64_MSR_TPR
- HV_X64_MSR_APIC_ASSIST_PAGE
- HV_X64_MSR_NPIEP_CONFIG
- HV_X64_MSR_SIRBP
- HV_X64_MSR_SCONTROL
- HV_X64_MSR_SVERSION
- HV_X64_MSR_SIEFP
- HV_X64_MSR_SIMP
- HV_X64_MSR_EOM
- HV_X64_MSR_SINT0 – HV_X64_MSR_SINT15
- HV_X64_MSR_STIMER0_CONFIG – HV_X64_MSR_STIMER3_CONFIG
- HV_X64_MSR_STIMER0_COUNT – HV_X64_MSR_STIMER3_COUNT
- ローカル APIC レジスタ (CR8/TPR を含む)

プライベートレジスタ:

- RIP、RSP
- RFLAGS
- CR0 レジスタ、CR3、CR4
- DR7
- IDTR、GDTR
- CS、DS、ES、FS、GS、SS、TR、LDTR
- TSC
- DR6 (* プロセッサの種類に依存します。 共有/プライベート状態を確認するには、HvRegisterVsmCapabilities 仮想レジスタを読み取ります)

### <a name="shared-state"></a>共有状態

コンテキストの切り替えのオーバーヘッドを削減するために、Vtl は状態を共有します。 また、状態の共有によって、Vtl 間でいくつかの必要な通信が可能になります 最も一般的な目的および浮動小数点レジスタは、ほとんどのアーキテクチャ MSRs と同様に共有されます。 すべての Vtl 間で共有される特定の MSRs とレジスタの一覧を次に示します。

共有 MSRs:

- HV_X64_MSR_TSC_FREQUENCY
- HV_X64_MSR_VP_INDEX
- HV_X64_MSR_VP_RUNTIME
- HV_X64_MSR_RESET
- HV_X64_MSR_TIME_REF_COUNT
- HV_X64_MSR_GUEST_IDLE
- HV_X64_MSR_DEBUG_DEVICE_OPTIONS
- MTRRs
- MCG_CAP
- MCG_STATUS

共有レジスタ:

- Rax、Rbx、Rcx、Rdx、Rsi、Rdi、Rbx
- CR2
- R8 – R15
- DR0 – DR5
- X87 浮動小数点状態
- XMM の状態
- AVX の状態
- XCR0 (XFEM)
- DR6 (* プロセッサの種類に依存します。 共有/プライベート状態を確認するには、HvRegisterVsmCapabilities 仮想レジスタを読み取ります)

### <a name="real-mode"></a>Real モード

Real モードは、0より大きいすべての VTL ではサポートされていません。 0より大きい Vtl は、32ビットモードまたは64ビットモードで実行できます。

## <a name="vtl-interrupt-management"></a>VTL 割り込み管理

仮想の信頼レベル間で高レベルの分離を実現するために、仮想保護モードでは、仮想プロセッサ上で有効になっている各 VTL に対して個別の割り込みサブシステムを提供します。 これにより、より安全性の低い VTL からの干渉なしに、確実に割り込みを送受信できるようになります。

各 VTL には独自の割り込みコントローラーがあり、仮想プロセッサがその特定の VTL で実行されている場合にのみアクティブになります。 仮想プロセッサが VTL の状態を切り替えると、プロセッサ上でアクティブになっている割り込みコントローラーも切り替えられます。

アクティブな vtl を上回る VTL をターゲットにした割り込みは、即座に VTL スイッチになります。 その後、より高い VTL で割り込みを受け取ることができます。 その TPR/CR8 値が原因で、より高い VTL が割り込みを受信できない場合、割り込みは "保留中" として保持され、VTL は切り替えられません。 割り込みが保留になっている複数の Vtl がある場合は、最高の VTL が優先されます (下位の VTL には注意が必要です)。

割り込みが下位の VTL を対象としている場合、その割り込みは、仮想プロセッサが次に対象となる VTL に移行するまでは配信されません。 より低い vtl が有効になっている仮想プロセッサでは、INIT と startup IPIs が下位の仮想プロセッサにドロップされます。 INIT/SIPI はブロックされるため、 [Hvcallstartvirtualprocessor](hypercalls/HvCallStartVirtualProcessor.md) ハイパーコールを使用してプロセッサを開始する必要があります。

### <a name="rflagsif"></a>RFLAGS。もし

Vtl を切り替えるために、RFLAGS を使用します。が、セキュリティで保護された割り込みによって VTL スイッチがトリガーされるかどうかに影響しない場合。 RFLAGS の場合は。割り込みをマスクするためにがオフになっている場合でも、より高い vtl に割り込むと、VTL はより高い VTL に切り替わります。 すぐに中断するかどうかを決定する際には、より高い VTL の TPR/CR8 値だけが考慮されます。

この動作は、VTL がを返すときの保留中の割り込みにも影響します。 RFLAGS の場合は。指定された VTL の割り込みをマスクするために bit がクリアされ、(下位の VTL に) VTL からが返された場合、ハイパーバイザーは保留中の割り込みを再評価します。 これにより、より高い VTL に直ちにコールバックします。

### <a name="virtual-interrupt-notification-assist"></a>仮想割り込み通知支援

同じ仮想プロセッサの下位の VTL に対する割り込みの即時配信をブロックしている場合は、より高い Vtl で通知を受け取ることができます。 より高い Vtl では、仮想レジスタ HvRegisterVsmVina を介して仮想割り込み通知支援 (VINA) を有効にすることができます。

```c
typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 Vector : 8;
        UINT64 Enabled : 1;
        UINT64 AutoReset : 1;
        UINT64 AutoEoi : 1;
        UINT64 ReservedP : 53;
    };
} HV_REGISTER_VSM_VINA;
```

各 VP の各 VTL には、独自の VINA インスタンスと HvRegisterVsmVina のバージョンがあります。 VINA ファシリティは、下位の VTL の割り込みが即時配信の準備ができている場合に、現在アクティブな上位の VTL に対してエッジによってトリガーされる割り込みを生成します。

このファシリティが有効になっているときに割り込みが大量に発生するのを防ぐために、VINA ファシリティには一定の状態が含まれています。 VINA の割り込みが生成されると、VINA ファシリティの状態は "アサート済み" に変更されます。 VINA ファシリティに関連付けられているシントに割り込みの終了を送信しても、"アサート" 状態はクリアされません。 アサート状態は、次の2つの方法のいずれかでのみ消去できます。

1. 状態は、 [HV_VP_VTL_CONTROL](datatypes/HV_VP_VTL_CONTROL.md) 構造体の VinaAsserted フィールドに書き込むことによって、手動で消去できます。
2. この状態は、HvRegisterVsmVina レジスタで [自動リセット (VTL) の自動リセット] オプションが有効になっている場合に、次の VTL へのエントリで自動的にクリアされます。

これにより、セキュリティで保護された VTL で実行されるコードは、下位の VTL に対して受信された最初の割り込みを通知するだけで済みます。 セキュリティで保護された VTL が追加の割り込みを通知する場合は、VP アシストページの VinaAsserted フィールドをクリアして、次の新しい割り込みが通知されます。

## <a name="secure-intercepts"></a>セキュリティで保護されたインターセプト

ハイパーバイザーを使用すると、より低い vtl のコンテキストで実行されるイベントに対して、より高い VTL でインターセプトをインストールできます。 これにより、より高いレベルの vtl が、より低い VTL リソースに対して高いレベルで制御できるようになります。 セキュリティで保護されたインターセプトを使用すると、システムに不可欠なリソースを保護し、下位の Vtl からの攻撃を防ぐことができます。

セキュリティで保護されたインターセプトは、より高い VTL をキューに配置し、その VTL を VP で実行可能にします。

### <a name="secure-intercept-types"></a>セキュリティで保護されたインターセプトの種類

| インターセプトの種類             | インターセプトの適用先                                                |
|----------------------------|---------------------------------------------------------------------|
| メモリアクセス              | より高い VTL によって確立された GPA 保護にアクセスしようとしています。   |
| レジスタアクセスの制御    | 上位の VTL によって指定された一連のコントロールレジスタにアクセスしようとしています。 |

### <a name="nested-intercepts"></a>入れ子になったインターセプト

複数の Vtl では、下位の VTL で同じイベントに対してセキュリティで保護されたインターセプトをインストールできます。 したがって、入れ子になったインターセプトが通知される場所を決定する階層が確立されます。 インターセプトが通知される順序は次のとおりです。

1. 下位の VTL
2. より高い VTL

### <a name="handling-secure-intercepts"></a>セキュリティで保護されたインターセプトの処理

VTL にセキュリティで保護されたインターセプトが通知されたら、下位の VTL を続行できるようにする必要があります。
より高い VTL では、例外の挿入、アクセスのエミュレート、アクセスへのプロキシの提供など、さまざまな方法でインターセプトを処理できます。 どのような場合でも、下位の VTL VP のプライベート状態を変更する必要がある場合は、 [HvCallSetVpRegisters](hypercalls/HvCallSetVpRegisters.md) を使用する必要があります。

### <a name="secure-register-intercepts"></a>Secure Register インターセプト

より高い VTL が特定のコントロールレジスタへのアクセスを傍受する可能性があります。 これを実現するには、 [HvCallSetVpRegisters](hypercalls/HvCallSetVpRegisters.md) ハイパーコールを使用して HvX64RegisterCrInterceptControl を設定します。 HvX64RegisterCrInterceptControl の制御ビットを設定すると、対応するコントロールレジスタのすべてのアクセスに対してインターセプトがトリガーされます。

```c
typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 Cr0Write : 1;
        UINT64 Cr4Write : 1;
        UINT64 XCr0Write : 1;
        UINT64 IA32MiscEnableRead : 1;
        UINT64 IA32MiscEnableWrite : 1;
        UINT64 MsrLstarRead : 1;
        UINT64 MsrLstarWrite : 1;
        UINT64 MsrStarRead : 1;
        UINT64 MsrStarWrite : 1;
        UINT64 MsrCstarRead : 1;
        UINT64 MsrCstarWrite : 1;
        UINT64 ApicBaseMsrRead : 1;
        UINT64 ApicBaseMsrWrite : 1;
        UINT64 MsrEferRead : 1;
        UINT64 MsrEferWrite : 1;
        UINT64 GdtrWrite : 1;
        UINT64 IdtrWrite : 1;
        UINT64 LdtrWrite : 1;
        UINT64 TrWrite : 1;
        UINT64 MsrSysenterCsWrite : 1;
        UINT64 MsrSysenterEipWrite : 1;
        UINT64 MsrSysenterEspWrite : 1;
        UINT64 MsrSfmaskWrite : 1;
        UINT64 MsrTscAuxWrite : 1;
        UINT64 MsrSgxLaunchControlWrite : 1;
        UINT64 RsvdZ : 39;
    };
} HV_REGISTER_CR_INTERCEPT_CONTROL;
```

#### <a name="mask-registers"></a>レジスタのマスク

細かい制御を可能にするために、コントロールレジスタのサブセットにも対応するマスクレジスタがあります。 マスクレジスタは、対応するコントロールレジスタのサブセットにインターセプトをインストールするために使用できます。 マスクレジスタが定義されていない場合は、(HvX64RegisterCrInterceptControl で定義されているように) すべてのアクセスでインターセプトがトリガーされます。

ハイパーバイザーでは、HvX64RegisterCrInterceptCr0Mask、HvX64RegisterCrInterceptCr4Mask、HvX64RegisterCrInterceptIa32MiscEnableMask の各マスクレジスタがサポートされています。

## <a name="dma-and-devices"></a>DMA とデバイス

デバイスは、VTL0 と同じ特権レベルを効果的に持ちます。 VSM が有効になっている場合、デバイスに割り当てられたすべてのメモリは VTL0 としてマークされます。 DMA アクセスには、VTL0 と同じ特権が与えられます。
