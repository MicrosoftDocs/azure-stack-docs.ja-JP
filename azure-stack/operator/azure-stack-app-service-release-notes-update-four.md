---
title: App Service on Azure Stack Update 4 のリリース ノート | Microsoft Docs
description: App Service on Azure Stack Update 4 の内容、既知の問題、更新プログラムをダウンロードする場所について説明します。
services: azure-stack
documentationcenter: ''
author: bryanla
manager: stefsch
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/25/2019
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 03/25/2019
ms.openlocfilehash: b65e7c3b1cf381db3e36567c9f9ccf71a2150cd7
ms.sourcegitcommit: 797dbacd1c6b8479d8c9189a939a13709228d816
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/28/2019
ms.locfileid: "66269133"
---
# <a name="app-service-on-azure-stack-update-4-release-notes"></a>App Service on Azure Stack update 4 のリリース ノート

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*

これらのリリース ノートでは、Azure App Service on Azure Stack Update 4 における機能強化と修正点、および既知の問題について説明します。 既知の問題は、デプロイおよび更新プロセスに直接関係する問題と、ビルド (インストール後) に関する問題に分けられています。

> [!IMPORTANT]
> Azure App Service 1.4 をデプロイする前に、Azure Stack 統合システムに 1809 更新プログラムを適用するか、または最新の Azure Stack 開発キットをデプロイします。
>
>

## <a name="build-reference"></a>ビルドのリファレンス

App Service on Azure Stack Update 4 のビルド番号は **78.0.13698.5** です。

### <a name="prerequisites"></a>前提条件

デプロイを開始する前に、[前提条件に関するドキュメント](azure-stack-app-service-before-you-get-started.md)をご覧ください。

Azure App Service on Azure Stack の 1.4 へのアップグレードを開始する前に:

- Azure Stack 管理ポータルにおいて Azure App Service の [管理] ですべてのロールが [準備完了] になっていることを確認します。

- App Service とマスター データベースをバックアップします。
  - AppService_Hosting
  - AppService_Metering
  - Master

- テナント アプリのコンテンツ ファイル共有をバックアップします。

- Marketplace からカスタム スクリプト拡張機能バージョン 1.9 を配信します。

### <a name="new-features-and-fixes"></a>新機能と修正

Azure App Service on Azure Stack Update 4 には、次の機能強化と修正が含まれています。

- [CVE 2018-8600](https://aka.ms/CVE20188600) クロスサイト スクリプトの脆弱性の解決策。

- App Service 2018-02-01 API バージョンのサポートが追加されました。

- **App Service のテナント ポータル、管理ポータル、Functions ポータル、Kudu ツール**の更新。 Azure Stack Portal SDK バージョンと一致しています。

- **Azure Functions ランタイム**が **v1.0.11959** に更新されました。

- 信頼性を高めるためのコア サービスと、一般的な問題を簡単に診断できるようにするエラー メッセージの更新。

- **次のアプリケーション フレームワークとツールの更新**:
  - NodeJS 10.6.0 を追加
  - NPM 6.1.0 を追加
  - Zulu OpenJDK 8.31.0.2 を追加
  - Tomcat 8.5.34 と 9.0.12 を追加
  - 次の PHP バージョンを追加
    - 5.6.37
    - 7.0.31
    - 7.1.20
    - 7.2.8
  - Python バージョンの更新:
    - 2.7.15
    - 3.6.6
  - Git for Windows を v 2.17.1.2 に更新
  - Kudu を 78.11022.3613 に更新
  
- **すべてのロールの基になっているオペレーティング システムの更新プログラム**:
  - [x64 ベース システム用 Windows Server 2016 に対する 2018-10 累積的更新プログラム (KB4462928)](https://support.microsoft.com/help/4462928/windows-10-update-kb4462928)

- Wordpress、DNN、および Orchard CMS ギャラリー アイテムをデプロイする際のテンプレートの検証に関する問題が解決されました。

- Azure Stack が Azure Resource Manager クライアント証明書をローテーションする際の構成の問題が解決されました。

- App Service テナント ポータルでの [クロス オリジン リソース共有] 設定の機能が復元されました。

- リソース プロバイダーのコントロール プレーンが構成済みの SQL Server インスタンスに接続できないとき、App Service 管理ポータル エクスペリエンスにエラー メッセージを表示します。

- エンドポイントが新しい関数アプリケーションに指定されたとき、確実にカスタム ストレージ接続文字列にエンドポイントが指定されるようにします。

### <a name="post-deployment-steps"></a>デプロイ後の手順

> [!IMPORTANT]  
> SQL Always On インスタンスを使用して App Service RP を提供している場合は、データベースのフェールオーバー時にサービスの停止を防ぐために、[appservice_hosting と appservice_metering データベースを可用性グループに追加](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/availability-group-add-a-database)し、それらのデータベースを同期する必要があります。

### <a name="post-update-steps-optional"></a>更新後の手順 (省略可)

既存の Azure App Service on Azure Stack デプロイに関して包含データベースへの移行を検討されている場合は、Azure App Service on Azure Stack 1.4 への更新が完了した後で、以下の手順を実行してください。

> [!IMPORTANT]
> 移行手順の所要時間は 5 分から 10 分程度です。  この手順には、既存のデータベース ログイン セッションを中止する作業が伴います。  Azure App Service on Azure Stack の移行とその後の検証に伴うダウンタイムを考慮してください。  Azure App Service on Azure Stack 1.3 に更新した後でこれらの手順を実行した場合、これらの手順は必要ありません。
>
>

1. [AppService データベース (appservice_hosting and appservice_metering) を可用性グループ](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/availability-group-add-a-database)に追加します

1. 包含データベースを有効にします。
    ```sql

        sp_configure 'contained database authentication', 1;
        GO
        RECONFIGURE;
            GO
    ```

1. データベースを部分的包含に変換すると、変換の際にすべてのアクティブなセッションを停止する必要があるため、ダウンタイムが引き起こされます。

    ```sql
        /******** [appservice_metering] Migration Start********/
            USE [master];

            -- kill all active sessions
            DECLARE @kill varchar(8000) = '';  
            SELECT @kill = @kill + 'kill ' + CONVERT(varchar(5), session_id) + ';'  
            FROM sys.dm_exec_sessions
            WHERE database_id  = db_id('appservice_metering')

            EXEC(@kill);

            USE [master]  
            GO  
            ALTER DATABASE [appservice_metering] SET CONTAINMENT = PARTIAL  
            GO  

        /********[appservice_metering] Migration End********/

        /********[appservice_hosting] Migration Start********/

            -- kill all active sessions
            USE [master];

            DECLARE @kill varchar(8000) = '';  
            SELECT @kill = @kill + 'kill ' + CONVERT(varchar(5), session_id) + ';'  
            FROM sys.dm_exec_sessions
            WHERE database_id  = db_id('appservice_hosting')

            EXEC(@kill);

            -- Convert database to contained
            USE [master]  
            GO  
            ALTER DATABASE [appservice_hosting] SET CONTAINMENT = PARTIAL  
            GO  

            /********[appservice_hosting] Migration End********/
    '''

1. Migrate Logins to Contained Database Users

    ```sql
        IF EXISTS(SELECT * FROM sys.databases WHERE Name=DB_NAME() AND containment = 1)
        BEGIN
        DECLARE @username sysname ;  
        DECLARE user_cursor CURSOR  
        FOR
            SELECT dp.name
            FROM sys.database_principals AS dp  
            JOIN sys.server_principals AS sp
                ON dp.sid = sp.sid  
                WHERE dp.authentication_type = 1 AND dp.name NOT IN ('dbo','sys','guest','INFORMATION_SCHEMA');
            OPEN user_cursor  
            FETCH NEXT FROM user_cursor INTO @username  
                WHILE @@FETCH_STATUS = 0  
                BEGIN  
                    EXECUTE sp_migrate_user_to_contained
                    @username = @username,  
                    @rename = N'copy_login_name',  
                    @disablelogin = N'do_not_disable_login';  
                FETCH NEXT FROM user_cursor INTO @username  
            END  
            CLOSE user_cursor ;  
            DEALLOCATE user_cursor ;
            END
        GO
    ```

検証

1. SQL Server で包含が有効になっているかどうかを確認します。

    ```sql
        sp_configure  @configname='contained database authentication'
    ```

1. 既存の包含の動作を確認します。
    ```sql
        SELECT containment FROM sys.databases WHERE NAME LIKE (SELECT DB_NAME())
    ```

### <a name="known-issues-post-installation"></a>既知の問題 (インストール後)

- App Service が既存の仮想ネットワークにデプロイされ、ファイル サーバーがプライベート ネットワークでしか使用できない場合、worker はファイル サーバーに到達することができません。Azure App Service on Azure Stack のデプロイ ドキュメントで説明されているとおりです。

ファイル サーバーに接続するために既存の仮想ネットワークと内部 IP アドレスへデプロイする場合は、送信セキュリティ規則を追加して、worker サブネットとファイル サーバー間の SMB トラフィックを有効にする必要があります。 管理者ポータルで WorkersNsg に移動し、次のプロパティを持つ送信セキュリティ規則を追加します。
 * ソース:任意
 * 送信元ポート範囲: *
 * 変換先:IP アドレス
 * 宛先 IP アドレス範囲:ファイル サーバーの IP の範囲
 * 送信先ポート範囲:445
 * プロトコル:TCP
 * アクション:ALLOW
 * 優先順位:700
 * 名前:Outbound_Allow_SMB445

### <a name="known-issues-for-cloud-admins-operating-azure-app-service-on-azure-stack"></a>Azure App Service on Azure Stack を運用するクラウド管理者に関する既知の問題

[Azure Stack 1809 リリース ノート](azure-stack-update-1903.md)内のドキュメントをご覧ください。

## <a name="next-steps"></a>次の手順

- Azure App Service の概要については、「[Azure Stack 上の App Service の概要](azure-stack-app-service-overview.md)」をご覧ください。
- App Service on Azure Stack のデプロイの準備をする方法の詳細については、「[App Service on Azure Stack を開始する前に](azure-stack-app-service-before-you-get-started.md)」をご覧ください。
