---
title: 'Azure Batch: Taskfehlerereignis'
description: Referenz zum Batch-Taskfehlerereignis. Dieses Ereignis wird zusätzlich zum Ereignis zum Abschluss eines Tasks ausgegeben und dient zum Ermitteln, wann ein Task fehlgeschlagen ist.
ms.topic: article
ms.date: 08/15/2019
ms.openlocfilehash: 0e973a7e0a2ab67300a0f6762c837336e12bae3b
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82116450"
---
# <a name="task-fail-event"></a>Taskfehlerereignis

 Dieses Ereignis wird ausgegeben, wenn ein Task mit einem Fehler abgeschlossen wird. Derzeit gelten alle Exitcodes ungleich null als Fehler. Dieses Ereignis wird *zusätzlich* zum Ereignis zum Abschluss eines Tasks ausgegeben und dient zum Ermitteln, wann ein Task fehlgeschlagen ist.


 Das folgende Beispiel zeigt den Text eines Taskfehlerereignisses.

```
{
    "jobId": "myJob",
    "id": "myTask",
    "taskType": "User",
    "systemTaskVersion": 0,
    "nodeInfo": {
        "poolId": "pool-001",
        "nodeId": "tvm-257509324_1-20160908t162728z"
    },
    "multiInstanceSettings": {
        "numberOfInstances": 1
    },
    "constraints": {
        "maxTaskRetryCount": 2
    },
    "executionInfo": {
        "startTime": "2016-09-08T16:32:23.799Z",
        "endTime": "2016-09-08T16:34:00.666Z",
        "exitCode": 1,
        "retryCount": 2,
        "requeueCount": 0
    }
}
```

|Elementname|type|Notizen|
|------------------|----------|-----------|
|`jobId`|String|Die ID des Auftrags, der den Task enthält.|
|`id`|String|Die ID des Tasks.|
|`taskType`|String|Der Typ des Tasks. Entweder „JobManager“, was bedeutet, dass dies ein Auftrags-Manager-Task ist, oder „User“, was bedeutet, dass dies nicht der Fall ist. Dieses Ereignis wird nicht für Auftragsvorbereitungstasks, Auftragsfreigabetasks oder Starttasks ausgegeben.|
|`systemTaskVersion`|Int32|Dies ist der interne Wiederholungszähler für einen Task. Der Batch-Dienst kann intern einen Task wiederholen, um vorübergehende Probleme zu berücksichtigen. Bei diesen Problemen kann es sich um interne Planungsfehler oder Versuche handeln, Computeknoten mit einem fehlerhaften Status wiederherzustellen.|
|[`nodeInfo`](#nodeInfo)|Komplexer Typ|Enthält Informationen zu den Computeknoten, auf dem der Task ausgeführt wurde.|
|[`multiInstanceSettings`](#multiInstanceSettings)|Komplexer Typ|Gibt an, dass der Task ein Task mit mehreren Instanzen ist, für den mehrere Computeknoten erforderlich sind.  Ausführliche Informationen finden Sie unter [`multiInstanceSettings`](https://docs.microsoft.com/rest/api/batchservice/get-information-about-a-task).|
|[`constraints`](#constraints)|Komplexer Typ|Die Ausführungseinschränkungen, die für diesen Task gelten.|
|[`executionInfo`](#executionInfo)|Komplexer Typ|Enthält Informationen zur Ausführung des Tasks.|

###  <a name="nodeinfo"></a><a name="nodeInfo"></a> nodeInfo

|Elementname|type|Notizen|
|------------------|----------|-----------|
|`poolId`|String|Die ID des Pools, auf den der Task angewendet wurde.|
|`nodeId`|String|Die ID des Knotens, auf dem der Task ausgeführt wurde.|

###  <a name="multiinstancesettings"></a><a name="multiInstanceSettings"></a> multiInstanceSettings

|Elementname|type|Notizen|
|------------------|----------|-----------|
|`numberOfInstances`|Int32|Die Anzahl der Computeknoten, die vom Task benötigt werden.|

###  <a name="constraints"></a><a name="constraints"></a> Einschränkungen

|Elementname|type|Notizen|
|------------------|----------|-----------|
|`maxTaskRetryCount`|Int32|Gibt an, wie oft der Task maximal wiederholt werden kann. Der Batch-Dienst wiederholt einen Task, wenn sein Exitcode ungleich null ist.<br /><br /> Beachten Sie, dass dieser Wert die Anzahl der Wiederholungen ausdrücklich steuert. Der Batch-Dienst wiederholt den Task einmal und kann ihn anschließend bis zu diesem Grenzwert wiederholen. Wenn beispielsweise die maximale Anzahl von Wiederholungsversuchen 3 ist, versucht der Batch-Dienst einen Task bis zu viermal (ein erster Versuch und drei Wiederholungsversuche).<br /><br /> Wenn die maximale Anzahl von Wiederholungsversuchen 0 ist, wiederholt der Batch-Dienst Tasks nicht.<br /><br /> Wenn die maximale Anzahl von Wiederholungsversuchen -1 ist, wiederholt der Batch-Dienst Tasks unbegrenzt.<br /><br /> Der Standardwert ist 0 (keine Wiederholungsversuche).|


###  <a name="executioninfo"></a><a name="executionInfo"></a> executionInfo

|Elementname|type|Notizen|
|------------------|----------|-----------|
|`startTime`|Datetime|Der Zeitpunkt, an dem die Ausführung des Tasks gestartet wurde. „Running“ entspricht dem Status **Wird ausgeführt**. Wenn also der Task Ressourcendateien oder Anwendungspakete angibt, reflektiert die Startzeit den Zeitpunkt, an dem der Task mit dem Herunterladen oder Bereitstellen dieser Elemente begonnen hat.  Wenn der Task neu gestartet oder wiederholt wurde, ist dies der letzte Zeitpunkt, an dem die Ausführung des Tasks gestartet wurde.|
|`endTime`|Datetime|Der Zeitpunkt, an dem die Ausführung des Tasks beendet wurde.|
|`exitCode`|Int32|Der Exitcode des Tasks.|
|`retryCount`|Int32|Die Häufigkeit, mit der der Task vom Batch-Dienst wiederholt wurde. Der Vorgang wird wiederholt, wenn der Exitcode ungleich null ist, und zwar bis zum angegebenen Wert von „MaxTaskRetryCount“.|
|`requeueCount`|Int32|Die Häufigkeit, mit der der Tasks vom Batch-Dienst als Ergebnis einer Benutzeranforderung erneut in die Warteschlange gestellt wurde.<br /><br /> Wenn der Benutzer Knoten aus einem Pool entfernt (durch Vergrößern oder Verkleinern des Pools) oder der Auftrag deaktiviert wird, kann der Benutzer angeben, dass auf den Knoten ausgeführte Tasks zur Ausführung erneut in die Warteschlange gestellt werden. Dieser Zähler überwacht, wie oft der Task aus diesen Gründen in die Warteschlange gestellt wurde.|
