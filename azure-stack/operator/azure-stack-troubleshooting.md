---
title: Azure Stack Hub のトラブルシューティングを行う
titleSuffix: Azure Stack
description: VM、ストレージ、App Service に関する問題を含む、Azure Stack Hub のトラブルシューティング方法について学習します。
author: PatAltimore
ms.topic: article
ms.date: 02/12/2021
ms.author: patricka
ms.reviewer: prchint
ms.lastreviewed: 12/10/2020
ms.openlocfilehash: b16582d6f4dad324d1c748034dad902907e669b4
ms.sourcegitcommit: e432e7f0a790bd6419987cbb5c5f3811e2e7a4a2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/09/2021
ms.locfileid: "102515721"
---
# <a name="troubleshoot-issues-in-azure-stack-hub"></a>Azure Stack Hub の問題のトラブルシューティングを行う

このドキュメントでは、Azure Stack Hub 統合環境のトラブルシューティング情報を提供します。 Azure Stack Development Kit のヘルプについては、[ASDK のトラブルシューティング](../asdk/asdk-troubleshooting.md)に関するページを参照するか、[Azure Stack Hub MSDN フォーラム](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)で専門家にお問い合わせください。

## <a name="frequently-asked-questions"></a>よく寄せられる質問

以下のセクションには、Microsoft サポートに送られてくる一般的な質問を取り上げたドキュメントのリンクが含まれています。

### <a name="purchase-considerations"></a>購入に関する考慮事項

* [購入方法](https://azure.microsoft.com/overview/azure-stack/how-to-buy/)
* [Azure Stack Hub の概要](azure-stack-overview.md)

### <a name="updates-and-diagnostics"></a>更新と診断

* [Azure Stack Hub で診断ツールを使用する方法](./diagnostic-log-collection.md)
* [Azure Stack Hub システムの状態を検証する方法](azure-stack-diagnostic-test.md)
* [更新プログラム パッケージのリリース周期](azure-stack-servicing-policy.md#update-package-release-cadence)
* [ノードの状態の確認とトラブルシューティング](azure-stack-node-actions.md)

### <a name="supported-operating-systems-and-sizes-for-guest-vms"></a>ゲスト VM でサポートされているオペレーティング システムとサイズ

* [Azure Stack Hub でサポートされているゲスト オペレーティング システム](azure-stack-supported-os.md)
* [Azure Stack Hub でサポートされている VM サイズ](../user/azure-stack-vm-sizes.md)

### <a name="azure-marketplace"></a>Azure Marketplace

* [Azure Stack Hub で使用できる Azure Marketplace 項目](azure-stack-marketplace-azure-items.md)

### <a name="manage-capacity"></a>キャパシティの管理

#### <a name="memory"></a>メモリ

Azure Stack Hub で使用可能な総メモリ容量を増やすために、メモリをさらに追加することができます。 Azure Stack Hub では、物理サーバーはスケール ユニット ノードと呼ばれることもあります。 単一のスケール ユニットに属しているすべてのスケール ユニット ノードには、[同量のメモリ](azure-stack-manage-storage-physical-memory-capacity.md)が確保されている必要があります。

#### <a name="retention-period"></a>保持期間

保有期間の設定では、クラウド オペレーターは削除されたアカウントを回復できる可能性がある期間を日数 (0 から 9999 日の間) で指定できます。 既定の保有期間は **0** 日に設定されています。 値を **0** に設定すると、削除されたすべてのアカウントがすぐに保有期間外になり、定期的なガベージ コレクションの対象としてマークされます。

* [保有期間の設定](azure-stack-manage-storage-accounts.md#set-the-retention-period)

### <a name="security-compliance-and-identity"></a>セキュリティ、コンプライアンス、ID  

#### <a name="manage-rbac"></a>RBAC の管理

Azure Stack Hub のユーザーは、サブスクリプション、リソース グループ、またはサービスの各インスタンスの閲覧者、所有者、または共同作成者になることができます。

* [Azure Stack Hub での RBAC の管理](azure-stack-manage-permissions.md)

Azure リソースの組み込みロールが組織の特定のニーズを満たさない場合は、独自のカスタム ロールを作成することができます。 このチュートリアルでは、Azure PowerShell を使用して、Reader Support Tickets というカスタム ロールを作成します。

* [チュートリアル:Azure PowerShell を使用して Azure リソースのカスタム ロールを作成する](/azure/role-based-access-control/tutorial-custom-role-powershell)

### <a name="manage-usage-and-billing-as-a-csp"></a>使用量と課金を CSP として管理する

* [使用量と課金を CSP として管理する](azure-stack-add-manage-billing-as-a-csp.md#create-a-csp-or-apss-subscription)
* [CSP または APSS サブスクリプションを作成する](azure-stack-add-manage-billing-as-a-csp.md#create-a-csp-or-apss-subscription)

Azure Stack Hub で使用する共有サービス アカウントの種類を選択します。 マルチテナント Azure Stack Hub の登録に使用できるサブスクリプションの種類は、次のとおりです。

* クラウド ソリューション プロバイダー
* Partner Shared Services サブスクリプション

### <a name="get-scale-unit-metrics"></a>スケール ユニットのメトリックを取得する

PowerShell を使用して、Microsoft サポートを利用せずに、スタンプ使用状況の情報を取得できます。 スタンプの使用状況を取得するには:

1. PEP セッションを作成します。
2. `test-azurestack` を実行します。
3. PEP セッションを終了します。
4. invoke-command 呼び出しを使用して、`get-azurestacklog -filterbyrole seedring` を実行します。
5. seedring .zip を抽出します。 `test-azurestack` を実行した ERCS フォルダーから、検証レポートを取得することができます。

詳細については、[Azure Stack Hub の診断](azure-stack-get-azurestacklog.md)に関する記述を参照してください。

## <a name="troubleshoot-virtual-machines-vms"></a>仮想マシン (VM) のトラブルシューティングを行う

### <a name="reset-linux-vm-password"></a>Linux VM のパスワードをリセットする

Linux VM のパスワードを忘れて、VMAccess 拡張機能の問題が原因で **[パスワードのリセット]** オプションが機能しない場合は、次の手順に従ってリセットを実行できます。

1. 復旧 VM として使用する Linux VM を選択します。

1. ユーザー ポータルにサインインします。
   1. VM サイズ、NIC、パブリック IP、NSG、データ ディスクをメモしておきます。
   1. 影響を受けた VM を停止します。
   1. 影響を受けた VM を削除します。
   1. 影響を受けた VM のディスクを、復旧 VM のデータ ディスクとしてアタッチします (ディスクが使用可能になるまでに数分かかる場合があります)。

1. 復旧 VM にサインインし、次のコマンドを実行します。

   ```
   sudo su –
   mkdir /tempmount
   fdisk -l
   mount /dev/sdc2 /tempmount /*adjust /dev/sdc2 as necessary*/
   chroot /tempmount/
   passwd root /*substitute root with the user whose password you want to reset*/
   rm -f /.autorelabel /*Remove the .autorelabel file to prevent a time consuming SELinux relabel of the disk*/
   exit /*to exit the chroot environment*/
   umount /tempmount
   ```

1. ユーザー ポータルにサインインします。

   1. 復旧 VM からディスクを切断します。
   1. ディスクから VM を再作成します。
   1. 必ず前の VM からパブリック IP を移行し、データ ディスクをアタッチするなどの操作を行ってください。


元のディスクで直接変更を行うのではなく、元のディスクのスナップショットを作成し、そこから新しいディスクを作成することもできます。 詳細については、以下のトピックを参照してください。

- [パスワードのリセット](/azure/virtual-machines/troubleshooting/reset-password)
- [スナップショットからディスクを作成する](/azure/virtual-machines/troubleshooting/troubleshoot-recovery-disks-portal-linux#create-a-disk-from-the-snapshot)
- [ルート パスワードの変更とリセット](https://access.redhat.com/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/sec-terminal_menu_editing_during_boot#sec-Changing_and_Resetting_the_Root_Password)


### <a name="license-activation-fails-for-windows-server-2012-r2-during-provisioning"></a>プロビジョニング中に Windows Server 2012 R2 のライセンス認証が失敗する

この場合、Windows で認証に失敗し、画面の右下隅に透かしが表示されます。 次のイベントが、C:\Windows\Panther の下にある WaSetup.xml ログに含まれます。

```xml
<Event time="2019-05-16T21:32:58.660Z" category="ERROR" source="Unattend">
    <UnhandledError>
        <Message>InstrumentProcedure: Failed to execute 'Call ConfigureLicensing()'. Will raise error to caller</Message>
        <Number>-2147221500</Number>
        <Description>Could not find the VOLUME_KMSCLIENT product</Description>
        <Source>Licensing.wsf</Source>
    </UnhandledError>
</Event>
```


ライセンス認証を行うには、認証する SKU に対する仮想マシンの自動ライセンス認証 (AVMA) キーをコピーします。

|Edition|AVMA キー|
|-|-|
|データセンター|Y4TGP-NPTV9-HTC2H-7MGQ3-DV4TW|
|Standard|DBGBW-NPF86-BJVTX-K3WKJ-MTB6V|
|要点|K2XGM-NMBT3-2R6Q8-WF2FK-P36R2|

VM で、次のコマンドを実行します。

```powershell
slmgr /ipk <AVMA_key>
```

詳細については、[仮想マシンの自動ライセンス認証](/windows-server/get-started-19/vm-activation-19)に関する記事をご覧ください。

### <a name="default-image-and-gallery-item"></a>既定のイメージとギャラリー アイテム

Azure Stack Hub に VM をデプロイする前に、Windows Server イメージとギャラリー アイテムを追加する必要があります。

### <a name="ive-deleted-some-vms-but-still-see-the-vhd-files-on-disk"></a>一部の VM の削除後に、まだディスクに VHD ファイルが表示される

この動作は仕様によるものです。

* VM を削除しても VHD は削除されません。 ディスクはリソース グループ内の個別のリソースです。
* ストレージ アカウントが削除されると、削除は Azure Resource Manager にすぐに表示されます。 しかし、含まれる可能性のあるディスクは、ガベージ コレクションが実行されるまで引き続きストレージに保持されます。

"孤立" VHD が表示された場合、それが削除されたストレージ アカウントのフォルダーの一部であるかどうかを知ることが重要です。 ストレージ アカウントが削除されていない場合は、まだそこにあっても正常です。

リテンション期間しきい値と、オンデマンドの再利用の設定について詳しくは、[ストレージ アカウントの管理](azure-stack-manage-storage-accounts.md)に関するページを参照してください。

## <a name="troubleshoot-storage"></a>ストレージのトラブルシューティング

### <a name="storage-reclamation"></a>記憶域の再利用

ポータルに再利用された容量が表示されるまで、最大で 14 時間かかる場合があります。 領域の再利用は、ブロック BLOB ストア内の内部コンテナー ファイルの使用率など、さまざまな要因に依存します。 そのため、削除されるデータの量によって、ガベージ コレクターの実行時に再利用可能になる領域の量に対する保証はありません。

### <a name="azure-storage-explorer-not-working-with-azure-stack-hub"></a>Azure Stack Hub で Azure Storage Explorer が動作しない

切断されたシナリオで統合システムを使用する場合は、エンタープライズ証明機関 (CA) を利用することをお勧めします。 ルート証明書を Base-64 形式でエクスポートしてから、Azure Storage Explorer にインポートします。 必ず、Resource Manager エンドポイントから末尾のスラッシュ (`/`) を削除してください。 詳細については、「[Azure Stack Hub への接続を準備する](../user/azure-stack-storage-connect-se.md)」を参照してください。

## <a name="troubleshoot-app-service"></a>App Service のトラブルシューティング

### <a name="create-aadidentityappps1-script-fails"></a>Create-AADIdentityApp.ps1 スクリプトが失敗する

App Service に必要な Create-AADIdentityApp.ps1 スクリプトが失敗する場合は、スクリプトの実行時に必須の `-AzureStackAdminCredential` パラメーターを必ず含めるようにしてください。 詳細については、「[App Service on Azure Stack Hub のデプロイの前提条件](azure-stack-app-service-before-you-get-started.md#create-an-azure-ad-app)」を参照してください。

## <a name="troubleshoot-azure-stack-hub-updates"></a>Azure Stack Hub の更新プログラムのトラブルシューティング

Azure Stack Hub の修正プログラムと更新プログラムのプロセスは、オペレーターが更新プログラム パッケージを一貫した合理的な方法で適用できるように設計されています。 まれに、修正プログラムや更新プログラムのプロセス中に問題が発生することがあります。 修正プログラムや更新プログラムのプロセス中に問題が発生した場合は、以下の手順を実行することをお勧めします。

0. **前提条件**:[更新プログラムのアクティビティのチェックリスト](release-notes-checklist.md)に従っていること、および [事前ログ収集を有効](./diagnostic-log-collection.md#send-logs-proactively)にしていることを確認してください。

1. 更新が失敗したときに作成されたエラー アラートの修復手順に従います。

2. 問題を解決できない場合は、[Azure Stack Hub のサポート チケット](./azure-stack-help-and-support-overview.md)を作成します。 問題が発生した期間に[収集されたログ](./diagnostic-log-collection.md#send-logs-now)があることを確認してください。 重大なアラートまたは警告が発生して更新が失敗した場合は、アラートの指示に従いエラーを調べて Microsoft カスタマー サポート サービスに連絡し、スケール ユニットが長時間エラー状態のままにならないようにすることが重要です。 スケール ユニットを長時間にわたって更新失敗状態のままにすると、解決するのがいっそう困難な別の問題が後で発生する可能性があります。

## <a name="common-azure-stack-hub-patch-and-update-issues"></a>Azure Stack Hub の修正プログラムと更新プログラムに関する一般的な問題

*適用対象:Azure Stack Hub 統合システム*

### <a name="preparationfailed"></a>PreparationFailed

**適用先**: この問題は、サポートされているすべてのリリースに適用されます。

**原因**:Azure Stack Hub の更新プログラムをインストールしようとしたときに、更新が失敗して、状態が `PreparationFailed` に変更される場合があります。 インターネットに接続されたシステムの場合、これは通常、インターネット接続が脆弱であるために、更新プログラム パッケージが適切にダウンロードされないことを示します。 

**対応策**: **[今すぐインストール]** をもう一度クリックすることで、この問題を回避できます。 問題が解決しない場合は、[更新プログラムのインストール](azure-stack-apply-updates.md?#install-updates-and-monitor-progress)に関するセクションに従って、更新プログラム パッケージを手動でアップロードすることをお勧めします。

**発生頻度**: 共通

### <a name="update-failed-check-and-enforce-external-key-protectors-on-csvs"></a>更新に失敗しました: CSV で外部キー プロテクターを確認して適用する

**適用先**: この問題は、サポートされているすべてのリリースに適用されます。

**原因**: ベースボード管理コントローラー (BMC) のパスワードが正しく設定されていません。

**修復**: [BMC 資格情報を更新](/azure-stack/operator/azure-stack-rotate-secrets#update-the-bmc-credential)し、更新を再開します。

### <a name="warnings-and-errors-reported-while-update-is-in-progress"></a>更新の進行中に報告される警告とエラー

**適用先**: この問題は、サポートされているすべてのリリースに適用されます。

**原因**:Azure Stack Hub の更新の状態が **進行中** の場合、ポータルで警告とエラーが報告されることがあります。 コンポーネントは、アップグレード中に他のコンポーネントを待機しているためにタイムアウトし、エラーが発生する場合があります。 Azure Stack Hub には、断続的なエラーが原因による一部のタスクを再試行または修復するメカニズムがあります。

**対応策**: Azure Stack Hub の更新の状態が **進行中** の場合、ポータルで報告された警告とエラーは無視できます。

**発生頻度**: 共通

::: moniker range="azs-2002"
### <a name="2002-update-failed"></a>2002 更新プログラムが失敗

**適用先**: この問題は 2002 リリースにのみ適用されます。

**原因**:2002 更新プログラムを実行しようとすると、更新プログラムが失敗し、`The private network parameter is missing from cloud parameters. Please use set-azsprivatenetwork cmdlet to set private networkTrace` というメッセージが表示されることがあります。

**対応策**: [プライベート内部ネットワークを設定します](./azure-stack-network.md?view=azs-2002&preserve-view=true#private-network)。
::: moniker-end
