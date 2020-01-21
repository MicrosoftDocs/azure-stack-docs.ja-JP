---
title: Azure Stack 上の App Service をバックアップする | Microsoft Docs
description: Azure Stack 上の App Services をバックアップする方法について説明します。
services: azure-stack
documentationcenter: ''
author: bryanla
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: app-service
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/23/2019
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 03/21/2019
ms.openlocfilehash: a41943a598545b1a4c5dbe6325307a8fa3594cd5
ms.sourcegitcommit: 245a4054a52e54d5989d6148fbbe386e1b2aa49c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/13/2019
ms.locfileid: "70975033"
---
# <a name="back-up-app-service-on-azure-stack"></a>Azure Stack 上の App Service をバックアップする

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*  

このドキュメントでは、Azure Stack 上の App Service をバックアップする方法の手順を説明します。

> [!IMPORTANT]
> Azure Stack 上の App Service は、[Azure Stack のインフラストラクチャのバックアップ](azure-stack-backup-infrastructure-backup.md)の一部としてバックアップされません。 Azure Stack のオペレーターとして、必要な場合に App Service を正常に復元できるように確実に手順を行う必要があります。

Azure Stack 上の Azure App Service のディザスター リカバリーを計画する場合、
1. リソース プロバイダーのインフラストラクチャ、サーバー ロール、worker 階層などの 4 つの主な要素を考慮する必要があります。 
2. App Service のシークレット。
3. App Service SQL Server ホスティングおよびメータリング データベース。
4. App Service のファイル共有に格納された App Service ユーザー ワークロード コンテンツ。

## <a name="back-up-app-service-secrets"></a>App Service のシークレットのバックアップ
App Service をバックアップから復旧する場合、初期デプロイで使用した App Service キーを入力する必要があります。 この情報は、App Service が正常にデプロイされ、安全な場所に格納されたら、すぐに保存する必要があります。 リソース プロバイダーのインフラストラクチャ構成は、App Service リカバリー PowerShell コマンドレットを使用して、復旧中にバックアップから再作成されます。

次の手順に従って、管理ポータルからアプリ サービスのシークレットをバックアップします。 

1. サービス管理者として Azure Stack 管理者ポータルにサインインします。

2. **[App Service]**  ->  **[シークレット]** に移動します。 

3. **[シークレットのダウンロード]** を選択します。

   ![Azure Stack 管理者ポータルでシークレットをダウンロードする](./media/app-service-back-up/download-secrets.png)

4. シークレットのダウンロードの準備が整ったら、 **[保存]** をクリックして安全な場所に App Service のシークレット (**SystemSecrets.JSON**) ファイルを保存します。 

   ![Azure Stack 管理者ポータルでシークレットを保存する](./media/app-service-back-up/save-secrets.png)

> [!NOTE]
> これらの手順は、App Service のシークレットを更新するたびに繰り返します。

## <a name="back-up-the-app-service-databases"></a>App Service のデータベースのバックアップ
App Service を復元するには、**Appservice_hosting** データベースおよび **Appservice_metering** データベースのバックアップが必要です。 これらのデータベースが確実に定期的にバックアップされ安全な場所に保存されるようにするには、SQL Server メンテナンス プランまたは Azure Backup Server を使用することをお勧めします。 しかし、必ず定期的に SQL のバックアップが作成されるのであれば、いかなる方法でも使用できます。

SQL Server へのログイン中にこれらのデータベースを手動でバックアップするには、次の PowerShell のコマンドを使用します。

  ```powershell
  $s = "<SQL Server computer name>"
  $u = "<SQL Server login>" 
  $p = read-host "Provide the SQL admin password"
  sqlcmd -S $s -U $u -P $p -Q "BACKUP DATABASE appservice_hosting TO DISK = '<path>\hosting.bak'"
  sqlcmd -S $s -U $u -P $p -Q "BACKUP DATABASE appservice_metering TO DISK = '<path>\metering.bak'"
  ```

> [!NOTE]
> SQL AlwaysOn データベースをバックアップする必要がある場合、[これらの手順](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/configure-backup-on-availability-replicas-sql-server?view=sql-server-2017)に従ってください。 

すべてのデータベースを正常にバックアップできたら、.bak ファイルを App Service のシークレット情報と共に安全な場所にコピーします。

## <a name="back-up-the-app-service-file-share"></a>App Service のファイル共有のバックアップ
App Service では、ファイル共有にテナント アプリの情報が格納されます。 復元を行う必要がある場合に失われるデータを最小限にするには、このファイル共有を App Service データベースと共に定期的にバックアップする必要があります。

App Service のファイル共有のコンテンツをバックアップするには Azure Backup Server を使用するか、以前のすべての復旧情報を保存した場所にファイル共有のコンテンツを定期的にコピーする別の方法を使用します。

たとえば、次の手順を使用して、(PowerShell ISE ではなく) Windows PowerShell コンソール セッションから Robocopy を使用します。

```powershell
$source = "<file share location>"
$destination = "<remote backup storage share location>"
net use $destination /user:<account to use to connect to the remote share in the format of domain\username> *
robocopy $source $destination
net use $destination /delete
```

## <a name="next-steps"></a>次の手順
[Azure Stack 上の App Service の復元](app-service-recover.md)
