---
title: "Azure デプロイ用のリンク テンプレート | Microsoft Docs"
description: "Azure リソース マネージャー テンプレートでリンクされたテンプレートを使用して、モジュール構造のテンプレート ソリューションを作成する方法について説明します。 パラメーターの値を渡す方法、パラメーター ファイルを指定する方法、および URL を動的に作成する方法を示します。"
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: 27d8c4b2-1e24-45fe-88fd-8cf98a6bb2d2
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/28/2017
ms.author: tomfitz
ms.openlocfilehash: a86d4d8705c7093e3900a9738ddbd364db8bd3b8
ms.sourcegitcommit: 29bac59f1d62f38740b60274cb4912816ee775ea
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/29/2017
---
# <a name="using-linked-templates-when-deploying-azure-resources"></a>Azure リソース デプロイ時のリンクされたテンプレートの使用

ソリューションをデプロイする場合、単一テンプレートか、複数のテンプレートがリンクされたメイン テンプレートのいずれかを使用できます。 中小規模のソリューションの場合、テンプレートを 1 つにするとわかりやすく、保守も簡単になります。 すべてのリソースと値を 1 つのファイルで参照できます。 高度なシナリオの場合、リンクされたテンプレートを使用することで、対象となるコンポーネントにソリューションを分割し、テンプレートを再利用できます。

リンクされたテンプレートを使用する場合は、配置時にパラメーター値を受け取るメインのテンプレートを作成します。 メインのテンプレートには、すべてのリンクされたテンプレートが含まれていて、必要に応じて、これらのテンプレートに値を渡します。

![リンク済みテンプレート](./media/resource-group-linked-templates/nestedTemplateDesign.png)

## <a name="link-to-a-template"></a>テンプレートにリンクする

別のテンプレートにリンクするには、メイン テンプレートに**展開**リソースを追加します。

```json
"resources": [
  {
      "apiVersion": "2017-05-10",
      "name": "linkedTemplate",
      "type": "Microsoft.Resources/deployments",
      "properties": {
          "mode": "Incremental",
          <inline-template-or-external-template>
      }
  }
]
```

展開リソースに指定するプロパティは、外部のテンプレートにリンクしているか、メインのテンプレートでインライン テンプレートを埋め込んでいるかによって異なります。

### <a name="inline-template"></a>インライン テンプレート

リンク済みテンプレートを埋め込むには、**テンプレート** プロパティを使用してテンプレートを含めます。

```json
"resources": [
  {
    "apiVersion": "2017-05-10",
    "name": "nestedTemplate",
    "type": "Microsoft.Resources/deployments",
    "properties": {
      "mode": "Incremental",
      "template": {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "variables": {},
        "resources": [
          {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[variables('storageName')]",
            "apiVersion": "2015-06-15",
            "location": "West US",
            "properties": {
              "accountType": "Standard_LRS"
            }
          }
        ]
      },
      "parameters": {}
    }
  }
]
```

### <a name="external-template-and-external-parameters"></a>外部テンプレートと外部パラメーター

外部のテンプレートおよびパラメーター ファイルにリンクするには、**templateLink** と **parametersLink** を使用します。 テンプレートにリンクするには、Resource Manager サービスからそのテンプレートにアクセスできる必要があります。 ローカル ファイルや、ローカル ネットワークだけで使用可能なファイルは指定できません。 **http** または **https** のいずれかを含む URI 値のみを指定できます。 1 つと選択肢として、ストレージ アカウントにリンク済みテンプレートを配置し、その項目の URI を使用できます。

```json
"resources": [
  {
     "apiVersion": "2017-05-10",
     "name": "linkedTemplate",
     "type": "Microsoft.Resources/deployments",
     "properties": {
       "mode": "incremental",
       "templateLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",
          "contentVersion":"1.0.0.0"
       },
       "parametersLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.parameters.json",
          "contentVersion":"1.0.0.0"
       }
     }
  }
]
```

### <a name="external-template-and-inline-parameters"></a>外部テンプレートとインライン パラメーター

または、パラメーターをインラインで指定できます。 メイン テンプレートから、リンクされたテンプレートに値を渡すには、**パラメーター**を使用します。

```json
"resources": [
  {
     "apiVersion": "2017-05-10",
     "name": "linkedTemplate",
     "type": "Microsoft.Resources/deployments",
     "properties": {
       "mode": "incremental",
       "templateLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",
          "contentVersion":"1.0.0.0"
       },
       "parameters": {
          "StorageAccountName":{"value": "[parameters('StorageAccountName')]"}
        }
     }
  }
]
```

## <a name="using-variables-to-link-templates"></a>変数を使用したテンプレートのリンク

前の例では、URL の値をハード コーディングしてテンプレートをリンクする方法について説明しました。 この方法は簡単なテンプレートには適していますが、モジュール構造の大規模な一連のテンプレートを使用する場合にはあまり適していません。 その場合は、メイン テンプレートのベース URL を格納する静的変数を作成し、リンクされたテンプレートの URL をそのベース URL から動的に作成することができます。 この方法の利点としては、テンプレートを簡単に移動したり、フォークしたりできることが挙げられます。 メイン テンプレート内の静的変数を変更するだけで、正しい URI が、メイン テンプレートから、分解されたテンプレート全体に渡されます。

次の例では、ベース URL を使用して、リンクされたテンプレート (**sharedTemplateUrl** と **vmTemplate**) の 2 つの URL を作成する方法を示しています。

```json
"variables": {
    "templateBaseUrl": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/postgresql-on-ubuntu/",
    "sharedTemplateUrl": "[concat(variables('templateBaseUrl'), 'shared-resources.json')]",
    "vmTemplateUrl": "[concat(variables('templateBaseUrl'), 'database-2disk-resources.json')]"
}
```

[deployment()](resource-group-template-functions-deployment.md#deployment) を使用して、現在のテンプレートのベース URL を取得したり、同じ場所にある他のテンプレートの URL を取得したりすることもできます。 この方法は、テンプレートの場所が変更された場合 (バージョン管理などのため) や、テンプレート ファイルのハード コーディング URL を回避する必要がある場合に便利です。

```json
"variables": {
    "sharedTemplateUrl": "[uri(deployment().properties.templateLink.uri, 'shared-resources.json')]"
}
```

## <a name="get-values-from-linked-template"></a>リンク済みテンプレートから値を取得する

リンクされたテンプレートから出力値を取得するには、`"[reference('<name-of-deployment>').outputs.<property-name>.value]"` のような構文でプロパティ値を取得します。

次の例では、リンクされたテンプレートを参照して、出力値を取得する方法を示します。 リンクされたテンプレートは、単純なメッセージを返します。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [],
    "outputs": {
        "greetingMessage": {
            "value": "Hello World",
            "type" : "string"
        }
    }
}
```

親テンプレートは、リンクされたテンプレートを展開し、返される値を取得します。 展開リソースを名前で参照し、リンクされたテンプレートによって返されるプロパティの名前を使用していることに注意してください。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [
        {
            "apiVersion": "2017-05-10",
            "name": "linkedTemplate",
            "type": "Microsoft.Resources/deployments",
            "properties": {
                "mode": "incremental",
                "templateLink": {
                    "uri": "[uri(deployment().properties.templateLink.uri, 'helloworld.json')]",
                    "contentVersion": "1.0.0.0"
                }
            }
        }
    ],
    "outputs": {
        "messageFromLinkedTemplate": {
            "type": "string",
            "value": "[reference('linkedTemplate').outputs.greetingMessage.value]"
        }
    }
}
```

他の種類のリソース同様、リンクされたテンプレートとその他のリソース間の依存関係を設定できます。 したがって、その他のリソースにリンクされたテンプレートからの出力値が必要な場合、リンクされたテンプレートが他のリソースの前にデプロイされるようにすることができます。 または、リンクされたテンプレートが他のリソースに依存する場合は、リンクされたテンプレートの前に他のリソースがデプロイされるようにすることができます。

次の例では、パブリック IP アドレスをデプロイし、リソース ID を返すテンプレートを示します。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publicIPAddresses_name": {
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "name": "[parameters('publicIPAddresses_name')]",
            "apiVersion": "2017-06-01",
            "location": "eastus",
            "properties": {
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Dynamic",
                "idleTimeoutInMinutes": 4
            },
            "dependsOn": []
        }
    ],
    "outputs": {
        "resourceID": {
            "type": "string",
            "value": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIPAddresses_name'))]"
        }
    }
}
```

ロード バランサーの展開時に、前のテンプレートからのパブリック IP アドレスを使用するには、テンプレートにリンクし、展開リソースに依存関係を追加します。 ロード バランサーのパブリック IP アドレスは、リンクされているテンプレートからの出力値に設定されます。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "loadBalancers_name": {
            "defaultValue": "mylb",
            "type": "string"
        },
        "publicIPAddresses_name": {
            "defaultValue": "myip",
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "name": "[parameters('loadBalancers_name')]",
            "apiVersion": "2017-06-01",
            "location": "eastus",
            "properties": {
                "frontendIPConfigurations": [
                    {
                        "name": "LoadBalancerFrontEnd",
                        "properties": {
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIPAddress": {
                                "id": "[reference('linkedTemplate').outputs.resourceID.value]"
                            }
                        }
                    }
                ],
                "backendAddressPools": [],
                "loadBalancingRules": [],
                "probes": [],
                "inboundNatRules": [],
                "outboundNatRules": [],
                "inboundNatPools": []
            },
            "dependsOn": [
                "linkedTemplate"
            ]
        },
        {
            "apiVersion": "2017-05-10",
            "name": "linkedTemplate",
            "type": "Microsoft.Resources/deployments",
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                    "uri": "[uri(deployment().properties.templateLink.uri, 'publicip.json')]",
                    "contentVersion": "1.0.0.0"
                },
                "parameters":{
                    "publicIPAddresses_name":{"value": "[parameters('publicIPAddresses_name')]"}
                }
            }
        }
    ]
}
```

## <a name="linked-templates-in-deployment-history"></a>デプロイメント履歴の中のリンク済みテンプレート

Resource Manager では、各リンク済みテンプレートは個別のデプロイメントとして処理されます。 したがって、3 つのリンク済みテンプレートのある親テンプレートは、デプロイ履歴で次のように表示されます。

![デプロイ履歴](./media/resource-group-linked-templates/deployment-history.png)

履歴内のこれらの個別のエントリを使用して、展開後に出力値を取得できます。 次のテンプレートは、パブリック IP アドレスを作成し、IP アドレスを出力します。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publicIPAddresses_name": {
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "name": "[parameters('publicIPAddresses_name')]",
            "apiVersion": "2017-06-01",
            "location": "southcentralus",
            "properties": {
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Static",
                "idleTimeoutInMinutes": 4,
                "dnsSettings": {
                    "domainNameLabel": "[concat(parameters('publicIPAddresses_name'), uniqueString(resourceGroup().id))]"
                }
            },
            "dependsOn": []
        }
    ],
    "outputs": {
        "returnedIPAddress": {
            "type": "string",
            "value": "[reference(parameters('publicIPAddresses_name')).ipAddress]"
        }
    }
}
```

次のテンプレートは、前のテンプレートにリンクします。 3 つのパブリック IP アドレスが作成されます。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
    },
    "variables": {},
    "resources": [
        {
            "apiVersion": "2017-05-10",
            "name": "[concat('linkedTemplate', copyIndex())]",
            "type": "Microsoft.Resources/deployments",
            "properties": {
              "mode": "Incremental",
              "templateLink": {
                "uri": "[uri(deployment().properties.templateLink.uri, 'static-public-ip.json')]",
                "contentVersion": "1.0.0.0"
              },
              "parameters":{
                  "publicIPAddresses_name":{"value": "[concat('myip-', copyIndex())]"}
              }
            },
            "copy": {
                "count": 3,
                "name": "ip-loop"
            }
        }
    ]
}
```

展開後、次の PowerShell スクリプトを使用して、出力値を取得できます。

```powershell
$loopCount = 3
for ($i = 0; $i -lt $loopCount; $i++)
{
    $name = 'linkedTemplate' + $i;
    $deployment = Get-AzureRmResourceGroupDeployment -ResourceGroupName examplegroup -Name $name
    Write-Output "deployment $($deployment.DeploymentName) returned $($deployment.Outputs.returnedIPAddress.value)"
}
```

または、次の Azure CLI スクリプトを使用します。

```azurecli
for i in 0 1 2;
do
    name="linkedTemplate$i";
    deployment=$(az group deployment show -g examplegroup -n $name);
    ip=$(echo $deployment | jq .properties.outputs.returnedIPAddress.value);
    echo "deployment $name returned $ip";
done
```

## <a name="securing-an-external-template"></a>外部テンプレートのセキュリティ保護

リンクされたテンプレートは外部から利用可能でなければなりませんが、一般公開する必要はありません。 ストレージ アカウント所有者のみがアクセス可能なプライベート ストレージ アカウントに、テンプレートを追加できます。 次に、デプロイ時にアクセスできるように、Shared Access Signature (SAS) トークンを作成します。 リンクされたテンプレートの URI に SAS トークンを追加します。 トークンがセキュリティで保護された文字列として渡された場合でも、SAS トークンを含むリンクされたテンプレートの URI が、デプロイ操作中にログに記録されます。 公開を制限するには、トークンの有効期限を設定します。

パラメーター ファイルは SAS トークンを使ったアクセスに制限することができます。

次の例は、テンプレートにリンクするときに、SAS トークンを渡す方法を示します。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "containerSasToken": { "type": "string" }
  },
  "resources": [
    {
      "apiVersion": "2017-05-10",
      "name": "linkedTemplate",
      "type": "Microsoft.Resources/deployments",
      "properties": {
        "mode": "incremental",
        "templateLink": {
          "uri": "[concat(uri(deployment().properties.templateLink.uri, 'helloworld.json'), parameters('containerSasToken'))]",
          "contentVersion": "1.0.0.0"
        }
      }
    }
  ],
  "outputs": {
  }
}
```

PowerShell では、コンテナーのトークンを取得し、以下を使用してテンプレートをデプロイします。

```powershell
Set-AzureRmCurrentStorageAccount -ResourceGroupName ManageGroup -Name storagecontosotemplates
$token = New-AzureStorageContainerSASToken -Name templates -Permission r -ExpiryTime (Get-Date).AddMinutes(30.0)
$url = (Get-AzureStorageBlob -Container templates -Blob parent.json).ICloudBlob.uri.AbsoluteUri
New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateUri ($url + $token) -containerSasToken $token
```

Azure CLI では、コンテナーのトークンを取得し、次のコードを使用してテンプレートをデプロイします。

```azurecli
expiretime=$(date -u -d '30 minutes' +%Y-%m-%dT%H:%MZ)
connection=$(az storage account show-connection-string \
    --resource-group ManageGroup \
    --name storagecontosotemplates \
    --query connectionString)
token=$(az storage container generate-sas \
    --name templates \
    --expiry $expiretime \
    --permissions r \
    --output tsv \
    --connection-string $connection)
url=$(az storage blob url \
    --container-name templates \
    --name parent.json \
    --output tsv \
    --connection-string $connection)
parameter='{"containerSasToken":{"value":"?'$token'"}}'
az group deployment create --resource-group ExampleGroup --template-uri $url?$token --parameters $parameter
```

## <a name="example-templates"></a>サンプル テンプレート

### <a name="hello-world-from-linked-template"></a>リンク済みテンプレートからの Hello World

[親テンプレート](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/helloworldparent.json)と[リンク済みテンプレート](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/helloworld.json)を展開するには、次の PowerShell を使用します。

```powershell
New-AzureRmResourceGroupDeployment `
  -ResourceGroupName examplegroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/helloworldparent.json
```

または、次の Azure CLI を使います。

```azurecli-interactive
az group deployment create \
  -g examplegroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/helloworldparent.json
```

### <a name="load-balancer-with-public-ip-address-in-linked-template"></a>リンクされているテンプレートのパブリック IP アドレスを使用する Azure Load Balancer

[親テンプレート](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip-parentloadbalancer.json)と[リンク済みテンプレート](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip.json)を展開するには、次の PowerShell を使用します。

```powershell
New-AzureRmResourceGroupDeployment `
  -ResourceGroupName examplegroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/linkedtemplates/public-ip-parentloadbalancer.json
```

または、次の Azure CLI を使います。

```azurecli-interactive
az group deployment create \
  -g examplegroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/linkedtemplates/public-ip-parentloadbalancer.json
```

### <a name="multiple-public-ip-addresses-in-linked-template"></a>リンクされているテンプレートに複数のパブリック IP アドレス

[親テンプレート](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/static-public-ip-parent.json)と[リンク済みテンプレート](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/static-public-ip.json)を展開するには、次の PowerShell を使用します。

```powershell
New-AzureRmResourceGroupDeployment `
  -ResourceGroupName examplegroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/linkedtemplates/static-public-ip-parent.json
```

または、次の Azure CLI を使います。

```azurecli-interactive
az group deployment create \
  -g examplegroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/linkedtemplates/static-public-ip-parent.json
```

## <a name="next-steps"></a>次のステップ

* リソースのデプロイの順序の定義については、「[Azure Resource Manager テンプレートでの依存関係の定義](resource-group-define-dependencies.md)」を参照してください。
* リソースを 1 つ定義し、そのリソースの複数のインスタンスを作成する方法については、「 [Azure Resource Manager でリソースの複数のインスタンスを作成する](resource-group-create-multiple.md)」を参照してください。
* ストレージ アカウントにテンプレートを設定し SAS トークンを生成する手順については、「[Resource Manager テンプレートと Azure PowerShell を使用したリソースのデプロイ](resource-group-template-deploy.md)」または「[Resource Manager テンプレートと Azure CLI を使用したリソースのデプロイ](resource-group-template-deploy-cli.md)」を参照してください。