---
title: Azure Automation での変数アセット
description: 変数アセットとは、Azure Automation のすべての Runbook と DSC 構成に使用できる値です。  この記事では、変数の詳細およびテキスト作成とグラフィカル作成の両方で変数を使用する方法について説明します。
services: automation
ms.service: automation
ms.subservice: shared-capabilities
author: mgoedtel
ms.author: magoedte
ms.date: 05/14/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: d4a4a92feb3e1b400c0f40076148f7898c4bdef1
ms.sourcegitcommit: 07d62796de0d1f9c0fa14bfcc425f852fdb08fb1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/27/2020
ms.locfileid: "80365823"
---
# <a name="variable-assets-in-azure-automation"></a>Azure Automation での変数アセット

変数アセットとは、Automation アカウント内のすべての Runbook と DSC 構成に使用できる値です。 それらの管理は、Azure portal または PowerShell を使用して行うか、Runbook または DSC 構成内で行うことができます。

Automation 変数は、次のシナリオで役立ちます。

- 複数の Runbook または DSC 構成で値を共有する。

- 同じ Runbook または DSC 構成の複数のジョブで値を共有する。

- Runbook または DSC 構成によって使用される値を、ポータルまたは PowerShell コマンド ラインから管理する。 例としては、共通する構成アイテムのセット (VM 名の詳細なリスト、特定のリソース グループ、AD ドメイン名など) があります。  

Azure Automation では、変数が保存されるので、Runbook または DSC 構成が失敗した場合でも使用できます。 この動作により、ある Runbook または DSC 構成で設定した値を、別の Runbook で使用したり、次の実行時に同じ Runbook または DSC 構成で使用したりすることができます。

Azure Automation では、暗号化された各変数を安全に保存します。 変数の作成時に、Azure Automation を使用して、セキュリティで保護された資産となるようにその変数の暗号化とストレージを指定できます。 セキュリティで保護されるその他の資産としては、資格情報、証明書、接続などがあります。 Azure Automation では、Automation アカウントごとに生成される一意のキーを使って、これらの資産を暗号化し、保存します。 このキーは、システムで管理されるキー コンテナーに格納されます。 セキュリティで保護された資産を保存する前に、Azure Automation ではキー コンテナーからキーを読み込み、それを使用して資産を暗号化します。 

>[!NOTE]
>この記事は、新しい Azure PowerShell Az モジュールを使用するために更新されました。 AzureRM モジュールはまだ使用でき、少なくとも 2020 年 12 月までは引き続きバグ修正が行われます。 Az モジュールと AzureRM の互換性の詳細については、「[Introducing the new Azure PowerShell Az module (新しい Azure PowerShell Az モジュールの概要)](https://docs.microsoft.com/powershell/azure/new-azureps-module-az?view=azps-3.5.0)」を参照してください。 Hybrid Runbook Worker での Az モジュールのインストール手順については、「[Azure PowerShell モジュールのインストール](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-3.5.0)」を参照してください。 Automation アカウントについては、「[Azure Automation の Azure PowerShell モジュールを更新する方法](../automation-update-azure-modules.md)」に従って、モジュールを最新バージョンに更新できます。

## <a name="variable-types"></a>変数の型

Azure portal を使用して変数を作成する場合、変数値を入力するための適切なコントロールをポータルに表示できるように、ドロップダウン ボックスの一覧からデータ型を指定する必要があります。 Azure Automation で使用できる変数の型を次に示します。

* String
* Integer
* DateTime
* Boolean
* [Null]

変数は、ここに指定されているデータ型に限定されません。 別の型の値を指定する場合は、Windows PowerShell を使用して変数を設定する必要があります。 `Not defined` を指定した場合、変数の値は Null に設定され、[Set-AzAutomationVariable](https://docs.microsoft.com/powershell/module/az.automation/set-azautomationvariable?view=azps-3.5.0) コマンドレットまたは `Set-AutomationVariable` アクティビティを使用して値を設定する必要があります。

Azure portal を使用して、複合型の変数の値を作成したり変更したりすることはできません。 ただし、Windows PowerShell を使用すると、任意の型の値を指定できます。 複合型は [PSCustomObject](/dotnet/api/system.management.automation.pscustomobject) として取得されます。

配列またはハッシュ テーブルを作成し、それを変数に保存することによって、複数の値を 1 つの変数に格納することができます。

## <a name="powershell-cmdlets-that-create-and-manage-variable-assets"></a>変数資産を作成および管理するための PowerShell コマンドレット

Az モジュールの場合、Windows PowerShell で Automation 変数資産を作成および管理するには、次の表のコマンドレットを使用します。 これらのコマンドレットは、Automation Runbook と DSC 構成に使用できる [Az.Automation モジュール](/powershell/azure/overview)に付属しています。

| コマンドレット | 説明 |
|:---|:---|
|[Get-AzAutomationVariable](https://docs.microsoft.com/powershell/module/az.automation/get-azautomationvariable?view=azps-3.5.0) | 既存の変数の値を取得します。 暗号化された変数の値を取得するために、このコマンドレットを使用することはできません。 それを行う唯一の方法は、Runbook または DSC 構成で `Get-AutomationVariable` アクティビティを使用することです。 |
|[New-AzAutomationVariable](https://docs.microsoft.com/powershell/module/az.automation/new-azautomationvariable?view=azps-3.5.0) | 新しい変数を作成し、その値を設定します。|
|[Remove-AzAutomationVariable](https://docs.microsoft.com/powershell/module/az.automation/remove-azautomationvariable?view=azps-3.5.0)| 既存の変数を削除します。|
|[Set-AzAutomationVariable](https://docs.microsoft.com/powershell/module/az.automation/set-azautomationvariable?view=azps-3.5.0)| 既存の変数の値を設定します。 |

## <a name="activities-to-access-variables-in-runbooks-and-dsc-configurations"></a>Runbook および DSC 構成内の変数にアクセスするためのアクティビティ

Runbook および DSC 構成内の変数にアクセスするには、次の表のアクティビティを使用します。 これらのアクティビティのコマンドレットには、グローバル モジュール `Orchestrator.AssetManagement.Cmdlets` が付属しています。

| アクティビティ | 説明 |
|:---|:---|
|`Get-AutomationVariable`|既存の変数の値を取得します。|
|`Set-AutomationVariable`|既存の変数の値を設定します。|

> [!NOTE]
> Runbook または DSC 構成で、`Get-AutomationVariable` の `Name` パラメーターに変数を使用することは避けてください。 このパラメーターを使用すると、デザイン時に、Runbook または DSC 構成と Automation 変数との間の依存関係の検出が複雑になる可能性があります。

`Get-AutomationVariable` は PowerShell では機能しませんが、Runbook または DSC 構成でのみ機能します。 たとえば、暗号化された変数の値を表示するには、Runbook を作成して変数を取得し、それを出力ストリームに書き込むことができます。
 
```powershell
$mytestencryptvar = Get-AutomationVariable -Name TestVariable
Write-output "The encrypted value of the variable is: $mytestencryptvar"
```

## <a name="functions-to-access-variables-in-python-2-runbooks"></a>Python 2 Runbook の変数にアクセスするための関数

Python 2 Runbook の変数にアクセスするには、次の表の関数を使用します。

|Python 2 関数|説明|
|:---|:---|
|`automationassets.get_automation_variable`|既存の変数の値を取得します。 |
|`automationassets.set_automation_variable`|既存の変数の値を設定します。 |

> [!NOTE]
> 資産関数にアクセスするには、お使いの Python Runbook の上部にある `automationassets` モジュールをインポートする必要があります。

## <a name="working-with-automation-variables"></a>Automation 変数の操作

>[!NOTE]
>変数の暗号化を削除する場合は、変数を削除し、暗号化せずに再作成する必要があります。

### <a name="create-a-new-variable-using-the-azure-portal"></a>Azure portal を使用して新しい変数を作成する

1. Automation アカウントから、 **[資産]** タイルをクリックして **[資産]** ブレードで、 **[変数]** を選択します。
2. **[変数]** タイルで、 **[変数の追加]** を選びます。
3. **[新しい変数]** ブレードでオプションを設定してから、 **[作成]** をクリックして新しい変数を保存します。

> [!NOTE]
> 暗号化された変数を保存したら、ポータルでそれを表示することはできません。 更新のみ可能です。

### <a name="create-and-use-a-variable-in-windows-powershell"></a>Windows PowerShell で変数を作成して使用する

PowerShell スクリプトでは、`New-AzAutomationVariable` コマンドレットまたはそれと同等の AzureRM モジュールを使用して、新しい変数を作成し、その初期値を設定します。 変数が暗号化されている場合、呼び出しで `Encrypted` パラメーターを使用する必要があります。

このスクリプトでは [Get-AzAutomationVariable](https://docs.microsoft.com/powershell/module/az.automation/get-azautomationvariable?view=azps-3.5.0) を使用して変数の値を取得します。 値が単純型の場合、コマンドレットではその同じ型が取得されます。 それが複合型の場合は、`PSCustomObject` 型が取得されます。

>[!NOTE]
>PowerShell スクリプトでは、暗号化された値を取得できません。 それを行う唯一の方法は、Runbook または DSC 構成で `Get-AutomationVariable` アクティビティを使用することです。

次の例は、文字列型の変数を作成してから、その値を返す方法を示しています。

```powershell
New-AzAutomationVariable -ResourceGroupName "ResourceGroup01" 
–AutomationAccountName "MyAutomationAccount" –Name 'MyStringVariable' `
–Encrypted $false –Value 'My String'
$string = (Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAccount" –Name 'MyStringVariable').Value
```

次の例は、複合型の変数を作成してから、そのプロパティを取得する方法を示しています。 ここでは、[Get-AzVM](https://docs.microsoft.com/powershell/module/Az.Compute/Get-AzVM?view=azps-3.5.0) の仮想マシン オブジェクトが使用されます。

```powershell
$vm = Get-AzVM -ResourceGroupName "ResourceGroup01" –Name "VM01"
New-AzAutomationVariable –AutomationAccountName "MyAutomationAccount" –Name "MyComplexVariable" –Encrypted $false –Value $vm

$vmValue = (Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAccount" –Name "MyComplexVariable").Value
$vmName = $vmValue.Name
$vmIpAddress = $vmValue.IpAddress
```

### <a name="create-and-use-a-variable-in-a-runbook-or-dsc-configuration"></a>Runbook または DSC 構成で変数を作成して使用する

Runbook または DSC 構成内から新しい変数を作成する唯一の方法は、`New-AzAutomationVariable` コマンドレットまたは、それと同等の AzureRM モジュールを使用することです。 このスクリプトでは、このコマンドレットを使用して、変数の初期値を設定します。 その後、スクリプトでは `Get-AzAutomationVariable` を使用して値を取得できます。 値が単純型である場合、その同じ型が取得されます。 それが複合型の場合は、`PSCustomObject` 型が取得されます。

>[!NOTE]
>暗号化された値を取得する唯一の方法は、Runbook または DSC 構成で `Get-AutomationVariable` アクティビティを使用することです。 

### <a name="textual-runbook-samples"></a>テキスト形式の Runbook のサンプル

#### <a name="set-and-retrieve-a-simple-value-from-a-variable"></a>変数を設定して単純値を取得する

次のサンプル コマンドは、テキスト形式の Runbook で変数を設定し取得する方法を示しています。 このサンプルでは、`NumberOfIterations` と `NumberOfRunnings` という名前の整数型の変数と、`SampleMessage` という名前の文字列変数を作成することを想定しています。

```powershell
$NumberOfIterations = Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" -Name 'NumberOfIterations'
$NumberOfRunnings = Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" -Name 'NumberOfRunnings'
$SampleMessage = Get-AutomationVariable -Name 'SampleMessage'

Write-Output "Runbook has been run $NumberOfRunnings times."

for ($i = 1; $i -le $NumberOfIterations; $i++) {
    Write-Output "$i`: $SampleMessage"
}
Set-AzAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" –Name NumberOfRunnings –Value ($NumberOfRunnings += 1)
```

#### <a name="set-and-retrieve-a-variable-in-a-python-2-runbook"></a>Python 2 Runbook で変数を設定して取得する

次のサンプルは、変数の使用方法、変数の設定方法、および Python 2 Runbook に存在しない変数の例外処理方法を示しています。

```python
import automationassets
from automationassets import AutomationAssetNotFound

# get a variable
value = automationassets.get_automation_variable("test-variable")
print value

# set a variable (value can be int/bool/string)
automationassets.set_automation_variable("test-variable", True)
automationassets.set_automation_variable("test-variable", 4)
automationassets.set_automation_variable("test-variable", "test-string")

# handle a non-existent variable exception
try:
    value = automationassets.get_automation_variable("nonexisting variable")
except AutomationAssetNotFound:
    print "variable not found"
```

### <a name="graphical-runbook-samples"></a>グラフィカルな Runbook のサンプル

グラフィカル Runbook では、`Get-AutomationVariable` または `Set-AutomationVariable` アクティビティを追加できます。 グラフィカル エディターの [ライブラリ] ペインで変数を右クリックし、目的のアクティビティを選択するだけです。

![キャンバスへの変数の追加](../media/variables/runbook-variable-add-canvas.png)

#### <a name="set-values-in-a-variable"></a>変数に値を設定する

次の図は、グラフィカルな Runbook の単純値で変数を更新するサンプル アクティビティを示しています。 このサンプルでは、`Get-AzVM` は、1 つの Azure 仮想マシンを取得し、コンピューター名を既存の Automation 文字列変数に保存します。 このコードでは出力に 1 つのオブジェクトのみが含まれると予期されるため、[リンクがパイプラインとシーケンス](../automation-graphical-authoring-intro.md#links-and-workflow)のいずれであっても関係ありません。

![単純変数の設定](../media/variables/runbook-set-simple-variable.png)

## <a name="next-steps"></a>次のステップ

- グラフィカル作成でアクティビティを接続する方法については、[グラフィカル作成でのリンク](../automation-graphical-authoring-intro.md#links-and-workflow)に関するページをご覧ください。
- グラフィカル Runbook の使用を開始するには、「[初めてのグラフィカルな Runbook](../automation-first-runbook-graphical.md)」を参照してください。
