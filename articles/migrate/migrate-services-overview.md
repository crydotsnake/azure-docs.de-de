---
title: Informationen zu Azure Migrate
description: Erfahren Sie mehr über den Azure Migrate-Dienst.
ms.topic: overview
ms.date: 04/15/2020
ms.custom: mvc
ms.openlocfilehash: fe6386346282cf182397f6420c541d629ba0aab3
ms.sourcegitcommit: d57d2be09e67d7afed4b7565f9e3effdcc4a55bf
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/22/2020
ms.locfileid: "81768403"
---
# <a name="about-azure-migrate"></a>Informationen zu Azure Migrate

Dieser Artikel enthält eine kurze Übersicht über den Azure Migrate-Dienst.

Azure Migrate verfügt über einen zentralisierten Hub für die Bewertung und die Migration von lokalen Servern sowie der lokalen Infrastruktur, Anwendungen und Daten zu Azure.

Azure Migrate enthält die folgenden Features:

- **Vereinheitlichte Migrationsplattform**: Ein einzelnes Portal zum Starten, Ausführen und Nachverfolgen Ihrer Migration zu Azure
- **Verfügbare Tools**: Eine Reihe von Tools für Bewertung und Migration. Beispiele hierfür sind Azure Migrate-Serverbewertung Serverbewertung und Azure Migrate: Servermigration. Azure Migrate kann in andere Azure-Dienste und andere Tools und Angebote unabhängiger Softwareanbieter (Independent Software Vendors, ISVs) integriert werden.
- **Bewertung und Migration**: Im Azure Migrate-Hub können Sie Folgendes bewerten und migrieren:
    - **Server**: Bewerten Sie lokale Server, und migrieren Sie sie zu virtuellen Azure-Computern.
    - **Datenbanken**: Bewerten Sie lokale Datenbanken, und migrieren Sie sie zu Azure SQL-Datenbank oder zu einer verwalteten Azure SQL-Datenbank-Instanz.
    - **Webanwendungen**: Bewerten Sie mit dem Azure App Service-Migrations-Assistenten lokale Webanwendungen, und migrieren Sie sie zu Azure App Service.
    - **Virtuelle Desktops**: Bewerten Sie die lokale VDI-Instanz (Virtual Desktop Infrastructure), und migrieren Sie sie zu Windows Virtual Desktop in Azure.
    - **Data:** Große Datenmengen werden mit Azure Data Box-Produkten schnell und kostengünstig zu Azure migriert.

## <a name="integrated-tools"></a>Integrierte Tools

Der Azure Migrate-Hub umfasst die folgenden Tools:

**Tool** | **Bewerten und Migrieren** | **Details**
--- | --- | ---
**Azure Migrate: Serverbewertung** | Bewerten von Servern. | Als Vorbereitung für die Migration zu Azure werden lokale VMware-VMs, Hyper-V-VMs und physische Server ermittelt und bewertet.
**Azure Migrate: Servermigration** | Migrieren von Servern. | VMware-VMs, Hyper-V-VMs, physische Server und andere virtualisierte Computer und VMs der öffentlichen Cloud werden zu Azure migriert.
**Data Migration Assistant** | Lokale SQL Server-Datenbanken werden für die Migration zu Azure SQL-Datenbank, zu einer verwalteten Azure SQL-Datenbank-Instanz oder zu Azure-VMs, auf denen SQL Server ausgeführt wird, bewertet. | Mit dem Datenmigrations-Assistenten können mögliche Probleme ermittelt werden, die die Migration blockieren. Er identifiziert nicht unterstützte Features, neue Features, von denen Sie nach der Migration profitieren können, sowie den richtigen Pfad für die Datenbankmigration. [Weitere Informationen](https://docs.microsoft.com/sql/dma/dma-overview?view=sql-server-2017)
**Azure Database Migration Service** | Lokale Datenbanken werden zu Azure-VMs migriert, auf denen SQL Server, Azure SQL-Datenbank oder verwaltete Azure SQL-Datenbank-Instanzen ausgeführt werden. | [Weitere Informationen](https://docs.microsoft.com/azure/dms/dms-overview) zu Database Migration Service
**Movere** | Bewerten von Servern. | Weitere Informationen zu Movere finden Sie [hier](#movere).
**Migrations-Assistent für Web-Apps** | Lokale Web-Apps werden bewertet und zu Azure migriert. |  Verwenden Sie den Migrations-Assistenten von Azure App Service, um lokale Websites für die Migration zu Azure App Service zu bewerten.<br/><br/> Verwenden Sie den Migrations-Assistenten zum Migrieren von .NET- und PHP-Web-Apps zu Azure. [Weitere Informationen](https://appmigration.microsoft.com/) zum Azure App Service-Migrations-Assistenten
**Azure Data Box** | Migrieren Sie Offlinedaten. | Verschieben Sie große Mengen an Offlinedaten mit Azure Data Box-Produkten zu Azure. [Weitere Informationen](https://docs.microsoft.com/azure/databox/)

> [!NOTE]
> In Azure Government können von externen integrierten Tools und ISV-Angeboten keine Daten an Azure Migrate-Projekte gesendet werden. Tools können unabhängig verwendet werden.

## <a name="isv-integration"></a>ISV-Integration

Azure Migrate kann in verschiedene ISV-Angebote integriert werden. 

**ISV**    | **Feature**
--- | ---
[Carbonite](https://www.carbonite.com/globalassets/files/datasheets/carb-migrate4azure-microsoft-ds.pdf) | Migrieren von Servern.
[Cloudamize](https://www.cloudamize.com/platform) | Bewerten von Servern.
[Corent Technology](https://www.corenttech.com/AzureMigrate/) | Bewerten und Migrieren von Servern
[Device42](https://docs.device42.com/) | Bewerten von Servern.
[Lakeside](https://go.microsoft.com/fwlink/?linkid=2104908) | Bewerten der VDI
[RackWare](https://go.microsoft.com/fwlink/?linkid=2102735) | Migrieren von Servern.
[Turbonomic](https://learn.turbonomic.com/azure-migrate-portal-free-trial) | Bewerten von Servern.
[UnifyCloud](https://www.cloudatlasinc.com/cloudrecon/) | Bewerten von Servern und Datenbanken

## <a name="azure-migrate-server-assessment-tool"></a>Von der Azure Serverbewertungstool

Azure Migrate: Das Serverbewertungstool ermittelt und bewertet lokale VMware-VMs, Hyper-V-VMs und physische Server für die Migration zu Azure.

Vom Tool wird Folgendes durchgeführt:

- **Azure-Bereitschaft**: Bewertet, ob lokale Computer bereit für die Migration zu Azure sind.
- **Azure-Dimensionierung:** Schätzt die Größe von Azure-VMs nach der Migration.
- **Azure-Kostenschätzung:** Schätzt die Kosten für die Ausführung lokaler Server in Azure.
- **Abhängigkeitsanalyse:** Hiermit werden serverübergreifende Abhängigkeiten und Optimierungsstrategien für das Verschieben untereinander abhängiger Server in Azure ermittelt. Unter [Abhängigkeitsanalyse](concepts-dependency-visualization.md) erfahren Sie mehr über die Serverbewertung.

Bei der Serverbewertung wird eine einfache [Azure Migrate-Appliance](migrate-appliance.md) verwendet, die Sie lokal bereitstellen.

- Die Appliance wird auf einer VM oder einem physischen Server ausgeführt. Sie können sie mit einer heruntergeladenen Vorlage leicht installieren.
- Die Appliance ermittelt lokale Computer. Computermetadaten und Leistungsdaten werden kontinuierlich an Azure Migrate gesendet.
- Die Applianceermittlung erfolgt ohne Agent. Auf ermittelten Computern wird nichts installiert.
- Nach der Ermittlung der Appliance fassen Sie die ermittelten Computer in Gruppen zusammen und führen Bewertungen für jede Gruppe durch.

## <a name="azure-migrate-server-migration-tool"></a>Von der Azure Tool für die Servermigration

Das Tool für die Azure Migrate- Servermigration unterstützt Sie bei der Migration zu Azure:

- Lokale VMware-VMs
- Virtuelle Hyper-V-Computer
- Physische Server
- Andere virtualisierte Computer
- VMs der öffentlichen Cloud

Computer können nach der Bewertung oder ohne Bewertung migriert werden.

Für die Migration von VMware-VMs ohne Agent und die Migration von Hyper-V-VMs wird bei der Servermigration eine Azure Migrate-Appliance verwendet, die Sie lokal bereitstellen. Die Appliance wird auch verwendet, wenn Sie die Serverbewertung einrichten. Dies wird im vorherigen Abschnitt beschrieben.

## <a name="selecting-assessment-and-migration-tools"></a>Auswählen von Bewertungs- und Migrationstools

Wählen Sie im Azure Migrate-Hub das Tool aus, das Sie für die Bewertung oder die Migration verwenden möchten, und fügen Sie es einem Azure Migrate-Projekt hinzu. Falls Sie ein ISV-Tool oder Movere hinzufügen, gehen Sie wie folgt vor:

- Beziehen Sie zunächst eine Lizenz, oder registrieren Sie sich für eine kostenlose Testversion (gemäß Anweisungen im Tool). Für jeden ISV bzw. jedes Tool ist eine Toollizenzierung angegeben.
- In jedem Tool ist eine Option zum Herstellen einer Verbindung mit Azure Migrate vorhanden. Befolgen Sie zur Verbindungsherstellung die Anweisungen im Tool.
- Verfolgen Sie Ihre Migration über alle Tools im Azure Migrate-Projekt nach.

## <a name="movere"></a>Movere

Bei Movere handelt es sich um eine SaaS-Plattform (Software-as-a-Service). Sie trägt durch die präzise Darstellung vollständiger IT-Umgebungen innerhalb eines Tages zur Verbesserung der Business Intelligence bei. Organisationen wachsen, verändern sich und werden digital optimiert. Mit Movere können diese Organisationen darauf vertrauen, dass sie dabei über die nötige Transparenz und Kontrolle für ihre Umgebungen verfügen – unabhängig von Plattform, Anwendung oder Geografie.

Movere wurde von Microsoft [übernommen](https://azure.microsoft.com/blog/microsoft-acquires-movere-to-help-customers-unlock-cloud-innovation-with-seamless-migration-tools/) und wird nicht mehr als eigenständiges Angebot verkauft. Movere steht im Rahmen des Microsoft-Programms für Lösungsbewertungen und Wirtschaftlichkeit in der Cloud zur Verfügung. Weitere Informationen zu Movere finden Sie [hier](https://www.movere.io).

Informieren Sie sich auch über Azure Migrate, unseren integrierten Migrationsdienst. Azure Migrate bietet einen zentralen Hub, um die Cloudmigration zu vereinfachen. Der Hub bietet umfassende Unterstützung für Workloads wie physische und virtuelle Server, Datenbanken und Anwendungen. Die End-to-End-Transparenz ermöglicht eine mühelose Nachverfolgung des Status während der Ermittlung, Bewertung und Migration.

Dank der integrierten Azure- und Partner-ISV-Tools bietet Azure Migrate ein breites Spektrum an Features, u. a.:

- Ermittlung virtueller und physischer Server
- Leistungsbasierte Größenanpassung
- Kostenplanung
- Importbasierte Bewertung
- Abhängigkeitsanalyse der Anwendungen ohne Agent

Sollten Sie professionelle Hilfe bei Ihren ersten Schritten benötigen: Microsoft verfügt über kompetente [Azure Expert Managed Service Provider](https://azure.microsoft.com/partners), die Sie auf Ihrem Weg begleiten. Besuchen Sie die [Azure Migrate-Website](https://azure.microsoft.com/services/azure-migrate/). 

## <a name="azure-migrate-versions"></a>Azure Migrate-Versionen

Es sind zwei Versionen des Azure Migrate-Diensts verfügbar:

- **Aktuelle Version**: Verwenden Sie diese Version, um Azure Migrate-Projekte zu erstellen, lokale Computer zu ermitteln und Bewertungen und Migrationen zu orchestrieren. [Erfahren Sie mehr](whats-new.md) über die Neuerungen in dieser Version.
- **Vorherige Version**: Die vorherige Version von Azure Migrate unterstützt nur die Bewertung von lokalen VMware-VMs. Falls Sie die vorherige Version verwendet haben, sollten Sie jetzt die aktuelle Version verwenden. Mit der vorherigen Version können keine Azure Migrate-Projekte mehr erstellt werden. Außerdem wird empfohlen, damit keine neuen Ermittlungen auszuführen.

    Wenn Sie auf bereits vorhandene Projekte im Azure-Portal zugreifen möchten, suchen Sie nach **Azure Migrate**, und wählen Sie die entsprechende Option aus. Auf dem Dashboard **Azure Migrate** finden Sie eine Benachrichtigung und einen Link für den Zugriff auf alte Azure Migrate-Projekte.

## <a name="next-steps"></a>Nächste Schritte

- Probieren Sie unsere Tutorials zum Bewerten von [VMware-VMs](tutorial-prepare-vmware.md), [Hyper-V-VMs](tutorial-prepare-hyper-v.md) und [physischen Servern](tutorial-prepare-physical.md) aus.
- Sehen Sie sich die [häufig gestellten Fragen](resources-faq.md) zu Azure Migrate an.
