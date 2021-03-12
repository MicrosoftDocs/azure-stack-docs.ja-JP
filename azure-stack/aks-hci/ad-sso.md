---
title: AKS on Azure Stack HCI 内で Active Directory シングル サインオンを使用する
description: Active Directory 認証を使用して、SSO 資格情報によって API サーバーに安全に接続する
author: v-susbo
ms.topic: how-to
ms.date: 02/12/2021
ms.author: v-susbo
ms.reviewer: ''
ms.openlocfilehash: fd4b8982ecaa9de1b3b63dd97cb460772e25971d
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874523"
---
# <a name="use-active-directory-single-sign-in-credentials-for-aks-on-azure-stack-hci"></a>AKS on Azure Stack HCI に対して Active Directory シングル サインイン資格情報を使用する

> 適用対象:Azure Stack HCI 上の AKS、Windows Server 2019 Datacenter 上の AKS ランタイム

Active Directory 認証を使用せずに、`kubectl` コマンドを使用して API サーバーに接続する場合、ユーザーは証明書ベースの _kubeconfig_ ファイルに依存する必要があります。 _kubeconfig_ ファイルには、秘密キーや証明書など、慎重に配布する必要があるシークレットが含まれています。 これは重大なセキュリティ リスクになる可能性があります。

証明書ベースの _kubeconfig_ を使用する代わりに、API サーバーへの安全な接続方法として、Active Directory (AD) シングル サインオン (SSO) 資格情報を使用できます。 Azure Kubernetes Service on Azure Stack HCI と AD の統合により、Windows ドメイン参加済みマシン上のユーザーが、自分の SSO 資格情報を使用して (`kubectl` 経由で) API サーバーに接続することができます。 これにより、秘密キーを含む証明書ベースの _kubeconfig_ ファイルを管理および配布する必要がなくなります。

AD 統合によって使用される AD _kubeconfig_ は、証明書ベースの _kubeconfig_ ファイルとは異なり、シークレットが一切含まれません。 ただし、Active Directory 資格情報を使用した接続に問題がある場合は、トラブルシューティングなどのバックアップの目的で、証明書ベースの _kubeconfig_ ファイルを使用できます。

また、ユーザーとグループが[セキュリティ識別子 (SID)](https://docs.microsoft.com/troubleshoot/windows-server/identity/security-identifiers-in-windows) として格納されることも、AD 統合のセキュリティ上のメリットの 1 つです。 グループ名とは異なり、SID は不変かつ一意であるため、名前が競合することはありません。

> [!Note] 
> このプレビュー リリースでは、ワークロード クラスターに対してのみ AD SSO 接続を提供します。 

このドキュメントでは、次の手順に従って、Active Directory を ID プロバイダーとして設定し、`kubectl` 経由で SSO を有効にします。

- API サーバーに対して AD アカウントを作成し、そのアカウントに関連付けられた [keytab](https://web.mit.edu/kerberos/krb5-devel/doc/basic/keytab_def.html) ファイルを作成します。 「[keytab ファイルを使用して AD 認証を作成する](#create-ad-auth-using-the-keytab-file)」を参照して、AD アカウントを作成し、keytab ファイルを生成します。
- [keytab](https://web.mit.edu/kerberos/krb5-devel/doc/basic/keytab_def.html) ファイルを使用して、Kubernetes クラスター上で AD 認証をインストールします。 この手順の一部として、既定のロールベースのアクセス制御 (RBAC) 構成が自動的に作成されます。
- RBAC 構成を作成または編集するときは、グループ名を SID に変換するか、SID をグループ名に変更します。手順については、「[AD グループの役割のバインドを作成および更新する](#create-and-update-the-ad-group-role-binding)」をご覧ください。
 
## <a name="before-you-begin"></a>開始する前に

Active Directory SSO 資格情報を構成するプロセスを開始する前に、次の項目を確認する必要があります。

 - 最新の **Aks-Hci PowerShell** モジュールがインストールされている必要があります。 インストールされていない場合は、「[AksHci PowerShell モジュールをダウンロードしてインストールする](./setup-powershell.md#step-1-download-and-install-the-akshci-powershell-module)」をご覧ください。 
 - AKS ホストがインストールされ、構成されている。 これがない場合は、[デプロイを構成](./setup-powershell.md#step-4-configure-your-deployment)する手順に従ってください。  
 - keytab ファイルが使用できることを確認します。 できない場合は、「[API サーバー AD アカウントと keytab ファイルを作成する](#create-the-api-server-ad-account-and-the-keytab-file)」をご覧ください。 
 - keytab ファイルが特定のサービス プリンシパル名 (SPN) に対して生成されます。この SPN は、ワークロード クラスター用の API サーバー AD アカウントに対応している必要があります。 また、AD 認証プロセス全体で同じ SPN が使用されていることを確認する必要もあります。 keytab ファイルの名前は _current.keytab_ にします。
 - AKS on Azure Stack HCI ワークロード クラスターごとに、使用可能な API サーバー AD アカウントが 1 つあることを確認します。
 - クライアント マシンは、Windows ドメイン参加済みマシンである必要があります。

## <a name="create-ad-auth-using-the-keytab-file"></a>keytab ファイルを使用して AD 認証を作成する

### <a name="step-1-create-the-workload-cluster-and-enable-ad-authentication"></a>手順 1: ワークロード クラスターを作成し、AD 認証を有効にする

AD 認証をインストールする前に、AKS on Azure Stack HCI ワークロード クラスターを作成し、そのクラスター上で AD 認証アドオンを有効にする必要があります。 新しいクラスターを作成するときに AD 認証を有効にしなかった場合、後で有効にすることはできません。

管理者として PowerShell を開き、次のように `New-AksHciCluster` コマンドを **-enableADAuth** パラメーターを指定して実行し ます。

```powershell
New-AksHciCluster -name mynewcluster1 -enableADAuth
```

ワークロード クラスターごとに、使用可能な API サーバー AD アカウントが 1 つあることを確認します。

ワークロード クラスター作成の詳細については、「[Windows PowerShell を使用して Kubernetes クラスターを作成する](./create-kubernetes-cluster-powershell.md)」をご覧ください。

### <a name="step-2-install-ad-authentication"></a>手順 2: AD 認証をインストールする

AD 認証をインストールする前に、ワークロード クラスターがインストールされていること、およびクラスター上で AD 認証が有効になっていることを確認してください。 AD 認証をインストールするには、次のいずれかのオプションを使用します。

#### <a name="option-1"></a>方法 1

ドメイン参加済み Azure Stack HCI クラスターの場合は、管理者として PowerShell を開き、次のコマンドを実行します。

```powershell
Install-AksHciAdAuth -name mynewcluster1 -keytab .\current.keytab -SPN k8s/apiserver@CONTOSO.COM -adminUser contoso\bob
```  

#### <a name="option-2"></a>方法 2

クラスター ホストがドメイン参加済みでない場合は、次の例に示すように、SID 形式で管理者ユーザー名またはグループ名を使用します。
 
```powershell
Install-AksHciAdAuth -name mynewcluster1 -keytab .\current.keytab -SPN k8
```  

ユーザー アカウントの SID を検索するには、「[ユーザーまたはグループのセキュリティ識別子を決定する](#determine-the-user-or-group-security-identifier)」をご覧ください。 

次の手順に進む前に、以下の項目をご確認ください。

- keytab ファイルの名前が _current.keytab_ であることを確認します。
- ご自身の環境に対応する SPN を置き換えます。
- **-adminUser** を指定すると、クラスター管理特権を使用して、指定された AD グループに対応するロール バインドが作成されます。 _contoso\bob_ を、ご自身の環境に対応する AD グループまたはユーザーに置き換えます。

### <a name="step-3-test-the-ad-webhook-and-keytab-file"></a>手順 3: AD Webhook と keytab ファイルをテストする

AD webhook が API サーバー上で実行されていること、および keytab ファイルが Kubernetes シークレットとして格納されていることを確認する必要があります。 ワークロード クラスター用の証明書ベースの _kubeconfig_ ファイルを取得するには、次の手順に従います。

1. 証明書ベースの _kubeconfig_ ファイルを取得します。 _kubeconfig_ ファイルを使用して、次のコマンドによってローカル ホストとしてクラスターに接続します。

   ```powershell
   Get-AksHciCredential -name mynewcluster1
   ```  

2. (前に作成した証明書ベースの _kubeconfig_ ファイルを使用して) 接続したサーバー上で `kubectl` を実行し、AD Webhook デプロイをチェックして、それが _ad-auth-webhook-xxxx_ 形式であることを確認します。  

    ```bash
    kubectl get pods -n=kube-system
    ```

3. `kubectl` を実行して、keytab ファイルがシークレットとしてデプロイされ、Kubernetes シークレットとして表示されていることを確認します。 

   ```bash
   kubectl get secrets -n=kube-system
   ```

### <a name="step-4-create-the-ad-kubeconfig-file"></a>手順 4: AD kubeconfig ファイルを作成する

AD Webhook と keytab が正常にデプロイされたら、AD _kubeconfig_ ファイルを作成します。
ファイルが作成された後、AD _kubeconfig_ ファイルをクライアント マシンにコピーし、それを使用して、API サーバーに対して認証します。 証明書ベースの _kubeconfig_ ファイルとは異なり、AD _kubeconfig_ はシークレットではなく、プレーン テキストとしてコピーしても問題ありません。

管理者として PowerShell を開き、次のコマンドを実行します。  

   ```powershell
   Get-AksHciCredential -name mynewcluster1 -outputLocation .\AdKubeconfig -adAuth
   ```

### <a name="step-5-copy-kubeconfig-and-other-files-to-the-client-machine"></a>手順 5: kubeconfig およびその他のファイルをクライアント マシンにコピーする

次に示す 3 つのファイルを、Azure Stack HCI クラスターからご自身のクライアント マシンにコピーする必要があります。

- 前の手順で作成した AD _kubeconfig_ ファイルを $env:USERPROFILE\.kube\config にコピーします。

- フォルダー パス `c:\adsso` を作成し、次のファイルを、Azure Stack HCI クラスターからご自身のクライアント マシンにコピーします。
  - `$env:ProgramFiles\AksHci` の下の Kubectl.exe を c:\adsso に
  - `$env:ProgramFiles\AksHci` の下の Kubectl-adsso.exe を c:\adsso に

### <a name="step-6-connect-to-the-api-server-from-the-client-machine"></a>手順 6: クライアント マシンから API サーバーに接続する

前の手順を完了したら、ご自身の SSO 資格情報を使用して、Windows ドメイン参加済みのクライアント マシンにログインします。 PowerShell を開き、`kubectl` を使用して、API サーバーへのアクセスを試みます。 認証を求めるメッセージが表示されたら、AD SSO が正常に設定されています。

## <a name="create-and-update-the-ad-group-role-binding"></a>AD グループのロール バインドを作成および更新する

手順 2 で説明したように、クラスター管理特権を持つ既定のロール バインドが、インストール中に提供されたユーザーやグループに対して作成されます。 Kubernetes 内のロール バインドにより、AD グループのアクセス ポリシーが定義されます。 この手順では、RBAC を使用して、Kubernetes 内で新しい AD グループのロール バインドを作成し、既存のロール バインドを編集する方法について説明します。 たとえば、クラスター管理者は、AD グループを使用して、追加の特権をユーザーに付与することができます (これにより、プロセスがさらに効率的になります)。 RBAC の詳細については、「[RBAC 認証の使用](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)」をご覧ください。

さらに多くの AD グループ RBAC エントリを作成または編集する場合、サブジェクト名は、"`microsoft:activedirectory:CONTOSO\\group name`" によって事前に修正する必要があります。 名前には、二重引用符で囲まれたドメイン名とプレフィックスが含まれている必要があることに注意してください。

次に 2 つの例を示します。

**例 1**

```bash
apiVersion: rbac.authorization.k8s.io/v1 
 kind: ClusterRoleBinding 
 metadata: 
   name: ad-user-cluster-admin 
 roleRef: 
   apiGroup: rbac.authorization.k8s.io 
   kind: ClusterRole 
   name: cluster-admin 
 subjects: 
 - apiGroup: rbac.authorization.k8s.io 
   kind: User 
   name: "microsoft:activedirectory:CONTOSO\Bob" 
```

**例 2**

以下の例は、AD グループを使用して、[名前空間](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) に対してカスタム役割とロール バインドを作成する方法を示しています。 この例の "SREGroup" は、Contoso Active Directory 内の既存のグループです。 ユーザーが AD グループに追加されると、すぐに特権が付与されます。

```bash
kind: Role 
 apiVersion: rbac.authorization.k8s.io/v1 
 metadata: 
   name: sre-user-full-access 
   namespace: sre 
 rules: 
 - apiGroups: ["", "extensions", "apps"] 
   resources: ["*"] 
   verbs: ["*"] 
 - apiGroups: ["batch"] 
   resources: 
   - jobs 
   - cronjobs 
   verbs: ["*"] 
 apiVersion: rbac.authorization.k8s.io/v1 
 kind: RoleBinding 
 namespace: sre 
 metadata: 
   name: ad-user-cluster-admin 
 roleRef: 
   apiGroup: rbac.authorization.k8s.io 
   kind: Role 
   name: sre-user-full-access 
 subjects: 
 - apiGroup: rbac.authorization.k8s.io 
   kind: User 
   name: "microsoft:activedirectory:CONTOSO\SREGroup" 
```

yaml ファイルを適用する前に、次のコマンドを使用して、**グループ名とユーザー名** が常に SID に変換されるようにする必要があります。

```bash
kubectl-adsso nametosid <rbac.yml>
```  

同様に、既存の RBAC を更新するために、変更を行う前に、SID をユーザー フレンドリなグループ名に変換することができます。 SID を変換するには、次のコマンドを使用します。 

```bash
kubectl-adsso sidtoname <rbac.yml>
```

## <a name="change-the-ad-account-password-associated-with-the-api-server-account"></a>API サーバー アカウントに関連付けられている AD アカウントのパスワードを変更する

API サーバー アカウント用のパスワードが変更された場合は、AD 認証アドオンをアンインストールし、更新された現在の keytab と前の keytab を使用して再インストールする必要があります。 

パスワードを変更するたびに、現在の keytab の名前 (_current.keytab_) を _previous.keytab_ に変更し、新しいパスワードの名前を _current.keytab_ にする必要があります。

> [!Important] 
> これらのファイルには、それぞれ _current.keytab_ と _previous.keytab_ という名前を付ける必要があります。 既存のロール バインドは、この影響を受けません。

### <a name="uninstall-and-reinstall-ad-authentication"></a>AD 認証をアンインストールして再インストールする

API サーバーのアカウントが変更されたとき、パスワードが更新されたとき、またはエラーのトラブルシューティングを行うときに、AD SSO の再インストールが必要になる場合があります。

AD 認証をアンインストールするには、管理者として PowerShell を開き、次のコマンドを実行します。  

```powershell
Uninstall-AksHciAdAuth -name mynewcluster1
```

AD 認証を再インストールするには、管理者として PowerShell を開き、次のコマンドを実行します。

```powershell
Install-AksHciAdAuth -name mynewcluster1 -keytab <.\current.keytab> -previousKeytab <.\previous.keytab> -SPN <service/principal@CONTOSO.COM> -adminUser CONTOSO\Bob
```

> [!Note] 
> この `-previousKeytab` パラメーターは、パスワードを変更した場合にのみ必要です。 その目的は、クライアントによってチケットがキャッシュされた場合に、ダウンタイムを回避することです。

## <a name="create-the-api-server-ad-account-and-the-keytab-file"></a>API サーバー AD アカウントと keytab ファイルを作成する

このプロセスには 2 つの手順があります。 最初に、サービス プリンシパル名 (SPN) を使用して API サーバーに対して新しい AD アカウント/ユーザーを作成し、次に、AD アカウントに対して keytab ファイルを作成します。

### <a name="step-1-create-a-new-ad-account-or-user-for-the-api-server"></a>手順 1: API サーバーに対して新しい AD アカウントまたはユーザーを作成する

[New-ADUser](https://docs.microsoft.com/powershell/module/addsadministration/new-aduser?view=win10-ps&preserve-view=true) PowerShell コマンドを使用して、SPN を含む新しい AD アカウント/ユーザーを作成します。 次に例を示します。 

```powershell 
New-ADUser -Name apiserver -ServicePrincipalNames k8s/apiserver -AccountPassword (ConvertTo-SecureString "password" -AsPlainText -Force) -KerberosEncryptionType AES128 -Enabled 1
```

### <a name="step-2-create-the-keytab-file-for-the-ad-account"></a>手順 2: AD アカウントに対して keytab ファイルを作成する

keytab ファイルを作成するには、[ktpass](https://docs.microsoft.com/windows-server/administration/windows-commands/ktpass) Windows コマンドを使用します。 

ktpass を使用する例を次に示します。

```bash
ktpass /out current.keytab /princ k8s/apiserver@BCONTOSO.COM /mapuser contoso\apiserver_acct /crypto all /pass p@$$w0rd /ptype KRB5_NT_PRINCIPAL
```

> [!NOTE]
> "**DsCrackNames が名前エントリで 0x2 を返しました**" というエラーが表示される場合は、`/mapuser` のパラメーターのフォームが `mapuser DOMAIN\user` であることをご確認ください。ここで、DOMAIN はエコー `%userdomain%` の出力です。

  
## <a name="determine-the-user-or-group-security-identifier"></a>ユーザーまたはグループのセキュリティ識別子を決定する

次の 2 つのオプションのいずれかを使用して、ご自身のアカウントまたはその他のアカウントの SID を検索します。

ご自身のアカウントに関連付けられている SID を検索するには、ホーム ディレクトリのコマンド プロンプトで、次のように入力します。

```bash
whoami /user
``` 

他のアカウントに関連付けられている SID を検索するには、管理者として PowerShell を開き、次を実行します。

```powershell
(New-Object System.Security.Principal.NTAccount(<CONTOSO\Bob>)).Translate([System.Security.Principal.SecurityIdentifier]).value
```

## <a name="next-steps"></a>次のステップ 

この攻略ガイドでは、SSO 資格情報を使用して、API サーバーに安全に接続するように AD 認証を構成する方法について学習しました。 次に、以下を実行できます。

- [Kubernetes クラスターに Linux アプリケーションをデプロイします](./deploy-linux-application.md)。
- [Kubernetes クラスターに Windows Server アプリケーションをデプロイします](./deploy-windows-application.md)。
