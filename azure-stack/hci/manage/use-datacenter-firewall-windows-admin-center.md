---
title: データセンターのファイアウォールを使用して Windows Admin Center で ACL を構成する
description: このトピックでは、データセンターのファイアウォールとソフトウェア定義ネットワーク (SDN) の論理および仮想ネットワーク上の ACL を使用してデータ トラフィック フローを管理するために、Windows Admin Center を使用してアクセス制御リスト (ACL) を作成し、構成する方法について説明します。
author: AnirbanPaul
ms.author: anpaul
ms.topic: how-to
ms.date: 03/04/2021
ms.openlocfilehash: 342f2771d74a2347cd53fa9364089a934885c671
ms.sourcegitcommit: f194f9ca4297864500e62d8658674a0625b29d1d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102194334"
---
# <a name="use-datacenter-firewall-to-configure-acls-with-windows-admin-center"></a>データセンターのファイアウォールを使用して Windows Admin Center で ACL を構成する

>適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019、Windows Server 2016

このトピックでは、データセンターのファイアウォールを使用してデータ トラフィック フローを管理するために、Windows Admin Center を使用してアクセス制御リスト (ACL) を作成および構成する方法について、手順を追って説明します。 また、ソフトウェア定義ネットワーク (SDN) の論理および仮想ネットワーク上で ACL を管理する方法についても説明します。 データセンターのファイアウォールを有効にして構成するには、ACL を作成してサブネットまたはネットワーク インターフェイスに適用します。 詳細については、「[データセンターのファイアウォールとは](../concepts/datacenter-firewall-overview.md)」を参照してください。 PowerShell スクリプトを使用してこれを行うには、「[データ センターのファイアウォールを使用して PowerShell で ACL を構成する](use-datacenter-firewall-powershell.md)」を参照してください。

ACL を構成する前に、ネットワーク コントローラーを展開する必要があります。 ネットワーク コントローラーについては、「[ネットワーク コントローラーの概要](../concepts/network-controller-overview.md)」を参照してください。 PowerShell スクリプトを使用してネットワーク コントローラーを展開する方法については、[SDN インフラストラクチャの展開](sdn-express.md)に関するページを参照してください。

また、SDN 論理ネットワークに ACL を適用する場合は、最初に論理ネットワークを作成する必要があります。 同様に、SDN 仮想ネットワークに ACL を適用する場合は、最初に仮想ネットワークを作成する必要があります。 詳細については、次を参照してください。
- [テナントの論理ネットワークを管理する](tenant-logical-networks.md)
- [テナントの仮想ネットワークを管理する](tenant-virtual-networks.md)

## <a name="create-an-acl"></a>ACL を作成する
ACL は Windows Admin Center で簡単に作成できます。

:::image type="content" source="./media/access-control-lists/create-acl.png" alt-text="アクセス制御リストの名前のボックスが表示されている Windows Admin Center ホーム画面のスクリーンショット。" lightbox="./media/access-control-lists/create-acl.png":::

1. Windows Admin Center ホーム画面の **[すべての接続]** で、ACL を作成するクラスターを選択します。
1. **[ツール]** の **[ネットワーク]** 領域まで下にスクロールし、 **[アクセス制御リスト]** を選択します。
1. **[アクセス制御リスト]** で、 **[インベントリ]** タブを選択し、 **[新規]** を選択します。
1. **[アクセス制御リスト]** ペインで、ACL の名前を入力し、 **[送信]** を選択します。
1. **[アクセス制御リスト]** で、新しい ACL の **[プロビジョニングの状態]** が **[成功]** と表示されていることを確認します。

## <a name="create-acl-rules"></a>ACL 規則を作成する
ACL を作成したら、ACL 規則を作成する準備が整います。 受信トラフィックと送信トラフィックの両方に ACL 規則を適用する場合は、2 つの規則を作成する必要があります。

:::image type="content" source="./media/access-control-lists/create-acl-rules.png" alt-text="アクセス制御規則のペインが表示されている Windows Admin Center のスクリーンショット。" lightbox="./media/access-control-lists/create-acl-rules.png":::

1. Windows Admin Center ホーム画面の **[すべての接続]** で、ACL を作成するクラスターを選択します。
1. **[ツール]** の **[ネットワーク]** 領域まで下にスクロールし、 **[アクセス制御リスト]** を選択します。
1. **[アクセス制御リスト]** で、 **[インベントリ]** タブを選択し、先ほど作成した ACL を選択します。
1. **[Access Control Rule]\(アクセス制御規則\)** で、 **[新規]** を選択します。
1. **[Access Control Rule]\(アクセス制御規則\)** ペインで、次の情報を指定します。
    1. 規則の **[名前]** 。
    1. 規則の **[優先度]** – 指定できる値は **101** から **65000** までです。 値が小さいほど、優先度が高くなります。
    1. **[種類]** – 受信または送信にすることができます。
    1. **[プロトコル]** – 受信または送信パケットのいずれかに一致するプロトコルを指定します。 指定できる値は、 **[すべて]** 、 **[TCP]** 、 **[UDP]** です。
    1. **[発信元アドレスのプレフィックス]** – 受信または送信パケットのいずれかに一致する発信元アドレスのプレフィックスを指定します。 \* を指定すると、すべての発信元アドレスを表します。
    1. **[Source Port Range]\(発信元ポートの範囲\)** – 受信または送信パケットのいずれかに一致する発信元ポート範囲を指定します。 \* を指定すると、すべての発信元ポートを表します。
    1. **[Destination Address Prefix]\(宛先アドレスのプレフィックス\)** – 受信または送信パケットのいずれかに一致する宛先アドレスのプレフィックスを指定します。 \* を指定すると、すべての宛先アドレスを表します。
    1. **[Destination Port Range]\(宛先ポートの範囲\)** – 受信または送信パケットのいずれかに一致する宛先ポート範囲を指定します。 \* を指定すると、すべての宛先ポートを表します。
    1. **[アクション]** – 上記の条件に一致する場合にパケットを許可するかブロックするかを指定します。 指定できる値は、 **[許可]** と **[拒否]** です。
    1. **[ログ]** – 規則のログ記録を有効にするか無効にするかを指定します。 ログ記録を有効にすると、この規則に一致するすべてのトラフィックがホスト コンピューター上でログに記録されます。
1. **[Submit]\(送信\)** をクリックします。

## <a name="apply-an-acl-to-a-virtual-network"></a>仮想ネットワークに ACL を適用する
ACL とその規則を作成したら、それらを仮想ネットワークのサブネット、論理ネットワークのサブネット、またはネットワーク インターフェイスのいずれかに適用する必要があります。

:::image type="content" source="./media/access-control-lists/apply-acl-virtual-network.png" alt-text="仮想サブネットのペインが表示されている Windows Admin Center のスクリーンショット。" lightbox="./media/access-control-lists/apply-acl-virtual-network.png":::

1. **[ツール]** の下で **[ネットワーク]** 領域まで下にスクロールし、 **[仮想ネットワーク]** を選択します。
1. **[インベントリ]** タブを選択し、仮想ネットワークを選択します。 後続のページで、仮想ネットワークのサブネットを選択し、 **[設定]** を選択します。
1. ドロップダウン リストから ACL を選択し、 **[送信]** を選択します。

    最後の手順を完了すると、ACL が仮想ネットワークのサブネットに関連付けられ、その仮想ネットワークのサブネットに接続されているすべてのコンピューターに適用されます。

## <a name="apply-an-acl-to-a-network-interface"></a>ネットワーク インターフェイスに ACL を適用する
ACL は、仮想マシン (VM) の作成時に、または後から、ネットワー クインターフェイスに適用できます。

:::image type="content" source="./media/access-control-lists/apply-acl-network-interface.png" alt-text="ACL をネットワーク インターフェイスに関連付けるためのネットワーク設定オプションが表示されている Windows Admin Center のスクリーンショット。" lightbox="./media/access-control-lists/apply-acl-network-interface.png":::

1. **[ツール]** の **[ネットワーク]** 領域まで下にスクロールし、 **[仮想マシン]** を選択します。
1. **[インベントリ]** タブを選択し、VM を選択して、 **[設定]** を選択します。
1. **[設定]** ページで **[ネットワーク]** を選択します。
1. 下へスクロールして **[アクセス制御リスト]** に移動し、ドロップダウン リストを展開して、ACL を選択し、 **[Save network settings]\(ネットワーク設定の保存\)** を選択します。

    最後の手順を完了すると、ACL がネットワーク インターフェイスに関連付けられ、ネットワーク インターフェイスのすべての送受信トラフィックに適用されます。

## <a name="get-a-list-of-acls"></a>ACL の一覧を取得する
クラスター内のすべての ACL を簡単に一覧に表示できます。

:::image type="content" source="./media/access-control-lists/get-acl-list.png" alt-text="[インベントリ] タブに ACL の一覧が表示されている Windows Admin Center のスクリーンショット。" lightbox="./media/access-control-lists/get-acl-list.png":::

1. Windows Admin Center ホーム画面の **[すべての接続]** で、ACL の一覧を表示するクラスターを選択します。
1. **[ツール]** の **[ネットワーク]** 領域まで下にスクロールし、 **[アクセス制御リスト]** を選択します。
1. **[インベントリ]** タブには、クラスターに対して使用可能な ACL の一覧が表示されます。また、一覧にある個々の ACL を管理するために使用できるコマンドが示されます。 次の操作を行うことができます。
    - ACL の一覧を表示します。
    - 各 ACL の規則の数、各 ACL に適用されている適用対象サブネットと NIC の数を表示します。
    - 各 ACL の **プロビジョニングの状態** (**成功**、**失敗**) を表示します。
    - ACL を削除します。
    - 一覧で ACL を選択すると、その規則を表示できます。 その後、ACL 規則の設定を追加、削除、または変更できます。

## <a name="delete-an-acl"></a>ACL を削除する
不要になった ACL は削除できます。

>[!NOTE]
> ACL の一覧から ACL を削除した後、サブネットまたはネットワーク インターフェイスのいずれにも関連付けられていないことを確認します。

:::image type="content" source="./media/access-control-lists/delete-acl.png" alt-text="ACL を削除するための削除の確認プロンプトが表示されている Windows Admin Center のスクリーンショット。" lightbox="./media/access-control-lists/delete-acl.png":::

1. **[ツール]** の **[ネットワーク]** 領域まで下にスクロールし、 **[アクセス制御リスト]** を選択します。
1. **[インベントリ]** タブを選択し、一覧にある ACL を選択して、 **[削除]** を選択します。
1. **削除の確認** プロンプトで **[はい]** を選択します。
1. 検索ボックスの横にある **[最新の情報に更新]** を選択して、ACL が削除されていることを確認します。

## <a name="next-steps"></a>次のステップ
詳細については、次のトピックも参照してください。
- [Azure Stack HCI および Windows Server 内のソフトウェアによるネットワーク制御 (SDN)](../concepts/software-defined-networking.md)
