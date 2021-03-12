---
title: AksHci PowerShell モジュール
description: AksHci モジュールのコマンドを使用して、AKS on Azure Stack HCI を管理する方法について説明します
author: jessicaguan
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: 511b3ad0dcf791550f40136a73f47114e9784bbe
ms.sourcegitcommit: 2c6418ee465e67edd417961b1f5211b2e09dbd5f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102116752"
---
# <a name="akshci"></a>AksHci 

Azure Kubernetes Service on Azure Stack HCI と対話するためのコマンド。

## <a name="akshci-cmdlets"></a>AksHci コマンドレット

|         |            |
| ------- | ---------- |
| [get-akshcicluster](get-akshcicluster.md) | Azure Kubernetes Service ホストを含むデプロイ済みのクラスターを一覧表示します。 |
| [get-akshciclusterupgrades](get-akshciclusterupgrades.md) | Azure Kubernetes Service クラスターの利用可能なアップグレードを取得します。 |
| [get-akshciconfig](get-akshciconfig.md) | Azure Kubernetes Service ホストの現在の構成設定を一覧表示します。 |
| [get-akshcicredential](get-akshcicredential.md) | kubectl を使用してご自身のクラスターにアクセスします。 |
| [get-akshcieventlog](get-akshcieventlog.md) | AKS HCI PowerShell モジュールからすべてのイベント ログを取得します。 |
| [get-akshcikubernetesversion](get-akshcikubernetesversion.md) | マネージド Kubernetes クラスターを作成するための利用可能なバージョンを一覧表示します。 |
| [get-akshcilogs](get-akshcilogs.md) | お使いのすべてのポッドからログを含む zip 形式のフォルダーを作成します。 |
| [get-akshciupdates](get-akshciupdates.md) | Azure Kubernetes Service on Azure Stack HCI 用の利用可能な更新プログラムを一覧表示します。 |
| [get-akshciversion](get-akshciversion.md) | Azure Kubernetes Service on Azure Stack HCI の現在のバージョンを取得します。 |
| [get-akshcivmsize](get-akshcivmsize.md) | サポートされる VM のサイズを一覧表示します。 |
| [initialize-akshcinode](initialize-akshcinode.md) | すべての物理ノードでチェックを実行して、Azure Kubernetes Service on Azure Stack HCI をインストールするためのすべての要件が満たされているかどうかを確認します。 |
| [install-akshci](install-akshci.md) | Azure Kubernetes Service on Azure Stack HCI エージェント/サービスおよびホストをインストールします。 |
| [install-akshciadauth](install-akshciadauth.md) | Active Directory 認証をインストールします。 |
| [install-akshciarconboarding](install-akshciarconboarding.md) | Kubernetes コマンドライン ツール、kubectl をダウンロードしてインストールします。 |
| [new-akshcicluster](new-akshcicluster.md) | 新しいマネージド Kubernetes クラスターを作成します。 |
| [new-akshcinetworksetting](new-akshcinetworksetting.md) | 新しい仮想ネットワークに対してオブジェクトを作成します。 |
| [remove-akshcicluster](remove-akshcicluster.md) | マネージド Kubernetes クラスターを削除します。 |
| [restart-akshci](restart-akshci.md) | Azure Kubernetes Service on Azure Stack HCI を再起動し、デプロイされているすべての Kubernetes クラスターを削除します。 |
| [set-akshciclusternodecount](set-akshciclusternodecount.md) | クラスター内のコントロール プレーン ノードまたはワーカー ノードの数をスケーリングします。 |
| [set-akshciconfig](set-akshciconfig.md) | Azure Kubernetes Service ホストの構成を設定または更新します。 |
| [uninstall-akshci](uninstall-akshci.md) | Azure Kubernetes Service on Azure Stack HCI を削除します。 |
| [uninstall-akshciadauth](uninstall-akshciadauth.md) | Active Directory 認証を削除します。 |
| [update-akshci](update-akshci.md) | Azure Kubernetes Service ホストを最新の Kubernetes バージョンに更新します。 |
| [update-akshcicluster](update-akshcicluster.md) | マネージド Kubernetes クラスターを新しい Kubernetes または OS バージョンに更新します。 |

