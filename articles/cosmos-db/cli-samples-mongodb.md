---
title: 'Azure CLI-Beispiele für die Azure Cosmos DB: MongoDB-API'
description: 'Azure CLI-Beispiele für die Azure Cosmos DB: MongoDB-API'
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: sample
ms.date: 9/25/2019
ms.author: mjbrown
ms.openlocfilehash: f7807a4c2024e16f563adbcb46a5c60e5c542dda
ms.sourcegitcommit: 0947111b263015136bca0e6ec5a8c570b3f700ff
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/24/2020
ms.locfileid: "77524562"
---
# <a name="azure-cli-samples-for-azure-cosmos-db-mongodb-api"></a>Azure CLI-Beispiele für die Azure Cosmos DB: MongoDB-API

Die folgende Tabelle enthält Links zu Azure CLI-Beispielskripts für die Azure Cosmos DB-MongoDB-API. Referenzseiten für alle CLI-Befehle für Azure Cosmos DB sind in der [Referenz zur Azure CLI](/cli/azure/cosmosdb) verfügbar. Alle CLI-Skriptbeispiele für Azure Cosmos DB finden Sie im [GitHub-Repository zur Azure Cosmos DB-CLI](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).

> [!NOTE]
> Derzeit können Sie mithilfe von PowerShell, CLI und Resource Manager-Vorlagen nur Version 3.2 der Azure Cosmos DB-API für MongoDB-Konten (d. h. Konten mit dem Endpunkt im Format `*.documents.azure.com`) erstellen. Verwenden Sie zum Erstellen von Konten der Version 3.6 stattdessen das Azure-Portal.

| |  |
|---|---|
| [Erstellen eines Kontos, einer Datenbank und einer Sammlung in Azure Cosmos](scripts/cli/mongodb/create.md?toc=%2fcli%2fazure%2ftoc.json)| Erstellen eines Kontos, einer Datenbank und einer Sammlung in Azure Cosmos DB für die MongoDB-API |
| [Ändern des Durchsatzes](scripts/cli/mongodb/throughput.md?toc=%2fcli%2fazure%2ftoc.json) | Aktualisieren der Anforderungseinheiten pro Sekunde (RU/s) für eine Datenbank und Sammlung|
| [Hinzufügen von Failoverregionen](scripts/cli/common/regions.md?toc=%2fcli%2fazure%2ftoc.json) | Hinzufügen einer Region, Ändern der Failoverpriorität und Auslösen eines manuellen Failovers|
| [Kontoschlüssel und Verbindungszeichenfolgen](scripts/cli/common/keys.md?toc=%2fcli%2fazure%2ftoc.json) | Auflisten von Kontoschlüsseln und schreibgeschützten Schlüsseln, erneutes Generieren von Schlüsseln und Auflisten von Verbindungszeichenfolgen|
| [Sichern mit einer IP-Firewall](scripts/cli/common/ipfirewall.md?toc=%2fcli%2fazure%2ftoc.json)| Erstellen eines Cosmos-Kontos mit konfigurierter IP-Firewall|
| [Sichern eines neuen Kontos mit Dienstendpunkten](scripts/cli/common/service-endpoints.md?toc=%2fcli%2fazure%2ftoc.json)| Erstellen eines Cosmos-Kontos und Sichern des Kontos mit Dienstendpunkten|
| [Sichern eines vorhandenen Kontos mit Dienstendpunkten](scripts/cli/common/service-endpoints-ignore-missing-vnet.md?toc=%2fcli%2fazure%2ftoc.json)| Aktualisieren eines Cosmos-Kontos beim Konfigurieren des Subnetzes, um es mit Dienstendpunkten zu sichern|
|||
