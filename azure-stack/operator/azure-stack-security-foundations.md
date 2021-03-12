---
title: Azure Stack Hub のセキュリティ コントロール
titleSuffix: Azure Stack Hub
description: Azure Stack Hub に適用されるセキュリティの体制とコントロールについて説明します。
author: BryanLa
ms.topic: article
ms.date: 06/10/2019
ms.author: bryanla
ms.reviewer: fiseraci
ms.lastreviewed: 04/07/2020
ms.openlocfilehash: 7706b453ba9666664e6269a2ba90f4fd0f6e3ab4
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231168"
---
# <a name="azure-stack-hub-infrastructure-security-controls"></a>Azure Stack Hub インフラストラクチャのセキュリティ コントロール

セキュリティに関する考慮事項と法令遵守規定は、ハイブリッド クラウドを使用する際の中核となる推進力です。 Azure Stack Hub は、これらのシナリオに適しています。 この記事では、Azure Stack Hub 用のセキュリティ コントロールについて説明しています。

Azure Stack Hub には、2 つのセキュリティ体制レイヤーが共存しています。 1 つ目のレイヤーは Azure Stack Hub インフラストラクチャであり、Azure Resource Manager へのハードウェア コンポーネントを含みます。 1 つ目のレイヤーには、管理者ポータルおよびユーザー ポータルが含まれています。 2 つ目のレイヤーは、テナントによって作成、デプロイ、および管理されるワークロードで構成されています。 2 つ目のレイヤーには、仮想マシンおよび App Services の Web サイトなどの項目が含まれています。

## <a name="security-approach"></a>セキュリティ手法

Azure Stack Hub のセキュリティ体制は、最新の脅威から防御するために設計され、主要なコンプライアンス標準の要件を満たすように構築されています。 その結果、Azure Stack Hub インフラストラクチャのセキュリティ体制が、次の 2 本の柱で構成されます。

- **セキュリティ侵害の想定**  
    システムが既に侵害されているものと想定して、"*侵害の影響の検出と抑制*" および攻撃の防止に重点を置きます。

- **既定でのセキュリティ機能の組込み**  
    適切に定義されたハードウェアとソフトウェア上でインフラストラクチャが実行されているため、Azure Stack Hub では既定で "*すべてのセキュリティ機能を有効にして、構成および検証*" しています。

Azure Stack Hub は統合システムとして提供されるため、Azure Stack Hub インフラストラクチャのセキュリティ体制は Microsoft によって定義されます。 Azure と同様に、テナントは、テナント ワークロードのセキュリティ体制を定義する責任を負います。 このドキュメントでは、Azure Stack Hub インフラストラクチャのセキュリティ体制に関する基本的知識について説明します。

## <a name="data-at-rest-encryption"></a>保存データの暗号化

Azure Stack Hub インフラストラクチャとテナントのすべてのデータは、保存時に BitLocker を使用して暗号化されます。 この暗号化により、Azure Stack Hub のストレージ コンポーネントは物理的損失や盗難から保護されます。 詳細については、「[Azure Stack Hub における保存データの暗号化](azure-stack-security-bitlocker.md)」を参照してください。

## <a name="data-in-transit-encryption"></a>転送中データの暗号化

Azure Stack Hub インフラストラクチャのコンポーネントは、TLS 1.2 で暗号化されたチャネルを使用して通信します。 暗号化証明書は、インフラストラクチャが自己管理します。

REST エンドポイントや Azure Stack Hub ポータルなど、すべての外部インフラストラクチャ エンドポイントでは、セキュリティで保護された通信用の TLS 1.2 がサポートされます。 これらのエンドポイントには、サード パーティまたはエンタープライズ証明機関のいずれかが発行する暗号化証明書を指定する必要があります。

自己署名証明書は、これらの外部エンドポイントに使用できますが、Microsoft では、自己署名証明書の使用を控えるよう強く推奨します。
Azure Stack Hub の外部エンドポイントに対して TLS 1.2 を適用する方法の詳細については、「[Azure Stack Hub セキュリティ コントロールを構成する](azure-stack-security-configuration.md)」を参照してください。

## <a name="secret-management"></a>シークレットの管理

Azure Stack Hub インフラストラクチャが機能するうえで、パスワードや証明書など、さまざまなシークレット情報が使用されます。 内部サービス アカウントに関連付けられているパスワードのほとんどは、24 時間おきに自動的にローテーションされます。これはアカウントが[グループの管理されたサービス アカウント (gMSA)](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) であるためです。gMSA は、内部ドメイン コントローラーによって直接管理される種類のドメイン アカウントです。

Azure Stack Hub インフラストラクチャでは、すべての内部証明書に 4096 ビット RSA キーが使用されます。 外部エンドポイントに対しても、同じキー長の証明書を使用できます。 シークレットと証明書のローテーションの詳細については、「[Azure Stack Hub でシークレットをローテーションする](azure-stack-rotate-secrets.md)」を参照してください。

## <a name="windows-defender-application-control"></a>Windows Defender アプリケーション制御

Azure Stack Hub には、最新の Windows Server セキュリティ機能が使用されています。 これらの 1 つに Windows Defender アプリケーション制御があります (WDAC: 以前はコード整合性と呼ばれていました)。これにより、実行可能ファイル フィルタリングが提供され、承認済みのコードだけが Azure Stack Hub インフラストラクチャ内で実行されることが保証されます。

承認済みのコードは Microsoft または OEM パートナーのいずれかによって署名されます。 署名され認証されたコードは、Microsoft が定義したポリシーで指定されている認定ソフトウェアのリストに含まれています。 つまり、Azure Stack Hub インフラストラクチャでの実行が承認されているソフトウェアのみを実行できます。 未承認のコードを実行しようとしてもブロックされ、アラートが生成されます。 Azure Stack Hub では、User Mode Code Integrity (UMCI) と Hypervisor Code Integrity (HVCI) の両方が適用されます。

WDAC ポリシーによっても、Azure Stack Hub インフラストラクチャでサード パーティ製のエージェントまたはソフトウェアを実行することが禁止されています。
WDAC の詳細については、「[Windows Defender アプリケーション コントロールと仮想化ベースのコードの整合性の保護](/windows/security/threat-protection/device-guard/introduction-to-device-guard-virtualization-based-security-and-windows-defender-application-control)」を参照してください。

## <a name="credential-guard"></a>資格情報の保護

Azure Stack Hub の別の Windows Server セキュリティ機能として Windows Defender Credential Guard があります。この機能は、Azure Stack Hub インフラストラクチャの資格情報を Pass-the-Hash 攻撃や Pass-the-Ticket 攻撃から保護するために使用されます。

## <a name="antimalware"></a>マルウェア対策

Azure Stack Hub のすべてのコンポーネント (Hyper-V ホストと仮想マシンの両方) は、Windows Defender ウイルス対策によって保護されています。

接続されているシナリオでは、ウイルス対策の定義とエンジンの更新プログラムが、1 日に複数回適用されます。 接続されていないシナリオでは、マルウェア対策の更新プログラムが、月次の Azure Stack Hub 更新プログラムの一部として適用されます。 切断されたシナリオで Windows Defender の定義をより頻繁に更新する必要がある場合、Azure Stack Hub では Windows Defender の更新プログラムをインポートすることもサポートされています。 詳細については、「[Azure Stack Hub 上で Windows Defender ウイルス対策を更新する](azure-stack-security-av.md)」を参照してください。

## <a name="secure-boot"></a>セキュア ブート

Azure Stack Hub では、すべての Hyper-V ホストおよびインフラストラクチャの仮想マシンに対してセキュア ブートが適用されます。 

## <a name="constrained-administration-model"></a>制約付き管理モデル

Azure Stack Hub 内の管理は、それぞれが特定の目的を持つ、次の 3 つのエントリ ポイントを使用して制御されます。

- [管理者ポータル](azure-stack-manage-portals.md)では、日常の管理操作にポイントアンドクリック エクスペリエンスが提供されます。
- Azure Resource Manager では、PowerShell および Azure CLI で使用される、REST API を介した管理者ポータルのすべての管理操作が公開されます。
- データ センターの統合やサポート シナリオなど、特定の低レベルの操作では、Azure Stack Hub により、[特権エンドポイント](azure-stack-privileged-endpoint.md)と呼ばれる PowerShell エンドポイントが公開されます。 このエンドポイントでは、コマンドレットの許可されたセットのみが公開され、これは厳重に監査されます。

## <a name="network-controls"></a>ネットワーク制御

Azure Stack Hub インフラストラクチャには、ネットワーク アクセス制御リスト (ACL) の複数のレイヤーが付属しています。 ACL は、インフラストラクチャ コンポーネントへの不正アクセスを防止し、インフラストラクチャがその機能を果たすために必要なパスとだけ通信するように制限します。

ネットワークの ACL には、次の 3 つのレイヤーが適用されます。

- レイヤー 1:ラック スイッチの最上部
- レイヤー 2:ソフトウェア定義ネットワーク
- レイヤー 3:ホストおよび VM のオペレーティング システムのファイアウォール

## <a name="regulatory-compliance"></a>規制に対するコンプライアンス

Azure Stack Hub はサード パーティの独立した監査法人による正式な機能評価を受けています。 その結果、Azure Stack Hub インフラストラクチャが、いくつかの主要なコンプライアンス標準の該当するコントロールをどのように満たしているかについてのドキュメントを入手できます。 標準には、複数の担当者に関連する、およびプロセスに関連するコントロールが複数含まれるため、このドキュメントは Azure Stack Hub の認定資格ではありません。 代わりに、お客様は、このドキュメントを利用して、認定プロセスをすぐに開始できます。

評価は次のような標準が含まれます。

- [PCI DSS](https://www.pcisecuritystandards.org/pci_security/) ペイメント カード業界に対応します。
- [CSA Cloud Control Matrix](https://cloudsecurityalliance.org/group/cloud-controls-matrix/#_overview) は、FedRAMP Moderate、ISO27001、HIPAA、HITRUST、ITAR、NIST SP800-53 などの複数の標準間での包括的なマッピングです。
- 政府顧客向け [FedRAMP High](https://www.fedramp.gov/fedramp-releases-high-baseline/)。

このコンプライアンス ドキュメントは [Microsoft Service Trust Portal](https://aka.ms/azurestackcompliance) で確認できます。 コンプライアンス ガイドは、保護されたリソースであり、Azure クラウド サービスの資格情報でサインインする必要があります。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub セキュリティ コントロールを構成する](azure-stack-security-configuration.md)
- [Azure Stack Hub でシークレットをローテーションする方法を確認する](azure-stack-rotate-secrets.md)
- [Azure Stack Hub 用の PCI-DSS および CSA-CCM ドキュメント](https://aka.ms/azurestackcompliance)
