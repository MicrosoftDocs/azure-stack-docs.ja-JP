---
title: Azure App Service on Azure Stack の概要
description: App Service on Azure Stack Hub と Azure Functions の概要。
author: BryanLa
ms.topic: article
ms.date: 05/05/2020
ms.author: BryanLa
ms.reviewer: anwestg
ms.lastreviewed: 01/13/2019
ms.openlocfilehash: f30a8b82a07aafed80fc75329516bd3b3d91df04
ms.sourcegitcommit: f9be5640dd445b3d926c9ce3e2165e96c72ece89
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2021
ms.locfileid: "100009045"
---
# <a name="azure-app-service-and-azure-functions-on-azure-stack-hub-overview"></a>App Service on Azure Stack Hub と Azure Functions の概要

Azure App Service on Azure Stack Hub は、Azure Stack Hub で利用できる Microsoft Azure の、サービスとしてのプラットフォーム (PaaS) サービスです。 このサービスにより、内部または外部の顧客はあらゆるプラットフォームやデバイス用の Web および Azure Functions アプリを作成できるようになります。 お客様のアプリとオンプレミスのアプリを統合し、ビジネス プロセスを自動化できます。 Azure Stack Hub クラウドの運用者は、指定した共有仮想マシン (VM) リソースまたは専用 VM を使用して、フル マネージドの VM 上でユーザーのアプリを実行できます。

Azure App Service を使用すると、ビジネス プロセスとホスト クラウド API を自動化できます。 単一の統合サービスである Azure App Service では、さまざまなコンポーネント (Web サイト、REST API、ビジネス プロセスなど) を 1 つのソリューションにまとめることができます。

## <a name="why-offer-azure-app-service-on-azure-stack-hub"></a>Azure App Service on Azure Stack Hub を提供する理由

Azure App Service のいくつかの主要機能を次に示します。

- **複数の言語とフレームワーク**:Azure App Service では、ASP.NET、Node.js、Java、PHP、Python が最高レベルでサポートされています。 また、App Service VM では、Windows PowerShell などのスクリプトや実行可能ファイルを実行することもできます。
- **DevOps の最適化**:継続的インテグレーションとデプロイを GitHub、ローカルの Git または BitBucket で設定できます。 テスト環境やステージング環境を介して更新を昇格させ、Azure PowerShell またはクロスプラットフォーム コマンド ライン インターフェイス (CLI) を使用して App Service でアプリを管理します。
- **Visual Studio 統合**:Visual Studio の専用ツールを使えば、アプリの作成やデプロイ作業が効率的になります。

## <a name="app-types-in-app-service"></a>App Service でのアプリの種類

App Service にはアプリの種類が複数用意されており、それぞれ特定のワークロードをホストするよう設計されています。

- Web サイト、Web アプリ、および REST API をホストする [Web Apps](/azure/app-service/overview)。
- イベント ドリブンでサーバーレスのワークロードをホスティングするための [Azure Functions v1](/azure/azure-functions)。

"*アプリ*" とは、ワークロードの実行専用のホスティング リソースを指しています。 *Web アプリ* を例にとると、Web アプリをコンピューティング リソースとアプリ コードの両方の観点から捉えることにおそらく慣れているでしょう。この両者が合わさって、ブラウザーに機能が提供されます。 Azure App Service において、Web アプリとは、アプリ コードをホストするために Azure Stack Hub で提供されるコンピューティング リソースです。

アプリは、さまざまな種類の複数の App Service アプリで構成できます。 たとえば、アプリが Web フロントエンドと REST API バックエンドで構成されている場合は、次のようなことを実行できます。

- 1 つの Web アプリに両方 (フロントエンドと API) をデプロイする。
- フロントエンドのコードを Web アプリに、バックエンドのコードを API アプリにデプロイする

   [![データの監視を表示した App Service の概要](media/azure-stack-app-service-overview/image01.png "データの監視を表示した App Service の [概要]")](media/azure-stack-app-service-overview/image01.png#lightbox)

## <a name="what-is-an-app-service-plan"></a>App Service プランとは

App Service のリソース プロバイダーは、Azure App Service で使用するものと同じコードを使用するので、一部の共通概念を共有します。 App Service では、アプリの価格コンテナーを *App Service プラン* と呼びます。 これは、アプリを保持するために使用される専用の VM のセットを表します。 特定のサブスクリプション内で、複数の App Service プランを利用できます。

Azure には、共有ワーカーと専用ワーカーがあります。 共有ワーカーは高密度のマルチテナント アプリ ホスティングをサポートします。この共有ワーカーは 1 セットしかありません。 専用サーバーは 1 つのテナントでのみ使用され、小、中、大という 3 つのサイズがあります。 オンプレミスの顧客のニーズは、必ずしもこれらの用語を使用して説明されるわけではありません。 Azure App Service on Azure Stack Hub では、リソース プロバイダー管理者が、使用可能にするワーカー層を定義します。 独自のホスティング ニーズに基づき、複数の共有ワーカー セットや、異なる専用ワーカー セットを定義できます。 これらのワーカー層の定義を使用することで、独自の価格 SKU を定義できます。

## <a name="portal-features"></a>ポータルの機能

Azure App Service on Azure Stack Hub では、Azure App Service で使用するのと同じユーザー インターフェイスが使用されます。 同じことがバック エンドにも当てはまります。 ただし、Azure Stack Hub では一部の機能が無効になります。 Azure Stack Hub では現在、このような機能に必要な Azure 固有の期待値やサービスはまだ使用できません。

## <a name="next-steps"></a>次のステップ

- [App Service on Azure Stack Hub のデプロイの前提条件](azure-stack-app-service-before-you-get-started.md)
- [Azure App Service リソース プロバイダーをインストールする](azure-stack-app-service-deploy.md)

[SQL Server リソース プロバイダー](azure-stack-sql-resource-provider-deploy.md)や [MySQL リソース プロバイダー](azure-stack-mysql-resource-provider-deploy.md)など、他の[サービスとしてのプラットフォーム (PaaS) サービス](service-plan-offer-subscription-overview.md)を試すこともできます。
