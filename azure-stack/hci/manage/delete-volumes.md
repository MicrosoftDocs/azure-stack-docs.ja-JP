---
title: Azure Stack HCI および Windows Server のボリュームを削除する
description: Windows Admin Center と PowerShell を使用して Azure Stack HCI および Windows Server のボリュームを削除する方法について説明します。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 03/09/2021
ms.openlocfilehash: fdc7fd4ae102871d534d846dd653af077a75d49a
ms.sourcegitcommit: 02a4c34fb829e053016912a4fffcc51e32685425
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102532488"
---
# <a name="deleting-volumes-in-azure-stack-hci-and-windows-server"></a>Azure Stack HCI および Windows Server のボリュームを削除する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

このトピックでは、Windows Admin Center または PowerShell のいずれかを使用してボリュームを削除する手順について説明します。

## <a name="delete-volumes-with-windows-admin-center"></a>Windows Admin Center を使用してボリュームを削除する

1. Windows Admin Center で、クラスターに接続し、左側の **[ツール]** ウィンドウで **[ボリューム]** を選択します。
2. **[ボリューム]** ページで **[インベントリ]** タブを選択し、削除するボリュームを選択します。
3. ボリュームの詳細ページの上部で、 **[削除]** を選択します。
4. 確認のダイアログで、ボリュームの削除を確認するチェック ボックスをオンにして、 **[削除]** を選択します。

   :::image type="content" source="media/delete-volumes/delete-volume.png" alt-text="削除するボリュームを選択し、[削除] を選択し、ボリューム上のすべてのデータを消去することを確認します。" lightbox="media/delete-volumes/delete-volume.png":::

## <a name="delete-volumes-with-powershell"></a>PowerShell を使用してボリュームを削除する

**VirtualDisk** オブジェクトを削除し、それが使用していた領域を、**VirtualDisk** オブジェクトを公開する記憶域プールに返すには、**Remove-VirtualDisk** コマンドレットを使用します。

まず、管理 PC で PowerShell を起動し、**Get-VirtualDisk** コマンドレットを **CimSession** パラメーターを指定して実行します。このパラメーターは、クラスターまたはサーバー ノードの名前 (例: *clustername.contoso.com*) です。

```PowerShell
Get-VirtualDisk -CimSession clustername.contoso.com
```

これにより、クラスター上のボリューム名に対応する、 **-FriendlyName** パラメーターに使用可能な値の一覧が返されます。

### <a name="example"></a>例

*Volume1* というミラー ボリュームを削除するには、PowerShell で次のコマンドを実行します。

```PowerShell
Remove-VirtualDisk -FriendlyName "Volume1"
```

操作を実行してボリュームに含まれるすべてのデータを消去するかどうかを確認するメッセージが表示されます。 Y または N を選択します。

   > [!WARNING]
   > これは回復可能な操作ではありません。 この例では、**VirtualDisk** Volume オブジェクトが完全に削除されます。

## <a name="next-steps"></a>次のステップ

その他の重要な記憶域管理タスクの詳細な手順については、以下も参照してください。

- [ボリュームを計画する](../concepts/plan-volumes.md)
- [ボリュームを作成する](create-volumes.md)
- [ボリュームを拡張する](extend-volumes.md)