---
title: Transformation für bedingtes Teilen im Zuordnungsdatenfluss
description: Es wird beschrieben, wie Sie Daten auf verschiedene Streams aufteilen, indem Sie die Transformation für bedingtes Teilen im Azure Data Factory-Zuordnungsdatenfluss verwenden.
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 10/16/2019
ms.openlocfilehash: bd9241e526d7cf42f0697afb8635c085a08c80d8
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81606480"
---
# <a name="conditional-split-transformation-in-mapping-data-flow"></a>Transformation für bedingtes Teilen im Zuordnungsdatenfluss

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Bei der Transformation für bedingtes Teilen werden Datenzeilen basierend auf Übereinstimmungsbedingungen an verschiedene Streams geleitet. Die Transformation für bedingtes Teilen ist mit einer CASE-Entscheidungsstruktur in einer Programmiersprache vergleichbar. Die Transformation wertet Ausdrücke aus und leitet dann die Datenzeile basierend auf den Ergebnissen an den angegebenen Datenstrom weiter.

## <a name="configuration"></a>Konfiguration

Mit der Einstellung **Teilen bei** wird bestimmt, ob die Zeile mit den Daten an den ersten übereinstimmenden Stream oder an alle übereinstimmenden Streams geleitet wird.

Verwenden Sie den Ausdrucks-Generator für Datenflüsse, um einen Ausdruck für die Teilungsbedingung einzugeben. Klicken Sie zum Hinzufügen einer neuen Bedingung in einer vorhandenen Zeile auf das Pluszeichen. Darüber hinaus kann für Zeilen, für die sich keine Übereinstimmung mit einer Bedingung ergibt, auch ein Standardstream hinzugefügt werden.

![Bedingtes Teilen](media/data-flow/conditionalsplit1.png "Optionen für bedingtes Teilen")

## <a name="data-flow-script"></a>Datenflussskript

### <a name="syntax"></a>Syntax

```
<incomingStream>
    split(
        <conditionalExpression1>
        <conditionalExpression2>
        ...
        disjoint: {true | false}
    ) ~> <splitTx>@(stream1, stream2, ..., <defaultStream>)
```

### <a name="example"></a>Beispiel

Das folgende Beispiel ist eine Transformation für bedingtes Teilen mit dem Namen `SplitByYear`, bei der der eingehende Stream `CleanData` verwendet wird. Diese Transformation verfügt über die beiden Teilungsbedingungen `year < 1960` und `year > 1980`. `disjoint` ist „false“, weil die Daten an die erste übereinstimmende Bedingung geleitet werden. Alle Zeilen, die die erste Bedingung erfüllen, werden an den Ausgabestream `moviesBefore1960` geleitet. Alle verbleibenden Zeilen, die die zweite Bedingung erfüllen, werden an den Ausgabestream `moviesAFter1980` geleitet. Für alle anderen Zeilen wird der Standardstream `AllOtherMovies` verwendet.

Auf der Data Factory-Benutzeroberfläche sieht diese Transformation wie folgt aus:

![Bedingtes Teilen](media/data-flow/conditionalsplit1.png "Optionen für bedingtes Teilen")

Das Datenflussskript für diese Transformation befindet sich im folgenden Codeausschnitt:

```
CleanData
    split(
        year < 1960,
        year > 1980,
        disjoint: false
    ) ~> SplitByYear@(moviesBefore1960, moviesAfter1980, AllOtherMovies)
```

## <a name="next-steps"></a>Nächste Schritte

Häufig verwendete Datenflusstransformationen, die zusammen mit dem bedingten Teilen verwendet werden, sind die [Join-Transformation](data-flow-join.md), [Suchtransformation](data-flow-lookup.md) und [Auswahltransformation](data-flow-select.md).
