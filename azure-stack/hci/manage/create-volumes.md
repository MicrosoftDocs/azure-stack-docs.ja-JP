---
title: Azure Stack HCI でボリュームを作成する
description: Windows Admin Center と PowerShell を使用して Azure Stack HCI でボリュームを作成する方法について説明します。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 02/04/2021
ms.openlocfilehash: 9bb0ff34863f8262d5919e5eae6f735709097bf5
ms.sourcegitcommit: 283b1308142e668749345bf24b63d40172559509
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/05/2021
ms.locfileid: "99570737"
---
# <a name="create-volumes-in-azure-stack-hci"></a>Azure Stack HCI でボリュームを作成する

> 適用対象:Azure Stack HCI バージョン 20H2

このトピックでは、Windows Admin Center と Windows PowerShell を使用して Azure Stack HCI クラスターにボリュームを作成する方法、ボリューム上のファイルを操作する方法、およびボリュームでデータ重複除去と圧縮を有効にする方法について説明します。 ストレッチ クラスター用のボリュームを作成し、レプリケーションを設定する方法を学習するには、[拡張ボリュームの作成](create-stretched-volumes.md)に関する記事を参照してください。

## <a name="create-a-three-way-mirror-volume"></a>3 方向ミラー ボリュームを作成する

Windows Admin Center を使用して 3 方向ミラー ボリュームを作成するには、次のようにします。

1. Windows Admin Center で、クラスターに接続し、 **[ツール]** ウィンドウで **[ボリューム]** を選択します。
2. **[ボリューム]** ページで **[インベントリ]** タブを選択し、 **[ボリュームの作成]** を選択します。
3. **[ボリュームの作成]** ペインで、ボリュームの名前を入力し、 **[回復性]** を **[Three-way mirror]\(3 方向ミラー\)** のままにします。
4. **[Size on HDD]\(HDD 上のサイズ\)** で、ボリュームのサイズを指定します。 たとえば、5 TB (テラバイト) のように指定します。
5. **［作成］** を選択します

サイズによっては、ボリュームの作成に数分かかることがあります。 右上の通知により、ボリュームが作成されたことを知ることができます。 新しいボリュームが [インベントリ] の一覧に表示されます。

3 方向ミラー ボリュームの作成方法に関する簡単な動画をご覧ください。

> [!VIDEO https://www.youtube-nocookie.com/embed/o66etKq70N8]

## <a name="create-a-mirror-accelerated-parity-volume"></a>ミラー高速パリティ ボリュームの作成

ミラー高速パリティ (MAP) によって、HDD 上のボリュームのフットプリントが減少します。 たとえば、3 方向ミラー ボリュームでは、10 テラバイトのサイズごとに、フットプリントとして 30 テラバイトが必要になることを意味します。 フットプリントのオーバーヘッドを削減するには、ミラー高速パリティを使用してボリュームを作成します。 これにより、最もアクティブな 20 % のデータをミラー化し、よりスペース効率の高いパリティを使用して残りを保管することで、4 台のサーバーのみでも、フットプリントを 30 テラバイトから 22 テラバイトまで削減できます。 このパリティとミラーの比率を調整して、ワークロードに適したパフォーマンスと容量のトレードオフを実現できます。 たとえば、90 % のパリティと 10 % のミラーでは、パフォーマンスが低下しますが、フットプリントはさらに合理化されます。

  >[!NOTE]
  >ミラー高速パリティ ボリュームには、Resilient File System (ReFS) が必要です。

Windows Admin Center でミラー高速パリティを使用するボリュームを作成するには、次のようにします。

1. Windows Admin Center で、クラスターに接続し、 **[ツール]** ウィンドウで **[ボリューム]** を選択します。
2. [ボリューム] ページで **[インベントリ]** タブを選択し、 **[ボリュームの作成]** を選択します。
3. **[ボリュームの作成]** ペインで、ボリュームの名前を入力します。
4. **[回復性]** で、 **[Mirror-accelerated parity]\(ミラー高速パリティ\)** を選択します。
5. **[Parity percentage]\(パリティの割合\)** で、パリティの割合を選択します。
6. **［作成］** を選択します

ミラー高速パリティ ボリュームを作成する方法についての簡単な動画をご覧ください。

> [!VIDEO https://www.youtube-nocookie.com/embed/R72QHudqWpE]

## <a name="open-volume-and-add-files"></a>ボリュームを開いてファイルを追加する

Windows Admin Center でボリュームを開き、ボリュームにファイルを追加するには、次のようにします。

1. Windows Admin Center で、クラスターに接続し、 **[ツール]** ウィンドウで **[ボリューム]** を選択します。
2. **[ボリューム]** ページで、 **[インベントリ]** タブを選択します。
2. ボリュームの一覧で、開くボリュームの名前を選択します。

    ボリュームの詳細ページで、ボリュームのパスを確認できます。

4. ページの最上部で **[開く]** を選択します。 これにより、Windows dmin Center の **ファイル** ツールが起動します。
5. ボリュームのパスに移動します。 ここで、ボリューム内のファイルを参照できます。
6. **[アップロード]** を選択し、アップロードするファイルを選択します。
7. ブラウザーの **[戻る]** ボタンを使用して、Windows Admin Center の **[ツール]** ペインに戻ります。

ボリュームを開いてファイルを追加する方法に関する簡単な動画をご覧ください。

> [!VIDEO https://www.youtube-nocookie.com/embed/j59z7ulohs4]

## <a name="turn-on-deduplication-and-compression"></a>重複除去と圧縮を有効にする

重複除去と圧縮はボリュームごとに管理されます。 重複除去と圧縮では後処理モデルが使用されるため、実行されるまで節約は確認されません。 実行されると、以前から存在するファイルであっても、すべてのファイルが処理されます。

詳細については、「[ボリュームの暗号化、重複除去、圧縮を有効にする](volume-encryption-deduplication.md)」をご覧ください。

## <a name="create-volumes-using-windows-powershell"></a>Windows PowerShell を使用してボリュームを作成する

最初に、Windows の [スタート] メニューから Windows PowerShell を起動します。 **New-Volume** コマンドレットを使用して、Azure Stack HCI のボリュームを作成することをお勧めします。 これは、高速で操作が非常に分かりやすいです。 このコマンドレット 1 つだけで、仮想ディスクが作成されて、パーティション化およびフォーマットされ、一致する名前を持つボリュームが作成されて、クラスター共有ボリュームに追加されます。すべて 1 つの簡単な手順で行うことができます。

**New-Volume** コマンドレットには、次の 4 つのパラメーターを常に指定する必要があります。

- **FriendlyName:** 任意の文字列 (たとえば *"Volume1")*
- **FileSystem:**  **CSVFS_ReFS** (すべてのボリュームに推奨されます。ミラー高速パリティ ボリュームの場合は必須です) または **CSVFS_NTFS** のいずれか
- **StoragePoolFriendlyName:** 記憶域プールの名前 (たとえば *"S2D on ClusterName"* )
- **Size:** ボリュームのサイズ (たとえば *"10 TB"* )

   > [!NOTE]
   > PowerShell を含む Windows では、バイナリ (基数 2) の数値を使用してカウントされますが、ドライブには 10 進数 (基数 10) の数値でラベルが付けれていることがよくあります。 これが理由で、1 兆バイトとして定義された "1 テラバイト" のドライブは Windows で約 "909 GB" として表示されます。 これは予期されることです。 **New-Volume** を使用してボリュームを作成する場合は、**Size** パラメーターをバイナリ (基数 2) の数値で指定する必要があります。 たとえば、"909 GB" または "0.909495 TB" を指定すると、約 1 兆バイトのボリュームが作成されます。

### <a name="example-with-2-or-3-servers"></a>例:2 台または 3 台のサーバー

設定を簡単にするために、デプロイにサーバーが 2 台しかない場合は、回復性を確保するために、記憶域スペース ダイレクトによって双方向のミラーリングが自動的に使用されます。 デプロイにサーバーが 3 台しかない場合は、自動的に 3 方向のミラーリングが使用されます。

```PowerShell
New-Volume -FriendlyName "Volume1" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -Size 1TB
```

### <a name="example-with-4-servers"></a>例:4 台以上のサーバー

4 台以上のサーバーがある場合は、オプションの **ResiliencySettingName** パラメーターを使用して、回復性の種類を選択できます。

-   **ResiliencySettingName:** **Mirror** または **Parity** のいずれか。

次の例では *"Volume2"* で 3 方向ミラーリングを使用し、 *"Volume3"* でデュアル パリティを使用しています ("イレージャー コーディング" とよく呼ばれます)。

```PowerShell
New-Volume -FriendlyName "Volume2" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -Size 1TB -ResiliencySettingName Mirror
New-Volume -FriendlyName "Volume3" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -Size 1TB -ResiliencySettingName Parity
```

## <a name="using-storage-tiers"></a>ストレージ層の使用

3 種類のドライブを使用したデプロイでは、1 つのボリュームが SSD および HDD 階層にまたがり、それぞれに部分的に存在することができます。 同様に、4 台以上のサーバーを使用したデプロイでは、1 つのボリュームにミラーリングとデュアル パリティを混在させ、両方に部分的に配置できます。

このようなボリュームの作成を支援するために、Azure Stack HCI には、**MirrorOn*MediaType*** および **NestedMirrorOn*MediaType*** (パフォーマンス用) と、**ParityOn*MediaType*** および **NestedParityOn*MediaType*** (容量用) (この *MediaType* は HDD または SSD です) という既定の階層テンプレートが用意されています。 テンプレートは、メディアの種類に基づいてストレージ層を表し、高速な容量のドライブの 3 方向ミラーリング (該当する場合)、および低速な容量のドライブのデュアル パリティ (該当する場合) の定義をカプセル化したものです。

これらを確認するには、クラスター内の任意のサーバーで **Get-StorageTier** コマンドレットを実行します。

```PowerShell
Get-StorageTier | Select FriendlyName, ResiliencySettingName, PhysicalDiskRedundancy
```

たとえば、HDD のみを搭載した 2 ノード クラスターがある場合、出力は次のようになります。

```PowerShell
FriendlyName      ResiliencySettingName PhysicalDiskRedundancy
------------      --------------------- ----------------------
NestedParityOnHDD Parity                                     1
Capacity          Mirror                                     1
NestedMirrorOnHDD Mirror                                     3
MirrorOnHDD       Mirror                                     1
```

階層化ボリュームを作成するには、**New-Volume** コマンドレットの **StorageTierFriendlyNames** と **StorageTierSizes** パラメーターを使用して、これらの層テンプレートを参照します。 たとえば、次のコマンドレットは、3 方向のミラーリングとデュアル パリティを30:70 の比率で混合するボリュームを 1 つ作成します。

```PowerShell
New-Volume -FriendlyName "Volume1" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -StorageTierFriendlyNames MirrorOnHDD, Capacity -StorageTierSizes 300GB, 700GB
```

必要なだけ繰り返して、複数のボリュームを作成します。

### <a name="nested-resiliency-volumes"></a>入れ子の回復性ボリューム

入れ子の回復性は、2 サーバー クラスターにのみ適用されます。クラスターに 3 つ以上のサーバーがある場合、入れ子の回復性を使用することはできません。 入れ子の回復性により、2 サーバー クラスターは、ストレージの可用性を失うことなく同時に複数のハードウェア障害に耐えられます。また、ユーザー、アプリ、および仮想マシンを中断することなく継続して実行することができます。 詳細については、「[ボリュームの計画: 回復性の種類の選択](../concepts/plan-volumes.md#choosing-the-resiliency-type)」を参照してください。

#### <a name="create-nested-storage-tiers"></a>入れ子になったストレージ層を作成する

NestedMirror 層を作成するには:

```PowerShell
New-StorageTier -StoragePoolFriendlyName S2D* -FriendlyName NestedMirror -ResiliencySettingName Mirror -NumberOfDataCopies 4 -MediaType HDD -CimSession 2nodecluster
```

NestedParity 層を作成するには:

```PowerShell
New-StorageTier -StoragePoolFriendlyName S2D* -FriendlyName NestedParity -ResiliencySettingName Parity -NumberOfDataCopies 2 -PhysicalDiskRedundancy 1 -NumberOfGroups 1 -FaultDomainAwareness StorageScaleUnit -ColumnIsolation PhysicalDisk -MediaType HDD -CimSession 2nodecluster
```

#### <a name="create-nested-volumes"></a>入れ子になったボリュームを作成

NestedMirror ボリュームを作成するには:

```PowerShell
New-Volume -StoragePoolFriendlyName S2D* -FriendlyName MyMirrorNestedVolume -StorageTierFriendlyNames NestedMirror -StorageTierSizes 500GB -CimSession 2nodecluster
```

NestedParity ボリュームを作成するには:

```PowerShell
New-Volume -StoragePoolFriendlyName S2D* -FriendlyName MyParityNestedVolume -StorageTierFriendlyNames NestedMirror,NestedParity -StorageTierSizes 200GB, 1TB -CimSession 2nodecluster
```

### <a name="storage-tier-summary-table"></a>ストレージ層の概要テーブル

次の表は、Azure Stack HCI で作成される、または作成できるストレージ層をまとめたものです。

**NumberOfNodes: 2**

| FriendlyName      | MediaType | ResiliencySettingName | NumberOfDataCopies | PhysicalDiskRedundancy | NumberOfGroups | FaultDomainAwareness | ColumnIsolation | Note         |
| ----------------- | :-------: | :-------------------: | :----------------: | :--------------------: |:--------------:| :------------------: | :-------------: | :----------: |
| MirrorOnHDD       | HDD       | ミラー                | 2                  | 1                      | 1              | StorageScaleUnit     | 物理ディスク    | 自動作成 |
| MirrorOnSSD       | SSD       | ミラー                | 2                  | 1                      | 1              | StorageScaleUnit     | 物理ディスク    | 自動作成 |
| MirrorOnSCM       | SCM       | ミラー                | 2                  | 1                      | 1              | StorageScaleUnit     | 物理ディスク    | 自動作成 |
| NestedMirrorOnHDD | HDD       | ミラー                | 4                  | 3                      | 1              | StorageScaleUnit     | 物理ディスク    | manual       |
| NestedMirrorOnSSD | SSD       | ミラー                | 4                  | 3                      | 1              | StorageScaleUnit     | 物理ディスク    | manual       |
| NestedMirrorOnSCM | SCM       | ミラー                | 4                  | 3                      | 1              | StorageScaleUnit     | 物理ディスク    | manual       |
| NestedParityOnHDD | HDD       | パリティ                | 2                  | 1                      | 1              | StorageScaleUnit     | 物理ディスク    | manual       |
| NestedParityOnSSD | SSD       | パリティ                | 2                  | 1                      | 1              | StorageScaleUnit     | 物理ディスク    | manual       |
| NestedParityOnSCM | SCM       | パリティ                | 2                  | 1                      | 1              | StorageScaleUnit     | 物理ディスク    | manual       |

**NumberOfNodes: 3**

| FriendlyName      | MediaType | ResiliencySettingName | NumberOfDataCopies | PhysicalDiskRedundancy | NumberOfGroups | FaultDomainAwareness | ColumnIsolation | Note         |
| ----------------- | :-------: | :-------------------: | :----------------: | :--------------------: |:--------------:| :------------------: | :-------------: | :----------: |
| MirrorOnHDD       | HDD       | ミラー                | 3                  | 2                      | 1              | StorageScaleUnit     | 物理ディスク    | 自動作成 |
| MirrorOnSSD       | SSD       | ミラー                | 3                  | 2                      | 1              | StorageScaleUnit     | 物理ディスク    | 自動作成 |
| MirrorOnSCM       | SCM       | ミラー                | 3                  | 2                      | 1              | StorageScaleUnit     | 物理ディスク    | 自動作成 |

**NumberOfNodes: 4+**

| FriendlyName      | MediaType | ResiliencySettingName | NumberOfDataCopies | PhysicalDiskRedundancy | NumberOfGroups | FaultDomainAwareness | ColumnIsolation | Note         |
| ----------------- | :-------: | :-------------------: | :----------------: | :--------------------: |:--------------:| :------------------: | :-------------: | :----------: |
| MirrorOnHDD       | HDD       | ミラー                | 3                  | 2                      | 1              | StorageScaleUnit     | 物理ディスク    | 自動作成 |
| MirrorOnSSD       | SSD       | ミラー                | 3                  | 2                      | 1              | StorageScaleUnit     | 物理ディスク    | 自動作成 |
| MirrorOnSCM       | SCM       | ミラー                | 3                  | 2                      | 1              | StorageScaleUnit     | 物理ディスク    | 自動作成 |
| ParityOnHDD       | HDD       | パリティ                | 1                  | 2                      | 自動           | StorageScaleUnit     | StorageScaleUnit| 自動作成 |
| ParityOnSSD       | SSD       | パリティ                | 1                  | 2                      | 自動           | StorageScaleUnit     | StorageScaleUnit| 自動作成 |
| ParityOnSCM       | SCM       | パリティ                | 1                  | 2                      | 自動           | StorageScaleUnit     | StorageScaleUnit| 自動作成 |

## <a name="next-steps"></a>次のステップ

関連トピックおよびその他のストレージ管理タスクについては、次のトピックも参照してください。

- [記憶域スペース ダイレクトの概要](/windows-server/storage/storage-spaces/storage-spaces-direct-overview)
- [ボリュームの計画](../concepts/plan-volumes.md)
- [ボリュームの拡張](extend-volumes.md)
- [ボリュームの削除](delete-volumes.md)