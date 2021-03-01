---
title: Azure Stack HCI クラスターを更新する
description: Windows Admin Center と PowerShell を使用して、オペレーティング システムとファームウェアの更新プログラムを Azure Stack HCI に適用する方法。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 02/05/2021
ms.openlocfilehash: 2ed1a6c2443f7222a873c412e991b4a39b253309
ms.sourcegitcommit: ec19e8455b5cb90a071afb03ec1446b0a9aafb99
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/05/2021
ms.locfileid: "99588650"
---
# <a name="update-azure-stack-hci-clusters"></a>Azure Stack HCI クラスターを更新する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

Azure Stack HCI クラスターを更新する場合、その目標は、クラスター内のサーバーを一度に 1 つだけ更新することで可用性を維持することです。 オペレーティング システムの多くの更新プログラムでは、サーバーをオフラインにする必要があります。たとえば、再起動するためや、ネットワーク スタックなどのソフトウェアを更新するためなどです。 クラスター対応更新 (CAU) を使用することをお勧めします。これは、アプリケーションの実行を維持しながら、クラスター内のすべてのサーバーに更新プログラムを簡単にインストールできる機能です。 クラスター対応更新により、必要に応じて、更新プログラムをインストールしてサーバーを再起動する間に、サーバーのメンテナンス モードの切り替えが自動化されます。 クラスター対応更新は、Windows 管理センターで使用される既定の更新方法であり、PowerShell を使用して開始することもできます。

このトピックでは、オペレーティング システムとソフトウェアの更新に焦点を当てています。 ハードウェアのメンテナンスを実行するためにサーバーをオフラインにする必要がある場合は、[メンテナンスのためにサーバーをオフラインにする方法](maintain-servers.md)に関するページを参照してください。

## <a name="update-a-cluster-using-windows-admin-center"></a>Windows Admin Center を使用してクラスターを作成する

Windows Admin Center を使用すると、簡単なユーザー インターフェイスを使用して、クラスターを更新し、オペレーティング システムとソリューションの更新プログラムを簡単に適用することができます。 Microsoft のハードウェア パートナーから統合システムを購入した場合、適切なパートナーの更新プログラムの拡張機能をインストールすることにより、Windows Admin Center から直接、最新のドライバー、ファームウェア、およびその他の更新プログラムを簡単に入手できます。 統合システムとしてハードウェアを購入しなかった場合、ハードウェア ベンダーの推奨事項に従い、必要に応じてファームウェアとドライバーの更新プログラムを個別に実行します。

   > [!WARNING]
   > Windows Admin Center を使用して更新プロセスを開始する場合は、更新が完了するまでウィザードを使い続けてください。 Windows Admin Center で更新プロセスを部分的に完了した後に、クラスター対応更新ツールを使用したり、PowerShell を使用してクラスターを更新したりしないでください。 Windows Admin Center ではなく PowerShell を使用して更新を実行する場合は、「[PowerShell を使用してクラスターを更新する](#update-a-cluster-using-powershell)」に進んでください。

更新プログラムをインストールするには、次の手順のようにします。

1. クラスターに接続すると、1 つ以上のサーバーに更新プログラムをインストールする準備ができている場合は、Windows Admin Center ダッシュボードで通知され、すぐに更新するためのリンクが表示されます。 または、左側の **[ツール]** メニューから **[更新プログラム]** を選択することもできます。

2. クラスターを初めて更新する場合は、Windows Admin Center によって、クラスター対応更新を実行するようにクラスターが適切に構成されているかどうかが確認され、必要に応じて、Windows Admin Center による CAU の自動構成 (CAU クラスターの役割のインストールや、必要なファイアウォール規則の有効化など) を希望するかどうかが確認されます。 更新プロセスを開始するには、 **[作業の開始]** をクリックします。

   :::image type="content" source="media/update-cluster/add-cau-role.png" alt-text="Windows Admin Center により、クラスター対応更新を実行するようにクラスターが自動的に構成される" lightbox="media/update-cluster/add-cau-role.png":::

   > [!NOTE]
   > Windows Admin Center のクラスター対応更新ツールを使用するには、Credential Security Service Provider (CredSSP) を有効にし、明示的な資格情報を指定する必要があります。 CredSSP を有効にするかどうかを確認するメッセージが表示される場合は、 **[はい]** をクリックします。 ユーザー名とパスワードを指定して、 **[続行]** をクリックします。

3. クラスターの更新状態が表示されます。クラスター内の各サーバーで利用可能なオペレーティング システムの更新プログラムの一覧を取得するには、 **[更新プログラムの確認]** をクリックします。 場合によっては、管理者の資格情報を入力する必要があります。 利用可能なオペレーティング システムの更新プログラムがない場合は、 **[Next: hardware updates]\(次へ: ハードウェアの更新\)** をクリックして、ステップ 7 に進みます。

   > [!IMPORTANT]
   > 更新の進行中に [更新] 画面から移動すると、現在の実行が終了するまで [更新] ページの履歴セクションが正しく表示されないなど、予期しない動作が発生する可能性があります。 更新の進行中にアプリケーションを引き続き使用する場合は、新しいブラウザー タブまたはウィンドウで Windows Admin Center を開くことをお勧めします。

4. **Next:Install\(次へ: インストール\)** を選択してオペレーティング システムの更新プログラムのインストールに進むか、**スキップ** をクリックしてそれらを除外します。 

   :::image type="content" source="media/update-cluster/operating-system-updates.png" alt-text="[Next: Install]\(次へ: インストール\) を選択してオペレーティング システムの更新プログラムのインストールに進むか、[スキップ] をクリックしてそれらを除外する" lightbox="media/update-cluster/operating-system-updates.png":::

5. クラスター内の各サーバーにオペレーティング システムの更新プログラムをインストールするには、 **[インストール]** を選択します。 更新の状態が "更新プログラムをインストールしています" に変わります。 いずれかの更新プログラムで再起動が必要な場合は、一度に 1 つずつサーバーが再起動され、ダウンタイムを防ぐために、仮想マシンなどのクラスターの役割がサーバー間で移動されます。

   :::image type="content" source="media/update-cluster/install-os-updates.png" alt-text="クラスター内の各サーバーにオペレーティング システムの更新プログラムをインストールするには、[インストール] をクリックする" lightbox="media/update-cluster/install-os-updates.png":::

6. オペレーティング システムの更新が完了すると、更新の状態が "成功" に変わります。 **[Next: hardware updates]\(次へ: ハードウェアの更新\)** をクリックして、ハードウェア更新画面に進みます。

7. Windows Admin Center によって、特定のサーバー ハードウェアをサポートする拡張機能がクラスターにインストールされているかどうかが確認されます。 クラスター内の各サーバーにハードウェアの更新プログラムをインストールするには、 **[次へ: インストール]** をクリックします。 拡張機能または更新プログラムが見つからない場合は、 **[終了]** をクリックします。

8. セキュリティを向上させるには、更新プログラムのインストールが完了したらすぐに CredSSP を無効にします。
    - Windows Admin Center の **[すべての接続]** で、クラスター内の最初のサーバーを選択し、 **[接続]** を選択します。
    - **[概要]** ページで **[Disable CredSSP]\(CredSSP を無効にする\)** を選択し、次に **[Disable CredSSP]\(CredSSP を無効にする\)** ポップアップ ウィンドウで **[はい]** を選択します。

## <a name="update-a-cluster-using-powershell"></a>PowerShell を使用してクラスターを更新する

クラスター対応更新を使用してクラスターを更新する前に、**Remote Server Administration Tools (RSAT)** の一部であり、クラスター対応更新ソフトウェアが含まれる **フェールオーバー クラスタリング ツール** をインストールする必要があります。 既存のクラスターを更新する場合は、これらのツールが既にインストールされている可能性があります。

クラスター対応更新を使用してソフトウェアの更新プログラムを適用するようにフェールオーバー クラスターが適切に設定されているかどうかをテストするには、**Test-CauSetup** PowerShell コマンドレットを実行します。これにより、フェールオーバー クラスターとネットワーク環境の Best Practices Analyzer (BPA) スキャンが実行され、警告またはエラーが発生した場合は通知されます。

```PowerShell
Test-CauSetup -ClusterName Cluster1
```

機能、ツール、または役割をインストールする必要がある場合は、以下のセクションを参照してください。 それ以外の場合は、「[PowerShell を使用して更新プログラムを確認する](#check-for-updates-with-powershell)」に進んでください。

### <a name="install-failover-clustering-and-failover-clustering-tools-using-powershell"></a>PowerShell を使用してフェールオーバー クラスタリングとフェールオーバー クラスタリング ツールをインストールする

クラスターまたはサーバーにフェールオーバー クラスタリング機能とフェールオーバー クラスタリング ツールが既にインストールされているかどうかを確認するには、管理 PC から **`Get-WindowsFeature`** PowerShell コマンドレットを発行します (または、-ComputerName パラメーターを省略して、これをクラスターまたはサーバー上で直接実行します)。

```PowerShell
Get-WindowsFeature -Name Failover*, RSAT-Clustering* -ComputerName Server1
```

"Install State" (インストール状態) に "Installed" (インストール済み) と表示され、"Failover Clustering" (フェールオーバー クラスタリング) と "Failover Cluster Module for Windows PowerShell" (Windows PowerShell 用フェールオーバー クラスター モジュール) の両方の前に X が表示されていることを確認します。

```
Display Name                                            Name                       Install State
------------                                            ----                       -------------
[X] Failover Clustering                                 Failover-Clustering            Installed
        [X] Failover Clustering Tools                   RSAT-Clustering                Installed
            [X] Failover Cluster Module for Windows ... RSAT-Clustering-Powe...        Installed
            [ ] Failover Cluster Automation Server      RSAT-Clustering-Auto...        Available
            [ ] Failover Cluster Command Interface      RSAT-Clustering-CmdI...        Available
```

フェールオーバー クラスタリング機能がインストールされていない場合は、 **`Install-WindowsFeature`** コマンドレットで -IncludeAllSubFeature と -IncludeManagementTools のパラメーターを使用して、クラスター内の各サーバーにインストールします。

```PowerShell
Install-WindowsFeature –Name Failover-Clustering -IncludeAllSubFeature –IncludeManagementTools -ComputerName Server1
```

このコマンドを実行すると、フェールオーバー クラスターを管理するための PowerShell コマンドレットを含む PowerShell 用フェールオーバー クラスター モジュールと、フェールオーバー クラスターにソフトウェアの更新プログラムをインストールするための PowerShell 用クラスター対応更新モジュールもインストールされます。

フェールオーバー クラスタリング機能が既にインストールされていて、Windows PowerShell 用フェールオーバー クラスター モジュールがインストールされていない場合は、**Install-WindowsFeature** コマンドレットを使用して、クラスター内の各サーバーにインストールします。

```PowerShell
Install-WindowsFeature –Name RSAT-Clustering-PowerShell -ComputerName Server1
```

### <a name="choose-an-updating-mode"></a>更新モードを選択する

クラスター対応更新は、次の 2 つのモードで完全なクラスター更新操作を調整できます。  
  
-   **自己更新モード**: このモードでは、クラスター対応更新のクラスター化された役割を、更新対象となるフェールオーバー クラスター上のワークロードとして構成し、対応する更新スケジュールを定義します。 予定の時刻になると、既定またはカスタムの更新実行プロファイルを使用して、クラスターの更新が自動実行されます。 更新の実行中、クラスター対応更新の更新コーディネーター プロセスは、クラスター対応更新のクラスター化された役割を現在所有しているノード上で開始され、このプロセスによって各クラスター ノード上で更新が順番に実行されます。 現在のクラスター ノードを更新するには、クラスター対応更新のクラスター化された役割が別のクラスター ノードにフェールオーバーされ、そのノード上の新しい更新コーディネーター プロセスによって更新実行の制御が引き継がれます。 自己更新モードの場合、クラスター対応更新では、完全に自動化されたエンドツーエンドの更新プロセスを使用してフェールオーバー クラスターを更新することができます。 また、管理者は、このモードでオンデマンドで更新プログラムをトリガーすることや、必要に応じてリモート更新アプローチを使用することができます。
  
-   **リモート更新モード**: このモードでは、フェールオーバー クラスターへのネットワーク接続はあっても、フェールオーバー クラスターのメンバーではないリモート管理コンピューター (通常は Windows 10 PC) は、フェールオーバー クラスタリング ツールを使用して構成されます。 管理者は、更新コーディネーターと呼ばれるリモート管理コンピューターから、既定またはカスタムの更新実行プロファイルを使用して、オンデマンドの更新実行をトリガーします。 リモート更新モードは、更新実行の進捗状況をリアルタイムで監視する目的のほか、クラスターが Server Core 環境で実行されている場合などに有効活用できます。  

   > [!NOTE]
   > Windows 10 October 2018 Update 以降では、RSAT は Windows 10 から直接 "オンデマンド機能" のセットとして含まれています。 **[設定] > [アプリ] > [アプリと機能] > [オプション機能] > [RSAT: フェールオーバー クラスタリング ツール]** に移動し、 **[インストール]** を選択します。 インストールの進行状況を確認するには、[戻る] ボタンをクリックして、[オプション機能の管理] ページに状態を表示します。 インストールされた機能は、Windows 10 のバージョンのアップグレードをまたがって保持されます。 2018 年 10 月の更新プログラムの前に RSAT for Windows 10 をインストールするには、[RSAT パッケージをダウンロード](https://www.microsoft.com/download/details.aspx?id=45520)します。

### <a name="add-cau-cluster-role-to-the-cluster"></a>クラスターに CAU クラスターの役割を追加する

自己更新モードでは、クラスター対応更新クラスターの役割が必要です。 Windows Admin Center を使用して更新プログラムを実行している場合は、クラスターの役割が自動的に追加されます。

**`Get-CauClusterRole`** コマンドレットを使用すると、指定したクラスターのクラスター対応更新クラスターの役割の構成プロパティが表示されます。

```PowerShell
Get-CauClusterRole -ClusterName Cluster1
```

クラスターで役割がまだ構成されていない場合は、次のエラー メッセージが表示されます。

```Get-CauClusterRole : The current cluster is not configured with a Cluster-Aware Updating clustered role.```

PowerShell を使用して自己更新モードのクラスター対応更新クラスターの役割を追加するには、次の例のように、 **`Add-CauClusterRole`** コマンドレットを使用し、適切な [パラメーター](/powershell/module/clusterawareupdating/add-cauclusterrole#parameters)を指定します。

```PowerShell
Add-CauClusterRole -ClusterName Cluster1 -MaxFailedNodes 0 -RequireAllNodesOnline -EnableFirewallRules -VirtualComputerObjectName Cluster1-CAU -Force -CauPluginName Microsoft.WindowsUpdatePlugin -MaxRetriesPerNode 3 -CauPluginArguments @{ 'IncludeRecommendedUpdates' = 'False' } -StartDate "3/2/2020 3:00:00 AM" -DaysOfWeek 4 -WeeksOfMonth @(3) -verbose
```

   > [!NOTE]
   > 上記のコマンドは、管理 PC またはドメイン コントローラーから実行する必要があります。

### <a name="enable-firewall-rules-to-allow-remote-restarts"></a>リモートの再起動を許可するファイアウォール規則を有効にする

更新プロセス中に、サーバーがリモートで再起動できるようにする必要があります。 Windows Admin Center を使用して更新プログラムを実行している場合、各サーバー上の Windows ファイアウォール規則は自動的に更新され、リモートの再起動が可能になります。 PowerShell を使用して更新する場合は、Windows ファイアウォールで [リモート シャットダウン] ファイアウォール規則グループを有効にするか、上記の例のように -EnableFirewallRules パラメーターをコマンドレットに渡します。

## <a name="check-for-updates-with-powershell"></a>PowerShell を使用して更新プログラムを確認する

**`Invoke-CAUScan`** コマンドレットを使用してサーバーに適用できる更新プログラムをスキャンし、指定したクラスター内の各サーバーに適用可能な更新プログラムの初期セットの一覧を取得することができます。

```PowerShell
Invoke-CauScan -ClusterName Cluster1 -CauPluginName Microsoft.WindowsUpdatePlugin -Verbose
```

一覧の生成が完了するまでに数分かかる場合があります。 プレビュー一覧には、更新プログラムの初期セットのみが含まれます。初期の更新プログラムのインストール後に適用可能になる更新プログラムは含まれていません。

## <a name="install-updates-with-powershell"></a>PowerShell を使用して更新プログラムをインストールする

サーバーをスキャンして適用可能な更新プログラムをスキャンし、指定したクラスター上で完全な更新実行を行うには、 **`Invoke-CAURun`** コマンドレットを使用します。

```PowerShell
Invoke-CauRun -ClusterName Cluster1 -CauPluginName Microsoft.WindowsUpdatePlugin -MaxFailedNodes 1 -MaxRetriesPerNode 3 -RequireAllNodesOnline -EnableFirewallRules -Force
```

このコマンドを使用すると、Cluster1 という名前のクラスター上でスキャンと完全な更新実行が行われます。 このコマンドレットには Microsoft.WindowsUpdatePlugin プラグインが使用されます。このコマンドレットを実行する前に、すべてのクラスター ノードがオンラインになっている必要があります。 さらに、このコマンドレットでは、ノードが失敗とマークされるまでに、ノードごとに 3 回までの再試行が許容されます。また、更新実行全体が失敗とマークされるまでに、1 回のノードの失敗のみが許容されます。 また、ファイアウォール規則を使用して、サーバーをリモートで再起動できるようすることができます。 このコマンドには Force パラメーターを指定しているため、コマンドレットは確認プロンプトを表示せずに実行されます。

更新実行プロセスには、以下が含まれます。 
- クラスター内の各サーバーで適用可能な更新プログラムをスキャンしてダウンロードする
- 現在実行中のクラスター化された役割を各サーバーから移動する
- 各サーバーに更新プログラムをインストールする
- インストールされている更新プログラムで必要な場合はサーバーを再起動する
- クラスター化された役割を元のサーバーに戻す。

更新実行プロセスには、クォーラムが維持されていることの確認、更新プログラムの初期セットのインストール後にのみインストールできる追加の更新プログラムの確認、実行されたアクションのレポートの保存も含まれています。

## <a name="check-on-the-status-of-an-updating-run"></a>更新実行の状態を確認する

管理者は、 **`Get-CauRun`** コマンドレットを実行して、進行中の更新実行に関する概要情報を取得することができます。

```PowerShell
Get-CauRun -ClusterName Cluster1
```

出力例を次に示します。

```
RunId                   : 834dd11e-584b-41f2-8d22-4c9c0471dbad 
RunStartTime            : 10/13/2019 1:35:39 PM 
CurrentOrchestrator     : NODE1 
NodeStatusNotifications : { 
Node      : NODE1 
Status    : Waiting 
Timestamp : 10/13/2019 1:35:49 PM 
} 
NodeResults             : { 
Node                     : NODE2 
Status                   : Succeeded 
ErrorRecordData          : 
NumberOfSucceededUpdates : 0 
NumberOfFailedUpdates    : 0 
InstallResults           : Microsoft.ClusterAwareUpdating.UpdateInstallResult[] 
}
```

## <a name="perform-a-fast-offline-update-of-all-servers-in-a-cluster"></a>クラスター内のすべてのサーバーに対して高速のオフライン更新を実行する

この方法では、クラスター内のすべてのサーバーを一度に停止し、すべてのサーバーを同時に更新することができます。 これにより、更新プロセス中の時間が節約されますが、トレードオフとして、ホストされるリソースにダウンタイムが生じます。

迅速に適用する必要がある重要なセキュリティ更新プログラムがある場合、または更新プログラムがメンテナンス期間内に確実に完了する必要がある場合は、この方法をお勧めします。 このプロセスによって、Azure Stack HCI クラスターが停止し、サーバーが更新されて、再び起動されます。

1. メンテナンス期間を計画します。
2. 仮想ディスクをオフラインにします。
3. クラスターを停止して、記憶域プールをオフラインにします。 **Stop-Cluster** コマンドレットを実行するか、または Windows 管理センターを使用して、クラスターを停止します。
4. クラスター サービスを、各サーバーの Services.msc で **無効** に設定します。 これにより、更新中にクラスター サービスが起動しなくなります。
5. すべてのサーバーに対して、Windows Server の累積的な更新プログラムと必要なすべてのサービス スタック更新プログラムを適用します。 すべてのサーバーを同時に更新することができます。クラスターがダウンしているため、待機する必要はありません。
6. サーバーを再起動し、すべてが正常であることを確認します。
7. 各サーバーで、クラスター サービスを **自動** に戻します。
8. クラスターを起動します。 **Start-Cluster** コマンドレットを実行するか、または Windows 管理センターを使用します。

   数分待ちます。  記憶域プールが正常であることを確認してください。

9. 仮想ディスクをオンラインに戻します。
10. **Get-Volume** コマンドレットと **Get-VirtualDisk** コマンドレットを実行して、仮想ディスクの状態を監視します。

## <a name="next-steps"></a>次のステップ

関連情報については、以下もご覧ください。

- [クラスター対応更新 (CAU)](/windows-server/failover-clustering/cluster-aware-updating)
- [記憶域スペース ダイレクトでのドライブのファームウェアの更新](/windows-server/storage/update-firmware)
- [Azure Stack HCI クラスターを検証する](../deploy/validate.md)