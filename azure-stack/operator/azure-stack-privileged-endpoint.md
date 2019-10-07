---
title: Azure Stack での特権エンドポイントの使用 | Microsoft Docs
description: オペレーターが Azure Stack で特権エンドポイント (PEP) を使用する方法を説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/18/2019
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 09/18/2019
ms.openlocfilehash: 3730da9d185f1c38411453a6bef965ab5df7d3ae
ms.sourcegitcommit: 28c8567f85ea3123122f4a27d1c95e3f5cbd2c25
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/02/2019
ms.locfileid: "71829367"
---
# <a name="use-the-privileged-endpoint-in-azure-stack"></a>Azure Stack で特権エンドポイントを使用する

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*

Azure Stack オペレーターは、管理ポータル、PowerShell、または Azure Resource Manager API を使用して、ほとんどの日常的な管理タスクを実行します。 ただし、あまり一般的でない一部の操作については、*特権エンドポイント* (PEP) を使用する必要があります。 この PEP は、あらかじめ構成されたリモート PowerShell コンソールであり、必要なタスクを実行するために十分な機能だけを提供します。 エンドポイントは [PowerShell JEA (Just Enough Administration)](https://docs.microsoft.com/powershell/scripting/learn/remoting/jea/overview) を使用して、コマンドレットの限定的なセットのみを公開します。 PEP にアクセスしてコマンドレットの限定的なセットを起動するために、低権限のアカウントが使用されます。 管理者アカウントは必要ありません。 セキュリティ強化のため、スクリプトは許可されません。

PEP を使用すると、次のタスクを実行できます。

- [診断ログの収集](azure-stack-configure-on-demand-diagnostic-log-collection.md#using-pep-to-collect-diagnostic-logs)などの低レベル タスク。
- デプロイ後のドメイン ネーム システム (DNS) フォワーダーの追加や、Microsoft Graph 統合、Active Directory フェデレーション サービス (AD FS) 統合、証明書ローテーションの設定など、統合システムのための多くのデプロイ後データセンター統合タスク。
- 統合システムの詳細なトラブルシューティングのために、サポート部門と連携して一時的な高レベル アクセスを取得。

PEP では、PowerShell セッションで実行するすべてのアクション (および、それに対応する出力) がログに記録されます。 これにより、完全な透明性と操作の完全な監査が提供されます。 これらのログ ファイルは将来の監査のために保持できます。

> [!NOTE]
> Azure Stack Development Kit (ASDK) では、PEP で利用可能なコマンドの一部を、開発キットのホスト上の PowerShell セッションから直接実行できます。 ただし、これは統合システム環境で特定の操作を実行するために利用可能な唯一の手段であるため、ログ収集など、PEP を使用した一部の操作をテストすることが必要な場合があります。

## <a name="access-the-privileged-endpoint"></a>特権エンドポイントへのアクセス

PEP には、PEP をホストする仮想マシン (VM) 上のリモート PowerShell セッションを介してアクセスします。 ASDK では、この VM の名前は **AzS-ERCS01** です。 統合システムを使用している場合、PEP の 3 つのインスタンスがあり、それぞれ異なるホスト上の VM (*Prefix*-ERCS01、*Prefix*-ERCS02、または *Prefix*-ERCS03) 内で動作することで、回復性を確保しています。

統合システムに対してこの手順を開始する前に、IP アドレスによって、または DNS を介して PEP にアクセスできることを確認してください。 Azure Stack の初期デプロイ後は、DNS 統合がまだセットアップされていないため、IP アドレスでしか PEP にアクセスできません。 PEP の IP アドレス を含む **AzureStackStampDeploymentInfo** という名前の JSON ファイルが、OEM ハードウェア ベンダーから提供されます。


> [!NOTE]
> セキュリティ上の理由から、PEP への接続は、ハードウェア ライフサイクル ホスト上で実行されているセキュリティ強化された VM から、または[特権アクセス ワークステーション](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations)のような専用のセキュリティで保護されたコンピューターからに限定して行う必要があります。 ハードウェア ライフサイクル ホストは、元の構成から変更しないようにし (新しいソフトウェアのインストールするなど)、PEP への接続にも使わないようにする必要があります。

1. 信頼関係を確立します。

    - 統合システムで、管理者特権の Windows PowerShell セッションから次のコマンドを実行して、ハードウェア ライフサイクル ホストまたは特権アクセス ワークステーションで実行されているセキュリティ強化された VM の信頼されたホストとして PEP を追加します。

      ```powershell
        winrm s winrm/config/client '@{TrustedHosts="<IP Address of Privileged Endpoint>"}'
      ```
    - ASDK を実行している場合、開発キットのホストにサインインします。

2. ハードウェア ライフサイクル ホストまたは特権アクセス ワークステーションで実行されているセキュリティ強化された VM で、Windows PowerShell セッションを開きます。 次のコマンドを実行して、PEP をホストする VM 上でリモート セッションを確立します。
 
   - 統合システム上で:
     ```powershell
       $cred = Get-Credential

       Enter-PSSession -ComputerName <IP_address_of_ERCS> `
         -ConfigurationName PrivilegedEndpoint -Credential $cred
     ```
     `ComputerName` パラメーターには、PEP をホストする 1 つの VM の IP アドレスまたは DNS 名を指定できます。

     >[!NOTE]
     >Azure Stack は、PEP 資格情報の検証時にリモート呼び出しを行いません。 ローカルに保存された RSA 公開キーに依存して、それを行います。
     
   - ASDK を実行している場合:
     
     ```powershell
       $cred = Get-Credential

       Enter-PSSession -ComputerName azs-ercs01 `
         -ConfigurationName PrivilegedEndpoint -Credential $cred
     ``` 
     入力を求められたら、次の資格情報を使用します。

     - **[ユーザー名]** : **&lt;*Azure Stack ドメイン*&gt;\cloudadmin** の形式で CloudAdmin アカウントを指定します。 (ASDK の場合、ユーザー名は **azurestack\cloudadmin** です。)
     - **Password**:インストール中に AzureStackAdmin ドメイン管理者アカウントのパスワードとして指定したものと同じパスワードを入力します。

     > [!NOTE]
     > ERCS エンドポイントに接続できない場合は、別の ERCS VM の IP アドレスを使用して、手順 1 と手順 2 を再試行してください。

3. 接続後、環境に応じて **[*IP アドレスまたは ERCS VM 名*]: PS>** または **[azs-ercs01]:PS>** プロンプトが変わります。 ここから `Get-Command` を実行して、利用可能なコマンドレットの一覧を表示します。

   これらのコマンドレットの多くは、統合システム環境での使用のみが意図されています (データセンター統合に関連するコマンドレットなど)。 ASDK では、次のコマンドレットが検証済みです。

   - Clear-Host
   - Close-PrivilegedEndpoint
   - Exit-PSSession
   - Get-AzureStackLog
   - Get-AzureStackStampInformation
   - Get-Command
   - Get-FormatData
   - Get-Help
   - Get-ThirdPartyNotices
   - Measure-Object
   - New-CloudAdminUser
   - Out-Default
   - Remove-CloudAdminUser
   - Select-Object
   - Set-CloudAdminUserPassword
   - Test-AzureStack
   - Stop-AzureStack
   - Get-ClusterLog

## <a name="tips-for-using-the-privileged-endpoint"></a>特権エンドポイントを使用するためのヒント 

前述のとおり、PEP は、[PowerShell JEA](https://docs.microsoft.com/powershell/scripting/learn/remoting/jea/overview) エンドポイントです。 JEA エンドポイントにより、強力なセキュリティ層が提供される一方で、スクリプトやタブ補完などの基本的な PowerShell の機能の一部が失われます。 何らかの種類のスクリプト操作を試みると、エラー **ScriptsNotAllowed** で操作は失敗します。 このエラーは予想される動作です。

たとえば、特定のコマンドレットについてパラメーターの一覧を取得するには、次のコマンドを実行します。

```powershell
    Get-Command <cmdlet_name> -Syntax
```

または、[Import-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Utility/Import-PSSession?view=powershell-5.1) コマンドレット使用して、ローカル コンピューターの現在のセッションにすべての PEP コマンドレットをインポートすることもできます。 これにより、PEP のすべてコマンドレットと関数を、タブ補完や、より一般にはスクリプトと共に、ローカル コンピューターで利用できるようになります。

ローカル コンピューターに PEP セッションをインポートするには、次の手順を実行します。

1. 信頼関係を確立します。

    統合システムで、管理者特権の Windows PowerShell セッションから次のコマンドを実行して、ハードウェア ライフサイクル ホストまたは特権アクセス ワークステーションで実行されているセキュリティ強化された VM の信頼されたホストとして PEP を追加します。

      ```powershell
        winrm s winrm/config/client '@{TrustedHosts="<IP Address of Privileged Endpoint>"}'
      ```
    - ASDK を実行している場合、開発キットのホストにサインインします。

2. ハードウェア ライフサイクル ホストまたは特権アクセス ワークステーションで実行されているセキュリティ強化された VM で、Windows PowerShell セッションを開きます。 次のコマンドを実行して、PEP をホストする仮想マシン上でリモート セッションを確立します。
 
   - 統合システム上で:
     ```powershell
       $cred = Get-Credential

       $session = New-PSSession -ComputerName <IP_address_of_ERCS> `
         -ConfigurationName PrivilegedEndpoint -Credential $cred
     ```
     `ComputerName` パラメーターには、PEP をホストする 1 つの VM の IP アドレスまたは DNS 名を指定できます。
   - ASDK を実行している場合:
     
     ```powershell
      $cred = Get-Credential

      $session = New-PSSession -ComputerName azs-ercs01 `
         -ConfigurationName PrivilegedEndpoint -Credential $cred
     ``` 
     入力を求められたら、次の資格情報を使用します。

     - **[ユーザー名]** : **&lt;*Azure Stack ドメイン*&gt;\cloudadmin** の形式で CloudAdmin アカウントを指定します。 (ASDK の場合、ユーザー名は **azurestack\cloudadmin** です。)
     - **Password**:インストール中に AzureStackAdmin ドメイン管理者アカウントのパスワードとして指定したものと同じパスワードを入力します。

3. ローカル コンピューターに PEP セッションをインポートします
     ```powershell 
        Import-PSSession $session
   ```
4. これで、Azure Stack のセキュリティ対策を損なうことなく、PEP のすべての関数およびコマンドレットと共に、ローカルの PowerShell セッションで通常どおりにタブ補完を使用し、スクリプトを実行できるようになりました。 機能を有効にご活用ください。


## <a name="close-the-privileged-endpoint-session"></a>特権エンドポイント セッションを閉じる

 前述のとおり、PEP では、PowerShell セッションで実行するすべてのアクション (および、それに対応する出力) がログに記録されます。 `Close-PrivilegedEndpoint` コマンドレットを使ってセッションを閉じる必要があります。 このコマンドレットは、エンドポイントを正しく閉じて、ログ ファイルを保管用の外部ファイル共有に転送します。

エンドポイント セッションを閉じるには:

1. PEP からアクセス可能な外部ファイル共有を作成します。 開発キット環境では、開発キットのホスト上にファイル共有を作成することができます。
2. 次のコマンドレットを実行します。
     ```powershell
     Close-PrivilegedEndpoint -TranscriptsPathDestination "\\fileshareIP\SharedFolder" -Credential Get-Credential
     ```
   このコマンドレットでは次の表のパラメーターを使用します。

   | パラメーター | 説明 | データ型 | 必須 |
   |---------|---------|---------|---------|
   | *TranscriptsPathDestination* | "fileshareIP\sharefoldername" として定義されている外部ファイル共有へのパス | string | はい|
   | *資格情報* | ファイル共有にアクセスするための資格情報 | SecureString |   はい |


トランスクリプト ログ ファイルがファイル共有に正常に転送された後、それらのファイルは PEP から自動的に削除されます。 

> [!NOTE]
> コマンドレット `Exit-PSSession` または `Exit` を使用して PEP セッションを閉じた、または単に PowerShell コンソールを閉じた場合、トランスクリプト ログはファイル共有に転送されません。 それらは PEP に残ります。 次に `Close-PrivilegedEndpoint` を実行してファイル共有をインクルードしたときに、以前のセッションのトランスクリプト ログも併せて転送されます。 `Exit-PSSession` または `Exit` を使って PEP セッションを閉じないでください。代わりに、`Close-PrivilegedEndpoint` を使ってください。


## <a name="next-steps"></a>次の手順

[Azure Stack の診断ツール](azure-stack-configure-on-demand-diagnostic-log-collection.md#using-pep-to-collect-diagnostic-logs)
