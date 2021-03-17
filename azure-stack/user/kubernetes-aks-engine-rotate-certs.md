---
title: Azure Stack Hub 上で Kubernetes 証明書をローテーションする
description: Azure Stack Hub 上で Kubernetes 証明書をローテーションする方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 3/1/2021
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 3/1/2021
ms.openlocfilehash: fa46de76f59e664fb3dd4a1c771ad7b835f97ca4
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102233891"
---
# <a name="rotate-kubernetes-certificates-on-azure-stack-hub"></a>Azure Stack Hub 上で Kubernetes 証明書をローテーションする

このドキュメントでは、既存の AKS エンジン クラスター上で証明書をローテーションする方法に関するガイダンスと、`aks-engine rotate-certs` をツールとして採用する場合の推奨事項を示します。

> [!IMPORTANT]  
> 現在、この機能はパブリック プレビュー段階にあります。
> このプレビュー バージョンはサービス レベル アグリーメントなしで提供されています。運用環境のワークロードに使用することはお勧めできません。 特定の機能はサポート対象ではなく、機能が制限されることがあります。 詳しくは、[Microsoft Azure プレビューの追加使用条件](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)に関するページをご覧ください。

## <a name="prerequisites"></a>前提条件

このガイドでは、AKS エンジンを使用してクラスターが既にデプロイされていて、そのクラスターが正常な状態にあることを前提としています。
## <a name="planning-for-certificate-rotation"></a>証明書のローテーションの計画

この機能の使用を検討している場合は、更新、検証、再起動の各ステップの間、Kubernetes コントロール プレーンが使用できなくなることに注意してください。 このメンテナンス操作は、必要に応じて計画してください。 また、運用環境で試す前に、ステージング環境において運用環境と同じ構成を使用してこの操作を実施することも計画してください。

この操作を試行する前に、以下の考慮事項を確認してください。

-  `aks-engine deploy` または `aks-engine generate` コマンドによって生成された API モデル (`apimodel.json`) にアクセスする必要があります。 既定では、このファイルは `_output/<clustername>/` などの相対ディレクトリに配置されます。
-  `aks-engine rotate-certs` 操作を実行すると、API サーバーのダウンタイムが発生します。
-  `aks-engine rotate-certs` では、クラスターの現在の状態に準拠した API モデルが必要です。 `aks-engine rotate-certs` では、クラスター ノード上でリモート コマンドを実行し、API モデルの情報を使用してセキュリティで保護された SSH 接続を確立します。 `aks-engine rotate-certs` はまた、元の `aks-engine` デプロイに従って名前が付けられる一部のリソースにも依存しています。たとえば、VM には `aks-engine` によって指定された名前付けを適用する必要があります。
-  `aks-engine rotate-certs` では、証明書のローテーション中は、クラスター コントロール プレーンへの動作中の接続に依存して、次のことを行います。
    - プロセスの各ステップを検証する。
    - **kube-system ポッド** やサービス アカウント トークンなどのクラスター リソースを再起動または再作成する。

    外部へのアクセスが閉じられている VNet 内のクラスターの証明書をローテーションする場合は、コントロール プレーンにネットワーク アクセスできるホスト VM (たとえば、マスター VM と同じ VNet 内に存在するジャンプボックス VM) から `aks-engine rotate-certs` を実行する必要があります。

- 運用環境で `aks-engine rotate-certs` を使用する場合は、同じ仕様に基づいて構築されたクラスター上で証明書ローテーション テストをステージすることをお勧めします。 つまり、ご利用の運用クラスターと同じクラスター構成、同じバージョンの AKS エンジン コマンドライン ツール、同じ有効になっているアドオン セットを使用して、そのクラスターを構築してから、証明書のローテーションを実行します。 AKS エンジンでは、さまざまなクラスター構成がサポートされています。しかし、AKS エンジン チームが実行するエンドツーエンド テスト範囲で、考えられるすべての構成が網羅されるのは実質的に不可能です。 そのため、ご利用の運用クラスター上で操作を試行する前に、ステージング環境においてお客様固有のクラスター構成が `aks-engine rotate-certs` で機能するように確認することをお勧めします。
-  `aks-engine rotate-certs` では、下位互換性は保証 **されません**。 aks-engine バージョン 0.60.x を使用してデプロイした場合は、バージョン 0.60.x を使用して証明書ローテーション プロセスを実行することをお勧めします。
-  現時点では、Key Vault からの新しい証明書セットの取り込みはサポートされていません。
- 信頼できるネットワーク接続を使用してください。 `aks-engine rotate-certs` では、複数のリモート コマンドを実行する必要があります。そのため、クラスター ノードへの接続に信頼性がない場合は特に、障害が発生する可能性があります。 ターゲット Azure Stack スタンプ上で実行されている VM から `aks-engine rotate-certs` を実行すると、一時的な問題の発生を減らすことができます。

## <a name="parameters"></a>パラメーター

| パラメーター           | 必須 | 説明 |
| --- | --- | --- |
| --api-model             | はい          | 想定されるクラスター構成を宣言する API モデル (クラスター定義) への相対パス。       |
| --ssh-host              | はい          | クラスター内のすべてのノードに到達できる SSH リスナーの完全修飾ドメイン名 (FQDN) または IP アドレス。                            |
| --linux-ssh-private-key | はい          | クラスターの Linux ノードにアクセスするための有効なプライベート SSH キーへのパス。                                        |
| --location              | はい          | クラスターがデプロイされている Azure の場所。                                                               |
| --subscription-id       | はい          | クラスター インフラストラクチャがデプロイされている Azure サブスクリプション。                                                     |
| --resource-group        | はい          | クラスター インフラストラクチャがデプロイされている Azure リソース グループ。                                                   |
| --client-id             | 場合により異なる      | サービス プリンシパルのクライアント ID。 auth-method が client_secret または client_certificate に設定されている場合は必須。 |
| --client-secret         | 場合により異なる      | サービス プリンシパルのクライアント シークレット。 auth-method が client_secret に設定されている場合は必須。                   |
| --azure-env             | 場合により異なる      | ターゲット クラウド名。 ターゲット クラウドが AzureCloud である場合は省略可能。                                              |
| --certificate-profile   | no           | 新しい証明書セットを含む JSON ファイルへの相対パス。                                        |
| --force                 | no           | API サーバーが応答していない場合でも、強制的に実行する。                                                       |

## <a name="simple-steps-to-rotate-certificates"></a>証明書をローテーションするためのシンプルなステップ

すべての[要件](https://github.com/Azure/aks-engine/blob/master/docs/topics/rotate-certs.md#pre-requirements)を読み終えたら、次のように適切な引数を使用して `aks-engine rotate-certs` を実行します。

```bash  
./bin/aks-engine rotate-certs \
  --location <resource-group-location> \
  --api-model <generated-apimodel.json> \
  --linux-ssh-private-key <private-SSH-key> \
  --ssh-host <apiserver-URI> \
  --resource-group <resource-group-name> \
  --client-id <service-principal-id> \
  --client-secret <service-principal-secret> \
  --subscription-id <subscription-id> \
  --azure-env <cloud-name>
```

次に例を示します。

```bash  
./bin/aks-engine rotate-certs \
  --location "westus2" \
  --api-model "_output/my-cluster/apimodel.json" \
  --linux-ssh-private-key "~/.ssh/id_rsa" \
  --ssh-host "my-cluster.westus2.cloudapp.azure.com"\
  --resource-group "my-cluster" \
  --client-id "12345678-XXXX-YYYY-ZZZZ-1234567890ab" \
  --client-secret "12345678-XXXX-YYYY-ZZZZ-1234567890ab" \
  --subscription-id "12345678-XXXX-YYYY-ZZZZ-1234567890ab" \
  --azure-env "AzureStackCloud" # optional if targeting AzureCloud
```

## <a name="troubleshooting"></a>トラブルシューティング

障害または一時的な問題 (ネットワーク接続など) が原因で、証明書ローテーション プロセスが完了前に停止した場合は、`--force` フラグを使用して `aks-engine rotate-certs` を再実行するのが安全です。

また、`/var/log/azure/rotate-certs.log` (Linux)、`c:\\k\\rotate-certs.log` (Windows) の各ファイル内にすべてのステップの出力が `aks-engine rotate-certs` によってログされることに注目してください。

この操作を実行したときの内部の動作の詳細、またはさらなるカスタマイズについては、「[Under The Hood](https://github.com/Azure/aks-engine/blob/master/docs/topics/rotate-certs.md#under-the-hood)」(内部のしくみ) を参照してください。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 上の AKS エンジン](azure-stack-kubernetes-aks-engine-overview.md)を確認してください  
