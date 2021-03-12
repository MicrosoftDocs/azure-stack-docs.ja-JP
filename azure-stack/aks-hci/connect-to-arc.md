---
title: Azure Stack HCI クラスター上の Azure Kubernetes Service を Azure Arc for Kubernetes に接続する
description: Azure Stack HCI クラスター上の Azure Kubernetes Service を Azure Arc for Kubernetes に接続する
author: abha
ms.topic: how-to
ms.date: 12/02/2020
ms.author: abha
ms.reviewer: ''
ms.openlocfilehash: 96e1996cbf22e354b960b1a46a8848543942b0cc
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101839863"
---
# <a name="connect-an-azure-kubernetes-service-on-azure-stack-hci-cluster-to-azure-arc-for-kubernetes"></a>Azure Stack HCI クラスター上の Azure Kubernetes Service を Azure Arc for Kubernetes に接続する

> 適用対象:Azure Stack HCI 上の AKS、Windows Server 2019 Datacenter 上の AKS ランタイム

Azure Stack HCI クラスター上の Azure Kubernetes Service が Azure Arc に接続されると、Azure portal に表示されます。 また、Azure Resource Manager ID とマネージド ID が割り当てられます。 クラスターは、標準の Azure サブスクリプションに接続され、リソース グループ内に存在し、他の Azure リソースと同様にタグを受け取ることができます。

Kubernetes クラスターを Azure に接続するには、クラスター管理者がエージェントをデプロイする必要があります。 これらのエージェントは、`azure-arc` という名前の Kubernetes 名前空間で実行され、標準の Kubernetes デプロイです。 エージェントは、Azure への接続、Azure Arc ログおよびメトリックの収集、構成要求の監視を担当します。

Azure Arc 対応 Kubernetes では、転送中のデータをセキュリティで保護する業界標準の SSL がサポートされます。 また、保存中は Azure Cosmos DB データベースに暗号化された状態で格納されるので、データの機密性が確保されます。

以下の手順では、Azure Stack HCI クラスター上の Azure Kubernetes Service を Azure Arc にオンボードするためにサービス プリンシパルを使用する方法について説明します。**既に Windows Admin Center を通して Kubernetes クラスターを Azure Arc にオンボードしている場合は、これらの手順をスキップできます。**

## <a name="before-you-begin"></a>開始する前に

次の要件の準備ができていることを確認します。

* 少なくとも 1 つの Linux ワーカー ノードが稼働している Azure Stack HCI クラスター上の Azure Kubernetes Service。 

* Arc 対応の Kubernetes エージェントをデプロイするには、クラスターとクラスター上のクラスター管理者ロールにアクセスするための kubeconfig ファイルが必要です。
* Azure Stack HCI 上の Azure Kubernetes Service 用の PowerShell モジュールがインストールされている。
* Azure Arc 対応 Kubernetes CLI 拡張機能をインストールするには、Azure CLI バージョン 2.3 以降が必要です。 [Azure CLI のインストール](/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true)。 Azure CLI バージョン 2.3 以降を確実に用意するために、最新バージョンに更新することもできます。
* あなたが所有者または共同作成者となっている Azure サブスクリプション。 
* PowerShell 管理ウィンドウでこのドキュメントのコマンドを実行します。


## <a name="network-requirements"></a>ネットワークの要件

Azure Arc エージェントが機能するには、次のプロトコル、ポート、送信 URL が必要です。

* ポート 443 での TCP --> `https://:443`
* ポート 9418 での TCP --> `git://:9418`

| エンドポイント (DNS)                                                                                               | 説明                                                                                                                 |
| ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| `https://management.azure.com`                                                                                 | エージェントが Azure に接続してクラスターを登録するために必要です                                                        |
| `https://eastus.dp.kubernetesconfiguration.azure.com`, `https://westeurope.dp.kubernetesconfiguration.azure.com` | エージェントが状態をプッシュして構成情報をフェッチするためのデータ プレーン エンドポイント                                      |
| `https://docker.io`                                                                                            | コンテナー イメージをプルするために必要です                                                                                         |
| `https://github.com`、git://github.com                                                                         | GitOps リポジトリの例は、GitHub でホストされています。 構成エージェントには、指定する git エンドポイントへの接続が必要です。 |
| `https://login.microsoftonline.com`                                                                            | Azure Resource Manager トークンをフェッチして更新するために必要です                                                                                    |
| `https://azurearcfork8s.azurecr.io`                                                                            | Azure Arc エージェント用のコンテナー イメージをプルするために必要です                                                                  |
| `https://eus.his.arc.azure.com`, `https://weu.his.arc.azure.com`                                                                            |  システムによって割り当てられたマネージド ID 証明書をプルするために必須                                                                  |

## <a name="step-1-log-in-to-azure"></a>手順 1:Azure にログインする

Azure にログインし、ログイン後、あなたが所有者または共同作成者となっている Azure サブスクリプションを、既定のサブスクリプションとして設定します。

```console
az login
az account set --subscription "00000000-aaaa-bbbb-cccc-000000000000"
```

## <a name="step-2-register-the-two-providers-for-azure-arc-enabled-kubernetes"></a>手順 2:Azure Arc 対応 Kubernetes 用の 2 つのプロバイダーを登録する:

Azure Arc 対応 Kubernetes サービスの 2 つのプロバイダーが、既にサブスクリプションに登録されている場合は、この手順をスキップできます。 登録は非同期プロセスであり、サブスクリプションごとに 1 回行う必要があります。 登録には、10 分ほどかかる場合があります。 

```console
az provider register --namespace Microsoft.Kubernetes
az provider register --namespace Microsoft.KubernetesConfiguration
```

以下のコマンドを使用して、登録されているかどうかを確認できます。

```console
az provider show -n Microsoft.Kubernetes -o table
az provider show -n Microsoft.KubernetesConfiguration -o table
```

## <a name="step-3-create-a-resource-group"></a>手順 3:リソース グループを作成する

接続されたクラスター リソースを保持するリソース グループを作成する必要があります。 米国東部または西ヨーロッパの場所にある既存のリソース グループを使用できます。 米国東部または西ヨーロッパの場所に既存のリソース グループがない場合は、次のコマンドを使用して新しいリソース グループを作成します。

```console
az group create --name AzureArcTest -l EastUS -o table
```

## <a name="step-4-create-a-new-service-principal"></a>手順 4:新しいサービス プリンシパルを作成する


`contributor` ロールを持つサービス プリンシパルを既に作成していて、サービス プリンシパルの appID、パスワード、およびテナントの値がわかっている場合は、この手順をスキップできます。

わかりやすい名前で新しいサービス プリンシパルを作成します。 この名前は、Azure Active Directory テナントで一意である必要があります。 サービス プリンシパルの既定のロールは`Contributor`です。 このロールには、Azure アカウントの読み取りと書き込みを行うための完全なアクセス許可が付与されます。 このサービス プリンシパルを再利用して、複数のクラスターを Azure Arc にオンボードすることもできます。サービス プリンシパルのスコープを *subscriptions/resource-group* に設定します。 *サービス プリンシパルの appID、パスワード、およびテナントの値は、後の手順でそれらの詳細が必要になるため、必ず保存しておいてください。*

```console
az ad sp create-for-RBAC --name "azure-arc-for-k8s" --scope /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}
```

**出力:**

```
{
  "appId": "00000000-0000-0000-0000-000000000000",
  "displayName": "azure-arc-for-k8s",
  "name": "https://azure-arc-for-k8s",
  "password": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
  "tenant": "ffffffff-gggg-hhhh-iiii-jjjjjjjjjjjj"
}
```
## <a name="step-5-save-service-principal-details"></a>手順 5:サービス プリンシパルの詳細を保存する
作成したサービス プリンシパルの appId、パスワードとテナントの値、クラスター名、Azure サブスクリプション ID、リソース グループの名前と場所を、PowerShell 変数に保存します。 これにより、他のチュートリアルで詳細を再利用できるようになります。 PowerShell セッションを閉じる場合に備えて、これらの値をメモ帳で保存しておいてください。

```PowerShell
$clusterName = #<name of your Kubernetes cluster>
$resourceGroup = #<Azure resource group to store your connected Kubernetes cluster in Azure Arc>
$location = #<Azure resource group location. This can only be eastus or westeurope for Azure Arc for Kubernetes>
$subscriptionId = #<Azure subscription Id>
$appId = #<appID from the service principal created above>
$password = #<password from the service principal created above>
$tenant = #<tenant from the service principal created above>
```
次を実行して、変数に正しい値を割り当てたことを確認します。

```PowerShell
echo $clusterName 
echo $resourceGroup
echo $location 
echo $subscriptionId 
echo $appId 
echo $password 
echo $tenant 
```

## <a name="step-6-connect-to-azure-arc-using-service-principal-and-the-aks-hci-powershell-module"></a>手順 6:サービス プリンシパルと Aks-Hci PowerShell モジュールを使用して Azure Arc に接続する

次に、サービス プリンシパルと Aks-Hci PowerShell モジュールを使用して、Kubernetes クラスターを Azure に接続します。 この手順で、Kubernetes 用の Azure Arc エージェントを `azure-arc` 名前空間にデプロイします。

新しく作成されたサービス プリンシパルを参照し、Aks-Hci PowerShell モジュールで使用可能な `Install-AksHciArcOnboarding` コマンドを実行します。

```PowerShell
Install-AksHciArcOnboarding -name $clusterName -resourcegroup $resourceGroup -location $location -subscriptionid $subscriptionId -clientid $appId -clientsecret $password -tenantid $tenant
```
## <a name="verify-connected-cluster"></a>接続されたクラスターを検証する

[Azure portal](https://portal.azure.com/) で Kubernetes クラスター リソースを表示できます。 ブラウザーでポータルを開いたら、先ほど `Install-AksHciArcOnboarding` PowerShell コマンドで使用したリソース名とリソース グループ名の入力に基づいて、リソース グループと Azure Arc 対応の Kubernetes リソースに移動します。

> [!NOTE]
> クラスターをオンボードした後、Azure portal の Azure Arc 対応の Kubernetes リソースの概要ページで、クラスターのメタデータ (クラスターのバージョン、エージェントのバージョン、ノードの数) が画面に表示されるまでに約 5 分から 10 分かかります。

クラスターを削除したり、クラスターが送信プロキシ サーバーの背後にある場合にクラスターを接続したりするには、[Azure Arc 対応 Kubernetes クラスターの接続](/azure/azure-arc/kubernetes/connect-cluster)に関するページを参照してください。

## <a name="azure-arc-agents-for-kubernetes"></a>Kubernetes 用 Azure Arc エージェント

Azure Arc 対応の Kubernetes では、`azure-arc` 名前空間にいくつかのオペレーターがデプロイされます。 これらのデプロイとポッドを表示するには、以下のように `kubectl` を使用します。 

```console
kubectl -n azure-arc get deployments,pods
```

Azure Arc 対応の Kubernetes は、`azure-arc` 名前空間にデプロイされたクラスター内で実行されるいくつかのエージェント (オペレーター) で構成されています。

* `deployment.apps/config-agent`: クラスターに適用されるソース管理構成リソースの接続されたクラスターを監視し、コンプライアンス状態を更新します
* `deployment.apps/controller-manager`: オペレーターのオペレーターであり、Azure Arc コンポーネント間の相互作用を統制します
* `deployment.apps/metrics-agent`: 他の Arc エージェントのメトリックを収集し、これらのエージェントのパフォーマンスが最適であることを確認します
* `deployment.apps/cluster-metadata-operator`: クラスターのメタデータを収集します (クラスターのバージョン、ノード数、Azure Arc エージェントのバージョン)
* `deployment.apps/resource-sync-agent`: 前述のクラスター メタデータを Azure に同期します
* `deployment.apps/clusteridentityoperator`: Azure Arc 対応 Kubernetes では、現在、システムによって割り当てられた ID がサポートされています。 clusteridentityoperator では、他のエージェントが Azure との通信に使用する管理サービス ID (MSI) 証明書が保持されます。
* `deployment.apps/flux-logs-agent`: ソース管理構成の一部としてデプロイされる Flux オペレーターからログを収集します

## <a name="next-steps"></a>次のステップ

* [接続されたクラスターで GitOps を使用する](/azure/azure-arc/kubernetes/use-gitops-connected-cluster)
* [Azure Policy を使用してクラスター構成を管理する](/azure/azure-arc/kubernetes/use-azure-policy)
* [Azure Arc 対応 Kubernetes クラスターで Azure Monitor を有効にする](/azure/azure-monitor/insights/container-insights-enable-arc-enabled-clusters)
