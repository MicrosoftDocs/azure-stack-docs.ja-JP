---
title: Azure Stack Hub で Azure Web アプリのデプロイ アクションを使用する
description: Azure Web アプリのデプロイ アクションを使用して、Azure Stack Hub への継続的インテグレーション、継続的デプロイ (CI/CD) ワークフローを作成します
author: mattbriggs
ms.topic: how-to
ms.date: 2/16/2021
ms.author: mabrigg
ms.reviewer: gara
ms.lastreviewed: 2/16/2021
ms.openlocfilehash: fec9acb7a3e156a8646aef88c53e681eed9e53da
ms.sourcegitcommit: 4c97ed2caf054ebeefa94da1f07cfb6be5929aac
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2021
ms.locfileid: "100655276"
---
# <a name="use-the-azure-web-app-deploy-action-with-azure-stack-hub"></a>Azure Stack Hub で Azure Web アプリのデプロイ アクションを使用する

Web アプリを Azure Stack Hub インスタンスにデプロイするように GitHub Actions を設定することができます。 これにより、アプリの継続的インテグレーションおよびデプロイを設定できるようになります。 この記事は、GitHub Actions と Azure Stack Hub を使用した自動デプロイを初めて実行する場合に役立ちます。 Web アプリを作成し、発行プロファイルを使用して、アプリをホストする Web アプリを作成します。

GitHub Actions は、コード リポジトリ内で自動化を有効にするアクションで構成されるワークフローです。 GitHub 開発プロセスにおいてイベントを含むワークフローをトリガーできます。 テスト、デプロイ、継続的インテグレーションなど、一般的な DevOps 自動化タスクを設定できます。

このワークフロー例には以下が含まれます。
- サービス プリンシパル (SPN) を作成および検証する手順。
- Web アプリ発行プロファイルの作成手順
- ランタイム固有のワークフローの追加
- Web アプリのデプロイでの照合ワークフローの追加
## <a name="get-service-principal"></a>サービス プリンシパルの取得

Azure 外部のプロセスでリソースに接続したり、リソースとやり取りしたりするために、SPN によってロールベースの資格情報が提供されます。 GitHub Actions で使用するために、共同作成者アクセス権を含む SPN と、以下の手順で指定される属性が必要になります。

Azure Stack Hub のユーザーには、SPN を作成するアクセス許可はありません。 このプリンシパルはクラウド オペレーターに要求する必要があります。 ここでは、クラウド オペレーターの場合に SPN を作成できるように手順を説明しています。 また、開発者の場合は、クラウド オペレーターから提供された SPN を検証できます。

クラウド オペレーターは Azure CLI を使用して SPN を作成する必要があります。

次のコード スニペットは、Azure CLI で PowerShell プロンプトを使用する Windows マシン用に作成されています。 Linux コンピューターと bash で CLI を使用している場合は、行継続を削除するか、`\` に置き換えます。

1. SPN の作成に使用する次のパラメーターの値を準備します。

    | パラメーター | 例 | 説明 |
    | --- | --- | --- |
    endpoint-resource-manager | "https://management.orlando.azurestack.corp.microsoft.com"  | リソース管理エンドポイント。 |
    suffix-storage-endpoint | "orlando.azurestack.corp.microsoft.com"  | ストレージ アカウントのエンドポイント サフィックス。 |
    suffix-keyvault-dns | ".vault.orlando.azurestack.corp.microsoft.com"  | Key Vault サービス dns サフィックス。 |
    endpoint-active-directory-graph-resource-id | "https://graph.windows.net/"  | Active Directory リソース ID。 |
    endpoint-sql-management | https://notsupported  | SQL サーバー管理エンドポイント。 これを `https://notsupported` |
    プロファイル | 2019-03-01-hybrid | このクラウドで使用するプロファイル。 |

2. Windows PowerShell や Bash などのコマンドライン ツールを開き、サインインします。 次のコマンドを使用します。

    ```azurecli  
    az login
    ```

3. 新しい環境の場合は `register` コマンドを使用し、既存の環境を使用する場合は `update` コマンドを使用します。 次のコマンドを使用します。

    ```azurecli  
    az cloud register `
        -n "AzureStackUser" `
        --endpoint-resource-manager "https://management.<local>.<FQDN>" `
        --suffix-storage-endpoint ".<local>.<FQDN>" `
        --suffix-keyvault-dns ".vault.<local>.<FQDN>" `
        --endpoint-active-directory-graph-resource-id "https://graph.windows.net/" `
        --endpoint-sql-management https://notsupported  `
        --profile 2019-03-01-hybrid
    ```

4. SPN に使用するサブスクリプション ID とリソース グループを取得します。

5. サブスクリプション ID とリソース グループを使用して、次のコマンドで SPN を作成します。

    ```azurecli  
    az ad sp create-for-rbac --name "myApp" --role contributor `
        --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} `
        --sdk-auth
    ```

    クラウド オペレーターの特権がない場合は、クラウド オペレーターから提供された SPN を使用してサインインすることもできます。 クライアント ID、シークレット、およびテナント ID が必要になります。 これらの値を使用し、次の Azure CLI コマンドを使用して、GitHub リポジトリにシークレットとして追加できる JSON オブジェクトを作成できます。

    ```azurecli  
    az login --service-principal -u "<client-id>" -p "<secret>" --tenant "<tenant-ID>" --allow-no-subscriptions
    az account show --sdk-auth
    ```

6. 結果の JSON オブジェクトを確認します。 JSON オブジェクトを使用して、アクションが含まれる GitHub リポジトリにシークレットを作成します。 JSON オブジェクトには、次の属性が必要です。

    ```json
    {
      "clientId": <Application ID for the SPN>,
      "clientSecret": <Client secret for the SPN>,
      "subscriptionId": <Subscription ID for the SPN>,
      "tenantId": <Tenant ID for the SPN>,
      "activeDirectoryEndpointUrl": "https://login.microsoftonline.com/",
      "resourceManagerEndpointUrl": "https://management.<FQDN>",
      "activeDirectoryGraphResourceId": "https://graph.windows.net/",
      "sqlManagementEndpointUrl": "https://notsupported",
      "galleryEndpointUrl": "https://providers.<FQDN>:30016/",
      "managementEndpointUrl": "https://management.<FQDN>"
    }
    ```

## <a name="create-the-web-app-publish-profile"></a>Web アプリの発行プロファイルを作成する

### <a name="open-the-create-web-app-blade"></a>[Web アプリの作成] ブレードを開く

1. Azure Stack Hub ポータルにサインインします。
1. **[リソースの作成]**  >  **[Web + モバイル]**  >  **[Web アプリ]** を選択します。
    ![Azure Stack Hub で Web アプリを作成する](\media\ci-cd-github-action-webapp\create-web-app.png)
### <a name="to-create-your-web-app"></a>Web アプリを作成するには

1. **サブスクリプション** を選択します。
1. **[リソース グループ]** を作成または選択します。
1. アプリの **[名前]** を入力します。 アプリの名前は、アプリの URL に表示されます (例: `yourappname.appservice.<region>.<FQDN>`)
1. アプリの **[ランタイム スタック]** を選択します。 ランタイムは、発行プロファイルをターゲットとして使用するワークフローと一致する必要があります。
1. ランタイムとアプリをホストする **[オペレーティング システム]** (OS) を選択します。
1. Azure Stack Hub インスタンスの **[リージョン]** を選択または入力します。
1. Azure Stack Hub のインスタンス、リージョン、およびアプリの OS に基づいてプランを選択します。
1. **[確認および作成]** を選択します。
1. Web アプリを確認します。 **［作成］** を選択します
    ![Azure Stack Hub で Web アプリを確認する](\media\ci-cd-github-action-webapp\review-azure-stack-hub-web-app.png)
1. **[リソースに移動]** を選択します。
    ![Azure Stack Hub で発行プロファイルを取得する](\media\ci-cd-github-action-webapp\get-azure-stack-hub-web-app-publish-profile.png)
1. **[発行プロファイルの取得]** を選択します。 発行プロファイルがダウンロードされます。`<yourappname>.PublishSettings` という名前です。 このファイルには、Web アプリのターゲット値が指定された XML が含まれています。
1. リポジトリのシークレットを作成するときにアクセスできるように、発行プロファイルを保存します。

## <a name="add-your-secrets-to-the-repository"></a>シークレットをリポジトリに追加する

GitHub シークレットを使用して、アクションで使用する機密情報を暗号化できます。 SPN を格納するシークレットと Web アプリ発行プロファイルを格納するシークレットを作成し、アクションから Azure Stack Hub インスタンスにサインインし、Web アプリ ターゲットに対してアプリをビルドできるようにします。

1. GitHub リポジトリを開くか作成します。 GitHub でリポジトリを作成するためのガイダンスが必要な場合は、[GitHub ドキュメントの手順](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/create-a-repo)を参照してください。
1. **[設定]** を選択します。
1. **[Secrets]\(シークレット\)** を選択します。
1. **[New repository secret]\(新しいリポジトリ シークレット\)** を選択します。
    ![GitHub Actions シークレットの追加](.\media\ci-cd-github-action-login-cli\github-action-secret.png)
1. シークレットに `AZURE_CREDENTIALS` という名前を付けます。
1. SPN を表す JSON オブジェクトを貼り付けます。
1. **[Add secret]\(シークレットの追加\)** を選択します。
1. **[New repository secret]\(新しいリポジトリ シークレット\)** を選択します。
1. シークレットに `AZURE_WEBAPP_PUBLISH_PROFILE` という名前を付けます。
1. `<yourappname>.PublishSettings` をテキスト エディターで開き、XML をコピーしてリポジトリ シークレットに貼り付けます。
1. **[Add secret]\(シークレットの追加\)** を選択します。

## <a name="add-a-runtime-workflow"></a>ランタイム ワークフローを追加する

1. Web アプリ ランタイムの表からテンプレートを選択します。

    | ランタイム | テンプレート |
    | --- | --- |
    | DotNet | [dotnet.yml](https://github.com/Azure/actions-workflow-samples/tree/master/AppService/asp.net-core-webapp-on-azure.yml) |
    | NodeJS | [node.yml](https://github.com/Azure/actions-workflow-samples/tree/master/AppService/node.js-webapp-on-azure.yml) |
    | Java | [java_jar.yml](https://github.com/Azure/actions-workflow-samples/tree/master/AppService/java-jar-webapp-on-azure.yml) |
    | Java | [java_war.yml](https://github.com/Azure/actions-workflow-samples/tree/master/AppService/java-war-webapp-on-azure.yml) |
    | Python | [python.yml](https://github.com/Azure/actions-workflow-samples/tree/master/AppService/python-webapp-on-azure.yml) |
    | PHP | [php.yml](https://github.com/Azure/actions-workflow-samples/blob/master/AppService/php-webapp-on-azure.yml) |
    | Docker | [docker.yml](https://github.com/Azure/actions-workflow-samples/blob/master/AppService/docker-webapp-container-on-azure.yml) |

2. テンプレート GitHub アクション ワークフロー ディレクトリをプロジェクト リポジトリに配置します。`.github/workflows/<runtime.yml>` ワークフロー ディレクトリには 2 つのワークフローが含まれます。

## <a name="add-the-web-app-deploy-action"></a>Web アプリのデプロイ アクションを追加する

このセクションの yaml を使用して 2 つ目のワークフローを作成します。 この例では、Python Web アプリをデプロイしています。 ワークフローに基づいて設定アクションを選択する必要があります。 アクションの作成に使用した手順の後のさまざまなランタイムのセットアップ アクションについては、「[さまざまなランタイムのセットアップ アクション](#setup-actions-for-different-runtimes)」の表を参照してください。

### <a name="example-github-action-workflow"></a>GitHub アクション ワークフローの例

1. GitHub リポジトリを開きます。 Web アプリのアプリケーション リソースをまだ追加していない場合は、ここで追加します。 この例では、[Python Flask Hello World](https://github.com/Azure-Samples/python-docs-hello-world) サンプルを使用しており、Python の `.gitignore`、`app.py`、および `requirements.txt` ファイルを追加しました。

    ![Azure Stack Hub で Python Flask を使用したリポジトリの例](\media\ci-cd-github-action-webapp\example-of-repo.png)
1. **[Actions]\(アクション\)** を選択します。
1. **[New workflow]\(新しいワークフロー\)** を選択します。
    - これが最初のワークフローである場合、 **[Choose a workflow template]\(ワークフロー テンプレートの選択\)** の下で **[set up a workflow yourself]\(自分でワークフローを設定\)** を選択します。
    - 既存のワークフローがある場合は、 **[New workflow]|\(新しいワークフロー\)**  >  **[Set up a workflow yourself]\(自分でワークフローを設定\)** を選択します。

4. パスで、ファイルに `workflow.yml` という名前を指定します。
5. ワークフロー yml をコピーして貼り付けます。
    ```yaml  
    # .github/workflows/worfklow.yml
    on: push

    jobs:
      build-and-deploy:
        runs-on: ubuntu-latest
        steps:
        # checkout the repo
        - name: 'Checkout Github Action' 
          uses: actions/checkout@master
    
        - name: Setup Python 3.6
          uses: actions/setup-node@v1
          with:
            python-version: '3.6'
        - name: 'create a virtual environment and install dependencies'
          run: |
            python3 -m venv .venv
            source .venv/bin/activate
            pip install -r requirements.txt
    
        - name: 'Run Azure webapp deploy action using publish profile credentials'
          uses: azure/webapps-deploy@v2
          with:
            app-name: <YOURAPPNAME>
            publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
    ```

6. workflow.yaml 内の `<YOURAPPNAME>` を実際のアプリ名に更新します。
7. **[Start commit]\(コミットの開始\)** を選択します。
8. コミットのタイトルと省略可能な詳細を追加し、 **[Commit new file]\(新しいファイルのコミット\)** を選択します。


### <a name="setup-actions-for-different-runtimes"></a>さまざまなランタイムのセットアップ アクション

特定の言語ベースの環境でアプリ コードをビルドするには、セットアップ アクションを使用します。

|  ランタイム | セットアップ アクション |
|------------|---------|
| DotNet     | [Setup DotNet](https://github.com/actions/setup-dotnet) |
| NodeJS     | [Setup Node](https://github.com/actions/setup-node) |
| Java     | [Setup Java  ](https://github.com/actions/setup-java) |
| Python     | [Setup Python ](https://github.com/actions/setup-python) |
| Docker | [docker-login ](https://github.com/Azure/docker-login) |

ログイン アクションが完了すると、ワークフロー内の次の一連のアクションで、コンテナーの構築、タグ付け、プッシュなどのタスクを実行できます。 詳細については、[Azure Webapp アクション](https://github.com/marketplace/actions/azure-webapp)のドキュメントを参照してください。
## <a name="trigger-your-deployment"></a>デプロイをトリガーする

アクションが実行されたら、正常に実行されたことを確認します。

1. GitHub リポジトリを開きます。 リポジトリにプッシュすることで、ワークフローをトリガーできます。
1. **[Actions]\(アクション\)** を選択します。
1. **[All workflows]\(すべてのワークフロー\)** の下でコミット名を選択します。 両方のワークフローによって状態がログに記録されています。
    ![GitHub Actions の状態を確認する](\media\ci-cd-github-action-webapp\workflow-success.png)
1. デプロイのジョブ名 `.github/workflows/workflow.yml` を選択します。
1. セクションを展開し、ワークフロー アクションの戻り値を確認します。 デプロイされた Web アプリの URL を見つけます。
    ![Azure Stack Hub Web アプリの URL を見つける](\media\ci-cd-github-action-webapp\review-cli-log-and-get-url.png)
1. Web ブラウザーを開き、URL を読み込みます。
## <a name="next-steps"></a>次のステップ

- その他のアクションについては、[GitHub Marketplace](https://github.com/marketplace) を参照してください。
- 「[Common deployments for Azure Stack Hub 向けの一般的なデプロイ](azure-stack-dev-start-deploy-app.md)」について学習する  
- 「[Azure Stack Hub で Azure Resource Manager テンプレートを使用する](azure-stack-arm-templates.md)」について学習する  
- DevOps ハイブリッド クラウド パターン、[DevOps パターン](https://docs.microsoft.com/hybrid/app-solutions/pattern-cicd-pipeline)を確認する
