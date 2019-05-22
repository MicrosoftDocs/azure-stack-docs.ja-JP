---
title: Azure Stack での Java による API バージョンのプロファイルの使用 | Microsoft Docs
description: Azure Stack での Java による API バージョンのプロファイルの使用について説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/26/2018
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 09/28/2018
ms.openlocfilehash: 175c684d8f92355c5669c0e37ff5a4dfb993ea61
ms.sourcegitcommit: 24d5c16132d4c40a760ad6f631739af86188a09f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/29/2019
ms.locfileid: "64910076"
---
# <a name="use-api-version-profiles-with-java-in-azure-stack"></a>Azure Stack での Java による API バージョンのプロファイルの使用

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*

Azure Stack Resource Manager 向けの Java SDK には､インフラストラクチャの構築と管理に役立つツールが用意されています。 SDK のリソース プロバイダーには、コンピューティング、ネットワーク、ストレージ、アプリ サービス、および [KeyVault](/azure/key-vault/key-vault-whatis) が含まれます。 Java SDK は、.java ファイルに適切なモジュールを読み込む Pom.xml ファイルに依存関係を含めることで、API プロファイルを組み込みます。 ただし、依存関係として複数のプロファイル (**2018-03-01-hybrid** など) を追加したり、Azure プロファイルとして**最新のもの**を追加したりできます。 これらの依存関係を使用すると適切なモジュールが読み込まれるため、リソースの種類を作成するときに、それらのプロファイルから使用したい API バージョンを選択することができます。 これにより、Azure Stack の最新の API バージョンに対する開発の際に、Azure で最新バージョンを使用することができます。 Java SDK を使用すると、真のハイブリッド クラウド開発者エクスペリエンスを実現できます。 Java SDK に含まれている API プロファイルを使用すると、グローバルな Azure リソースと Azure Stack 上のリソース間を切り替えることで、ハイブリッド クラウド開発を行うことができます。

## <a name="java-and-api-version-profiles"></a>Java と API バージョンのプロファイル

API プロファイルは、リソース プロバイダーと API バージョンを組み合わせたものです。 API プロファイルを使用すると、リソース プロバイダーのパッケージに含まれる各リソースの種類の、最も安定した最新バージョンを取得できます。

- すべてのサービスの最新バージョンを使用するには、依存関係として **latest** プロファイルを使用します。

  - latest プロファイルを使用する場合、依存関係は **com.microsoft.azure** です。

  - Azure Stack と互換性のあるサービスを使用するには、**com.microsoft.azure.profile\_2018\_03\_01\_hybrid** プロファイルを使用します。

    - これは、Pom.xml ファイルに依存関係として指定するためです。これにより、.NET と同様に、ドロップダウン リストから適切なクラスを選択すると、モジュールが自動的に読み込まれます。

    - 各モジュールの先頭は次のようになります: `Import com.microsoft.azure.management.resources.v2018_03_01.ResourceGroup`

  - 依存関係は次のようになります。

     ```xml
     <dependency>
     <groupId>com.microsoft.azure.profile_2018_03_01_hybrid</groupId>
     <artifactId>azure</artifactId>
     <version>1.0.0-beta</version>
     </dependency>
     ```

  - 特定のリソース プロバイダーのリソースの種類に合わせて特定の API バージョンを使用するには、Intellisense を使用して定義した特定の API バージョンを使用します。

すべてのオプションを同じアプリケーション内で組み合わせることができることに注意してください。

## <a name="install-the-azure-java-sdk"></a>Azure Java SDK のインストール

次の手順を使用して Java SDK をインストールします。

1. 正式な指示に従って、Git をインストールします。 手順については、「[Getting Started - Installing Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)」 (はじめに - Git をインストールする) をご覧ください。

2. 指示に従って、[Java SDK](https://zulu.org/download/) と [Maven](https://maven.apache.org/) をインストールします。 適切なバージョンは、Java Developer Kit のバージョン 8 です。 適切な Apache Maven はバージョン 3.0 以降です。 このクイック スタートを完了するには、JAVA_HOME 環境変数を Java Development Kit のインストール場所に設定する必要があります。 詳細については、「[Java と Maven を使用して初めての関数を作成する](/azure/azure-functions/functions-create-first-java-maven)」を参照してください。

3. 適切な依存関係パッケージをインストールするには、Java アプリケーションで Pom.xml ファイルを開きます。 次のコードに示されているように、依存関係を追加します。

   ```xml  
   <dependency>
   <groupId>com.microsoft.azure.profile_2018_03_01_hybrid</groupId>
   <artifactId>azure</artifactId>
   <version>1.0.0-beta</version>
   </dependency>
   ```

4. インストールする必要があるパッケージのセットは、使用したいプロファイル バージョンによって異なります。 プロファイル バージョンのパッケージ名は次のとおりです。

   - **com.microsoft.azure.profile\_2018\_03\_01\_hybrid**
   - **com.microsoft.azure**
     - **latest**

5. 使用できない場合は、サブスクリプションを作成し、後で使用するためにサブスクリプション ID を保存します。 サブスクリプションの作成方法については、「[Azure Stack でオファーのサブスクリプションを作成する](../operator/azure-stack-subscribe-plan-provision-vm.md)」を参照してください。

6. サービス プリンシパルを作成し、クライアント ID とクライアント シークレットを保存します。 Azure Stack 用のサービス プリンシパルの作成方法については、「[Azure Stack へのアクセスをアプリケーションに提供する](../operator/azure-stack-create-service-principals.md)」を参照してください。 クライアント ID は、サービス プリンシパルの作成時にはアプリケーション ID とも呼ばれることに注意してください。

7. サブスクリプションでサービス プリンシパルのロールが共同作成者/所有者であることを確認します。 サービス プリンシパルへのロールの割り当て手順については、「[Azure Stack へのアクセスをアプリケーションに提供する](../operator/azure-stack-create-service-principals.md)」を参照してください。

## <a name="prerequisites"></a>前提条件

Azure Stack で Azure Java SDK を使用するには、次の値を指定した後、環境変数に値を設定する必要があります。 環境変数を設定するには、使用しているオペレーティング システムの表の後にある手順を参照してください。

| 値                     | 環境変数 | 説明                                                                                                                                                                                                          |
| ------------------------- | --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| テナント ID                 | AZURE_TENANT_ID            | Azure Stack の[テナント ID](../operator/azure-stack-identity-overview.md) の値。                                                          |
| クライアント ID                 | AZURE_CLIENT_ID             | 前のセクションでサービス プリンシパルが作成されたときに保存した、サービス プリンシパル アプリケーション ID。                                                                                              |
| サブスクリプション ID           | AZURE_SUBSCRIPTION_ID      | [サブスクリプション ID](../operator/azure-stack-plan-offer-quota-overview.md#subscriptions) は Azure Stack 内のオファーにアクセスするために必要です。                |
| クライアント シークレット             | AZURE_CLIENT_SECRET        | サービス プリンシパルの作成時に保存した、サービス プリンシパル アプリケーション シークレット。                                                                                                                                   |
| Resource Manager エンドポイント | ARM_ENDPOINT              | 「[Azure Stack Resource Manager エンドポイント](../user/azure-stack-version-profiles-ruby.md#the-azure-stack-resource-manager-endpoint)」を参照してください。 |
| Location                  | RESOURCE_LOCATION    | Azure Stack 用の**ローカル**。                                                                                                                                                                                                |

Azure Stack のテナント ID を確認するには、[こちら](../operator/azure-stack-csp-ref-operations.md)の手順をご覧ください。 環境変数を設定するには、次の手順に従います。

### <a name="microsoft-windows"></a>Microsoft Windows

環境変数を設定するには、Windows コマンド プロンプトで次の形式を使用します。

```shell
Set AZURE_TENANT_ID=<Your_Tenant_ID>
```

### <a name="macos-linux-and-unix-based-systems"></a>macOS、Linux、および Unix ベースのシステム

Unix ベースのシステムでは、次のコマンドを使用します。

```shell
Export AZURE_TENANT_ID=<Your_Tenant_ID>
```

### <a name="trust-the-azure-stack-ca-root-certificate"></a>Azure Stack の CA ルート証明書を信頼する

ASDK を使用する場合は、リモート マシン上で CA ルート証明書を信頼する必要があります。 統合システムではそれを行う必要はありません。

#### <a name="windows"></a> Windows

1. Azure Stack の自己署名証明書を自分のデスクトップにエクスポートします。

1. コマンド プロンプトでディレクトリを %JAVA_HOME%\bin に変更します。

1. 次のコマンドを実行します。

   ```shell
   .\keytool.exe -importcert -noprompt -file <location of the exported certificate here> -alias root -keystore %JAVA_HOME%\lib\security\cacerts -trustcacerts -storepass changeit
   ```

### <a name="the-azure-stack-resource-manager-endpoint"></a>Azure Stack Resource Manager エンドポイント

Microsoft Azure Resource Manager は、管理者が Azure リソースのデプロイ、管理、監視を行えるようにするための管理フレームワークです。 Azure Resource Manager では、これらのタスクを個別に処理するのではなく、グループとして単一の操作で処理することができます。

メタデータ情報は、Resource Manager エンドポイントから取得できます。 エンドポイントは、コードを実行するために必要な情報と共に、JSON ファイルを返します。

次の考慮事項に注意してください。

- Azure Stack Development Kit (ASDK) の **ResourceManagerUrl** は https://management.local.azurestack.external/ です。

- 統合されたシステムの **ResourceManagerUrl** は `https://management.<location>.ext-<machine-name>.masd.stbtest.microsoft.com/` です。

必要なメタデータを取得するには、`<ResourceManagerUrl>/metadata/endpoints?api-version=1.0` になります。

サンプルの JSON ファイルは、次のようになります。

```json
{
   "galleryEndpoint": "https://portal.local.azurestack.external:30015/",
   "graphEndpoint": "https://graph.windows.net/",
   "portal Endpoint": "https://portal.local.azurestack.external/",
   "authentication":
      {
      "loginEndpoint": "https://login.windows.net/",
      "audiences": ["https://management.<yourtenant>.onmicrosoft.com/3cc5febd-e4b7-4a85-a2ed-1d730e2f5928"]
      }
}
```

## <a name="existing-api-profiles"></a>既存の API プロファイル

- **com.microsoft.azure.profile\_2018\_03\_01\_hybrid**:Azure Stack 用に作成された最新のプロファイル。 1808 以降のスタンプを使用している限り、Azure Stack との互換性に優れたサービスには、このプロファイルを使用します。

- **com.microsoft.azure**:すべてのサービスの最新バージョンで構成されているプロンプトファイル。 すべてのサービスの最新バージョンを使用してください。

Azure Stack および API プロファイルについて詳しくは、「[API プロファイルの概要](../user/azure-stack-version-profiles.md#summary-of-api-profiles)」をご覧ください。

## <a name="azure-java-sdk-api-profile-usage"></a>Azure Java SDK API プロファイルの使用

次のコードでは、Azure Stack でサービス プリンシパルを認証します。 これにより、テナント ID と、Azure Stack に固有の認証基準を使用して、トークンが作成されます。

```java
AzureTokenCredentials credentials = new ApplicationTokenCredentials(client, tenant, key, AZURE_STACK)
                    .withDefaultSubscriptionID(subscriptionID);
Azure azureStack = Azure.configure()
                    .withLogLevel(com.microsoft.rest.LogLevel.BASIC)
                    .authenticate(credentials, credentials.defaultSubscriptionID());
```

これにより、API プロファイルの依存関係を使用して、アプリケーションを Azure Stack に正常にデプロイすることができます。

## <a name="define-azure-stack-environment-setting-functions"></a>Azure Stack 環境設定関数の定義

適切なエンドポイントで Azure Stack クラウドを登録するには、次のコードを使用します。

```java
AzureEnvironment AZURE_STACK = new AzureEnvironment(new HashMap<String, String>() {
                {
                    put("managementEndpointUrl", settings.get("audience"));
                    put("resourceManagerEndpointUrl", armEndpoint);
                    put("galleryEndpointUrl", settings.get("galleryEndpoint"));
                    put("activeDirectoryEndpointUrl", settings.get("login_endpoint"));
                    put("activeDirectoryResourceID", settings.get("audience"));
                    put("activeDirectoryGraphResourceID", settings.get("graphEndpoint"));
                    put("storageEndpointSuffix", armEndpoint.substring(armEndpoint.indexOf('.')));
                    put("keyVaultDnsSuffix", ".vault" + armEndpoint.substring(armEndpoint.indexOf('.')));
                }
            });
```

次のコードの `getActiveDirectorySettings` 呼び出しは、メタデータ エンドポイントからエンドポイントを取得します。 これは、この呼び出しからの環境変数を示しています。

```java
public static HashMap<String, String>
getActiveDirectorySettings(String armEndpoint) {

HashMap<String, String> adSettings = new HashMap<String, String>();

try {

// create HTTP Client
HttpClient httpClient = HttpClientBuilder.create().build();

// Create new getRequest with below mentioned URL
HttpGet getRequest = new
HttpGet(String.format("%s/metadata/endpoints?api-version=1.0",
armEndpoint));

// Add additional header to getRequest which accepts application/xml data
getRequest.addHeader("accept", "application/xml");

// Execute request and catch response
HttpResponse response = httpClient.execute(getRequest);
```

## <a name="samples-using-api-profiles"></a>API プロファイルを使用したサンプル

.NET と Azure Stack API のプロファイルを使用してソリューションを作成する場合は、次の GitHub のサンプルを参考資料として使用できます。

- [リソース グループの管理](https://github.com/Azure-Samples/Hybrid-resources-java-manage-resource-group)

- [ストレージ アカウントの管理](https://github.com/Azure-Samples/hybrid-storage-java-manage-storage-accounts)

- [仮想マシンの管理](https://github.com/Azure-Samples/hybrid-compute-java-manage-vm)

### <a name="sample-unit-test-project"></a>単体テスト プロジェクトのサンプル

1. 次のコマンドを使用して、リポジトリを複製します。

   `git clone https://github.com/Azure-Samples/Hybrid-resources-java-manage-resource-group.git`

2. Azure サービス プリンシパルを作成し、サブスクリプションにアクセスするロールを割り当てます。 サービス プリンシパルの作成方法については、「[Azure PowerShell を使用して資格情報でのサービス プリンシパルを作成する](../operator/azure-stack-create-service-principals.md)」をご覧ください。

3. 次の必要な環境変数値を取得します。

   - AZURE_TENANT_ID
   - AZURE_CLIENT_ID
   - AZURE_CLIENT_SECRET
   - AZURE_SUBSCRIPTION_ID
   - ARM_ENDPOINT
   - RESOURCE_LOCATION

4. コマンド プロンプトを使用して作成したサービス プリンシパルから取得した情報を使用して、次の環境変数を設定します。

   - export AZURE_TENANT_ID={ご使用のテナント ID}
   - export AZURE_CLIENT_ID={ご使用のクライアント ID}
   - export AZURE_CLIENT_SECRET={ご使用のクライアント シークレット}
   - export AZURE_SUBSCRIPTION_ID={ご使用のサブスクリプション ID}
   - export ARM_ENDPOINT={ご使用の Azure Stack Resource Manager URL}
   - export RESOURCE_LOCATION={Azure Stack の場所}

   Windows では、**export** ではなく **set** を使用します。

5. `getactivedirectorysettings` コードを使用して ARM メタデータ エンドポイントを取得し、HTTP クライアントを使用してエンドポイント情報を設定します。

   ```java
   public static HashMap<String, String> getActiveDirectorySettings(String armEndpoint) {
   HashMap<String, String> adSettings = new HashMap<String,> String>();

   try {

   // create HTTP Client
   HttpClient httpClient = HttpClientBuilder.create().build();

   // Create new getRequest with below mentioned URL
   HttpGet getRequest = new
   HttpGet(String.format("%s/metadata/endpoints?api-version=1.0", armEndpoint));

   // Add additional header to getRequest which accepts application/xml data
   getRequest.addHeader("accept", "application/xml");

   // Execute request and catch response
   HttpResponse response = httpClient.execute(getRequest);
   ```

6. Pom.xml ファイルで、Azure Stack 用の **2018-03-01-hybrid** プロファイルを使用するために下の依存関係を追加します。 この依存関係では、コンピューティング、ネットワーク、ストレージ、KeyVault、App Services のリソース プロバイダー用の、このプロファイルに関連付けられているモジュールがインストールされます。

   ```xml
   <dependency>
   <groupId>com.microsoft.azure.profile_2018_03_01_hybrid</groupId>
   <artifactId>azure</artifactId>
   <vers1s.0.0-beta</version>
   </dependency>
   ```

7. 環境変数を設定するために開いていたコマンド プロンプトで、次のコマンドを入力します。

   ```shell
   mvn clean compile exec:java
   ```

## <a name="next-steps"></a>次の手順

API プロファイルの詳細については、以下を参照してください。

- [Azure Stack のバージョン プロファイル](azure-stack-version-profiles.md)
- [プロファイルでサポートされているリソース プロバイダー API バージョン](azure-stack-profiles-azure-resource-manager-versions.md)
