---
title: Verwenden von SQL On-Demand (Vorschauversion)
description: In dieser Schnellstartanleitung erfahren Sie, wie Sie mit SQL On-Demand (Vorschauversion) ganz einfach verschiedene Dateitypen abfragen.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: ''
ms.date: 04/15/2020
ms.author: v-stazar
ms.reviewer: jrasnick
ms.openlocfilehash: 0d543abc88c1e45f2c1f5503473d8e92566fc582
ms.sourcegitcommit: b55d7c87dc645d8e5eb1e8f05f5afa38d7574846
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/16/2020
ms.locfileid: "81457381"
---
# <a name="quickstart-using-sql-on-demand"></a>Schnellstart: Verwenden von SQL On-Demand

Synapse SQL On-Demand (Vorschauversion) ist ein serverloser Abfragedienst zum Ausführen von SQL-Abfragen für Dateien in Azure Storage. In dieser Schnellstartanleitung erfahren Sie, wie Sie mit SQL On-Demand verschiedene Arten von Dateien abfragen.

Die folgenden Dateitypen werden unterstützt: JSON, CSV, Apache Parquet

## <a name="prerequisites"></a>Voraussetzungen

Wählen Sie einen SQL-Client aus, der zum Ausgeben von Abfragen verwendet werden soll:

- [Azure Synapse Studio](quickstart-synapse-studio.md) ist ein Webtool, mit dem Sie Dateien im Speicher durchsuchen und SQL-Abfragen erstellen können.
- [Azure Data Studio](sql/get-started-azure-data-studio.md) ist ein Clienttool, mit dem Sie SQL-Abfragen und Notebooks für Ihre On-Demand-Datenbank ausführen können.
- [SQL Server Management Studio](sql/get-started-ssms.md) ist ein Clienttool, mit dem Sie SQL-Abfragen für Ihre On-Demand-Datenbank ausführen können.

Parameter für den Schnellstart:

| Parameter                                 | BESCHREIBUNG                                                   |
| ----------------------------------------- | ------------------------------------------------------------- |
| Dienstendpunktadresse von SQL On-Demand    | Wird als Servername verwendet.                                   |
| Dienstendpunktregion von SQL On-Demand     | Wird verwendet, um zu bestimmen, welcher Speicher in Beispielen genutzt wird. |
| Benutzername und Kennwort für den Endpunktzugriff | Wird für den Zugriff auf den Endpunkt verwendet.                               |
| Datenbank für die Erstellung von Sichten         | Die Datenbank wird in Beispielen als Ausgangspunkt verwendet.       |

## <a name="first-time-setup"></a>Erstmalige Einrichtung

Vorbereitung:

- Erstellen Sie eine Datenbank für Ihre Sichten (sofern Sie Sichten verwenden möchten).
- Erstellen Sie Anmeldeinformationen, die von SQL On-Demand für den Zugriff auf Dateien im Speicher verwendet werden können.

### <a name="create-database"></a>Erstellen einer Datenbank

Erstellen Sie zu Demonstrationszwecken eine eigene Datenbank. Hierbei handelt es sich um die Datenbank, in der Ihre Sichten erstellt werden. Verwenden Sie diese Datenbank in den Beispielabfragen dieses Artikels.

> [!NOTE]
> Die Datenbanken werden nicht für tatsächliche Daten, sondern nur für Sichtmetadaten verwendet.
>
> Notieren Sie sich den verwendeten Datenbanknamen. Er wird im weiteren Verlauf der Schnellstartanleitung benötigt.

Verwenden Sie die folgende Abfrage, und ändern Sie `mydbname` in einen Namen Ihrer Wahl:

```sql
CREATE DATABASE mydbname
```

### <a name="create-credentials"></a>Erstellen von Anmeldeinformationen

Erstellen Sie Anmeldeinformationen, mit denen SQL On-Demand auf Dateien im Speicher zugreifen kann, um Abfragen mit SQL On-Demand zu ermöglichen.

> [!NOTE]
> Beachten Sie, dass Sie Anmeldeinformationen für den Zugriff auf das Speicherkonto erstellen müssen. SQL On-Demand kann zwar auf Speicher aus unterschiedlichen Regionen zugreifen, die Leistung ist jedoch besser, wenn sich der Speicher und der Azure Synapse-Arbeitsbereich in derselben Region befinden.

Ändern Sie den folgenden Codeausschnitt, um Anmeldeinformationen für CSV-, JSON- und Parquet-Container zu erstellen:

```sql
-- create credentials for containers in our demo storage account
IF EXISTS
   (SELECT * FROM sys.credentials
   WHERE name = 'https://sqlondemandstorage.blob.core.windows.net')
   DROP CREDENTIAL [https://sqlondemandstorage.blob.core.windows.net]
GO

CREATE CREDENTIAL [https://sqlondemandstorage.blob.core.windows.net]
WITH IDENTITY='SHARED ACCESS SIGNATURE',  
SECRET = 'sv=2018-03-28&ss=bf&srt=sco&sp=rl&st=2019-10-14T12%3A10%3A25Z&se=2061-12-31T12%3A10%3A00Z&sig=KlSU2ullCscyTS0An0nozEpo4tO5JAgGBvw%2FJX2lguw%3D'
GO
```

## <a name="querying-csv-files"></a>Abfragen von CSV-Dateien

Die folgende Abbildung zeigt eine Vorschau der abzufragenden Datei:

![Die ersten zehn Zeilen der CSV-Datei (ohne Kopfzeile und mit Neue-Zeile-Zeichen im Windows-Stil).](./sql/media/query-single-csv-file/population.png)

Die folgende Abfrage zeigt, wie eine CSV-Datei ohne Kopfzeile gelesen wird, die Neue-Zeile-Zeichen im Windows-Stil und kommagetrennte Spalten enthält:

```sql
SELECT TOP 10 *
FROM OPENROWSET
  (
      BULK 'https://sqlondemandstorage.blob.core.windows.net/csv/population/*.csv'
    , FORMAT = 'CSV'
  )
WITH
  (
      country_code VARCHAR (5)
    , country_name VARCHAR (100)
    , year smallint
    , population bigint
  ) AS r
WHERE
  country_name = 'Luxembourg' AND year = 2017
```

Das Schema kann zum Zeitpunkt der Abfragekompilierung angegeben werden.
Weitere Beispiele finden Sie unter [Abfragen von CSV-Datei](sql/query-single-csv-file.md).

## <a name="querying-parquet-files"></a>Abfragen von Parquet-Dateien

Im folgenden Beispiel werden die Funktionen des automatischen Schemarückschlusses beim Abfragen von Parquet-Dateien gezeigt. Hierzu wird die Anzahl von Zeilen im September 2017 ohne Angabe eines Schemas zurückgegeben.

> [!NOTE]
> Beim Lesen von Parquet-Dateien müssen keine Spalten in der Klausel `OPENROWSET WITH` angegeben werden. In diesem Fall verwendet SQL On-Demand Metadaten in der Parquet-Datei und bindet Spalten anhand des Namens.

```sql
SELECT COUNT_BIG(*)
FROM OPENROWSET
  (
      BULK 'https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/year=2017/month=9/*.parquet'
    , FORMAT='PARQUET'
  ) AS nyc
```

Weitere Informationen zum Abfragen von Parquet-Dateien finden Sie [hier](sql/query-parquet-files.md).

## <a name="querying-json-files"></a>Abfragen von JSON-Daten

### <a name="json-sample-file"></a>JSON-Beispieldatei

Die Dateien sind im Container *json* im Ordner *books* gespeichert und enthalten einen einzelnen Bucheintrag mit folgender Struktur:

```json
{  
   "_id":"ahokw88",
   "type":"Book",
   "title":"The AWK Programming Language",
   "year":"1988",
   "publisher":"Addison-Wesley",
   "authors":[  
      "Alfred V. Aho",
      "Brian W. Kernighan",
      "Peter J. Weinberger"
   ],
   "source":"DBLP"
}
```

### <a name="querying-json-files"></a>Abfragen von JSON-Daten

Die folgende Abfrage zeigt, wie Sie mithilfe von [JSON_VALUE](/sql/t-sql/functions/json-value-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) skalare Werte (Titel, Verleger) aus einem Buch mit dem Titel *Probabilistic and Statistical Methods in Cryptology, An Introduction by Selected articles* abrufen:

```sql
SELECT
    JSON_VALUE(jsonContent, '$.title') AS title
  , JSON_VALUE(jsonContent, '$.publisher') as publisher
  , jsonContent
FROM OPENROWSET
  (
      BULK 'https://sqlondemandstorage.blob.core.windows.net/json/books/*.json'
    , FORMAT='CSV'
    , FIELDTERMINATOR ='0x0b'
    , FIELDQUOTE = '0x0b'
    , ROWTERMINATOR = '0x0b'
  )
WITH
  ( jsonContent varchar(8000) ) AS [r]
WHERE
  JSON_VALUE(jsonContent, '$.title') = 'Probabilistic and Statistical Methods in Cryptology, An Introduction by Selected Topics'
```

> [!IMPORTANT]
> Die gesamte JSON-Datei wird als einzelne Zeile/Spalte gelesen. FIELDTERMINATOR, FIELDQUOTE und ROWTERMINATOR werden deshalb auf „0x0b“ festgelegt, da wir nicht erwarten, sie in der Datei zu finden.

## <a name="next-steps"></a>Nächste Schritte

Nun können Sie mit den folgenden Schnellstartartikeln beginnen:

- [Abfragen einer einzelnen CSV-Datei](sql/query-single-csv-file.md)
- [Abfragen von Ordnern und mehreren CSV-Dateien](sql/query-folders-multiple-csv-files.md)
- [Abfragen bestimmter Dateien](sql/query-specific-files.md)
- [Abfragen von Parquet-Dateien](sql/query-parquet-files.md)
- [Abfragen von geschachtelten Parquet-Typen](sql/query-parquet-nested-types.md)
- [Abfragen von JSON-Dateien](sql/query-json-files.md)
- [Erstellen und Verwenden von Sichten](sql/create-use-views.md)
- [Erstellen und Verwenden externer Tabellen](sql/create-use-external-tables.md)
- [Speichern des Abfrageergebnisses in Azure Storage](sql/create-external-table-as-select.md)

Im nächsten Artikel erfahren Sie, wie Sie eine einzelne CSV-Datei abfragen:
> [!div class="nextstepaction"]
> [Abfragen einer einzelnen CSV-Datei](sql/query-single-csv-file.md)
