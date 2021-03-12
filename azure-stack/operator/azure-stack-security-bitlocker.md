---
title: 保存データの暗号化
titleSuffix: Azure Stack Hub
description: Azure Stack Hub が暗号化によって保存データをどのように保護するかを説明します。
author: BryanLa
ms.topic: how-to
ms.date: 03/04/2020
ms.author: bryanla
ms.reviewer: fiseraci
ms.lastreviewed: 08/06/2020
ms.openlocfilehash: 147da85fc75ffbe76c1216c402fcec10ba0eb60f
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231202"
---
# <a name="data-at-rest-encryption-in-azure-stack-hub"></a>Azure Stack Hub での保存データの暗号化

Azure Stack Hub は、保存時の暗号化を使用して、ストレージ サブシステム レベルのユーザー データとインフラストラクチャ データを保護します。 既定で、Azure Stack Hub のストレージ サブシステムは、128 ビット AES 暗号化による BitLocker を使用して暗号化されます。 BitLocker のキーは、内部のシークレット ストアに保存されます。 デプロイ時、256 ビット AES 暗号化を使用するように BitLocker を構成することもできます。

保存データの暗号化は、主要なコンプライアンス標準 (PCI-DSS、FedRAMP、HIPAA など) の多くで一般的な要件です。 Azure Stack Hub では、特別な作業や構成を行わなくても、これらの要件を満たすことができます。 Azure Stack Hub がコンプライアンス標準への準拠をどのように支援するかについて詳しくは、[Microsoft Service Trust Portal](https://aka.ms/AzureStackCompliance) を参照してください。

> [!NOTE]
> 保存データの暗号化は、ハード ドライブが物理的な盗難に遭ってもアクセスできないようデータを保護します。 保存データの暗号化は、ネットワーク上のデータ (転送中のデータ) の傍受や、使用中のデータ (メモリ内データ) の漏えい、また、より一般的にはシステム稼働中のデータの抽出からデータを保護するものではありません。

## <a name="retrieving-bitlocker-recovery-keys"></a>BitLocker 回復キーを取得する

Azure Stack Hub で保存データに使用される BitLocker キーは内部で管理されます。 通常の操作やシステム起動時にそれらを指定する必要はありません。 ただしサポートのシナリオで、システムをオンラインにするために BitLocker 回復キーが必要になることはあります。  

> [!WARNING]
> BitLocker 回復キーを取得して、Azure Stack Hub 外部の安全な場所に保管してください。 特定のサポート シナリオ時に回復キーがないと、データの損失につながったり、バックアップ イメージからのシステムの復元が必要になったりすることがあります。

BitLocker 回復キーを取得するには、[特権エンドポイント](azure-stack-privileged-endpoint.md) (PEP) にアクセスする必要があります。 PEP セッションから Get-AzsRecoveryKeys コマンドレットを実行します。

```powershell
##This cmdlet retrieves the recovery keys for all the volumes that are encrypted with BitLocker.
Get-AzsRecoveryKeys -raw
```

*Get-AzsRecoveryKeys* コマンドレットのパラメーター:

| パラメーター | 説明 | Type | 必須 |
|---------|---------|---------|---------|
|*raw* | 回復キーとコンピューター名と暗号化された各ボリュームのパスワード ID とにおけるデータのマッピングを返します。  | Switch | いいえ (ただし推奨) |

## <a name="troubleshoot-issues"></a>問題のトラブルシューティング

ごくまれな状況ですが、BitLocker ロック解除要求が失敗し、特定のボリュームが起動しない場合があります。 アーキテクチャを構成する一部のコンポーネントの利用の可否によっては、BitLocker 回復キーがないと、このエラーがダウンタイムやデータ損失のリスクを招く可能性があります。

> [!WARNING]
> BitLocker 回復キーを取得して、Azure Stack Hub 外部の安全な場所に保管してください。 特定のサポート シナリオ時に回復キーがないと、データの損失につながったり、バックアップ イメージからのシステムの復元が必要になったりすることがあります。

Azure Stack Hub が起動に失敗するなど、システムで BitLocker に関する問題が発生している可能性がある場合は、サポートに連絡してください。 サポートには、BitLocker 回復キーが必要です。 BitLocker に関連した問題の大半は、その特定の VM/ホスト/ボリュームに対する FRU 操作で解決できます。 その他のケースでは、BitLocker 回復キーを使用した手動でのロック解除手順を実施できます。 BitLocker 回復キーが利用できない場合、バックアップ イメージからの復元が唯一の選択肢となります。 最後にバックアップを実施した時期によっては、データの損失が生じることがあります。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub のセキュリティについて詳しく学習します](azure-stack-security-foundations.md)。
- BitLocker による CSV の保護の詳細については、[BitLocker を使用したクラスター共有ボリュームと記憶域ネットワークの保護](/windows/security/information-protection/bitlocker/protecting-cluster-shared-volumes-and-storage-area-networks-with-bitlocker)に関するページを参照してください。
