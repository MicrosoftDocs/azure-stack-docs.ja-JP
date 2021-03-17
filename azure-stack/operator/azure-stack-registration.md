---
title: Azure Stack Hub を Azure に登録する
titleSuffix: Azure Stack Hub
description: Azure Stack Hub 統合システムを Azure に登録することで、Azure Marketplace 項目をダウンロードしてデータ レポートを設定できるようにする方法について説明します。
author: PatAltimore
ms.topic: how-to
ms.date: 02/26/2021
ms.author: patricka
ms.reviewer: avishwan
ms.lastreviewed: 11/19/2020
ms.custom: contperf-fy20q4
zone_pivot_groups: state-connected-disconnected
ms.openlocfilehash: 53dd1a8b168f5c0e2285878f811a3e055cfd1929
ms.sourcegitcommit: 4f1d22747c02ae280609174496933fca8c04a6cf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102606365"
---
# <a name="register-azure-stack-hub-with-azure"></a>Azure Stack Hub を Azure に登録する

Azure Stack Hub を Azure に登録して、Azure から Azure Marketplace 項目をダウンロードしたり、Microsoft に報告を返すコマース データを設定したりできます。 Azure Stack Hub を登録した後は、Azure コマースに使用状況が報告され、登録に使用した Azure 課金サブスクリプション ID で確認できます。

この記事では、Azure を使用して Azure Stack Hub 統合システムを登録する方法について説明します。 Azure を使用して ASDK を登録する方法については、ASDK ドキュメントの [Azure Stack Hub の登録](../asdk/asdk-register.md)に関するページを参照してください。

> [!IMPORTANT]  
> マーケットプレースのオファー項目を含む、Azure Stack Hub のすべての機能をサポートするには、登録が必要です。 従量制課金モデルを使用している場合、登録しないと、Azure Stack Hub のライセンス条項違反になります。 Azure Stack Hub のライセンス モデルの詳細については、[購入方法のページ](https://azure.microsoft.com/overview/azure-stack/how-to-buy/)を参照してください。

## <a name="prerequisites"></a>前提条件

登録する前に、次の前提条件を完了してください。

- 資格情報を確認する。
- PowerShell 言語モードを設定する。
- PowerShell for Azure Stack Hub をインストールする。
- Azure Stack Hub ツールをダウンロードする。
- 課金モデルを決定する。
- 一意の登録名を決定する。

### <a name="verify-your-credentials"></a>資格情報を確認する

Azure を使用して Azure Stack Hub を登録する前に、以下のものが必要です。

::: zone pivot="state-connected"
- Azure サブスクリプションのサブスクリプション ID。 登録には、EA、CSP、CSP 共有サービス サブスクリプションだけがサポートされています。 CSP では、[CSP サブスクリプションまたは APSS サブスクリプションのどちらを使用するか](azure-stack-add-manage-billing-as-a-csp.md#create-a-csp-or-apss-subscription)を決める必要があります。<br><br>ID を取得するには、Azure にサインインして、 **[すべてのサービス]** をクリックします。 次に、 **[一般]** カテゴリの下で **[サブスクリプション]** を選び、使用するサブスクリプションをクリックすると、 **[要点]** の下にサブスクリプション ID が表示されます。 ベスト プラクティスとして、運用環境と開発環境またはテスト環境には、異なるサブスクリプションを使用してください。 
::: zone-end
::: zone pivot="state-disconnected"
- Azure サブスクリプションのサブスクリプション ID。 登録には、EA サブスクリプションだけがサポートされています。 

    ID を取得するには、Azure にサインインして、 **[すべてのサービス]** をクリックします。 次に、 **[一般]** カテゴリの下で **[サブスクリプション]** を選び、使用するサブスクリプションをクリックすると、 **[要点]** の下にサブスクリプション ID が表示されます。 ベスト プラクティスとして、運用環境と開発環境またはテスト環境には、異なるサブスクリプションを使用してください。 
::: zone-end
   > [!Note]  
   > Germany Cloud のサブスクリプションは現在サポートされていません。  

- サブスクリプションの所有者であるアカウントのユーザー名とパスワード。

- ユーザー アカウントは Azure サブスクリプションにアクセスできなければならず、そのサブスクリプションに関連付けられているディレクトリで ID アプリとサービス プリンシパルを作成するアクセス許可を持つ必要があります。 最低限の特権による管理を使用し、Azure Stack Hub を Azure に登録することをお勧めします。 登録のためのサブスクリプションへのアクセスを制限するカスタム ロール定義を作成する方法について詳しくは、[Azure Stack Hub の登録ロールの作成](azure-stack-registration-role.md)に関するページを参照してください。

- Azure Stack Hub リソース プロバイダーを登録しました (詳細については、Azure Stack Hub リソース プロバイダーの登録に関する次のセクションを参照してください)。

登録の後、Azure Active Directory (Azure AD) の全体管理者のアクセス許可は必要ありません。 ただし、一部の操作では、全体管理者の資格情報が必要な場合があります (たとえば、リソース プロバイダーのインストーラー スクリプトや、アクセス許可の付与を必要とする新機能です)。 アカウントの全体管理者のアクセス許可を一時的に復元するか、*既定のプロバイダー サブスクリプション* の所有者である別の全体管理者アカウントを使用します。

Azure Stack Hub を登録するユーザーは、Azure AD のサービス プリンシパルの所有者です。 Azure Stack Hub の登録を変更できるのは、Azure Stack Hub を登録したユーザーのみです。 登録サービス プリンシパルの所有者ではない管理者以外のユーザーが Azure Stack Hub を登録または再登録しようとすると、403 応答が発生する可能性があります。 403 応答は、操作を完了するための十分な権限がユーザーにないことを示します。

これらの要件を満たす Azure サブスクリプションがない場合は、[ここで無料の Azure アカウントを作成](https://azure.microsoft.com/free/?b=17.06)できます。 Azure Stack Hub を登録しても、Azure サブスクリプションに課金されることはありません。

> [!NOTE]
> 複数の Azure Stack Hub がある場合のベスト プラクティスは、各 Azure Stack Hub を専用のサブスクリプションに登録することです。 これにより、使用状況の追跡が容易になります。

### <a name="set-the-powershell-language-mode"></a>PowerShell 言語モードを設定する

Azure Stack Hub を正常に登録するには、PowerShell 言語モードを **FullLanguageMode** に設定する必要があります。  現在の言語モードが完全に設定されていることを確認するには、PowerShell ウィンドウを管理者特権で開いて、次の PowerShell コマンドレットを実行します。

```powershell  
$ExecutionContext.SessionState.LanguageMode
```

出力で **FullLanguageMode** が返されていることを確認します。 その他の言語モードが返されている場合は、別のマシン上で再登録を実行するか、言語モードを **FullLanguageMode** に設定してから作業を続行する必要があります。

### <a name="install-powershell-for-azure-stack-hub"></a>PowerShell for Azure Stack Hub をインストールする

Azure に登録するために、最新の PowerShell for Azure Stack Hub を使用します。

最新バージョンがまだインストールされていない場合は、[PowerShell for Azure Stack Hub のインストール](powershell-install-az-module.md)に関するページを参照してください。

### <a name="download-the-azure-stack-hub-tools"></a>Azure Stack Hub ツールをダウンロードする

Azure Stack Hub ツールの GitHub リポジトリには、Azure Stack Hub 機能 (登録機能を含む) をサポートする PowerShell モジュールが含まれています。 登録プロセス中に、**RegisterWithAzure.psm1** PowerShell モジュール (Azure Stack Hub ツール リポジトリにあります) をインポートおよび利用して、使用する Azure Stack Hub インスタンスを Azure に登録する必要があります。

最新バージョンを使用していることを確認するには、Azure に登録する前に、既存のバージョンの Azure Stack Hub ツールをすべて削除し、[GitHub から最新バージョンをダウンロード](azure-stack-powershell-download.md)します。

[!INCLUDE [Azure Stack Hub Operator Access Workstation](../includes/operator-note-owa.md)]

### <a name="determine-your-billing-model"></a>課金モデルを決定する
::: zone pivot="state-connected"
 接続されたデプロイでは、Azure Stack Hub はインターネットと Azure に接続できます。 また、ID ストアとして Azure AD または Active Directory フェデレーション サービス (AD FS) を使用して、従量課金制と容量ベースの 2 つの課金モデルのどちらかを選択することもできます。 課金モデルは、後で登録スクリプトの実行中に指定します。
::: zone-end

::: zone pivot="state-disconnected"
 切断されたデプロイでは、インターネットに接続していなくても、Azure Stack Hub を使用することができます。 切断されたデプロイでは、AD FS ID ストアおよび容量ベースの課金モデルに制限されます。 課金モデルは、後で登録スクリプトの実行中に指定します。
::: zone-end

### <a name="determine-your-unique-registration-name"></a>一意の登録名を決定する

登録スクリプトを実行するときに、一意の登録名を指定する必要があります。 Azure の登録に Azure Stack Hub サブスクリプションを関連付ける簡単な方法は、Azure Stack Hub の **クラウド ID** を使用することです。

> [!NOTE]
> 容量ベースの課金モデルを使用している Azure Stack Hub 登録では、[期限切れの登録を削除](#renew-or-change-registration)して Azure に再登録するのでない限り、年単位のサブスクリプションの期限が切れた後の再登録で一意の名前を変更する必要があります。

Azure Stack Hub デプロイのクラウド ID を確認する場合は、「[クラウド ID を検索する](azure-stack-find-cloud-id.md)」をご覧ください。

::: zone pivot="state-connected"
## <a name="register-with-pay-as-you-use-billing"></a>従量課金制の課金で登録する

従量制課金モデルを使用して Azure Stack Hub を Azure に登録するには、次の手順を使用します。

> [!Note]  
> これらのすべての手順は、特権エンドポイント (PEP) にアクセスできるコンピューターから実行する必要があります。 PEP の詳細については、[Azure Stack Hub での特権エンドポイントの使用](azure-stack-privileged-endpoint.md)に関するページを参照してください。

接続された環境では、インターネットと Azure にアクセスできます。 これらの環境では、Azure Stack Hub リソース プロバイダーを Azure に登録してから、課金モデルを構成する必要があります。

### <a name="az-modules"></a>[Az モジュール](#tab/az1)

1. Azure Stack Hub リソース プロバイダーを Azure に登録するには、PowerShell ISE を管理者として起動し、**EnvironmentName** パラメーターを適切な Azure サブスクリプション タイプ (以下のパラメーターを参照) に設定して、次の PowerShell コマンドレットを使用します。

2. Azure Stack Hub を登録するために使用した Azure アカウントを追加します。 アカウントを追加するには、**Connect-AzAccount** コマンドレットを実行します。 Azure アカウント資格情報の入力を求められ、お使いのアカウントの構成によっては 2 要素認証を使用する必要があります。

   ```powershell
   Connect-AzAccount -EnvironmentName "<environment name>"
   ```

   | パラメーター | 説明 |  
   |-----|-----|
   | EnvironmentName | Azure クラウド サブスクリプション環境名。 サポートされている環境名は **AzureCloud**、**AzureUSGovernment**、または中国の Azure サブスクリプションを使用している場合は **AzureChinaCloud** です。  |

   >[!Note]
   > セッションの有効期限が切れた、パスワードが変更された、または単にアカウントを切り替えたい場合は、Connect-AzAccount を使用してサインインする前に、次のコマンドレットを実行します: `Remove-AzAccount-Scope Process`

3. 複数のサブスクリプションがある場合は、次のコマンドを実行して、使用するものを選択します。  

   ```powershell  
   Get-AzSubscription -SubscriptionID '<Your Azure Subscription GUID>' | Select-AzSubscription
   ```

4. Azure サブスクリプションで Azure Stack Hub リソース プロバイダーを登録するには、次のコマンドを実行します。

   ```powershell  
   Register-AzResourceProvider -ProviderNamespace Microsoft.AzureStack
   ```

5. 管理者として PowerShell ISE を起動し、Azure Stack Hub ツールをダウンロードしたときに作成された **AzureStack-Tools-az** ディレクトリ内の **Registration** フォルダーに移動します。 PowerShell を使用して **RegisterWithAzure.psm1** モジュールをインポートします。

   ```powershell  
   Import-Module .\RegisterWithAzure.psm1
   ```

6. 次に、同じ PowerShell セッションで、正しい Azure PowerShell コンテキストにサインインしていることを確認します。 このコンテキストでは、前に Azure Stack Hub リソース プロバイダーを登録するために使用された Azure アカウントです。 実行する PowerShell:

   ```powershell  
   Connect-AzAccount -Environment "<environment name>"
   ```

   | パラメーター | 説明 |  
   |-----|-----|
   | EnvironmentName | Azure クラウド サブスクリプション環境名。 サポートされている環境名は **AzureCloud**、**AzureUSGovernment**、または中国の Azure サブスクリプションを使用している場合は **AzureChinaCloud** です。  |

7. 同じ PowerShell セッションで **Set-AzsRegistration** コマンドレットを実行します。 実行する PowerShell:  

   ```powershell  
   $CloudAdminCred = Get-Credential -UserName <Privileged endpoint credentials> -Message "Enter the cloud domain credentials to access the privileged endpoint."
   $RegistrationName = "<unique-registration-name>"
   Set-AzsRegistration `
      -PrivilegedEndpointCredential $CloudAdminCred `
      -PrivilegedEndpoint <PrivilegedEndPoint computer name> `
      -BillingModel PayAsYouUse `
      -RegistrationName $RegistrationName
   ```
   Set-AzsRegistration コマンドレットの詳細については、「[登録に関するリファレンス](#registration-reference)」を参照してください。

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm1)

1. Azure Stack Hub リソース プロバイダーを Azure に登録するには、PowerShell ISE を管理者として起動し、**EnvironmentName** パラメーターを適切な Azure サブスクリプション タイプ (以下のパラメーターを参照) に設定して、次の PowerShell コマンドレットを使用します。

2. Azure Stack Hub を登録するために使用した Azure アカウントを追加します。 アカウントを追加するには、**Add-AzureRMAccount** コマンドレットを実行します。 Azure アカウント資格情報の入力を求められ、お使いのアカウントの構成によっては 2 要素認証を使用する必要があります。

   ```powershell
   Add-AzureRMAccount -EnvironmentName "<environment name>"
   ```

   | パラメーター | 説明 |  
   |-----|-----|
   | EnvironmentName | Azure クラウド サブスクリプション環境名。 サポートされている環境名は **AzureCloud**、**AzureUSGovernment**、または中国の Azure サブスクリプションを使用している場合は **AzureChinaCloud** です。  |

   >[!Note]
   > セッションの有効期限が切れた、パスワードが変更された、または単にアカウントを切り替えたい場合は、Add-AzureRMAccount を使用してサインインする前に、次のコマンドレットを実行します。`Remove-AzureRMAccount-Scope Process`

3. 複数のサブスクリプションがある場合は、次のコマンドを実行して、使用するものを選択します。  

   ```powershell  
   Get-AzureRMSubscription -SubscriptionID '<Your Azure Subscription GUID>' | Select-AzureRMSubscription
   ```

4. Azure サブスクリプションで Azure Stack Hub リソース プロバイダーを登録するには、次のコマンドを実行します。

   ```powershell  
   Register-AzureRMResourceProvider -ProviderNamespace Microsoft.AzureStack
   ```

5. 管理者として PowerShell ISE を起動し、Azure Stack Hub ツールをダウンロードしたときに作成された **AzureStack-Tools-az** ディレクトリ内の **Registration** フォルダーに移動します。 PowerShell を使用して **RegisterWithAzure.psm1** モジュールをインポートします。

   ```powershell  
   Import-Module .\RegisterWithAzure.psm1
   ```

6. 次に、同じ PowerShell セッションで、正しい Azure PowerShell コンテキストにサインインしていることを確認します。 このコンテキストでは、前に Azure Stack Hub リソース プロバイダーを登録するために使用された Azure アカウントです。 実行する PowerShell:

   ```powershell  
   Connect-AzureRMAccount -Environment "<environment name>"
   ```

   | パラメーター | 説明 |  
   |-----|-----|
   | EnvironmentName | Azure クラウド サブスクリプション環境名。 サポートされている環境名は **AzureCloud**、**AzureUSGovernment**、または中国の Azure サブスクリプションを使用している場合は **AzureChinaCloud** です。  |

7. 同じ PowerShell セッションで **Set-AzsRegistration** コマンドレットを実行します。 実行する PowerShell:  

   ```powershell  
   $CloudAdminCred = Get-Credential -UserName <Privileged endpoint credentials> -Message "Enter the cloud domain credentials to access the privileged endpoint."
   $RegistrationName = "<unique-registration-name>"
   Set-AzsRegistration `
      -PrivilegedEndpointCredential $CloudAdminCred `
      -PrivilegedEndpoint <PrivilegedEndPoint computer name> `
      -BillingModel PayAsYouUse `
      -RegistrationName $RegistrationName
   ```
   Set-AzsRegistration コマンドレットの詳細については、「[登録に関するリファレンス](#registration-reference)」を参照してください。

---

   このプロセスには 10 - 15 分かかります。 コマンドが完了すると、「**Your environment is now registered and activated using the provided parameters. (提供されたパラメーターを使用して環境が登録され、アクティブ化されました。)** 」というメッセージが表示されます。

## <a name="register-with-capacity-billing"></a>容量課金で登録する

容量課金モデルを使用して Azure Stack Hub を Azure に登録するには、次の手順を使用します。

> [!Note]  
> これらのすべての手順は、特権エンドポイント (PEP) にアクセスできるコンピューターから実行する必要があります。 PEP の詳細については、[Azure Stack Hub での特権エンドポイントの使用](azure-stack-privileged-endpoint.md)に関するページを参照してください。

接続された環境では、インターネットと Azure にアクセスできます。 これらの環境では、Azure Stack Hub リソース プロバイダーを Azure に登録してから、課金モデルを構成する必要があります。

### <a name="az-modules"></a>[Az モジュール](#tab/az2)

1. Azure Stack Hub リソース プロバイダーを Azure に登録するには、PowerShell ISE を管理者として起動し、**EnvironmentName** パラメーターを適切な Azure サブスクリプション タイプ (以下のパラメーターを参照) に設定して、次の PowerShell コマンドレットを使用します。

2. Azure Stack Hub を登録するために使用した Azure アカウントを追加します。 アカウントを追加するには、**Connect-AzAccount** コマンドレットを実行します。 Azure アカウント資格情報の入力を求められ、お使いのアカウントの構成によっては 2 要素認証を使用する必要があります。

   ```powershell  
   Connect-AzAccount -Environment "<environment name>"
   ```

   | パラメーター | 説明 |  
   |-----|-----|
   | EnvironmentName | Azure クラウド サブスクリプション環境名。 サポートされている環境名は **AzureCloud**、**AzureUSGovernment**、または中国の Azure サブスクリプションを使用している場合は **AzureChinaCloud** です。  |

3. 複数のサブスクリプションがある場合は、次のコマンドを実行して、使用するものを選択します。  

   ```powershell  
   Get-AzSubscription -SubscriptionID '<Your Azure Subscription GUID>' | Select-AzSubscription
   ```

4. Azure サブスクリプションで Azure Stack Hub リソース プロバイダーを登録するには、次のコマンドを実行します。

   ```powershell  
   Register-AzResourceProvider -ProviderNamespace Microsoft.AzureStack
   ```

5. 管理者として PowerShell ISE を起動し、Azure Stack Hub ツールをダウンロードしたときに作成された **AzureStack-Tools-az** ディレクトリ内の **Registration** フォルダーに移動します。 PowerShell を使用して **RegisterWithAzure.psm1** モジュールをインポートします。

   ```powershell  
   $CloudAdminCred = Get-Credential -UserName <Privileged endpoint credentials> -Message "Enter the cloud domain credentials to access the privileged endpoint."
   $RegistrationName = "<unique-registration-name>"
   Set-AzsRegistration `
      -PrivilegedEndpointCredential $CloudAdminCred `
      -PrivilegedEndpoint <PrivilegedEndPoint computer name> `
      -AgreementNumber <EA agreement number> `
      -BillingModel Capacity `
      -RegistrationName $RegistrationName
   ```

    容量 SKU ライセンスを購入したときの "*EA 契約番号*" を使用します。

   > [!Note]  
   > **Set-AzsRegistration** コマンドレットの UsageReportingEnabled パラメーターを false に設定することで、使用状況レポートを無効にすることができます。 
   
   Set-AzsRegistration コマンドレットの詳細については、「[登録に関するリファレンス](#registration-reference)」を参照してください。

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm2)

1. Azure Stack Hub リソース プロバイダーを Azure に登録するには、PowerShell ISE を管理者として起動し、**EnvironmentName** パラメーターを適切な Azure サブスクリプション タイプ (以下のパラメーターを参照) に設定して、次の PowerShell コマンドレットを使用します。

2. Azure Stack Hub を登録するために使用した Azure アカウントを追加します。 アカウントを追加するには、**Add-AzureRMAccount** コマンドレットを実行します。 Azure アカウント資格情報の入力を求められ、お使いのアカウントの構成によっては 2 要素認証を使用する必要があります。

   ```powershell  
   Connect-AzureRMAccount -Environment "<environment name>"
   ```

   | パラメーター | 説明 |  
   |-----|-----|
   | EnvironmentName | Azure クラウド サブスクリプション環境名。 サポートされている環境名は **AzureCloud**、**AzureUSGovernment**、または中国の Azure サブスクリプションを使用している場合は **AzureChinaCloud** です。  |

3. 複数のサブスクリプションがある場合は、次のコマンドを実行して、使用するものを選択します。  

   ```powershell  
   Get-AzureRMSubscription -SubscriptionID '<Your Azure Subscription GUID>' | Select-AzureRMSubscription
   ```

4. Azure サブスクリプションで Azure Stack Hub リソース プロバイダーを登録するには、次のコマンドを実行します。

   ```powershell  
   Register-AzureRMResourceProvider -ProviderNamespace Microsoft.AzureStack
   ```

5. 管理者として PowerShell ISE を起動し、Azure Stack Hub ツールをダウンロードしたときに作成された **AzureStack-Tools-master** ディレクトリ内の **Registration** フォルダーに移動します。 PowerShell を使用して **RegisterWithAzure.psm1** モジュールをインポートします。

   ```powershell  
   $CloudAdminCred = Get-Credential -UserName <Privileged endpoint credentials> -Message "Enter the cloud domain credentials to access the privileged endpoint."
   $RegistrationName = "<unique-registration-name>"
   Set-AzsRegistration `
      -PrivilegedEndpointCredential $CloudAdminCred `
      -PrivilegedEndpoint <PrivilegedEndPoint computer name> `
      -AgreementNumber <EA agreement number> `
      -BillingModel Capacity `
      -RegistrationName $RegistrationName
   ```

    容量 SKU ライセンスを購入したときの "*EA 契約番号*" を使用します。

   > [!Note]  
   > **Set-AzsRegistration** コマンドレットの UsageReportingEnabled パラメーターを false に設定することで、使用状況レポートを無効にすることができます。 
   
   Set-AzsRegistration コマンドレットの詳細については、「[登録に関するリファレンス](#registration-reference)」を参照してください。

---

::: zone-end

::: zone pivot="state-disconnected"
## <a name="register-with-capacity-billing"></a>容量課金で登録する

接続されていない環境 (インターネットに接続されていない環境) に Azure Stack Hub を登録する場合は、Azure Stack Hub 環境から登録トークンを取得する必要があります。 次に、Azure に接続でき、PowerShell for Azure Stack Hub がインストールされているコンピューターでそのトークンを使用します。  

### <a name="get-a-registration-token-from-the-azure-stack-hub-environment"></a>Azure Stack Hub 環境から登録トークンを取得する

1. 管理者として PowerShell ISE を起動し、Azure Stack Hub ツールをダウンロードしたときに作成された **AzureStack-Tools-az** ディレクトリ内の **Registration** フォルダーに移動します。 **RegisterWithAzure.psm1** モジュールをインポートします。  

   ```powershell  
   Import-Module .\RegisterWithAzure.psm1
   ```

2. 登録トークンを取得するには、次の PowerShell コマンドレットを実行します。  

   ```Powershell
   $FilePathForRegistrationToken = "$env:SystemDrive\RegistrationToken.txt"
   $RegistrationToken = Get-AzsRegistrationToken -PrivilegedEndpointCredential $YourCloudAdminCredential -UsageReportingEnabled:$False -PrivilegedEndpoint $YourPrivilegedEndpoint -BillingModel Capacity -AgreementNumber '<EA agreement number>' -TokenOutputFilePath $FilePathForRegistrationToken
   ```

   容量 SKU ライセンスを購入したときの "*EA 契約番号*" を使用します。

   Get-AzsRegistrationToken コマンドレットの詳細については、「[登録に関するリファレンス](#registration-reference)」を参照してください。

   > [!Tip]  
   > 登録トークンは、 *$FilePathForRegistrationToken* に指定されたファイルに保存されます。 ファイル パスまたはファイル名は任意に変更できます。

3. 接続された Azure マシンで使用するためにこの登録トークンを保存します。 *$FilePathForRegistrationToken* からファイルまたはテキストをコピーできます。

### <a name="connect-to-azure-and-register"></a>Azure に接続して登録する

インターネットに接続されたコンピューターで同じ手順を実行して、RegisterWithAzure.psm1 モジュールをインポートし、正しい Azure PowerShell コンテキストにサインインします。 次に、Register-AzsEnvironment を呼び出します。 Azure に登録する登録トークンを指定します。 同じ Azure サブスクリプション ID を利用して Azure Stack Hub の複数のインスタンスを登録している場合は、一意の登録名を指定します。

自分の登録トークンと、一意のトークン名が必要です。

1. 管理者として PowerShell ISE を起動し、Azure Stack Hub ツールをダウンロードしたときに作成された **AzureStack-Tools-az** ディレクトリ内の **Registration** フォルダーに移動します。 **RegisterWithAzure.psm1** モジュールをインポートします。  

   ```powershell  
   Import-Module .\RegisterWithAzure.psm1
   ```

2. その後、PowerShell コマンドレットを実行します。  

    ```powershell  
    $RegistrationToken = "<Your Registration Token>"
    $RegistrationName = "<unique-registration-name>"
    Register-AzsEnvironment -RegistrationToken $RegistrationToken -RegistrationName $RegistrationName
    ```

オプションで、Get-Content コマンドレットを使用して、登録トークンが含まれているファイルを示すことができます。

自分の登録トークンと、一意のトークン名が必要です。

1. 管理者として PowerShell ISE を起動し、Azure Stack Hub ツールをダウンロードしたときに作成された **AzureStack-Tools-az** ディレクトリ内の **Registration** フォルダーに移動します。 **RegisterWithAzure.psm1** モジュールをインポートします。  

    ```powershell  
    Import-Module .\RegisterWithAzure.psm1
    ```

2. その後、PowerShell コマンドレットを実行します。  

    ```powershell  
    $RegistrationToken = Get-Content -Path '<Path>\<Registration Token File>'
    Register-AzsEnvironment -RegistrationToken $RegistrationToken -RegistrationName $RegistrationName
    ```

  > [!Note]  
  > 後で参照できるように登録リソース名および登録トークンを保存します。

### <a name="retrieve-an-activation-key-from-azure-registration-resource"></a>Azure の登録リソースからアクティブ化キーを取得する

次に、Register-AzsEnvironment 中に Azure で作成された登録リソースからアクティブ化キーを取得する必要があります。

アクティブ化キーを取得するには、次の PowerShell コマンドレットを実行します。  

  ```Powershell
  $RegistrationResourceName = "<unique-registration-name>"
  $KeyOutputFilePath = "$env:SystemDrive\ActivationKey.txt"
  $ActivationKey = Get-AzsActivationKey -RegistrationName $RegistrationResourceName -KeyOutputFilePath $KeyOutputFilePath
  ```

  > [!Tip]  
  > アクティブ化キーは *$KeyOutputFilePath*　に指定されたファイルに保存されます。 ファイル パスまたはファイル名は任意に変更できます。

### <a name="create-an-activation-resource-in-azure-stack-hub"></a>Azure Stack Hub にアクティブ化リソースを作成する

Get-AzsActivationKey から作成されたアクティブ化キーのファイルまたはテキストを使用して Azure Stack Hub 環境に戻ります。 次に、アクティブ化キーを使用して Azure Stack Hub にアクティブ化リソースを作成します。 アクティブ化リソースを作成するには、次の PowerShell コマンドレットを実行します。

  ```Powershell
  $ActivationKey = "<activation key>"
  New-AzsActivationResource -PrivilegedEndpointCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint -ActivationKey $ActivationKey
  ```

オプションで、Get-Content コマンドレットを使用して、登録トークンが含まれているファイルを示すことができます。

  ```Powershell
  $ActivationKey = Get-Content -Path '<Path>\<Activation Key File>'
  New-AzsActivationResource -PrivilegedEndpointCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint -ActivationKey $ActivationKey
  ```
::: zone-end

## <a name="verify-azure-stack-hub-registration"></a>Azure Stack Hub の登録を検証する

Azure Stack Hub の登録に成功したことは、 **[Region management]\(リージョン管理\)** タイルを使用して確認できます。 このタイルは、管理者ポータルの既定のダッシュボードにあります。 状態には、登録済みと未登録とがあります。 登録済みである場合は、Azure Stack Hub の登録に使用した Azure サブスクリプション ID が、登録のリソース グループおよび名前と共に表示されます。

1. Azure Stack Hub 管理者ポータル `https://adminportal.local.azurestack.external` にサインインします。

2. ダッシュボードで、 **[Region management]\(リージョン管理\)** を選択します。

3. **[プロパティ]** を選択します。 このブレードには、環境の状態と詳細が表示されます。 状態には、 **[登録済み]** 、 **[未登録]** または **[有効期限切れ]** があります。

    [![Azure Stack Hub 管理者ポータルの [リージョンの管理] タイル](media/azure-stack-registration/admin1sm.png "[Region management]\(リージョン管理\) タイル")](media/azure-stack-registration/admin1.png#lightbox)

    登録した場合、プロパティには以下が含まれます。
    
    - **登録サブスクリプション ID**:登録され、Azure Stack Hub に関連付けられた Azure サブスクリプション ID。
    - **登録リソース グループ**:Azure Stack Hub リソースを含む関連付けられているサブスクリプション内の Azure リソース グループ。

4. Azure portal を使用して Azure Stack Hub 登録リソースを表示し、登録が成功したことを確認できます。 Azure Stack Hub の登録に使用したサブスクリプションに関連付けられているアカウントを使用して、[Azure portal](https://portal.azure.com) にサインインします。 **[すべてのリソース]** を選択し、 **[非表示の型の表示]** チェックボックスをオンにして、登録名を選択します。

5. 登録が成功しなかった場合、この問題を解決するには、[こちらの手順](#change-the-subscription-you-use)に従って再登録する必要があります。  

また、登録が成功したかどうかは、Marketplace の管理機能を使用して確認することもできます。 [Marketplace management]\(Marketplace の管理\) ブレードに Marketplace 項目の一覧が表示されれば、登録は成功しています。 ただし、切断されている環境では、[Marketplace management]\(Marketplace の管理\) に Marketplace 項目は表示されません。

> [!NOTE]
> 登録が完了すると、登録されていないことを示すアクティブな警告は表示されなくなります。 1904 より前の Azure Stack Hub リリースの場合、接続されていないシナリオでは、正常に登録が完了しても、Azure Stack Hub を登録してアクティブ化するよう求めるメッセージが [Marketplace management]\(Marketplace の管理\) に表示されます。 このメッセージは、リリース 1904 以降では表示されません。

## <a name="renew-or-change-registration"></a>登録を更新または変更する

::: zone pivot="state-connected"
次の状況では、登録を更新する必要があります。

- 容量ベースの年単位サブスクリプションを更新した後。
- 課金モデルを変更したとき。
- 容量ベースの課金のために変更を調整したとき (ノードの追加/削除)。

### <a name="prerequisites"></a>前提条件

登録を更新または変更するには、[管理者ポータル](#verify-azure-stack-hub-registration)の次の情報が必要です。

| 管理者ポータル | コマンドレット パラメーター | Notes | 
|-----|-----|-----|
| 登録サブスクリプション ID | サブスクリプション | 以前の登録時に使用されたサブスクリプション ID |
| 登録リソース グループ | ResourceGroupName | 以前の登録リソースが存在するリソース グループ |
| 登録名 | RegistrationName | 以前の登録時に使用された登録名 |

### <a name="change-the-subscription-you-use"></a>使用するサブスクリプションを変更する

使用するサブスクリプションを変更する場合は、まず **Remove-AzsRegistration** コマンドレットを実行します。その後、正しい Azure PowerShell コンテキストにサインインしていることを確認します。 次に、`<billing model>` を含む変更されたパラメーターを使用して **Set-AzsRegistration** を実行します。 **Remove-AzsRegistration** の実行中、登録時に使用したサブスクリプションにサインインし、[管理者ポータル](#verify-azure-stack-hub-registration)に表示された `RegistrationName` パラメーターと `ResourceGroupName` パラメーターの値を使用する必要があります。

### <a name="az-modules"></a>[Az モジュール](#tab/az3)

```powershell  
# select the subscription used during the registration (shown in portal)
Select-AzSubscription -Subscription '<Registration subscription ID from portal>'
# unregister using the parameter values from portal
Remove-AzsRegistration -PrivilegedEndpointCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint -RegistrationName '<Registration name from portal>' -ResourceGroupName '<Registration resource group from portal>'
# switch to new subscription id
Select-AzSubscription -Subscription '<New subscription ID>'
# register 
Set-AzsRegistration -PrivilegedEndpointCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint -BillingModel '<Billing model>' -RegistrationName '<Registration name>' -ResourceGroupName '<Registration resource group name>'
```

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm3)

```powershell  
# select the subscription used during the registration (shown in portal)
Select-AzureRMSubscription -Subscription '<Registration subscription ID from portal>'
# unregister using the parameter values from portal
Remove-AzsRegistration -PrivilegedEndpointCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint -RegistrationName '<Registration name from portal>' -ResourceGroupName '<Registration resource group from portal>'
# switch to new subscription id
Select-AzureRMSubscription -Subscription '<New subscription ID>'
# register 
Set-AzsRegistration -PrivilegedEndpointCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint -BillingModel '<Billing model>' -RegistrationName '<Registration name>' -ResourceGroupName '<Registration resource group name>'
```

---

### <a name="change-billing-model-how-features-are-offered-or-re-register-your-instance"></a>課金モデルと機能の提供方法の変更、インスタンスの再登録

このセクションは、課金モデルや機能を提供する方法を変更する場合、またはインスタンスを再登録する場合に適用されます。 これらのすべてのケースで、登録関数を呼び出して新しい値を設定します。 最初に現在の登録を削除する必要はありません。 [管理者ポータル](#verify-azure-stack-hub-registration)に表示されているサブスクリプション ID にサインインし、`RegistrationName` パラメーターと `ResourceGroupName` パラメーターの値は[管理者ポータル](#verify-azure-stack-hub-registration)に表示された値のままで、新しい `BillingModel` の値を指定して登録を再実行します。

### <a name="az-modules"></a>[Az モジュール](#tab/az4)

```powershell  
# select the subscription used during the registration
Select-AzSubscription -Subscription '<Registration subscription ID from portal>'
# rerun registration with new BillingModel (or same billing model in case of re-registration) but using other parameters values from portal
Set-AzsRegistration -PrivilegedEndpointCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint -BillingModel '<New billing model>' -RegistrationName '<Registration name from portal>' -ResourceGroupName '<Registration resource group from portal>'
```

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm4)

```powershell  
# select the subscription used during the registration
Select-AzureRMSubscription -Subscription '<Registration subscription ID from portal>'
# rerun registration with new BillingModel (or same billing model in case of re-registration) but using other parameters values from portal
Set-AzsRegistration -PrivilegedEndpointCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint -BillingModel '<New billing model>' -RegistrationName '<Registration name from portal>' -ResourceGroupName '<Registration resource group from portal>'
```

---

::: zone-end

::: zone pivot="state-disconnected"
次の状況では、登録を更新または変更する必要があります。

- 容量ベースの年単位サブスクリプションを更新した後。
- 課金モデルを変更したとき。
- 容量ベースの課金のために変更を調整したとき (ノードの追加/削除)。

### <a name="remove-the-activation-resource-from-azure-stack-hub"></a>Azure Stack Hub からアクティブ化リソースを削除する

最初に Azure Stack Hub からアクティブ化リソースを削除してから、Azure の登録リソースを削除する必要があります。  

Azure Stack Hub のアクティブ化リソースを削除するには、Azure Stack Hub 環境で次の PowerShell コマンドレットを実行します。  

  ```powershell
  Remove-AzsActivationResource -PrivilegedEndpointCredential $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint
  ```

次に、Azure の登録リソースを削除するには、Azure に接続されたコンピューターを使用していることを確認して、正しい Azure PowerShell コンテキストにサインインし、以下の説明に従って、適切な PowerShell コマンドレットを実行します。

リソースの作成に使用した登録トークンを使用できます。  

  ```powershell
  $RegistrationToken = "<registration token>"
  Unregister-AzsEnvironment -RegistrationToken $RegistrationToken
  ```

または、[管理者ポータル](#verify-azure-stack-hub-registration)の登録名と登録リソース グループ名を使用することもできます。

  ```powershell
  Unregister-AzsEnvironment -RegistrationName '<Registration name from portal>' -ResourceGroupName '<Registration resource group from portal>'
  ```

### <a name="re-register-using-connected-steps"></a>接続された手順を使用して再登録する

課金モデルを切断状態の容量課金から接続された状態での従量課金に変更する場合は、[接続されたモデルの手順](azure-stack-registration.md?pivots=state-connected#change-billing-model-how-features-are-offered-or-re-register-your-instance)に従って再登録します。 

>[!Note] 
>これによって ID モデルが変更されるわけではなく、課金メカニズムのみが変更されます。ID ソースとして AD FS が引き続き使用されます。

### <a name="re-register-using-disconnected-steps"></a>切り離された手順を使用して再登録する

これで、接続が切断されたシナリオで登録が完全に解除されました。接続が切断されたシナリオで Azure Stack Hub 環境を登録する手順を繰り返す必要があります。
::: zone-end

### <a name="disable-or-enable-usage-reporting"></a>使用状況レポートを無効または有効にする

容量課金モデルを使用する Azure Stack Hub 環境では、**Set-AzsRegistration** または **Get-AzsRegistrationToken** コマンドレットのいずれかで **UsageReportingEnabled** パラメーターを使用して使用状況レポートをオフにします。 Azure Stack Hub では、既定で使用状況メトリックがレポートされます。 容量モデルを使用するオペレーターまたは切断された環境をサポートするオペレーターは、使用状況レポートをオフにする必要があります。

::: zone pivot="state-connected"
次の PowerShell コマンドレットを実行します。

   ```powershell  
   $CloudAdminCred = Get-Credential -UserName <Privileged endpoint credentials> -Message "Enter the cloud domain credentials to access the privileged endpoint."
   $RegistrationName = "<unique-registration-name>"
   Set-AzsRegistration `
      -PrivilegedEndpointCredential $CloudAdminCred `
      -PrivilegedEndpoint <PrivilegedEndPoint computer name> `
      -BillingModel Capacity
      -RegistrationName $RegistrationName
      -UsageReportingEnabled:$false
   ```
::: zone-end
::: zone pivot="state-disconnected"
1. 登録トークンを変更するには、次の PowerShell コマンドレットを実行します。  

   ```Powershell
   $FilePathForRegistrationToken = $env:SystemDrive\RegistrationToken.txt
   $RegistrationToken = Get-AzsRegistrationToken -PrivilegedEndpointCredential -UsageReportingEnabled:$False
   $YourCloudAdminCredential -PrivilegedEndpoint $YourPrivilegedEndpoint -BillingModel Capacity -AgreementNumber '<EA agreement number>' -TokenOutputFilePath $FilePathForRegistrationToken
   ```

    容量 SKU ライセンスを購入したときの "*EA 契約番号*" を使用します。

   > [!Tip]  
   > 登録トークンは、 *$FilePathForRegistrationToken* に指定されたファイルに保存されます。 ファイル パスまたはファイル名は任意に変更できます。

2. 接続された Azure マシンで使用するためにこの登録トークンを保存します。 *$FilePathForRegistrationToken* からファイルまたはテキストをコピーできます。
::: zone-end

## <a name="move-a-registration-resource"></a>登録リソースを移動する

同じサブスクリプションのリソース グループ間における登録リソースの移動は、すべての環境で **サポートされています**。 ただし、サブスクリプションをまたがる登録リソースの移動がサポートされるのは、両方のサブスクリプションで同じパートナー ID が解決される場合の CSP のみです。 新しいリソース グループへのリソースの移動については、「[新しいリソース グループまたはサブスクリプションへのリソースの移動](/azure/azure-resource-manager/resource-group-move-resources)」を参照してください。

> [!IMPORTANT]
> ポータルで登録リソースが誤って削除されないように、登録スクリプトではリソースに自動的にロックが追加されます。 移動または削除する前に、このロックを解除する必要があります。 誤って削除されないように、登録リソースにロックを追加することをお勧めします。

## <a name="registration-reference"></a>登録に関するリファレンス

### <a name="set-azsregistration"></a>Set-AzsRegistration

**Set-AzsRegistration** を使用すると、Azure Stack Hub を Azure に登録し、マーケットプレースでの項目のオファーと使用状況レポートを有効または無効にすることができます。

コマンドレットを実行するには、以下が必要です。

- 任意の種類のグローバルな Azure サブスクリプション。
- そのサブスクリプションの所有者または共同作成者であるアカウントを使用して Azure PowerShell にログインします。

```powershell
Set-AzsRegistration [-PrivilegedEndpointCredential] <PSCredential> [-PrivilegedEndpoint] <String> [[-AzureContext]
    <PSObject>] [[-ResourceGroupName] <String>] [[-ResourceGroupLocation] <String>] [[-BillingModel] <String>]
    [-MarketplaceSyndicationEnabled] [-UsageReportingEnabled] [[-AgreementNumber] <String>] [[-RegistrationName]
    <String>] [<CommonParameters>]
```

| パラメーター | Type | 説明 |
|-------------------------------|--------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| PrivilegedEndpointCredential | PSCredential | [特権エンドポイントへのアクセス](azure-stack-privileged-endpoint.md#access-the-privileged-endpoint)に使用する資格情報。 ユーザー名は、**AzureStackDomain\CloudAdmin** の形式です。 |
| PrivilegedEndpoint | String | ログ収集およびその他のデプロイ後タスクのような機能を提供する、あらかじめ構成されたリモート PowerShell コンソール。 詳細については、[特権エンドポイントの使用](azure-stack-privileged-endpoint.md#access-the-privileged-endpoint)の記事を参照してください。 |
| AzureContext | PSObject |  |
| ResourceGroupName | String |  |
| ResourceGroupLocation | String |  |
| BillingModel | String | 自分のサブスクリプションで使用する請求モデル。 このパラメーターの有効値は、Capacity、PayAsYouUse、および Development です。 |
| MarketplaceSyndicationEnabled | True または False | ポータル内で Marketplace 管理機能を使用できるようにするかどうかを決定します。 インターネット接続を使用して登録している場合は true に設定します。 切断された環境で登録している場合は false に設定します。 切断された環境で登録した場合は、[オフライン シンジケーション ツール](azure-stack-download-azure-marketplace-item.md?pivots=state-disconnected)を使用して、マーケットプレースの項目をダウンロードできます。 |
| UsageReportingEnabled | True または False | Azure Stack Hub では、既定で使用状況メトリックがレポートされます。 容量モデルを使用するオペレーターまたは切断された環境をサポートするオペレーターは、使用状況レポートをオフにする必要があります。 このパラメーターの有効値は、True、False。 |
| AgreementNumber | String | この Azure Stack の容量 SKU が注文された EA 契約の番号。 |
| RegistrationName | String | 同じ Azure サブスクリプション ID を利用し、複数の Azure Stack Hub インスタンスで登録スクリプトを実行している場合、登録に一意の名前を設定します。 このパラメーターの既定値は **AzureStackRegistration** です。 ただし、複数の Azure Stack Hub インスタンスに同じ名前を使用すると、スクリプトは失敗します。 |

### <a name="get-azsregistrationtoken"></a>Get-AzsRegistrationToken

Get-AzsRegistrationToken は、入力パラメーターから登録トークンを生成します。

```powershell  
Get-AzsRegistrationToken [-PrivilegedEndpointCredential] <PSCredential> [-PrivilegedEndpoint] <String>
    [-BillingModel] <String> [[-TokenOutputFilePath] <String>] [-UsageReportingEnabled] [[-AgreementNumber] <String>]
    [<CommonParameters>]
```
| パラメーター | Type | 説明 |
|-------------------------------|--------------|-------------|
| PrivilegedEndpointCredential | PSCredential | [特権エンドポイントへのアクセス](azure-stack-privileged-endpoint.md#access-the-privileged-endpoint)に使用する資格情報。 ユーザー名は、**AzureStackDomain\CloudAdmin** の形式です。 |
| PrivilegedEndpoint | String |  ログ収集およびその他のデプロイ後タスクのような機能を提供する、あらかじめ構成されたリモート PowerShell コンソール。 詳細については、[特権エンドポイントの使用](azure-stack-privileged-endpoint.md#access-the-privileged-endpoint)の記事を参照してください。 |
| AzureContext | PSObject |  |
| ResourceGroupName | String |  |
| ResourceGroupLocation | String |  |
| BillingModel | String | 自分のサブスクリプションで使用する請求モデル。 このパラメーターの有効値は、Capacity、PayAsYouUse、および Development です。 |
| MarketplaceSyndicationEnabled | True または False |  |
| UsageReportingEnabled | True または False | Azure Stack Hub では、既定で使用状況メトリックがレポートされます。 容量モデルを使用するオペレーターまたは切断された環境をサポートするオペレーターは、使用状況レポートをオフにする必要があります。 このパラメーターの有効値は、True、False。 |
| AgreementNumber | String |  |

## <a name="registration-failures"></a>登録エラー

Azure Stack Hub の登録を試みている間に、次のエラーのいずれかが発生する可能性があります。

- `$hostName` の必須ハードウェア情報を取得できませんでした。 物理ホストと接続を確認してから、登録を再実行してください。

- ハードウェア情報を取得するために `$hostName` に接続できません。 物理ホストと接続を確認してから、登録を再実行してください。

   原因: これは通常、アクティブ化を試みるためにホストから UUID、BIOS、CPU などのハードウェア詳細を取得しようとしたが、物理ホストに接続できないことでハードウェア詳細を取得不能だったことが原因です。

- Cloud identifier [`GUID`] is already registered. (クラウド識別子 [`GUID`] は既に登録されています。) Reusing cloud identifiers is not allowed. (クラウド識別子の再使用は許可されません。)

   原因: これは、Azure Stack 環境が既に登録されている場合に発生します。 別のサブスクリプションまたは課金モデルを使用して環境を再登録する場合は、[登録を更新または変更する手順](#renew-or-change-registration)に従ってください。

- Marketplace の管理にアクセスしして、製品を配信しようとするとエラーが発生します。

   原因: これは通常、Azure Stack Hub が登録リソースにアクセスできない場合に発生します。 これに関する一般的な理由の 1 つとして、Azure サブスクリプションのディレクトリ テナントが変更されると登録がリセットされることがあげられます。 サブスクリプションのディレクトリ テナントを変更した場合、Azure Stack Hub Marketplace へのアクセスおよび使用状況のレポートを行うことはできません。 この問題を解決するには再登録する必要があります。
::: zone pivot="state-disconnected"
- 切断されたプロセスを使用して既にスタンプを登録している場合でも、Marketplace の管理から、Azure Stack Hub の登録とアクティブ化を要求されます。

   原因: これは、切断された環境に関する既知の問題であり、[登録状態を確認](#verify-azure-stack-hub-registration)する必要があります。 Marketplace 管理を使用するには、[オフライン ツール](azure-stack-download-azure-marketplace-item.md?pivots=state-disconnected)を使用します。
::: zone-end

## <a name="next-steps"></a>次のステップ

[Azure から Marketplace の項目をダウンロードする](azure-stack-download-azure-marketplace-item.md)
