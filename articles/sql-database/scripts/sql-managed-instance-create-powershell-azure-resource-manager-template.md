---
title: 'Vorlagenbeispiel: Erstellen einer verwalteten Instanz in Azure SQL-Datenbank'
description: Verwenden Sie dieses Azure PowerShell-Beispielskript zum Erstellen einer verwalteten Instanz in Azure SQL-Datenbank.
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: seo-dt-2019
ms.devlang: PowerShell
ms.topic: sample
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 03/12/2019
ms.openlocfilehash: e5be8c9441be5ca441a5c0f7c4444c2edbdc7a95
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/29/2020
ms.locfileid: "80811214"
---
# <a name="use-powershell-with-azure-resource-manager-template-to-create-a-managed-instance-in-azure-sql-database"></a>Verwenden von PowerShell mit einer Azure Resource Manager-Vorlage zum Erstellen einer verwalteten Instanz in Azure SQL-Datenbank

Eine verwaltete Azure SQL-Datenbank-Instanz kann mit einer Azure PowerShell-Bibliothek und Azure Resource Manager-Vorlagen erstellt werden.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Wenn Sie PowerShell lokal installieren und nutzen möchten, müssen Sie für dieses Tutorial mindestens Version 1.4.0 von Azure PowerShell verwenden. Wenn Sie ein Upgrade ausführen müssen, finden Sie unter [Installieren des Azure PowerShell-Moduls](/powershell/azure/install-az-ps) Informationen dazu. Wenn Sie PowerShell lokal ausführen, müssen Sie auch `Connect-AzAccount` ausführen, um eine Verbindung mit Azure herzustellen.

Mit Azure PowerShell-Befehlen kann die Bereitstellung gestartet werden, indem eine vordefinierte Azure Resource Manager-Vorlage verwendet wird. Die folgenden Eigenschaften können in der Vorlage angegeben werden:

- Instanzname
- Benutzername und Kennwort des SQL-Administrators
- Größe der Instanz (Anzahl von Kernen und maximale Speichergröße)
- VNET und Subnetz, in dem die Instanz angeordnet wird
- Sortierung der Instanz auf Serverebene (Vorschauversion)

Der Instanzname, der Benutzername des SQL-Administrators, das VNET/Subnetz und die Sortierung können später nicht mehr geändert werden. Andere Eigenschaften der Instanz können geändert werden.

## <a name="prerequisites"></a>Voraussetzungen

In diesem Beispiel wird vorausgesetzt, dass Sie für Ihre verwaltete Instanz [eine gültige Netzwerkumgebung erstellt](../sql-database-managed-instance-create-vnet-subnet.md) oder [ein vorhandenes VNET geändert](../sql-database-managed-instance-configure-vnet-subnet.md) haben. Da im Beispiel die Cmdlets [New-AzResourceGroupDeployment](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroupdeployment) und [Get-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/get-azvirtualnetwork) verwendet werden, vergewissern Sie sich, dass Sie die folgenden PowerShell-Module installiert haben:

```powershell
Install-Module Az.Network
Install-Module Az.Resources
```

## <a name="azure-resource-manager-template"></a>Azure Resource Manager-Vorlage

Der folgende Inhalt sollte in einer Datei angeordnet werden, die als Vorlage für die Erstellung der Instanz dient:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.1",
    "parameters": {
        "instance": {
            "type": "string"
        },
        "user": {
            "type": "string"
        },
        "pwd": {
            "type": "securestring"
        },
        "subnetId": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[parameters('instance')]",
            "location": "West Central US",
            "tags": {
                "Description":"GP Instance with custom instance collation - Serbian_Cyrillic_100_CS_AS"
            },
            "sku": {
                "name": "GP_Gen5",
                "tier": "GeneralPurpose"
            },
            "properties": {
                "administratorLogin": "[parameters('user')]",
                "administratorLoginPassword": "[parameters('pwd')]",
                "subnetId": "[parameters('subnetId')]",
                "storageSizeInGB": 256,
                "vCores": 8,
                "licenseType": "LicenseIncluded",
                "hardwareFamily": "Gen5",
                "collation": "Serbian_Cyrillic_100_CS_AS"
            },
            "type": "Microsoft.Sql/managedInstances",
            "identity": {
                "type": "SystemAssigned"
            },
            "apiVersion": "2015-05-01-preview"
        }
    ]
}
```

Es wird vorausgesetzt, dass Azure VNET mit dem richtig konfigurierten Subnetz bereits vorhanden ist. Sollten Sie über kein ordnungsgemäß konfiguriertes Subnetz verfügen, bereiten Sie die Netzwerkumgebung mit einer separaten [Azure Resource Manager-Vorlage](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-managed-instance-azure-environment) vor. (Die Vorlage kann unabhängig ausgeführt oder in diese Vorlage einbezogen werden.)

Speichern Sie den Inhalt dieser Datei als JSON-Datei, fügen Sie den Dateipfad in das folgende PowerShell-Skript ein, und ändern Sie die Namen der Objekte im Skript:

```powershell
$subscriptionId = "ed827499-xxxx-xxxx-xxxx-xxxxxxxxxx"
Select-AzSubscription -SubscriptionId $subscriptionId

# Managed Instance properties
$resourceGroup = "rg_mi"
$location = "West Central US"
$name = "managed-instance-name"
$user = "miSqlAdmin"
$secpasswd = ConvertTo-SecureString "<Put some strong password here>" -AsPlainText -Force

# Network configuration
$vNetName = "my_vnet"
$vNetResourceGroup = "rg_mi_vnet"
$subnetName = "ManagedInstances"
$vNet = Get-AzVirtualNetwork -Name $vNetName -ResourceGroupName $vNetResourceGroup
$subnet = Get-AzVirtualNetworkSubnetConfig -Name $SubnetName -VirtualNetwork $vNet
$subnetId = $subnet.Id

# Deploy Instance using Azure Resource Manager template:
New-AzResourceGroupDeployment  -Name MyDeployment -ResourceGroupName $resourceGroup  `
                                    -TemplateFile 'C:\...\create-managed-instance.json' `
                                    -instance $name -user $user -pwd $secpasswd -subnetId $subnetId
```

Nach erfolgreicher Ausführung des Skripts können alle Azure-Dienste und die konfigurierte IP-Adresse auf die SQL-Datenbank zugreifen.

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zu Azure PowerShell finden Sie in der [Azure PowerShell-Dokumentation](/powershell/azure/overview).

Zusätzliche PowerShell-Skriptbeispiele für SQL-Datenbank finden Sie in den [Azure PowerShell samples for Azure SQL Database](../sql-database-powershell-samples.md) (Azure PowerShell-Beispiele für Azure SQL-Datenbank).
