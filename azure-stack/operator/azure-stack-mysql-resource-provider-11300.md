---
title: Azure Stack Hub MySQL リソース プロバイダー 1.1.30.0 のリリース ノート
description: このリリース ノートでは、Azure Stack Hub MySQL リソース プロバイダー 1.1.30.0 の更新プログラムの新機能を紹介しています。
author: bryanla
ms.topic: article
ms.date: 1/22/2020
ms.author: bryanla
ms.reviewer: jiahan
ms.lastreviewed: 12/10/2019
ms.openlocfilehash: 710b5d48e4ce906eb357c5e17cff4132bcd747ba
ms.sourcegitcommit: a630894e5a38666c24e7be350f4691ffce81ab81
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "77698931"
---
# <a name="mysql-resource-provider-11300-release-notes"></a>MySQL リソース プロバイダー 1.1.30.0 のリリース ノート

これらのリリース ノートでは、MySQL リソース プロバイダー バージョン 1.1.30.0 の機能強化と既知の問題について説明します。

## <a name="build-reference"></a>ビルドのリファレンス
MySQL リソース プロバイダー バイナリをダウンロードした後、自己展開ツールを実行してコンテンツを一時ディレクトリに展開します。 リソース プロバイダーには、対応する最低限の Azure Stack Hub のビルドがあります。 MySQL リソース プロバイダーのこのバージョンをインストールするために必要な Azure Stack Hub の最小リリース バージョンを次に示します。

> |Azure Stack Hub の最小バージョン|MySQL リソース プロバイダーのバージョン|
> |-----|-----|
> |Azure Stack Hub 1808 の更新プログラム (1.1808.0.97)|[1.1.30.0](https://aka.ms/azurestackmysqlrp11300)|
> |     |     |

> [!IMPORTANT]
> MySQL リソース プロバイダーの最新バージョンをデプロイする前に、サポートされている最低限の Azure Stack Hub 更新プログラムを Azure Stack Hub 統合システムに適用するか、最新の Azure Stack Development Kit (ASDK) をデプロイします。

## <a name="new-features-and-fixes"></a>新機能と修正
このバージョンの Azure Stack Hub MySQL リソース プロバイダーには、次の機能強化と修正プログラムが含まれています。

- **MySQL リソース プロバイダーのデプロイに対して有効になっているテレメトリ**。 MySQL リソース プロバイダーのデプロイに対してテレメトリ収集が有効になっています。 収集されるテレメトリには、リソース プロバイダーのデプロイ、開始時刻と終了時刻、終了状態、終了メッセージ、エラーの詳細 (該当する場合) が含まれています。

- **TLS 1.2 暗号化の更新**。 リソース プロバイダーと Azure Stack Hub の内部コンポーネントとの通信には、TLS 1.2 のみへのサポートが有効になっています。 

### <a name="fixes"></a>修正

- **MySQL リソース プロバイダーの Azure Stack Hub PowerShell との互換性**。 MySQL リソース プロバイダーは、Azure Stack Hub 2018-03-01-hybrid PowerShell プロファイルを使用でき、AzureRM 1.3.0 以降と互換性があるように更新されています。

- **MySQL ログインの [パスワードの変更] ブレード**。 [パスワードの変更] ブレードでパスワードを変更できないという問題が修正されました。 パスワード変更通知からリンクを削除しました。

## <a name="known-issues"></a>既知の問題

- **MySQL SKU はポータルに表示されるまで最大 1 時間かかることがあります**。 新しい MySQL データベースを作成するときに、新規に作成される SKU が表示されて使用できるようになるまで、最大 1 時間かかることがあります。

    **回避策**:[なし] :

- **再利用された MySQL ログイン**。 同じサブスクリプションの既存のログインと同じユーザー名で新しい MySQL ログインを作成しようとすると、同じログインと既存のパスワードが再利用されます。

    **回避策**:同じサブスクリプションに新しいログインを作成するときに別のユーザー名を使用するか、同じユーザー名のログインを異なるサブスクリプションに作成します。

- **TLS 1.2 サポート要件**。 TLS 1.2 が有効になっていないコンピューターから MySQL リソース プロバイダーをデプロイまたは更新しようとすると、操作が失敗する可能性があります。 リソース プロバイダーのデプロイまたは更新に使用するコンピューターで、次の PowerShell コマンドを実行して、TLS 1.2 がサポート対象であるかどうかを確認します。

  ```powershell
  [System.Net.ServicePointManager]::SecurityProtocol
  ```

  **Tls12** がコマンドの出力に含まれていない場合、TLS 1.2 はそのコンピューターでは有効ではありません。

    **回避策**:次の PowerShell コマンドを実行して TLS 1.2 を有効にし、同じ PowerShell セッションからリソース プロバイダーのデプロイを開始するかまたはスクリプトを更新します。

    ```powershell
    [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12
    ```
 
### <a name="known-issues-for-cloud-admins-operating-azure-stack-hub"></a>Azure Stack Hub を運用するクラウド管理者に関する既知の問題
[Azure Stack Hub リリース ノート](azure-stack-servicing-policy.md)内のドキュメントを参照してください。

## <a name="next-steps"></a>次のステップ
[MySQL リソース プロバイダーの詳細を確認します](azure-stack-mysql-resource-provider.md)。

[MySQL リソースプロバイダーのデプロイを準備します](azure-stack-mysql-resource-provider-deploy.md#prerequisites)。

[MySQL リソース プロバイダーを以前のバージョンからアップグレードします](azure-stack-mysql-resource-provider-update.md)。 