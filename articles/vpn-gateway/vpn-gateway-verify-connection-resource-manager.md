---
title: 'Azure-VPN Gateway: Überprüfen einer Gatewayverbindung'
description: Dieser Artikel zeigt, wie Sie eine Verbindung eines virtuellen Netzwerks mit einem VPN Gateway überprüfen.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: article
ms.date: 05/16/2017
ms.author: cherylmc
ms.openlocfilehash: 40417b078577424b1ba13ce60f4f773a1c8fd4dc
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/27/2020
ms.locfileid: "75780146"
---
# <a name="verify-a-vpn-gateway-connection"></a>Überprüfen einer VPN Gateway-Verbindung

In diesem Artikel erfahren Sie, wie Sie eine VPN Gateway-Verbindung für das klassische und das Resource Manager-Bereitstellungsmodell überprüfen.

## <a name="azure-portal"></a>Azure-Portal

[!INCLUDE [Azure portal](../../includes/vpn-gateway-verify-connection-portal-rm-include.md)]

## <a name="powershell"></a>PowerShell

Installieren Sie die aktuelle Version der [Azure Resource Manager-PowerShell-Cmdlets](/powershell/azure/overview), um eine VPN Gateway-Verbindung für das Resource Manager-Bereitstellungsmodell mit PowerShell zu überprüfen.

[!INCLUDE [PowerShell](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

## <a name="azure-cli"></a>Azure-Befehlszeilenschnittstelle

Installieren Sie die aktuelle Version der [CLI-Befehle](https://docs.microsoft.com/cli/azure/install-azure-cli) (2.0 oder höher), um eine VPN Gateway-Verbindung für das Resource Manager-Bereitstellungsmodell über die Azure-Befehlszeilenschnittstelle zu überprüfen.

[!INCLUDE [CLI](../../includes/vpn-gateway-verify-connection-cli-rm-include.md)]


## <a name="azure-portal-classic"></a>Azure-Portal (klassisch)

[!INCLUDE [Azure portal](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]

## <a name="powershell-classic"></a>PowerShell (klassisch)

Installieren Sie die aktuelle Version der Azure PowerShell-Cmdlets, um Ihre VPN Gateway-Verbindung für das klassische Bereitstellungsmodell mit PowerShell zu überprüfen. Laden Sie das Modul [Dienstverwaltung](https://docs.microsoft.com/powershell/azure/servicemanagement/install-azure-ps?view=azuresmps-4.0.0#azure-service-management-cmdlets) herunter, und installieren Sie es. Melden Sie sich mit „Add-AzureAccount“ beim klassischen Bereitstellungsmodell an.

[!INCLUDE [Classic PowerShell](../../includes/vpn-gateway-verify-connection-ps-classic-include.md)]

## <a name="next-steps"></a>Nächste Schritte

* Sie können Ihren virtuellen Netzwerken virtuelle Computer hinzufügen. Für diese Schritte finden Sie Informationen unter [Erstellen eines virtuellen Computers](../virtual-machines/virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) .
