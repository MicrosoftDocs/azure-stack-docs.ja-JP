---
title: Azure Stack HCI 上の Azure Kubernetes Service に関する既知の問題
description: Azure Stack HCI 上の Azure Kubernetes Service に関する既知の問題
author: abha
ms.topic: troubleshooting
ms.date: 09/22/2020
ms.author: abha
ms.reviewer: ''
ms.openlocfilehash: 5865c7cfea0231c811199ff4abbc05011e4d0a66
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101840050"
---
# <a name="known-issues-for-azure-kubernetes-service-on-azure-stack-hci-public-preview"></a>Azure Stack HCI 上の Azure Kubernetes Service のパブリック プレビューに関する既知の問題
この記事では、Azure Stack HCI 上の Azure Kubernetes Service のパブリック プレビュー リリースに関する既知の問題について説明します。

## <a name="recovering-from-a-failed-aks-on-azure-stack-hci-deployment"></a>Azure Stack HCI デプロイ上で失敗した AKS からの復旧
デプロイの問題が発生している場合や、デプロイをリセットする必要がある場合は、PowerShell 管理ウィンドウから Uninstall-AksHci を実行する前に、Azure Stack HCI 上の Azure Kubernetes Service に接続されている Windows Admin Center のすべてのインスタンスを必ず閉じてください。

## <a name="when-using-kubectl-to-delete-a-node-the-associated-vm-might-not-be-deleted"></a>kubectl を使用してノードを削除すると、関連付けられている VM が削除されないことがある
以下の手順に従った場合にこの問題が発生します。
* Kubernetes クラスターを作成する
* クラスターを 3 つ以上のノードにスケーリングする
* kubectl delete node <ノード名> を使用してノードを削除する 
* Run kubectl get nodes を実行する。 削除されたノードは出力に一覧表示されません
* PowerShell 管理ウィンドウを開く
* get-vm を実行する。 削除されたノードはまだ一覧表示されます

これにより、システムはノードがないことを認識しない状態になり、新しいノードが起動されません。 これは将来のリリースで修正される予定です

## <a name="time-synchronization-must-be-configured-across-all-physical-cluster-nodes-and-in-hyper-v"></a>すべての物理クラスター ノードにわたって Hyper-V 内で時刻の同期を構成する必要がある
gMSA と AD 認証を確実に機能させるには、Azure Stack HCI クラスター ノードが、時刻をドメイン コントローラーまたはその他のタイム ソースと同期するように構成されていること、および Hyper-V が、どの仮想マシンに対しても時刻を同期するように構成されていることを確認します。

## <a name="special-active-directory-permissions-are-needed-for-domain-joined-azure-stack-hci-nodes"></a>ドメインに参加した Azure Stack HCI ノードには特別な Active Directory アクセス許可が必要 
Azure Stack HCI 上に Azure Kubernetes Service をデプロイして構成するユーザーには、サーバーおよびサービス オブジェクトが作成される Active Directory コンテナーで AD オブジェクトを作成するための "フル コントロール" アクセス許可が必要です。 

## <a name="get-akshcilogs-command-may-fail"></a>Get-AksHciLogs コマンドが失敗することがある
大規模なクラスターでは、Get-AksHciLogs コマンドによって、例外がスローされること、ノードの列挙に失敗すること、c:\wssd\wssdlogs.zip 出力ファイルが生成されないことがあります。
これは、ファイルを zip 形式にする PowerShell コマンド `Compress-Archive` に、2 GB の出力ファイル サイズ制限があるためです。 この問題は将来のリリースで修正される予定です。

## <a name="azure-kubernetes-service-powershell-deployment-doesnt-check-for-available-memory-before-creating-a-new-target-cluster"></a>Azure Kubernetes Service の PowerShell によるデプロイで、新しいターゲット クラスターを作成する前に使用可能なメモリが調べられない
PowerShell コマンド Aks-Hci では、Kubernetes ノードを作成する前に、ホスト サーバーで使用可能なメモリが確認されません。 これにより、メモリ不足に至り、バーチャル マシンが起動しなくなる可能性があります。 このエラーは、現在正しく処理されておらず、明確なエラー メッセージを示さずにデプロイが応答しなくなります。
デプロイが応答しなくなった場合は、`Eventviewer` を開き、VM を起動するのに十分なメモリがないことを示す Hyper-V 関連のエラー メッセージがないか調べます。
この問題は将来のリリースで修正される予定です

## <a name="azure-kubernetes-service-deployment-fails-on-an-azure-stack-hci-configured-with-static-ips-vlans-sdn-or-proxies"></a>静的 IP、VLAN、SDN、またはプロキシを使用して構成された Azure Stack HCI では Azure Kubernetes Service のデプロイが失敗する。
静的 IP、VLAN、SDN、またはプロキシを使用する Azure Stack HCI クラスターに Azure Kubernetes Service をデプロイすると、クラスターの作成時にデプロイが失敗します。 この問題は将来のリリースで修正される予定です。

## <a name="ipv6-must-be-disabled-in-the-hosting-environment"></a>ホスティング環境では IPv6 を無効にする必要がある
IPv4 と IPv6 の両方のアドレスが物理 NIC にバインドされている場合、クラスタリング用の `cloudagent` サービスでは、通信に IPv6 アドレスが使用されます。 デプロイ フレームワークのその他のコンポーネントでは、IPv4 のみが使用されます。 これにより、Windows Admin Center はクラスターに接続できなくなり、マシンへの接続を試みるとリモート処理エラーが報告されます。
対処法:物理ネットワーク アダプターの IPv6 を無効にします。
この問題は将来のリリースで修正される予定です

## <a name="moving-virtual-machines-between-azure-stack-hci-cluster-nodes-quickly-leads-to-vm-startup-failures"></a>Azure Stack HCI クラスター ノード間で仮想マシンを移動するとすぐに VM 起動エラーが発生する
クラスター管理ツールを使用して、1 つのノード (ノード A) から Azure Stack HCI クラスターの別のノード (ノード B) に VM を移動すると、新しいノードで VM が起動に失敗することがあります。 その VM を元のノードに戻すと、そこでも起動に失敗します。
この問題は、最初の移行をクリーンアップするロジックが非同期に実行されるために発生します。 結果として、Azure Kubernetes Service の "VM の場所の更新" ロジックでは、ノード A 上の元の Hyper-V で VM が検出され、その VM は登録を解除する代わりに削除されます。
対処法:新しいノードで VM が正常に起動したことを確認してから、VM を元のノードに戻します。
この問題は将来のリリースで修正される予定です

## <a name="load-balancer-in-azure-kubernetes-service-requires-dhcp-reservation"></a>Azure Kubernetes Service でロード バランサーを使用には DHCP 予約が必要
Azure Stack HCI 上の Azure Kubernetes Service の負荷分散ソリューションでは、DHCP を使用してサービス エンドポイントに IP アドレスを割り当てます。 サービスの再起動によってサービス エンドポイントの IP アドレスが変わると、有効期限が短いために DHCP リースが期限切れになります。 したがって、Kubernetes 構成に含まれる IP アドレスが、エンド ポイント上にある IP アドレスと異なるために、サービスがアクセス不能になります。 これにより、Kubernetes クラスターが使用できなくなる状態に至る可能性があります。
この問題を回避するには、負荷分散されるサービス エンドポイントには MAC アドレス プールを使用し、プール内の各 MAC アドレスには特定の IP アドレスを予約します。
この問題は将来のリリースで修正される予定です。

## <a name="cannot-deploy-azure-kubernetes-service-to-an-environment-that-has-separate-storage-and-compute-clusters"></a>独立したストレージやコンピューティングのクラスターを持つ環境に Azure Kubernetes Service をデプロイできない
Windows Admin Center では、コンピューティングとストレージのリソースが同じクラスターによって提供されると予期されているため、独立したストレージやコンピューティングのクラスターを持つ環境には Azure Kubernetes Service がデプロイされません。 ほとんどの場合、コンピューティング クラスターによって公開される CSV が見つからず、デプロイの続行が拒否されます。
この問題は将来のリリースで修正される予定です。

## <a name="windows-admin-center-only-supports-azure-kubernetes-service-for-azure-stack-hci-in-desktop-mode"></a>Windows Admin Center では、Azure Kubernetes Service for Azure Stack HCI がデスクトップ モードでのみサポートされる
プレビューでは、すべての Azure Kubernetes Service for Azure Stack HCI の機能が、Windows Admin Center のデスクトップ モードでのみサポートされています。 Windows Admin Center ゲートウェイは、Windows 10 の PC にインストールする必要があります。 Windows Admin Center のインストール オプションの詳細については、[Windows Admin Center のドキュメント](/windows-server/manage/windows-admin-center/plan/installation-options)を参照してください。 追加のシナリオは、今後のリリースでサポートされる予定です。

## <a name="azure-kubernetes-service-host-setup-fails-in-windows-admin-center-if-reboots-are-required"></a>再起動が必要な場合、Windows Admin Center で Azure Kubernetes Service ホストのセットアップが失敗する
PowerShell や Hyper-V などの役割をインストールするために、使用している 1 つまたは複数のサーバーを再起動する必要がある場合、Azure Kubernetes Service ホストのセットアップ ウィザードは失敗します。 現在の回避策は、ウィザードを終了し、サーバーがオンラインに戻った後に同じシステムでもう一度実行します。 この問題は将来のリリースで修正される予定です。

## <a name="azure-registration-step-in-azure-kubernetes-service-host-setup-asks-to-try-again"></a>Azure Kubernetes Service ホスト セットアップの Azure 登録手順で、再試行を求められる
Windows Admin Center を使用して Azure Kubernetes Service ホストを設定するときに、Azure 登録ページで必要な情報を入力した後で再試行するように求められる場合があります。 この手順を続行するには、Windows Admin Center ゲートウェイでもう一度 Azure にサインインする必要がある場合があります。 この問題は将来のリリースで修正される予定です。

## <a name="windows-admin-center-doesnt-have-an-arc-offboarding-experience"></a>Windows Admin Center に Arc のオフボード エクスペリエンスがない
現在、Windows Admin Center には、Azure Arc からクラスターをオフボードするプロセスがありません。破棄されたクラスター上の Arc エージェントを削除するには、Azure portal でクラスターのリソース グループに移動し、Arc コンテンツを手動で削除します。 まだ稼働しているクラスター上の Arc エージェントを削除するには、ユーザーが次のコマンドを実行する必要があります。
```PowerShell
az connectedk8s delete
```

## <a name="when-setting-up-an-azure-kubernetes-service-host-using-windows-admin-center-setup-may-fail-if-file-explorer-is-open"></a>Windows Admin Center を使用して Azure Kubernetes Service ホストを設定するときに、エクスプローラーが開いていると設定が失敗する可能性があります
"確認と作成" 手順に達したときに、エクスプローラーが開いていて、**C:\Program Files\AksHci** ディレクトリにいる場合、作成は "プロセスでファイル 'C:\Program Files\AksHci\wssdcloudagent.exe' にアクセスできませんでした" というエラーで失敗する可能性があります。 これは、それが別のプロセスによって使用されているためです。 このエラーを回避するには、この手順に進む前に、エクスプローラーを閉じるか、別のディレクトリに移動します。 

## <a name="cannot-connect-windows-admin-center-to-azure-as-create-new-azure-app-id-fails"></a>新規 Azure アプリ ID の作成に失敗したため、Windows Admin Center を Azure に接続できません
ゲートウェイ上で Azure アプリ ID を自動的に作成して使用できないことが原因で、Windows Admin Center を Azure に接続できない場合は、Azure アプリ ID を作成し、ポータル上で適切なアクセス許可を割り当てます。 次に、 **[Use existing in the gateway]\(ゲートウェイで既存のものを使用\)** を選択します。 詳細については、[Azure へのゲートウェイの接続](/windows-server/manage/windows-admin-center/azure/azure-integration)に関するページをご覧ください。
