---
title: AKS on Azure Stack HCI 上で etcd シークレットを暗号化する
description: AKS on Azure Stack HCI 上で etcd シークレットを暗号化する方法について説明します
author: aabhathipsay
ms.topic: how-to
ms.date: 02/02/2021
ms.author: aabha
ms.reviewer: ''
ms.openlocfilehash: 82d40c2f97171196b90171e3dda850990bf2fbab
ms.sourcegitcommit: 2c6418ee465e67edd417961b1f5211b2e09dbd5f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102116718"
---
# <a name="encrypt-etcd-secrets-on-aks-on-azure-stack-hci-clusters"></a>AKS on Azure Stack HCI クラスター上で etcd シークレットを暗号化する

Kubernetes 内のシークレットは、パスワードや SSH キーなど、少量の機密データを含むオブジェクトです。 Kubernetes API サーバー内では、シークレットが  _etcd_ に格納されます。これは、すべてのクラスター データの Kubernetes バッキング ストアとして使用される可用性の高いキー値ストアです。 etcd シークレットの暗号化を有効にすると、[保存時の秘密データを暗号化](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)し、暗号化キーの管理とローテーションを自動化することもできます。 

> [!NOTE]
> このプレビュー リリースでは、etcd シークレットの暗号化は、新しいワークロード クラスターでのみ使用できます。 

## <a name="enable-encryption-of-etcd-secrets"></a>etcd シークレットの暗号化を有効にする

以下に示すように、[New-AksHciCluster](./new-akshcicluster.md) コマンドに `-enableSecretsEncryption` パラメーターを指定して、etcd シークレットの暗号化を有効にし、暗号化キーのローテーションを自動化します。 

```powershell
New-AksHciCluster -name mynewcluster -enableSecretsEncryption
```

`-enableSecretsEncryption` パラメーターを指定して新しいクラスターをデプロイしたら、この機能を無効にすることはできません。

## <a name="monitor-and-troubleshoot"></a>監視とトラブルシューティング

Kubernetes クラスター上でアプリケーションのデプロイを簡略化するには、使用可能な[ドキュメントとスクリプト](https://github.com/microsoft/AKS-HCI-Apps)を確認します。

- Elasticsearch、Fluent Bitビット、および Kibana を使用してログを設定するには、[ツールをインストールしてログを設定](https://github.com/microsoft/AKS-HCI-Apps/tree/main/Logging)する手順に従います
- 監視ツール Prometheus を使用するには、[Kubernetes クラスターに Prometheus をインストール](https://github.com/microsoft/AKS-HCI-Apps/tree/main/Monitoring#certs-and-keys-monitoring)する手順に従います。

> [!NOTE]
> ログは、`/var/log/pods` の下のコントロール プレーン ノード上でご確認いただけます。

## <a name="additional-resources"></a>その他のリソース

- [保存時の秘密データを暗号化する](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data)
- [データ暗号化に KMS プロバイダーを使用する](https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/)

## <a name="next-steps"></a>次のステップ

この攻略ガイドでは、新しいワークロード クラスターの etcd シークレットの暗号化を有効にする方法について学習しました。 次に、以下を実行できます。
- [Kubernetes クラスターに Linux アプリケーションをデプロイします](./deploy-linux-application.md)。
- [Kubernetes クラスターに Windows Server アプリケーションをデプロイします](./deploy-windows-application.md)。
