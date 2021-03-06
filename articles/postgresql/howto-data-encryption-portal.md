---
title: Azure portal を使用した Azure Database for PostgreSQL 単一サーバーのデータ暗号化
description: Azure portal を使用して Azure Database for PostgreSQL 単一サーバーのデータ暗号化を設定および管理する方法について説明します。
author: kummanish
ms.author: manishku
ms.service: postgresql
ms.topic: conceptual
ms.date: 01/13/2020
ms.openlocfilehash: 6028f5e618b4b480a2259241fc2380f0200cebc6
ms.sourcegitcommit: 668b3480cb637c53534642adcee95d687578769a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/07/2020
ms.locfileid: "78898363"
---
# <a name="data-encryption-for-azure-database-for-postgresql-single-server-by-using-the-azure-portal"></a>Azure portal を使用した Azure Database for PostgreSQL 単一サーバーのデータ暗号化

Azure portal を使用して Azure Database for PostgreSQL 単一サーバーのデータ暗号化を設定および管理する方法について説明します。

## <a name="prerequisites-for-azure-cli"></a>Azure CLI の前提条件

* Azure サブスクリプションがあり、そのサブスクリプションの管理者である必要があります。
* Azure Key Vault で、カスタマー マネージド キーに使用するキー コンテナーとキーを作成します。
* カスタマー マネージド キーとして使用するには、キー コンテナーに次のプロパティが必要です。
  * [論理的な削除](../key-vault/key-vault-ovw-soft-delete.md)

    ```azurecli-interactive
    az resource update --id $(az keyvault show --name \ <key_vault_name> -test -o tsv | awk '{print $1}') --set \ properties.enableSoftDelete=true
    ```

  * [消去保護](../key-vault/key-vault-ovw-soft-delete.md#purge-protection)

    ```azurecli-interactive
    az keyvault update --name <key_vault_name> --resource-group <resource_group_name>  --enable-purge-protection true
    ```

* カスタマー マネージド キーとして使用するには、キーに次の属性が必要です。
  * 有効期限がない
  * 無効化されていない
  * [取得]、[キーを折り返す]、[キーの折り返しを解除] の各操作を実行できる

## <a name="set-the-right-permissions-for-key-operations"></a>キー操作に対する適切なアクセス許可を設定する

1. Key Vault で、 **[アクセス ポリシー]**  >  **[アクセス ポリシーの追加]** の順に選択します。

   ![[アクセス ポリシー] と [アクセスポリシーの追加] が強調表示されている Key Vault のスクリーンショット](media/concepts-data-access-and-security-data-encryption/show-access-policy-overview.png)

2. **[キーのアクセス許可]** を選択し、 **[取得]** 、 **[キーを折り返す]** 、 **[キーの折り返しを解除]** 、および PostgreSQL サーバーの名前である**プリンシパル**を選択します。 お使いのサーバー プリンシパルが既存のプリンシパルの一覧に見つからない場合は、登録する必要があります。 初めてデータの暗号化を設定しようとして失敗したときに、サーバー プリンシパルを登録するように求められます。  

   ![アクセス ポリシーの概要](media/concepts-data-access-and-security-data-encryption/access-policy-wrap-unwrap.png)

3. **[保存]** を選択します。

## <a name="set-data-encryption-for-azure-database-for-postgresql-single-server"></a>Azure Database for PostgreSQL 単一サーバーのデータ暗号化を設定する

1. Azure Database for PostgreSQL で、 **[データの暗号化]** を選択して、カスタマー マネージド キーを設定します。

   ![[データの暗号化] が強調表示されている Azure Database for PostgreSQL のスクリーンショット](media/concepts-data-access-and-security-data-encryption/data-encryption-overview.png)

2. キー コンテナー とキーの組を選択するか、キー識別子を入力することができます。

   ![データの暗号化のオプションが強調表示されている Azure Database for PostgreSQL のスクリーンショット](media/concepts-data-access-and-security-data-encryption/setting-data-encryption.png)

3. **[保存]** を選択します。

4. 確実にすべてのファイル (一時ファイルを含む) が完全に暗号化されるように、サーバーを再起動します。

## <a name="restore-or-create-a-replica-of-the-server"></a>サーバーのレプリカを復元または作成する

Key Vault に格納されている顧客のマネージド キーで Azure Database for PostgreSQL 単一サーバーが暗号化されると、そのサーバーの新しく作成されたコピーも暗号化されます。 この新しいコピーは、ローカルまたは geo 復元操作を使用するか、レプリカ (ローカル/リージョン間) 操作を使用して作成できます。 そのため、暗号化された PostgreSQL サーバーの場合は、次の手順を使用して、暗号化済みの復元されたサーバーを作成できます。

1. サーバーで、 **[概要]**  >  **[復元]** の順に選択します。

   ![[概要] と [復元] が強調表示されている Azure Database for PostgreSQL のスクリーンショット](media/concepts-data-access-and-security-data-encryption/show-restore.png)

   または、レプリケーションが有効なサーバーでは、 **[設定]** 見出しの下にある **[レプリケーション]** を選択します。

   ![[レプリケーション] が強調表示されている Azure Database for PostgreSQL のスクリーンショット](media/concepts-data-access-and-security-data-encryption/postgresql-replica.png)

2. 復元操作が完了すると、作成された新しいサーバーは、プライマリ サーバーのキーで暗号化されます。 ただし、このサーバーの機能とオプションは無効になっており、サーバーにアクセスできません。 新しいサーバーの ID には、キー コンテナーへのアクセス許可がまだ付与されていないため、これによってデータ操作が抑止されます。

   ![[アクセス不可] 状態が強調表示されている Azure Database for PostgreSQL のスクリーンショット](media/concepts-data-access-and-security-data-encryption/show-restore-data-encryption.png)

3. サーバーにアクセスできるようにするには、復元されたサーバーでキーを再検証します。 **[データの暗号化]**  >  **[キーの再検証]** の順に選択します。

   > [!NOTE]
   > 新しいサーバーのサービス プリンシパルにキー コンテナーへのアクセス権を付与する必要があるため、最初の再検証の試行は失敗します。 サービス プリンシパルを生成するには、 **[キーの再検証]** を選択します。これにより、エラーが表示されますが、サービス プリンシパルが生成されます。 その後、この記事で前述した[こちらの手順](#set-the-right-permissions-for-key-operations)を参照してください。

   ![再検証の手順が強調表示されている Azure Database for PostgreSQL のスクリーンショット](media/concepts-data-access-and-security-data-encryption/show-revalidate-data-encryption.png)

   新しいサーバーへのアクセス権をキー コンテナーに付与する必要があります。

4. サービス プリンシパルを登録した後、もう一度キーを再検証すると、サーバーの通常の機能が再開されます。

   ![復元された機能を示している Azure Database for PostgreSQL のスクリーンショット](media/concepts-data-access-and-security-data-encryption/restore-successful.png)

## <a name="using-an-azure-resource-manager-template-to-enable-data-encryption"></a>Azure Resource Manager テンプレートを使用したデータ暗号化の有効化

Azure portal とは別に、新規および既存のサーバー用の Azure Resource Manager テンプレートを使用して、Azure Database for PostgreSQL の単一サーバー上でデータ暗号化を有効にすることもできます。

### <a name="for-a-new-server"></a>新しいサーバーの場合

事前に作成したいずれかの Azure Resource Manager テンプレートを使用して、データ暗号化を有効にしてサーバーをプロビジョニングします。[データの暗号化を使用した例](https://github.com/Azure/azure-postgresql/tree/master/arm-templates/ExampleWithDataEncryption)

この Azure Resource Manager テンプレートでは、Azure Database for PostgreSQL の単一サーバーを作成し、パラメーターとして渡された **KeyVault** および **Key** を使用して、サーバーのデータ暗号化を有効にします。

### <a name="for-an-existing-server"></a>既存のサーバーの場合
また、Azure Resource Manager テンプレートを使用して、既存の Azure Database for PostgreSQL 単一サーバー上でデータ暗号化を有効にすることもできます。

* properties オブジェクトの `keyVaultKeyUri` プロパティで以前コピーした Azure Key Vault キーの URI を渡します。

* API バージョンとして *2020-01-01-preview* を使用します。

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string"
    },
    "serverName": {
      "type": "string"
    },
    "keyVaultName": {
      "type": "string",
      "metadata": {
        "description": "Key vault name where the key to use is stored"
      }
    },
    "keyVaultResourceGroupName": {
      "type": "string",
      "metadata": {
        "description": "Key vault resource group name where it is stored"
      }
    },
    "keyName": {
      "type": "string",
      "metadata": {
        "description": "Key name in the key vault to use as encryption protector"
      }
    },
    "keyVersion": {
      "type": "string",
      "metadata": {
        "description": "Version of the key in the key vault to use as encryption protector"
      }
    }
  },
  "variables": {
    "serverKeyName": "[concat(parameters('keyVaultName'), '_', parameters('keyName'), '_', parameters('keyVersion'))]"
  },
  "resources": [
    {
      "type": "Microsoft.DBforPostgreSQL/servers",
      "apiVersion": "2017-12-01",
      "kind": "",
      "location": "[parameters('location')]",
      "identity": {
        "type": "SystemAssigned"
      },
      "name": "[parameters('serverName')]",
      "properties": {
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-05-01",
      "name": "addAccessPolicy",
      "resourceGroup": "[parameters('keyVaultResourceGroupName')]",
      "dependsOn": [
        "[resourceId('Microsoft.DBforPostgreSQL/servers', parameters('serverName'))]"
      ],
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "resources": [
            {
              "type": "Microsoft.KeyVault/vaults/accessPolicies",
              "name": "[concat(parameters('keyVaultName'), '/add')]",
              "apiVersion": "2018-02-14-preview",
              "properties": {
                "accessPolicies": [
                  {
                    "tenantId": "[subscription().tenantId]",
                    "objectId": "[reference(resourceId('Microsoft.DBforPostgreSQL/servers/', parameters('serverName')), '2017-12-01', 'Full').identity.principalId]",
                    "permissions": {
                      "keys": [
                        "get",
                        "wrapKey",
                        "unwrapKey"
                      ]
                    }
                  }
                ]
              }
            }
          ]
        }
      }
    },
    {
      "name": "[concat(parameters('serverName'), '/', variables('serverKeyName'))]",
      "type": "Microsoft.DBforPostgreSQL/servers/keys",
      "apiVersion": "2020-01-01-preview",
      "dependsOn": [
        "addAccessPolicy",
        "[resourceId('Microsoft.DBforPostgreSQL/servers', parameters('serverName'))]"
      ],
      "properties": {
        "serverKeyType": "AzureKeyVault",
        "uri": "[concat(reference(resourceId(parameters('keyVaultResourceGroupName'), 'Microsoft.KeyVault/vaults/', parameters('keyVaultName')), '2018-02-14-preview', 'Full').properties.vaultUri, 'keys/', parameters('keyName'), '/', parameters('keyVersion'))]"
      }
    }
  ]
}
```

## <a name="next-steps"></a>次のステップ

 データ暗号化の詳細については、「[カスタマー マネージド キーを使用した Azure Database for PostgreSQL 単一サーバーのデータの暗号化](concepts-data-encryption-postgresql.md)」を参照してください。
