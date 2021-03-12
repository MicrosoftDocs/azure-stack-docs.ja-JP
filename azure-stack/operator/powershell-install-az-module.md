---
title: Azure Stack Hub 用の PowerShell Az モジュールをインストールする
description: PowerShell for Azure Stack Hub をインストールする方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 02/18/2021
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 02/18/2021
ms.openlocfilehash: 031a1695f8ba11db5a8787ef1b38c40763614b88
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101840866"
---
# <a name="install-powershell-az-module-for-azure-stack-hub"></a>Azure Stack Hub 用の PowerShell Az モジュールをインストールする

この記事では、PowerShellGet を使用して Azure PowerShell Az および互換性のある Azure Stack Hub 管理者モジュールをインストールする方法について説明します。 Az modules は、Windows、macOS、Linux の各プラットフォームにインストールできます。

Docker コンテナー内の Azure Stack Hub に対して Az モジュールを実行することもできます。 手順については、[Docker を使用して Azure Stack Hub に対して PowerShell を実行する](../user/azure-stack-powershell-user-docker.md)方法に関するページを参照してください。

Azure Stack Hub 用の PowerShell Resource Modules (AzureRM) モジュールをインストールする場合は、「[Azure Stack Hub 用の PowerShell AzureRM モジュールをインストールする](azure-stack-powershell-install.md)」を参照してください。

> [!IMPORTANT]
> Azure Resource Modules モジュールには新しいリリースがない可能性があります。 Azure Resource Modules モジュールは、重要な修正についてのみサポートされています。 今後は、Azure Stack Hub 用の Az リリースのみになります。

"*API プロファイル*" を使用して、互換性のある Azure Stack Hub リソース プロバイダーのエンドポイントを指定することができます。

API プロファイルには、Azure と Azure Stack Hub のバージョンの違いを管理するための方法が用意されています。 API バージョンのプロファイルは、特定の API バージョンを持つ一連の Azure Resource Manager PowerShell モジュールです。 各クラウド プラットフォームでは、一連の API バージョンのプロファイルがサポートされています。 たとえば、Azure Stack Hub では、**2019-03-01-hybrid** などの特定のプロファイル バージョンがサポートされます。 プロファイルをインストールすると、指定されたプロファイルに対応する Azure Resource Manager PowerShell モジュールがインストールされます。

Azure Stack Hub と互換性のある PowerShell Az モジュールは、インターネットに接続されている、部分的に接続されている、または接続が切断されているシナリオでインストールできます。 この記事では、これらのシナリオの詳細な手順について説明します。

## <a name="1-verify-your-prerequisites"></a>1.前提条件を確認する

Az モジュールは、更新プログラム 2002 以降および現在の修正プログラムがインストールされている Azure Stack Hub でサポートされています。 詳細については、「[Azure Stack Hub のリリース ノート](release-notes.md)」をご覧ください。

Azure PowerShell の Az モジュールは、Windows 上の PowerShell 5.1 以降、またはすべてのプラットフォーム上の PowerShell Core 6.x 以降で動作します。 お使いのオペレーティング システムで利用できる[最新バージョンの PowerShell Core](/powershell/scripting/install/installing-powershell#powershell-core) をインストールする必要があります。 PowerShell Core 上で実行する場合、Azure PowerShell にその他の要件はありません。

PowerShell のバージョンを確認するには、次のコマンドを実行します。

```powershell  
$PSVersionTable.PSVersion
```

### <a name="prerequisites-for-windows"></a>Windows の前提条件
Windows 上の PowerShell 5.1 で Azure PowerShell を使用するには、次の手順に従います。

1. 必要に応じて [Windows PowerShell 5.1](/powershell/scripting/windows-powershell/install/installing-windows-powershell#upgrading-existing-windows-powershell) に更新します。 Windows 10 の場合は、あらかじめ PowerShell 5.1 がインストールされています。
2. [.NET Framework 4.7.2 以降](/dotnet/framework/install)をインストールします。
3. 最新バージョンの PowerShellGet がインストールされていることを確認します。 `Install-Module PowerShellGet -MinimumVersion 2.2.3 -Force` を実行します。 

## <a name="2-prerequisites-for-linux-and-mac"></a>2.Linux および Mac の前提条件
PowerShell Core 6.x 以降のバージョンが必要です。 手順については、[リンク](/powershell/scripting/install/installing-powershell-core-on-windows)に従ってください。

## <a name="3-uninstall-existing-versions-of-the-azure-stack-hub-powershell-modules"></a>3.既存のバージョンの Azure Stack Hub PowerShell モジュールをアンインストールする

必要なバージョンをインストールする前に、必ず以前にインストールした Azure Stack Hub Azure Resource Manager または Az PowerShell モジュールをすべてアンインストールしてください。 モジュールをアンインストールするには、次の 2 つの方法のいずれかを使用します。

1. 既存の Azure Resource Manager モジュールと Az PowerShell モジュールをアンインストールするには、アクティブな PowerShell セッションをすべて閉じ、次のコマンドレットを実行します。

    ```powershell
    Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    Get-Module -Name Azs.* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    Get-Module -Name Az.* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    ```
    「モジュールが既に使用されています」などのエラーが発生した場合には、そのモジュールを使用している PowerShell セッションを終了してから、上のスクリプトをもう一度実行してください。

2. `C:\Program Files\WindowsPowerShell\Modules` と `C:\Users\{yourusername}\Documents\WindowsPowerShell\Modules` の各フォルダーから、`Azure`、`Az`、または `Azs.` で始まるすべてのフォルダーを削除します。 これらのフォルダーを削除すると、既存の PowerShell モジュールがすべて削除されます。

## <a name="4-connected-install-with-internet-connectivity"></a>4.接続済みの場合: インターネット接続を使用したインストール

Azure Stack Az モジュールは、Azure Stack Hub 2002 以降で動作します。 さらに、Azure Stack Az モジュールは、Windows マシン上では PowerShell 5.1 以上、Linux または macOS プラットフォームでは PowerShell 6.x 以上で動作します。 推奨されるインストール方法は、PowerShellGet コマンドレットを使用することです。 この方法は、サポートされているプラットフォームでも同様に機能します。

1. PowerShell セッションから次のコマンドを実行して、PowerShellGet をバージョン 2.2.3 以上に更新します

    ```powershell  
    Install-Module PowerShellGet -MinimumVersion 2.2.3 -Force
    ```

2. PowerShell セッションを閉じてから、更新を有効プログラムをできるように新しい PowerShell セッションを開きます。

3. PowerShell セッションから次のコマンドを実行します。

    ```powershell  
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

> [!Note]  
> Azure Stack Hub モジュール バージョン 2.0.0 は破壊的変更を伴うリリースです。 詳細については、「[Azure Stack Hub での AzureRM から Azure PowerShell Az への移行](migrate-azurerm-az.md)」を参照してください。

> [!WARNING]
> Windows 用の PowerShell 5.1 で Azure Resource Manager (AzureRM) と Az の両方のモジュールを同時にインストールすることはできません。 Azure Resource Manager をシステムで引き続き使用できるようにしておく必要がある場合は、PowerShell Core 6.x 以降用の Az モジュールをインストールします。 そのためには、[PowerShell Core 6.x 以降をインストール](/powershell/scripting/install/installing-powershell-core-on-windows)してから、PowerShell Core ターミナルで以下の手順に従ってください。

## <a name="5-disconnected-install-without-internet-connection"></a>5.切断状態の場合: インターネット接続を使用しないインストール

接続が切断されたシナリオでは、まずインターネット接続が確立されたマシンに PowerShell モジュールをダウンロードします。 次に、それらを Azure Stack Development Kit (ASDK) に転送してインストールします。

インターネット接続が確立されたコンピューターにサインインし、次のスクリプトを使用して、Azure Stack Hub のバージョンに応じた Azure Resource Manager および Azure Stack Hub パッケージをダウンロードします。

インストールには 5 つの手順があります。

1. 接続されているマシンに Azure Stack Hub PowerShell をインストールします。
2. 追加のストレージ機能を有効にします。
3. 接続が切断されたワークステーションに PowerShell パッケージを転送します。
4. 切断されたワークステーション上で NuGet プロバイダーを手動でブートストラップします。
5. PowerShell のインストールを確認します。

### <a name="install-azure-stack-hub-powershell"></a>Azure Stack Hub PowerShell のインストール

::: moniker range=">=azs-2002"
Azure Stack Hub 2002 以降。

Azure Resource Manager または Az モジュールのいずれかを使用できます。 Azure Resource Manager については、[PowerShell AzureRM モジュールのインストール](powershell-install-az-module.md)に関する記事の手順を参照してください。 次のコードでは、信頼できるオンライン リポジトリ https://www.powershellgallery.com/ からのモジュールが保存されます。

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

Install-module -Name PowerShellGet -MinimumVersion 2.2.3 -Force
Import-Module -Name PackageManagement -ErrorAction Stop

$savedModulesPath = "<Path that is used to save the packages>"
Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name Az -Path $savedModulesPath -Force -RequiredVersion 0.10.0-preview
Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureStack -Path $savedModulesPath -Force -RequiredVersion 2.0.2-preview
```
::: moniker-end

> [!NOTE]  
> インターネット接続のないマシンでは、次のコマンドレットを実行して利用統計情報の収集を無効にすることをお勧めします。 利用統計情報の収集を無効にしないと、コマンドレットのパフォーマンスが低下することがあります。 これは、インターネット接続のないマシンのみ該当します。
> ```powershell
> Disable-AzDataCollection
> ```

### <a name="add-your-packages-to-your-workstation"></a>パッケージをワークステーションに追加する

1. ダウンロードしたパッケージを USB デバイスにコピーします。

2. 接続が切断されたワークステーションにサインインし、パッケージを USB デバイスからワークステーション上の場所にコピーします。

3. 切断されたワークステーション上で NuGet プロバイダーを手動でブートストラップします。 手順については、「[インターネットに接続されていないマシンで NuGet プロバイダーを手動でブートストラップする](/powershell/scripting/gallery/how-to/getting-support/bootstrapping-nuget#manually-bootstrapping-the-nuget-provider-on-a-machine-that-is-not-connected-to-the-internet)」をご覧ください。

4. この場所を既定のレポジトリとして登録し、このレポジトリから `AzureRM` モジュールと `AzureStack` モジュールをインストールします。

   ```powershell
   # requires -Version 5
   # requires -RunAsAdministrator
   # requires -Module PowerShellGet
   # requires -Module PackageManagement

   $SourceLocation = "<Location on the development kit that contains the PowerShell packages>"
   $RepoName = "MyNuGetSource"

   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

   Register-PSRepository -Name $RepoName -SourceLocation $SourceLocation -InstallationPolicy Trusted

   Install-Module -Name AzureStack -Repository $RepoName -RequiredVersion 2.0.2-preview -AllowPrerelease -Scope AllUsers

   Install-Module -Name Az -Repository $RepoName -RequiredVersion 0.10.0-preview -AllowPrerelease -Scope AllUsers
   ```

### <a name="confirm-the-installation-of-powershell"></a>PowerShell のインストールを確認する

次のコマンドを実行してインストールを確認します。

```powershell
Get-Module -Name "Az*" -ListAvailable
Get-Module -Name "Azs*" -ListAvailable
```

## <a name="6-configure-powershell-to-use-a-proxy-server"></a>6.プロキシ サーバーを使用するように PowerShell を構成する

インターネットへのアクセスにプロキシ サーバーを必要とするシナリオでは、まず既存のプロキシ サーバーを使用するように PowerShell を構成します。

1. 管理者特権の PowerShell プロンプトを開きます。
2. 次のコマンドを実行します。

   ```powershell
   #To use Windows credentials for proxy authentication
   [System.Net.WebRequest]::DefaultWebProxy.Credentials = [System.Net.CredentialCache]::DefaultCredentials

   #Alternatively, to prompt for separate credentials that can be used for #proxy authentication
   [System.Net.WebRequest]::DefaultWebProxy.Credentials = Get-Credential
   ```

## <a name="7-use-the-az-module"></a>7.Az モジュールを使用する

Azure Resource Manager に基づくコマンドレットとコード サンプルを使用できます。 ただし、モジュールとコマンドレットの名前を変更する必要があります。 モジュール名は、`AzureRM` および Azure が `Az` になるように変更されており、コマンドレットについても同様です。 たとえば、`AzureRM.Compute` モジュールは名前が `Az.Compute` に変更されています。` New-AzureRMVM` は ` New-AzVM` になり、`Get-AzureStorageBlob` は `Get-AzStorageBlob` になっています。

Az への AzurRM スクリプトの移動、および Azure Stack Hub の AZ モジュールにおける破壊的変更に関する詳しい説明とガイダンスについては、[AzureRM から Azure PowerShell Az への移行](migrate-azurerm-az.md)に関する記事を参照してください。

## <a name="known-issues"></a>既知の問題

[!Include[Known issue for install - one](../includes/known-issue-az-install-1.md)]

[!Include[Known issue for install - two](../includes/known-issue-az-install-2.md)]

## <a name="next-steps"></a>次のステップ

- [GitHub からの Azure Stack Hub ツールのダウンロード](azure-stack-powershell-download.md)
- [Azure Stack Hub ユーザーの PowerShell 環境の構成](../user/azure-stack-powershell-configure-user.md)
- [Azure Stack Hub オペレーターの PowerShell 環境の構成](azure-stack-powershell-configure-admin.md)
- [Azure Stack Hub での API バージョンのプロファイルの管理](../user/azure-stack-version-profiles.md)
