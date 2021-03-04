---
title: アプリ ID を使用してリソースにアクセスする
description: アプリ ID を使用して Azure Stack Hub リソースにアクセスする方法について説明します。アプリ ID は、リソースへのサインインおよびアクセスのためにロールベースのアクセス制御で使用できます。
author: BryanLa
ms.author: bryanla
ms.topic: how-to
ms.date: 11/16/2020
ms.lastreviewed: 11/16/2020
ms.custom: contperf-fy20q4
zone_pivot_groups: state-connected-disconnected
ms.openlocfilehash: 57ab28cbc548c8b5b8329fb7c9c8c73aee3b7d93
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101840730"
---
# <a name="use-an-app-identity-to-access-azure-stack-hub-resources"></a>アプリ ID を使用して Azure Stack Hub リソースにアクセスする

アプリケーションのリソースのデプロイや構成を Azure Resource Manager を通じて行う必要がある場合は、そのアプリケーションをその ID で表す必要があります。 ユーザー プリンシパルと呼ばれているセキュリティ プリンシパルでユーザーが表されるように、アプリはサービス プリンシパルで表されます。 サービス プリンシパルは、開発者が開発するアプリの ID となり、開発者は必要なアクセス許可のみをそのアプリに委任することができます。  

たとえば、Azure Resource Manager を使用して Azure リソースのインベントリを作成する構成管理アプリがあります。 このシナリオでは、サービス プリンシパルを作成し、これに "閲覧者" ロールを付与して、構成管理アプリのアクセスを読み取り専用に制限します。

## <a name="overview"></a>概要

ユーザーと同様に、アプリでは認証時に資格情報を提示する必要があります。 この認証は、次の 2 つの要素で構成されます。

- **アプリケーション ID**。クライアント ID と呼ばれることもあります。 Active Directory テナント内でのそのアプリの登録を一意に識別する GUID です。
- アプリケーション ID に関連付けられている **シークレット**。 クライアント シークレット文字列 (パスワードに似ています) を生成することも、X509 証明書 (証明書の公開キーを使用します) を指定することもできます。

独自の ID の下でアプリを実行することは、次の理由から、ユーザーの ID の下で実行することより好ましくなります。

 - **資格情報の強度が上がる** - アプリでは、文字の共有シークレットまたはパスワードの代わりに X509 証明書を利用してサインインできます。  
 - **アクセス許可の制限が増える** - 制限の厳しいアクセス許可をアプリに割り当てることができます。 一般的に、このアクセス許可はそのアプリでの実行に必要なものだけに限定され、このことを *最小限の特権の原則* といいます。
 - アプリの場合、ユーザーの資格情報ほど **頻繁には資格情報とアクセス許可は変更されません**。 たとえば、ユーザーの責務が変わるときや、パスワードの要件で変更が要求されているときや、ユーザーが退職したときです。

開発者は初めに、使用するディレクトリ内に新しいアプリ登録を作成します。これによって、対応する[サービス プリンシパル オブジェクト](/azure/active-directory/develop/developer-glossary#service-principal-object)が作成され、これがそのディレクトリの中でアプリの ID を表します。 

この記事ではまず、Azure Stack Hub インスタンス用に選択したディレクトリに応じて、サービス プリンシパルを作成および管理するプロセスについて説明します。

- **Azure Active Directory (Azure AD)** 。 Azure AD は、マルチテナントに対応したクラウドベースのディレクトリおよび ID の管理サービスです。 Azure AD は、接続された Azure Stack Hub インスタンスで使用できます。
- **Active Directory フェデレーション サービス (AD FS)** 。 AD FS は、シンプルかつ安全な ID フェデレーション機能と Web シングル サインオン (SSO) 機能を実現します。 AD FS は、接続された Azure Stack Hub インスタンスでも、切断された Azure Stack Hub インスタンスでも使用できます。

その後、サービス プリンシパルをロールに割り当ててリソースへのアクセスを制限する方法を説明します。

::: zone pivot="state-disconnected"
<!-- this is intentionally a noop -->
::: zone-end

::: zone pivot="state-connected"
## <a name="manage-an-azure-ad-app-identity"></a>Azure AD アプリ ID の管理

Azure Stack Hub をデプロイするときに Azure AD を ID 管理サービスとして選択した場合は、サービス プリンシパルを Azure の場合と同様に作成します。 このセクションでは、このことを Azure portal で行う手順を説明します。 始める前に、[Azure AD で必要なアクセス許可](/azure/active-directory/develop/howto-create-service-principal-portal#required-permissions)があることを確認してください。

### <a name="create-a-service-principal-that-uses-a-client-secret-credential"></a>クライアント シークレット資格情報を使用するサービス プリンシパルを作成する

このセクションでは、Azure portal を使用してアプリを登録します。これによってサービス プリンシパル オブジェクトが Azure AD テナント内に作成されます。 この例では、クライアント シークレット資格情報を指定しますが、ポータルでは X509 証明書ベースの資格情報もサポートされます。

1. Azure アカウントを使用して [Azure Portal](https://portal.azure.com) にサインインします。
2. **[Azure Active Directory]**  >  **[アプリの登録]**  >  **[新規登録]** の順に選択します。
3. アプリの **名前** を指定します。
4. **[サポートされているアカウントの種類]** で適切なものを選択します。
5. **[リダイレクト URI]** で、アプリの種類として **[Web]** を選択し、このアプリにリダイレクト URI が必要な場合はここで指定します。
6. 値を設定したら、 **[登録]** を選択します。 アプリ登録が作成されて **[概要]** ページが表示されます。
7. **アプリ ID** をコピーします。この ID をアプリ コードで使用するためです。 この値は、クライアント ID と呼ばれることもあります。
8. クライアント シークレットを生成するために、 **[証明書とシークレット]** ページを選択します。 **[新しいクライアント シークレット]** を選択します。
9. シークレットの **説明** と **有効期限** を指定します。
10. 完了したら、 **[追加]** をクリックします。
11. シークレットの値が表示されます。 この値をコピーして別の場所に保存します。後でこの値を取得することはできないからです。 このシークレットは、サインインするときに、クライアント アプリ内でアプリ ID と共に提示します。

    ![クライアント シークレットに保存されたキー](./media/azure-stack-create-service-principal/create-service-principal-in-azure-stack-secret.png)

次に、「[ロールの割り当て](#assign-a-role)」に進み、アプリの ID のロールベースのアクセス制御を確立する方法について説明します。
::: zone-end

## <a name="manage-an-ad-fs-app-identity"></a>AD FS アプリ ID を管理する

Azure Stack Hub をデプロイするときに AD FS を ID 管理サービスとして選択した場合は、アプリ ID の管理には PowerShell を使用する必要があります。 次に示すサービス プリンシパル資格情報の管理の例では、X509 証明書とクライアント シークレットの両方について説明します。

スクリプトは、管理者特権のある ("管理者として実行") PowerShell コンソールで実行する必要があります。これにより、Azure Stack Hub インスタンスの特権エンドポイントをホストする VM への別のセッションが開きます。 特権エンドポイント セッションが確立した後に、追加のコマンドレットを実行してサービス プリンシパルの管理を行います。 特権エンドポイントの詳細については、[Azure Stack Hub での特権エンドポイントの使用](azure-stack-privileged-endpoint.md)に関する記事を参照してください。

### <a name="create-a-service-principal-that-uses-a-certificate-credential"></a>証明書資格情報を使用するサービス プリンシパルを作成する

証明書資格情報を作成するときは、次の要件を満たす必要があります。

 - 運用環境では、証明書は、内部の証明機関または公的証明機関のどちらかから発行されている必要があります。 公的機関を利用している場合は、Microsoft の信頼されたルート機関プログラムの一部としてその機関を基本オペレーティング システム イメージに含める必要があります。 完全な一覧は、「[参加者の一覧 - Microsoft の信頼されたルート プログラム](/security/trusted-root/participants-list)」を参照してください。 "自己署名" テスト証明書の作成の例については、後で「[証明書資格情報を更新する](#update-a-certificate-credential)」でも示しています。 
 - 暗号化プロバイダーを、Microsoft レガシ暗号化サービス プロバイダー (CSP) のキー プロバイダーとして指定する必要があります。
 - 公開キーと秘密キーの両方が必要なため、証明書は PFX ファイル形式である必要があります。 Windows サーバーでは、公開キー ファイル (TLS または SSL 証明書ファイル) と関連付けられている秘密キー ファイルが含まれている .pfx ファイルが使用されます。
 - お使いの Azure Stack Hub インフラストラクチャは、証明書において公開されている証明機関の証明書失効リスト (CRL) の場所にネットワークでアクセスできる必要があります。 この CRL は、HTTP エンドポイントである必要があります。

証明書を用意できたら、下記の PowerShell スクリプトを使用してアプリを登録し、サービス プリンシパルを作成します。 また、このサービス プリンシパルを使用して Azure にサインインします。 次のプレースホルダーを実際の値で置き換えてください。

| プレースホルダー | 説明 | 例 |
| ----------- | ----------- | ------- |
| \<PepVM\> | Azure Stack Hub インスタンス上にある特権エンドポイント VM の名前。 | "AzS-ERCS01" |
| \<YourCertificateLocation\> | ローカル証明書ストア内の X509 証明書の場所。 | "Cert:\CurrentUser\My\AB5A8A3533CC7AA2025BF05120117E06DE407B34" |
| \<YourAppName\> | 新しいアプリ登録のわかりやすい名前。 | "My management tool" |

### <a name="az-modules"></a>[Az モジュール](#tab/az1)

1. Windows PowerShell セッションを管理者特権で開き、次のスクリプトを実行します。

    ```powershell  
    # Sign in to PowerShell interactively, using credentials that have access to the VM running the Privileged Endpoint (typically <domain>\cloudadmin)
    $Creds = Get-Credential
    
    # Create a PSSession to the Privileged Endpoint VM
    $Session = New-PSSession -ComputerName "<PepVm>" -ConfigurationName PrivilegedEndpoint -Credential $Creds
    
    # Use the Get-Item cmdlet to retrieve your certificate.
    # If you don't want to use a managed certificate, you can produce a self signed cert for testing purposes: 
    # $Cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=<YourAppName>" -KeySpec KeyExchange
    $Cert = Get-Item "<YourCertificateLocation>"
    
    # Use the privileged endpoint to create the new app registration (and service principal object)
    $SpObject = Invoke-Command -Session $Session -ScriptBlock {New-GraphApplication -Name "<YourAppName>" -ClientCertificates $using:cert}
    $AzureStackInfo = Invoke-Command -Session $Session -ScriptBlock {Get-AzureStackStampInformation}
    $Session | Remove-PSSession
    
    # Using the stamp info for your Azure Stack Hub instance, populate the following variables:
    # - Az endpoint used for Azure Resource Manager operations 
    # - Audience for acquiring an OAuth token used to access Graph API 
    # - GUID of the directory tenant
    $ArmEndpoint = $AzureStackInfo.TenantExternalEndpoints.TenantResourceManager
    $GraphAudience = "https://graph." + $AzureStackInfo.ExternalDomainFQDN + "/"
    $TenantID = $AzureStackInfo.AADTenantID
    
    # Register and set an Az environment that targets your Azure Stack Hub instance
    Add-AzEnvironment -Name "AzureStackUser" -ArmEndpoint $ArmEndpoint
    
    # Sign in using the new service principal
    $SpSignin = Connect-AzAccount -Environment "AzureStackUser" `
    -ServicePrincipal `
    -CertificateThumbprint $SpObject.Thumbprint `
    -ApplicationId $SpObject.ClientId `
    -TenantId $TenantID
    
    # Output the service principal details
    $SpObject
    
    ```

2. このスクリプトが完了すると、アプリ登録の情報が表示され、この中にサービス プリンシパルの資格情報も含まれます。 `ClientID` と `Thumbprint` が認証され、その後、Azure Resource Manager で管理されるリソースへのアクセスが承認されます。

   ```shell
   ApplicationIdentifier : S-1-5-21-1512385356-3796245103-1243299919-1356
   ClientId              : 3c87e710-9f91-420b-b009-31fa9e430145
   Thumbprint            : 30202C11BE6864437B64CE36C8D988442082A0F1
   ApplicationName       : Azurestack-MyApp-c30febe7-1311-4fd8-9077-3d869db28342
   ClientSecret          :
   PSComputerName        : azs-ercs01
   RunspaceId            : a78c76bb-8cae-4db4-a45a-c1420613e01b
   ```

PowerShell コンソール セッションは開いたままにします。`ApplicationIdentifier` の値とともに次のセクションで使用するからです。

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm1)

1. Windows PowerShell セッションを管理者特権で開き、次のスクリプトを実行します。

    ```powershell  
    # Sign in to PowerShell interactively, using credentials that have access to the VM running the Privileged Endpoint (typically <domain>\cloudadmin)
    $Creds = Get-Credential
    
    # Create a PSSession to the Privileged Endpoint VM
    $Session = New-PSSession -ComputerName "<PepVm>" -ConfigurationName PrivilegedEndpoint -Credential $Creds
    
    # Use the Get-Item cmdlet to retrieve your certificate.
    # If you don't want to use a managed certificate, you can produce a self signed cert for testing purposes: 
    # $Cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=<YourAppName>" -KeySpec KeyExchange
    $Cert = Get-Item "<YourCertificateLocation>"
    
    # Use the privileged endpoint to create the new app registration (and service principal object)
    $SpObject = Invoke-Command -Session $Session -ScriptBlock {New-GraphApplication -Name "<YourAppName>" -ClientCertificates $using:cert}
    $AzureStackInfo = Invoke-Command -Session $Session -ScriptBlock {Get-AzureStackStampInformation}
    $Session | Remove-PSSession
    
    # Using the stamp info for your Azure Stack Hub instance, populate the following variables:
    # - AzureRM endpoint used for Azure Resource Manager operations 
    # - Audience for acquiring an OAuth token used to access Graph API 
    # - GUID of the directory tenant
    $ArmEndpoint = $AzureStackInfo.TenantExternalEndpoints.TenantResourceManager
    $GraphAudience = "https://graph." + $AzureStackInfo.ExternalDomainFQDN + "/"
    $TenantID = $AzureStackInfo.AADTenantID
    
    # Register and set an AzureRM environment that targets your Azure Stack Hub instance
    Add-AzureRMEnvironment -Name "AzureStackUser" -ArmEndpoint $ArmEndpoint
    
    # Sign in using the new service principal
    $SpSignin = Connect-AzureRMAccount -Environment "AzureStackUser" `
    -ServicePrincipal `
    -CertificateThumbprint $SpObject.Thumbprint `
    -ApplicationId $SpObject.ClientId `
    -TenantId $TenantID
    
    # Output the service principal details
    $SpObject
    ```

2. このスクリプトが完了すると、アプリ登録の情報が表示され、この中にサービス プリンシパルの資格情報も含まれます。 `ClientID` と `Thumbprint` が認証され、その後、Azure Resource Manager で管理されるリソースへのアクセスが承認されます。

   ```shell
   ApplicationIdentifier : S-1-5-21-1512385356-3796245103-1243299919-1356
   ClientId              : 3c87e710-9f91-420b-b009-31fa9e430145
   Thumbprint            : 30202C11BE6864437B64CE36C8D988442082A0F1
   ApplicationName       : Azurestack-MyApp-c30febe7-1311-4fd8-9077-3d869db28342
   ClientSecret          :
   PSComputerName        : azs-ercs01
   RunspaceId            : a78c76bb-8cae-4db4-a45a-c1420613e01b
   ```

PowerShell コンソール セッションは開いたままにします。`ApplicationIdentifier` の値とともに次のセクションで使用するからです。

---

### <a name="update-a-certificate-credential"></a>証明書資格情報を更新する

サービス プリンシパルが作成されたので、このセクションでは次のことを行う方法を説明します。

1. 新しい自己署名 X509 証明書をテスト用に生成します。
2. サービス プリンシパルの資格情報を更新します。具体的には、**Thumbprint** プロパティを新しい証明書に合わせて更新します。

PowerShell を使用して証明書資格情報を更新します。次のプレースホルダーを実際の値で置き換えてください。

| プレースホルダー | 説明 | 例 |
| ----------- | ----------- | ------- |
| \<PepVM\> | Azure Stack Hub インスタンス上にある特権エンドポイント VM の名前。 | "AzS-ERCS01" |
| \<YourAppName\> | 新しいアプリ登録のわかりやすい名前。 | "My management tool" |
| \<YourCertificateLocation\> | ローカル証明書ストア内の X509 証明書の場所。 | "Cert:\CurrentUser\My\AB5A8A3533CC7AA2025BF05120117E06DE407B34" |
| \<AppIdentifier\> | アプリケーション登録に割り当てられた識別子。 | "S-1-5-21-1512385356-3796245103-1243299919-1356" |

1. 管理者特権のある Windows PowerShell セッションを使用して、次のコマンドレットを実行します。

     ```powershell
     # Create a PSSession to the PrivilegedEndpoint VM
     $Session = New-PSSession -ComputerName "<PepVM>" -ConfigurationName PrivilegedEndpoint -Credential $Creds

     # Create a self-signed certificate for testing purposes. 
     $NewCert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=<YourAppName>" -KeySpec KeyExchange
     # In production, use Get-Item and a managed certificate instead.
     # $Cert = Get-Item "<YourCertificateLocation>"

     # Use the privileged endpoint to update the certificate thumbprint, used by the service principal associated with <AppIdentifier>
     $SpObject = Invoke-Command -Session $Session -ScriptBlock {Set-GraphApplication -ApplicationIdentifier "<AppIdentifier>" -ClientCertificates $using:NewCert}
     $Session | Remove-PSSession

     # Output the updated service principal details
     $SpObject
     ```

2. このスクリプトが完了すると、更新後のアプリ登録の情報が表示され、この中に新しい自己署名証明書の拇印の値も含まれます。

     ```Shell  
     ApplicationIdentifier : S-1-5-21-1512385356-3796245103-1243299919-1356
     ClientId              : 
     Thumbprint            : AF22EE716909041055A01FE6C6F5C5CDE78948E9
     ApplicationName       : Azurestack-MyApp-c30febe7-1311-4fd8-9077-3d869db28342
     ClientSecret          : 
     PSComputerName        : azs-ercs01
     RunspaceId            : a580f894-8f9b-40ee-aa10-77d4d142b4e5
     ```

### <a name="create-a-service-principal-that-uses-client-secret-credentials"></a>クライアント シークレット資格情報を使用するサービス プリンシパルを作成する

> [!WARNING]
> クライアント シークレットを使用する場合は、X509 証明書資格情報を使用する場合に比べて安全性が低くなります。 認証メカニズムの安全性が低いだけでなく、一般的にはシークレットをクライアント アプリのソース コードに埋め込むことが必要になります。 そのため、運用環境のアプリでは証明書資格情報を使用することを強くお勧めします。

別のアプリ登録を作成しますが、今回はクライアント シークレット資格情報を指定します。 証明書資格情報とは異なり、クライアント シークレット資格情報はディレクトリが生成することができます。 開発者はクライアント シークレットを指定する代わりに、`-GenerateClientSecret` スイッチを使用してシークレットの生成を要求できます。 次のプレースホルダーを実際の値で置き換えてください。

| プレースホルダー | 説明 | 例 |
| ----------- | ----------- | ------- |
| \<PepVM\> | Azure Stack Hub インスタンス上にある特権エンドポイント VM の名前。 | "AzS-ERCS01" |
| \<YourAppName\> | 新しいアプリ登録のわかりやすい名前。 | "My management tool" |

### <a name="az-modules"></a>[Az モジュール](#tab/az2)

1. Windows PowerShell セッションを管理者特権で開き、次のコマンドレットを実行します。

    ```powershell  
    # Sign in to PowerShell interactively, using credentials that have access to the VM running the Privileged Endpoint (typically <domain>\cloudadmin)
    $Creds = Get-Credential
    
    # Create a PSSession to the Privileged Endpoint VM
    $Session = New-PSSession -ComputerName "<PepVM>" -ConfigurationName PrivilegedEndpoint -Credential $Creds
    
    # Use the privileged endpoint to create the new app registration (and service principal object)
    $SpObject = Invoke-Command -Session $Session -ScriptBlock {New-GraphApplication -Name "<YourAppName>" -GenerateClientSecret}
    $AzureStackInfo = Invoke-Command -Session $Session -ScriptBlock {Get-AzureStackStampInformation}
    $Session | Remove-PSSession
    
    # Using the stamp info for your Azure Stack Hub instance, populate the following variables:
    # - Az endpoint used for Azure Resource Manager operations 
    # - Audience for acquiring an OAuth token used to access Graph API 
    # - GUID of the directory tenant
    $ArmEndpoint = $AzureStackInfo.TenantExternalEndpoints.TenantResourceManager
    $GraphAudience = "https://graph." + $AzureStackInfo.ExternalDomainFQDN + "/"
    $TenantID = $AzureStackInfo.AADTenantID
    
    # Register and set an Az environment that targets your Azure Stack Hub instance
    Add-AzEnvironment -Name "AzureStackUser" -ArmEndpoint $ArmEndpoint
    
    # Sign in using the new service principal
    $securePassword = $SpObject.ClientSecret | ConvertTo-SecureString -AsPlainText -Force
    $credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $SpObject.ClientId, $securePassword
    $SpSignin = Connect-AzAccount -Environment "AzureStackUser" -ServicePrincipal -Credential $credential -TenantId $TenantID
    
    # Output the service principal details
    $SpObject
    ```

2. このスクリプトが完了すると、アプリ登録の情報が表示され、この中にサービス プリンシパルの資格情報も含まれます。 `ClientID` と `ClientSecret` が認証され、その後、Azure Resource Manager で管理されるリソースへのアクセスが承認されます。

     ```shell  
     ApplicationIdentifier : S-1-5-21-1634563105-1224503876-2692824315-2623
     ClientId              : 8e0ffd12-26c8-4178-a74b-f26bd28db601
     Thumbprint            : 
     ApplicationName       : Azurestack-YourApp-6967581b-497e-4f5a-87b5-0c8d01a9f146
     ClientSecret          : 6RUWLRoBw3EebBLgaWGiowCkoko5_j_ujIPjA8dS
     PSComputerName        : azs-ercs01
     RunspaceId            : 286daaa1-c9a6-4176-a1a8-03f543f90998
     ```

PowerShell コンソール セッションは開いたままにします。`ApplicationIdentifier` の値とともに次のセクションで使用するからです。
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm2)

1. Windows PowerShell セッションを管理者特権で開き、次のコマンドレットを実行します。

    ```powershell  
    # Sign in to PowerShell interactively, using credentials that have access to the VM running the Privileged Endpoint (typically <domain>\cloudadmin)
    $Creds = Get-Credential
    
    # Create a PSSession to the Privileged Endpoint VM
    $Session = New-PSSession -ComputerName "<PepVM>" -ConfigurationName PrivilegedEndpoint -Credential $Creds
    
    # Use the privileged endpoint to create the new app registration (and service principal object)
    $SpObject = Invoke-Command -Session $Session -ScriptBlock {New-GraphApplication -Name "<YourAppName>" -GenerateClientSecret}
    $AzureStackInfo = Invoke-Command -Session $Session -ScriptBlock {Get-AzureStackStampInformation}
    $Session | Remove-PSSession
    
    # Using the stamp info for your Azure Stack Hub instance, populate the following variables:
    # - AzureRM endpoint used for Azure Resource Manager operations 
    # - Audience for acquiring an OAuth token used to access Graph API 
    # - GUID of the directory tenant
    $ArmEndpoint = $AzureStackInfo.TenantExternalEndpoints.TenantResourceManager
    $GraphAudience = "https://graph." + $AzureStackInfo.ExternalDomainFQDN + "/"
    $TenantID = $AzureStackInfo.AADTenantID
    
    # Register and set an AzureRM environment that targets your Azure Stack Hub instance
    Add-AzureRMEnvironment -Name "AzureStackUser" -ArmEndpoint $ArmEndpoint
    
    # Sign in using the new service principal
    $securePassword = $SpObject.ClientSecret | ConvertTo-SecureString -AsPlainText -Force
    $credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $SpObject.ClientId, $securePassword
    $SpSignin = Connect-AzureRMAccount -Environment "AzureStackUser" -ServicePrincipal -Credential $credential -TenantId $TenantID
    
    # Output the service principal details
    $SpObject
    ```

2. このスクリプトが完了すると、アプリ登録の情報が表示され、この中にサービス プリンシパルの資格情報も含まれます。 `ClientID` と `ClientSecret` が認証され、その後、Azure Resource Manager で管理されるリソースへのアクセスが承認されます。

     ```shell  
     ApplicationIdentifier : S-1-5-21-1634563105-1224503876-2692824315-2623
     ClientId              : 8e0ffd12-26c8-4178-a74b-f26bd28db601
     Thumbprint            : 
     ApplicationName       : Azurestack-YourApp-6967581b-497e-4f5a-87b5-0c8d01a9f146
     ClientSecret          : 6RUWLRoBw3EebBLgaWGiowCkoko5_j_ujIPjA8dS
     PSComputerName        : azs-ercs01
     RunspaceId            : 286daaa1-c9a6-4176-a1a8-03f543f90998
     ```

PowerShell コンソール セッションは開いたままにします。`ApplicationIdentifier` の値とともに次のセクションで使用するからです。

---

### <a name="update-a-client-secret"></a>クライアント シークレットを更新する

PowerShell を使用してクライアント シークレット資格情報を更新します。**ResetClientSecret** パラメーターを使用するので、クライアント シークレットが即座に変更されます。 次のプレースホルダーを実際の値で置き換えてください。

| プレースホルダー | 説明 | 例 |
| ----------- | ----------- | ------- |
| \<PepVM\> | Azure Stack Hub インスタンス上にある特権エンドポイント VM の名前。 | "AzS-ERCS01" |
| \<AppIdentifier\> | アプリケーション登録に割り当てられた識別子。 | "S-1-5-21-1634563105-1224503876-2692824315-2623" |

1. 管理者特権のある Windows PowerShell セッションを使用して、次のコマンドレットを実行します。

     ```powershell
     # Create a PSSession to the PrivilegedEndpoint VM
     $Session = New-PSSession -ComputerName "<PepVM>" -ConfigurationName PrivilegedEndpoint -Credential $Creds

     # Use the privileged endpoint to update the client secret, used by the service principal associated with <AppIdentifier>
     $SpObject = Invoke-Command -Session $Session -ScriptBlock {Set-GraphApplication -ApplicationIdentifier "<AppIdentifier>" -ResetClientSecret}
     $Session | Remove-PSSession

     # Output the updated service principal details
     $SpObject
     ```

2. このスクリプトが完了すると、更新後のアプリ登録の情報が表示され、この中に新しく生成されたクライアント シークレットも含まれます。

     ```shell  
     ApplicationIdentifier : S-1-5-21-1634563105-1224503876-2692824315-2623
     ClientId              : 8e0ffd12-26c8-4178-a74b-f26bd28db601
     Thumbprint            : 
     ApplicationName       : Azurestack-YourApp-6967581b-497e-4f5a-87b5-0c8d01a9f146
     ClientSecret          : MKUNzeL6PwmlhWdHB59c25WDDZlJ1A6IWzwgv_Kn
     PSComputerName        : azs-ercs01
     RunspaceId            : 6ed9f903-f1be-44e3-9fef-e7e0e3f48564
     ```

### <a name="remove-a-service-principal"></a>サービス プリンシパルを削除する

ここでは、PowerShell を使用してアプリ登録をディレクトリから削除する方法を説明します。関連付けられたサービス プリンシパル オブジェクトも削除されます。 

次のプレースホルダーを実際の値で置き換えてください。

| プレースホルダー | 説明 | 例 |
| ----------- | ----------- | ------- |
| \<PepVM\> | Azure Stack Hub インスタンス上にある特権エンドポイント VM の名前。 | "AzS-ERCS01" |
| \<AppIdentifier\> | アプリケーション登録に割り当てられた識別子。 | "S-1-5-21-1634563105-1224503876-2692824315-2623" |

```powershell  
# Sign in to PowerShell interactively, using credentials that have access to the VM running the Privileged Endpoint (typically <domain>\cloudadmin)
$Creds = Get-Credential

# Create a PSSession to the PrivilegedEndpoint VM
$Session = New-PSSession -ComputerName "<PepVM>" -ConfigurationName PrivilegedEndpoint -Credential $Creds

# OPTIONAL: Use the privileged endpoint to get a list of applications registered in AD FS
$AppList = Invoke-Command -Session $Session -ScriptBlock {Get-GraphApplication}

# Use the privileged endpoint to remove the application and associated service principal object for <AppIdentifier>
Invoke-Command -Session $Session -ScriptBlock {Remove-GraphApplication -ApplicationIdentifier "<AppIdentifier>"}
```

Remove-GraphApplication コマンドレットを特権エンドポイントに対して呼び出したときに返される出力はありませんが、このコマンドレットの実行中に次のような詳細な確認出力がコンソールに表示されます。

```shell
VERBOSE: Deleting graph application with identifier S-1-5-21-1634563105-1224503876-2692824315-2623.
VERBOSE: Remove-GraphApplication : BEGIN on AZS-ADFS01 on ADFSGraphEndpoint
VERBOSE: Application with identifier S-1-5-21-1634563105-1224503876-2692824315-2623 was deleted.
VERBOSE: Remove-GraphApplication : END on AZS-ADFS01 under ADFSGraphEndpoint configuration
```

## <a name="assign-a-role"></a>ロールの割り当て

ユーザーやアプリによる Azure リソースへのアクセスは、ロールベースのアクセス制御 (RBAC) を通じて承認されます。 自分のサブスクリプションに含まれるリソースにアプリからアクセスできるようにするには、そのサービス プリンシパルを特定の "*リソース*" に対する "*ロール*" に "*割り当てる*" 必要があります。 初めに、どのロールがそのアプリにとって適切な "*アクセス許可*" を表すかを判断します。 利用可能なロールについては、「[Azure リソースの組み込みロール](/azure/role-based-access-control/built-in-roles)」を参照してください。

選択したリソースの種類に応じて、アプリの "*アクセス スコープ*" も決まります。 アクセス スコープはサブスクリプション、リソース グループ、またはリソースのレベルで設定できます。 アクセス許可は、スコープの下位レベルに継承されます。 たとえば、アプリをリソース グループの "閲覧者" ロールに追加すると、そのリソース グループと、その中にあるすべてのリソースを読み取ることができることになります。

1. Azure Stack Hub のインストール時に指定したディレクトリに基づいて、適切なポータルにサインインします (たとえば、Azure AD の場合は Azure portal、AD FS の場合は Azure Stack Hub ユーザー ポータル)。 この例では、ユーザーが Azure Stack Hub ユーザー ポータルにサインインしたとします。

   > [!NOTE]
   > リソースに対するロール割り当てを追加するには、この操作を行うユーザー アカウントが属しているロールで `Microsoft.Authorization/roleAssignments/write` の許可が宣言されている必要があります。 この条件を満たすのは、組み込みロールである[所有者](/azure/role-based-access-control/built-in-roles#owner)や[ユーザー アクセス管理者](/azure/role-based-access-control/built-in-roles#user-access-administrator)などです。  
2. どのリソースへのアクセスをアプリに許可するかを決定して、そのリソースの場所まで移動します。 この例では、サブスクリプションをスコープとしてアプリのサービス プリンシパルをロールに割り当てるので、 **[サブスクリプション]** を選択してから特定のサブスクリプションを選択します。 代わりに、リソース グループを選択することも、特定のリソース (仮想マシンなど) を選択することもできます。

     ![割り当てのためのサブスクリプションを選択する](./media/azure-stack-create-service-principal/select-subscription.png)

3. **[アクセス制御 (IAM)]** ページを選択します。このページは、RBAC をサポートするすべてのリソースにあります。
4. **[+ 追加]** を選択します。
5. **[ロール]** の下で、アプリに割り当てるロールを選択します。
6. **[選択]** の下で、アプリケーション名の全体または一部を入力してアプリを検索します。 登録時に、アプリケーション名は *Azurestack-\<YourAppName\>YourAppName-\<ClientId\>* として生成されます。 たとえば、作成時にアプリケーション名として *App2* を使用し、クライアント ID として *2bbe67d8-3fdb-4b62-87cf-cc41dd4344ff* が割り当てられた場合のフル ネームは *Azurestack-App2-2bbe67d8-3fdb-4b62-87cf-cc41dd4344ff* となります。 検索するときに、この文字列全体を正確に指定することも、その一部分 (たとえば *Azurestack* や *Azurestack-App2*) を指定することもできます。
7. アプリが見つかったら、そのアプリを選択すると **[選択したメンバー]** に表示されます。
8. **[保存]** を選択して、ロールの割り当てを完了します。

     [![ロールを割り当てる](media/azure-stack-create-service-principal/assign-role.png)](media/azure-stack-create-service-principal/assign-role.png#lightbox)

9. 完了すると、現在のスコープに割り当てられているプリンシパルの一覧の、指定のロールのセクションにアプリが表示されます。

     [![割り当てられたロール](media/azure-stack-create-service-principal/assigned-role.png)](media/azure-stack-create-service-principal/assigned-role.png#lightbox)

これでアプリに ID が与えられ、リソース アクセスが承認されたので、スクリプトやコードでサインインし、Azure Stack Hub リソースに安全にアクセスできます。  

## <a name="next-steps"></a>次のステップ

[ユーザー アクセス許可の管理](azure-stack-manage-permissions.md)  
[Azure Active Directory のドキュメント](/azure/active-directory)  
[Active Directory フェデレーション サービス (AD FS)](/windows-server/identity/active-directory-federation-services)
