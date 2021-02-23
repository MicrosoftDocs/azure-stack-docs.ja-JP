---
title: データ センターのファイアウォールを使用して PowerShell で ACL を構成する
description: このトピックを使用して、Azure Stack HCI、Windows Server 2019、および Windows Server 2016 内でソフトウェアによるネットワーク制御に対するデータセンター ファイアウォールの使用を開始します。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 02/09/2021
ms.openlocfilehash: 38f7511d4ccc11c4801abde82ec8ab1c7091eb69
ms.sourcegitcommit: 34babe5abf1bceee718011b5c5c25f75e1b03b0c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/17/2021
ms.locfileid: "100562645"
---
# <a name="use-datacenter-firewall-to-configure-acls-with-powershell"></a>データ センターのファイアウォールを使用して PowerShell で ACL を構成する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019、Windows Server 2016

このトピックでは、Windows PowerShell を使用して、Azure Stack HCI でソフトウェア定義ネットワーク (SDN) の[データセンター ファイアウォール](../concepts/datacenter-firewall-overview.md)によってデータ トラフィック フローを管理するためのアクセス制御リスト (ACL) を構成する手順について説明します。 データセンター ファイアウォールを有効にして構成するには、サブネットまたはネットワーク インターフェイスに適用される ACL を作成します。 このトピックのサンプル スクリプトでは、**NetworkController** モジュールからエクスポートされた Windows PowerShell コマンドを使用します。 Windows Admin Center を使用して ACL を構成および管理することもできます。

## <a name="configure-datacenter-firewall-to-allow-all-traffic"></a>すべてのトラフィックを許可するようにデータセンター ファイアウォールを構成する

SDN をデプロイしたら、新しい環境で基本的なネットワーク接続をテストする必要があります。 これを行うには、制限なしですべてのネットワーク トラフィックを許可するデータセンター ファイアウォールの規則を作成します。

次の表の項目を使用して、すべての受信および送信ネットワーク トラフィックを許可する一連の規則を作成します。

| 発信元 IP | 宛先 IP | Protocol | 発信元ポート | 宛先ポート | Direction | アクション | Priority |
|:---------:|:--------------:|:--------:|:-----------:|:----------------:|:---------:|:------:|:--------:|
|    \*     |       \*       |   All    |     \*      |        \*        |  受信  | Allow  |   100    |
|    \*     |       \*       |   All    |     \*      |        \*        | 送信  | Allow  |   110    |

---

この例では、次の 2 つの規則を持つ ACL を作成します。

1. **AllowAll_Inbound** - この ACL が構成されているネットワーク インターフェイスへのすべてのネットワーク トラフィックの通過を許可します。
2. **AllowAllOutbound** - すべてのトラフィックにネットワーク インターフェースの外部に渡されることを許可します。 リソース ID "AllowAll-1" で識別されるこの ACL が、仮想サブネットとネットワーク インターフェイスで使用できるようになりました。

まず、PowerShell セッションを開いて、いずれかのクラスター ノードに接続します。

```PowerShell
Enter-PSSession <server-name>
```

次に、以下のスクリプトを実行して ACL を作成します。

```PowerShell
$ruleproperties = new-object Microsoft.Windows.NetworkController.AclRuleProperties
$ruleproperties.Protocol = "All"
$ruleproperties.SourcePortRange = "0-65535"
$ruleproperties.DestinationPortRange = "0-65535"
$ruleproperties.Action = "Allow"
$ruleproperties.SourceAddressPrefix = "*"
$ruleproperties.DestinationAddressPrefix = "*"
$ruleproperties.Priority = "100"
$ruleproperties.Type = "Inbound"
$ruleproperties.Logging = "Enabled"
$aclrule1 = new-object Microsoft.Windows.NetworkController.AclRule
$aclrule1.Properties = $ruleproperties
$aclrule1.ResourceId = "AllowAll_Inbound"
$ruleproperties = new-object Microsoft.Windows.NetworkController.AclRuleProperties
$ruleproperties.Protocol = "All"
$ruleproperties.SourcePortRange = "0-65535"
$ruleproperties.DestinationPortRange = "0-65535"
$ruleproperties.Action = "Allow"
$ruleproperties.SourceAddressPrefix = "*"
$ruleproperties.DestinationAddressPrefix = "*"
$ruleproperties.Priority = "110"
$ruleproperties.Type = "Outbound"
$ruleproperties.Logging = "Enabled"
$aclrule2 = new-object Microsoft.Windows.NetworkController.AclRule
$aclrule2.Properties = $ruleproperties
$aclrule2.ResourceId = "AllowAll_Outbound"
$acllistproperties = new-object Microsoft.Windows.NetworkController.AccessControlListProperties
$acllistproperties.AclRules = @($aclrule1, $aclrule2)
New-NetworkControllerAccessControlList -ResourceId "AllowAll" -Properties $acllistproperties -ConnectionUri <NC REST FQDN>
```

>[!NOTE]
>ネットワーク コントローラーの Windows PowerShell コマンド リファレンスについては、[ネットワーク コントローラーのコマンドレット](/powershell/module/networkcontroller/)に関するトピックを参照してください。

## <a name="use-acls-to-limit-traffic-on-a-subnet"></a>ACL を使用してサブネット上のトラフィックを制限する
この例では、192.168.0.0/24 サブネット内の仮想マシン (VM) が相互に通信できないようにする ACL を作成します。 この種類の ACL は、攻撃者がサブネット内で水平展開することを制限しつつ、VM でサブネット外からの要求を受け取ったり、他のサブネット上の他のサービスと通信したりすることを引き続き許容するのに便利です。

|   発信元 IP    | 宛先 IP | Protocol | 発信元ポート | 宛先ポート | Direction | アクション | Priority |
|:--------------:|:--------------:|:--------:|:-----------:|:----------------:|:---------:|:------:|:--------:|
|  192.168.0.1   |       \*       |   All    |     \*      |        \*        |  受信  | Allow  |   100    |
|       \*       |  192.168.0.1   |   All    |     \*      |        \*        | 送信  | Allow  |   101    |
| 192.168.0.0/24 |       \*       |   All    |     \*      |        \*        |  受信  | ブロック  |   102    |
|       \*       | 192.168.0.0/24 |   All    |     \*      |        \*        | 送信  | ブロック  |   103    |
|       \*       |       \*       |   All    |     \*      |        \*        |  受信  | Allow  |   104    |
|       \*       |       \*       |   All    |     \*      |        \*        | 送信  | Allow  |   105    |

---

次のサンプル スクリプトを使用して作成された、リソース ID **Subnet-192-168-0-0** で識別される ACL を、"192.168.0.0/24" サブネット アドレスを使用する仮想ネットワーク サブネットに適用できるようになりました。 この仮想ネットワーク サブネットに接続されているすべてのネットワーク インターフェイスには、自動的に上記の ACL 規則が適用されます。

ネットワーク コントローラー REST API を使用してこの ACL を作成するスクリプトの例を次に示します。

```PowerShell
import-module networkcontroller
$ncURI = "https://mync.contoso.local"
$aclrules = @()

$ruleproperties = new-object Microsoft.Windows.NetworkController.AclRuleProperties
$ruleproperties.Protocol = "All"
$ruleproperties.SourcePortRange = "0-65535"
$ruleproperties.DestinationPortRange = "0-65535"
$ruleproperties.Action = "Allow"
$ruleproperties.SourceAddressPrefix = "192.168.0.1"
$ruleproperties.DestinationAddressPrefix = "*"
$ruleproperties.Priority = "100"
$ruleproperties.Type = "Inbound"
$ruleproperties.Logging = "Enabled"

$aclrule = new-object Microsoft.Windows.NetworkController.AclRule
$aclrule.Properties = $ruleproperties
$aclrule.ResourceId = "AllowRouter_Inbound"
$aclrules += $aclrule

$ruleproperties = new-object Microsoft.Windows.NetworkController.AclRuleProperties
$ruleproperties.Protocol = "All"
$ruleproperties.SourcePortRange = "0-65535"
$ruleproperties.DestinationPortRange = "0-65535"
$ruleproperties.Action = "Allow"
$ruleproperties.SourceAddressPrefix = "*"
$ruleproperties.DestinationAddressPrefix = "192.168.0.1"
$ruleproperties.Priority = "101"
$ruleproperties.Type = "Outbound"
$ruleproperties.Logging = "Enabled"

$aclrule = new-object Microsoft.Windows.NetworkController.AclRule
$aclrule.Properties = $ruleproperties
$aclrule.ResourceId = "AllowRouter_Outbound"
$aclrules += $aclrule

$ruleproperties = new-object Microsoft.Windows.NetworkController.AclRuleProperties
$ruleproperties.Protocol = "All"
$ruleproperties.SourcePortRange = "0-65535"
$ruleproperties.DestinationPortRange = "0-65535"
$ruleproperties.Action = "Deny"
$ruleproperties.SourceAddressPrefix = "192.168.0.0/24"
$ruleproperties.DestinationAddressPrefix = "*"
$ruleproperties.Priority = "102"
$ruleproperties.Type = "Inbound"
$ruleproperties.Logging = "Enabled"

$aclrule = new-object Microsoft.Windows.NetworkController.AclRule
$aclrule.Properties = $ruleproperties
$aclrule.ResourceId = "DenySubnet_Inbound"
$aclrules += $aclrule

$ruleproperties = new-object Microsoft.Windows.NetworkController.AclRuleProperties
$ruleproperties.Protocol = "All"
$ruleproperties.SourcePortRange = "0-65535"
$ruleproperties.DestinationPortRange = "0-65535"
$ruleproperties.Action = "Deny"
$ruleproperties.SourceAddressPrefix = "*"
$ruleproperties.DestinationAddressPrefix = "192.168.0.0/24"
$ruleproperties.Priority = "103"
$ruleproperties.Type = "Outbound"
$ruleproperties.Logging = "Enabled"

$aclrule = new-object Microsoft.Windows.NetworkController.AclRule
$aclrule.Properties = $ruleproperties
$aclrule.ResourceId = "DenySubnet_Outbound"

$ruleproperties = new-object Microsoft.Windows.NetworkController.AclRuleProperties
$ruleproperties.Protocol = "All"
$ruleproperties.SourcePortRange = "0-65535"
$ruleproperties.DestinationPortRange = "0-65535"
$ruleproperties.Action = "Allow"
$ruleproperties.SourceAddressPrefix = "*"
$ruleproperties.DestinationAddressPrefix = "*"
$ruleproperties.Priority = "104"
$ruleproperties.Type = "Inbound"
$ruleproperties.Logging = "Enabled"

$aclrule = new-object Microsoft.Windows.NetworkController.AclRule
$aclrule.Properties = $ruleproperties
$aclrule.ResourceId = "AllowAll_Inbound"
$aclrules += $aclrule

$ruleproperties = new-object Microsoft.Windows.NetworkController.AclRuleProperties
$ruleproperties.Protocol = "All"
$ruleproperties.SourcePortRange = "0-65535"
$ruleproperties.DestinationPortRange = "0-65535"
$ruleproperties.Action = "Allow"
$ruleproperties.SourceAddressPrefix = "*"
$ruleproperties.DestinationAddressPrefix = "*"
$ruleproperties.Priority = "105"
$ruleproperties.Type = "Outbound"
$ruleproperties.Logging = "Enabled"

$aclrule = new-object Microsoft.Windows.NetworkController.AclRule
$aclrule.Properties = $ruleproperties
$aclrule.ResourceId = "AllowAll_Outbound"
$aclrules += $aclrule

$acllistproperties = new-object Microsoft.Windows.NetworkController.AccessControlListProperties
$acllistproperties.AclRules = $aclrules

New-NetworkControllerAccessControlList -ResourceId "Subnet-192-168-0-0" -Properties $acllistproperties -ConnectionUri $ncURI
```

## <a name="add-an-acl-to-a-network-interface"></a>ネットワーク インターフェイスに ACL を追加する

ACL を作成して仮想サブネットに割り当てたら、仮想サブネット上の既定の ACL を個々のネットワーク インターフェイス用に特定の ACL で上書きすることができます。 Windows Server 2019 Datacenter 以降では、SDN 仮想ネットワークに加えて、SDN 論理ネットワークに接続されたネットワーク インターフェイスに特定の ACL を直接適用できます。 ネットワーク インターフェイスに接続された仮想サブネットに ACL が設定されている場合、両方の ACL が適用され、仮想サブネットの ACL よりもネットワーク インターフェイスの ACL が優先されます。

この例では、仮想ネットワークに ACL を追加する方法を示します。

>[!TIP]
>また、ネットワーク インターフェイスを作成するときに ACL を追加することもできます。

1. ACL を追加するネットワーク インターフェイスを取得または作成します。

   ```PowerShell
   $nic = get-networkcontrollernetworkinterface -ConnectionUri $uri -ResourceId "MyVM_Ethernet1"
   ```

2. ネットワーク インターフェイスに追加する ACL を取得または作成します。

   ```PowerShell
   $acl = get-networkcontrolleraccesscontrollist -ConnectionUri $uri -ResourceId "AllowAllACL"
   ```

3. ネットワーク インターフェイスの AccessControlList プロパティに ACL を割り当てます。

   ```PowerShell
    $nic.properties.ipconfigurations[0].properties.AccessControlList = $acl
   ```

4. ネットワーク コントローラーにネットワーク インターフェイスを追加します。

   ```PowerShell
   new-networkcontrollernetworkinterface -ConnectionUri $uri -Properties $nic.properties -ResourceId $nic.resourceid
   ```

## <a name="remove-an-acl-from-a-network-interface"></a>ネットワーク インターフェイスから ACL を削除する

この例では、ネットワーク インターフェイスから ACL を削除する方法について説明します。 ACL を削除すると、既定のルール セットがネットワーク インターフェイスに適用されます。 既定のルール セットでは、すべての送信トラフィックが許可されますが、すべての受信トラフィックはブロックされます。 すべての受信トラフィックを許可する場合は、前の[例](#add-an-acl-to-a-network-interface)に従って、すべての受信および送信トラフィックを許可する ACL を追加する必要があります。

1. ACL を削除するネットワーク インターフェイスを取得します。
   ```PowerShell
   $nic = get-networkcontrollernetworkinterface -ConnectionUri $uri -ResourceId "MyVM_Ethernet1"
   ```

2. ipConfiguration の AccessControlList プロパティに $null を割り当てます。
   ```PowerShell
   $nic.properties.ipconfigurations[0].properties.AccessControlList = $null
   ```

3. ネットワーク コントローラーにネットワーク インターフェイス オブジェクトを追加します。
   ```PowerShell
   new-networkcontrollernetworkinterface -ConnectionUri $uri -Properties $nic.properties -ResourceId $nic.resourceid
   ```

## <a name="firewall-auditing"></a>ファイアウォールの監査

Windows Server 2019 で導入されたファイアウォールの監査は、SDN ファイアウォール規則によって処理されるすべてのフローを記録するデータセンター ファイアウォールの新機能です。 ログが有効になっているすべての ACL が記録されます。 ログ ファイルは、[Azure Network Watcher フロー ログ](/azure/network-watcher/network-watcher-nsg-flow-logging-overview)と一致する構文に含まれている必要があります。 これらのログは、診断に使用したり、後で分析するためにアーカイブしたりすることができます。

ホスト サーバーでファイアウォールの監査を有効にするサンプル スクリプトを次に示します。 最初にある変数を更新し、[ネットワーク コントローラー](../concepts/network-controller-overview.md)が展開された Azure Stack HCI クラスターでこれを実行します。

```PowerShell
$logpath = "C:\test\log1"
$servers = @("sa18n22-2", "sa18n22-3", "sa18n22-4")
$uri = "https://sa18n22sdn.sa18.nttest.microsoft.com"

# Create log directories on the hosts
invoke-command -Computername $servers  {
    param(
        $Path
    )
    mkdir $path    -force
} -argumentlist $LogPath

# Set firewall auditing settings on Network Controller
$AuditProperties = new-object Microsoft.Windows.NetworkController.AuditingSettingsProperties
$AuditProperties.OutputDirectory = $logpath
set-networkcontrollerauditingsettingsconfiguration -connectionuri $uri -properties $AuditProperties -force  | out-null

# Enable logging on each server
$servers = get-networkcontrollerserver -connectionuri $uri
foreach ($s in $servers) {
    $s.properties.AuditingEnabled = @("Firewall")
    new-networkcontrollerserver -connectionuri $uri -resourceid $s.resourceid -properties $s.properties -force | out-null
}
```

有効にすると、各ホストの指定したディレクトリに新しいファイルが 1 時間ごとに 1 つずつ追加されます。  これらのファイルを定期的に処理し、ホストから削除する必要があります。  現在のファイルは長さが 0 で、次の 1 時間にフラッシュされるまでロックされます。

```syntax
PS C:\test\log1> dir

    Directory: C:\test\log1

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/19/2018   6:28 AM          17055 SdnFirewallAuditing.d8b3b697-5355-40e2-84d2-1bf2f0e0dc4a.20180719TL122803093.json
-a----        7/19/2018   7:28 AM           7880 SdnFirewallAuditing.d8b3b697-5355-40e2-84d2-1bf2f0e0dc4a.20180719TL132803173.json
-a----        7/19/2018   8:28 AM           7867 SdnFirewallAuditing.d8b3b697-5355-40e2-84d2-1bf2f0e0dc4a.20180719TL142803264.json
-a----        7/19/2018   9:28 AM          10949 SdnFirewallAuditing.d8b3b697-5355-40e2-84d2-1bf2f0e0dc4a.20180719TL152803360.json
-a----        7/19/2018   9:28 AM              0 SdnFirewallAuditing.d8b3b697-5355-40e2-84d2-1bf2f0e0dc4a.20180719TL162803464.json
```

これらのファイルには、フロー イベントのシーケンスが含まれています。次に例を示します。

```syntax
{
    "records": [
        {
            "properties":{
                "Version":"1.0",
                "flows":[
                    {
                        "flows":[
                            {
                                "flowTuples":["1531963580,192.122.0.22,192.122.255.255,138,138,U,I,A"],
                                "portId":"9",
                                "portName":"7290436D-0422-498A-8EB8-C6CF5115DACE"
                            }
                        ],
                        "rule":"Allow_Inbound"
                    }
                ]
            },
            "operationName":"NetworkSecurityGroupFlowEvents",
            "resourceId":"394f647d-2ed0-4c31-87c5-389b8c0c8132",
            "time":"20180719:L012620622",
            "category":"NetworkSecurityGroupFlowEvent",
            "systemId":"d8b3b697-5355-40e2-84d2-1bf2f0e0dc4a"
            },
```

ログ記録が行われるのは、**Logging** が **Enabled** に設定されているルールだけです。次に例を示します。

```syntax
{
    "Tags":  null,
    "ResourceRef":  "/accessControlLists/AllowAll",
    "InstanceId":  "4a63e1a5-3264-4986-9a59-4e77a8b107fa",
    "Etag":  "W/\"1535a780-0fc8-4bba-a15a-093ecac9b88b\"",
    "ResourceMetadata":  null,
    "ResourceId":  "AllowAll",
    "Properties":  {
                       "ConfigurationState":  null,
                       "ProvisioningState":  "Succeeded",
                       "AclRules":  [
                                        {
                                            "ResourceMetadata":  null,
                                            "ResourceRef":  "/accessControlLists/AllowAll/aclRules/AllowAll_Inbound",
                                            "InstanceId":  "ba8710a8-0f01-422b-9038-d1f2390645d7",
                                            "Etag":  "W/\"1535a780-0fc8-4bba-a15a-093ecac9b88b\"",
                                            "ResourceId":  "AllowAll_Inbound",
                                            "Properties":  {
                                                               "Protocol":  "All",
                                                               "SourcePortRange":  "0-65535",
                                                               "DestinationPortRange":  "0-65535",
                                                               "Action":  "Allow",
                                                               "SourceAddressPrefix":  "*",
                                                               "DestinationAddressPrefix":  "*",
                                                               "Priority":  "101",
                                                               "Description":  null,
                                                               "Type":  "Inbound",
                                                               "Logging":  "Enabled",
                                                               "ProvisioningState":  "Succeeded"
                                                           }
                                        },
                                        {
                                            "ResourceMetadata":  null,
                                            "ResourceRef":  "/accessControlLists/AllowAll/aclRules/AllowAll_Outbound",
                                            "InstanceId":  "068264c6-2186-4dbc-bbe7-f504c6f47fa8",
                                            "Etag":  "W/\"1535a780-0fc8-4bba-a15a-093ecac9b88b\"",
                                            "ResourceId":  "AllowAll_Outbound",
                                            "Properties":  {
                                                               "Protocol":  "All",
                                                               "SourcePortRange":  "0-65535",
                                                               "DestinationPortRange":  "0-65535",
                                                               "Action":  "Allow",
                                                               "SourceAddressPrefix":  "*",
                                                               "DestinationAddressPrefix":  "*",
                                                               "Priority":  "110",
                                                               "Description":  null,
                                                               "Type":  "Outbound",
                                                               "Logging":  "Enabled",
                                                               "ProvisioningState":  "Succeeded"
                                                           }
                                        }
                                    ],
                       "IpConfigurations":  [

                                            ],
                       "Subnets":  [
                                       {
                                           "ResourceMetadata":  null,
                                           "ResourceRef":  "/virtualNetworks/10_0_1_0/subnets/Subnet1",
                                           "InstanceId":  "00000000-0000-0000-0000-000000000000",
                                           "Etag":  null,
                                           "ResourceId":  null,
                                           "Properties":  null
                                       }
                                   ]
                   }
}
```

## <a name="next-steps"></a>次のステップ

関連情報については、以下もご覧ください。

- [データセンター ファイアウォールの概要](../concepts/datacenter-firewall-overview.md)
- [ネットワーク コント ローラーの概要](../concepts/network-controller-overview.md)
- [Azure Stack HCI および Windows Server 内の SDN](../concepts/software-defined-networking.md)