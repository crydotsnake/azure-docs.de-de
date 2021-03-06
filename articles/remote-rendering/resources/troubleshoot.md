---
title: Problembehandlung
description: Informationen zum Troubleshooting bei Azure Remote Rendering
author: florianborn71
ms.author: flborn
ms.date: 02/25/2020
ms.topic: troubleshooting
ms.openlocfilehash: 7ee219ae5ace0f0da398cc542f410d3c895c8bd4
ms.sourcegitcommit: 642a297b1c279454df792ca21fdaa9513b5c2f8b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2020
ms.locfileid: "80678828"
---
# <a name="troubleshoot"></a>Problembehandlung

Auf dieser Seite finden Sie allgemeine Probleme, die Azure Remote Rendering beeinträchtigen, sowie Möglichkeiten, diese zu beheben.

## <a name="client-cant-connect-to-server"></a>Der Client kann keine Verbindung mit dem Server herstellen.

Stellen Sie sicher, dass Ihre Firewalls (auf dem Gerät, in Routern usw.) die folgenden Ports nicht blockieren:

* **50051 (TCP):** für die erste Verbindung (HTTP-Handshake) erforderlich
* **8266 (TCP + UDP):** für die Datenübertragung erforderlich
* **5000 (TCP)** , **5433 (TCP)** , **8443 (TCP)** : für [ArrInspector](tools/arr-inspector.md) erforderlich

## <a name="error-disconnected-videoformatnotavailable"></a>Fehler „Getrennt: VideoFormatNotAvailable“

Vergewissern Sie sich, dass Ihre GPU-Hardware Videodecodierung unterstützt. Weitere Informationen finden Sie unter [Entwicklungs-PC](../overview/system-requirements.md#development-pc).

Wenn Sie an einem Laptop mit zwei GPUs arbeiten, kann es vorkommen, dass die GPU, die Sie standardmäßig für Ausführungen verwenden, keine Funktion für die Hardwarevideodecodierung bietet. Versuchen Sie in diesem Fall, die Verwendung der anderen GPU durch die App zu erzwingen. Dies ist häufig in den Einstellungen des GPU-Treibers möglich.

## <a name="h265-codec-not-available"></a>H.265-Codec nicht verfügbar

Der Server kann aus zwei Gründen die Verbindung mit dem Fehler **Codec nicht verfügbar** ablehnen.

**Der H.265-Codec ist nicht installiert:**

Stellen Sie zunächst sicher, dass die **HEVC-Videoerweiterungen** wie im Abschnitt [Software](../overview/system-requirements.md#software) der Systemanforderungen beschrieben installiert wurde.

Wenn weiterhin Probleme auftreten, stellen Sie sicher, dass Ihre Grafikkarte H.265 unterstützt und der neueste Grafiktreiber installiert ist. Herstellerspezifische Informationen finden Sie im Abschnitt [Entwicklungs-PC](../overview/system-requirements.md#development-pc) der Systemanforderungen.

**Der Codec ist installiert, kann aber nicht verwendet werden:**

Der Grund für dieses Problem ist eine falsche Sicherheitseinstellung in den DLLs. Dieses Problem tritt beim Versuch, mit H.265 codierte Videos anzusehen, nicht auf. Das Problem wird durch eine Neuinstallation des Codecs auch nicht behoben. Führen Sie stattdessen die folgenden Schritte aus:

1. Öffnen Sie eine **PowerShell-Instanz mit Administratorrechten**, und führen Sie folgenden Befehl aus:

    ```PowerShell
    Get-AppxPackage -Name Microsoft.HEVCVideoExtension
    ```
  
    Dieser Befehl sollte die `InstallLocation` des Codecs ausgeben, z. B.:
  
    ```cmd
    InstallLocation   : C:\Program Files\WindowsApps\Microsoft.HEVCVideoExtension_1.0.23254.0_x64__5wasdgertewe
    ```

1. Öffnen Sie diesen Ordner in Windows-Explorer.
1. Es sollten die Unterordner **x86** und **x64** vorhanden sein. Klicken Sie mit der rechten Maustaste auf einen der Ordner, und wählen Sie **Eigenschaften** aus.
    1. Wählen Sie die Registerkarte **Sicherheit** aus, und klicken Sie auf die Schaltfläche **Erweitert**.
    1. Klicken Sie für **Besitzer** auf **Ändern**.
    1. Geben Sie in das Textfeld **Administratoren** ein.
    1. Klicken Sie auf **Namen überprüfen** und auf **OK**.
1. Wiederholen Sie die angegebenen Schritte für den anderen Ordner.
1. Wiederholen Sie außerdem die angegebenen Schritte für jede DLL-Datei in beiden Ordnern. Insgesamt sollten vier DLL-Dateien vorhanden sein.

Um zu überprüfen, ob die Einstellungen jetzt richtig sind, gehen Sie für jede der vier DLL-Dateien folgendermaßen vor:

1. Wählen Sie **Eigenschaften > Sicherheit > Bearbeiten** aus.
1. Gehen Sie die Liste aller **Gruppen/Benutzer** durch, und stellen Sie sicher, dass jeder Eintrag über die Berechtigung **Lesen und ausführen** verfügt (das Kontrollkästchen in der Spalte **Zulassen** muss aktiviert sein).

## <a name="low-video-quality"></a>Geringe Videoqualität

Die Videoqualität kann durch die Netzwerkqualität oder einen fehlenden H.265-Videocodec beeinträchtigt werden.

* Weitere Informationen finden Sie in den Schritten zum [Identifizieren von Netzwerkproblemen](#unstable-holograms).
* Informationen zum Installieren des aktuellen Grafiktreibers finden Sie in den [Systemanforderungen](../overview/system-requirements.md#development-pc).

## <a name="black-screen-after-successful-model-loading"></a>Schwarzer Bildschirm nach erfolgreichem Laden des Modells

Wenn Sie mit der Renderingruntime verbunden sind und erfolgreich ein Modell geladen haben, aber danach nur ein schwarzer Bildschirm angezeigt wird, kann dies verschiedene Gründe haben.

Es wird empfohlen, die folgenden Tests auszuführen, bevor Sie eine ausführlichere Analyse durchführen:

* Ist der H.265-Codec installiert? Obwohl ein Fallback auf den H.264-Codec erfolgen sollte, wurden bereits Fälle beobachtet, in denen dieses Fallback nicht ordnungsgemäß funktionierte. Informationen zum Installieren des aktuellen Grafiktreibers finden Sie in den [Systemanforderungen](../overview/system-requirements.md#development-pc).
* Wenn Sie ein Unity-Projekt verwenden, schließen Sie Unity, löschen Sie die temporären Ordner *library* und *obj* im Projektverzeichnis, laden Sie das Projekt erneut, und erstellen Sie es neu. In einigen Fällen konnte das Beispiel aufgrund von zwischengespeicherten Daten nicht ordnungsgemäß ausgeführt werden, ohne dass ein offensichtlicher Grund vorlag.

Wenn diese beiden Schritte nicht hilfreich waren, müssen Sie herausfinden, ob der Client Videoframes empfängt. Dies kann programmgesteuert abgefragt werden, wie im Kapitel zu [serverseitigen Leistungsabfragen](../overview/features/performance-queries.md) erläutert. Das `FrameStatistics struct` weist einen Member auf, der angibt, wie viele Videoframes empfangen wurden. Wenn diese Zahl größer als 0 ist und im Lauf der Zeit zunimmt, empfängt der Client Videoframes vom Server. Es muss sich demzufolge um ein Problem auf der Clientseite handeln.

### <a name="common-client-side-issues"></a>Häufige clientseitige Fehler

**Das Modell befindet sich nicht im Ansichtsfrustum:**

In vielen Fällen wird das Modell ordnungsgemäß dargestellt, befindet sich jedoch außerhalb des Kamerafrustums. Ein häufiger Grund dafür ist, dass das Modell mit einem Pivot deutlich außerhalb des Mittelpunkts exportiert wurde, sodass es von der Entfernungsclippingebene der Kamera abgeschnitten wird. Es hilft, den Begrenzungsrahmen des Modells programmgesteuert abzufragen und mit Unity als Linienrahmen zu visualisieren oder die entsprechenden Werte im Debugprotokoll auszugeben.

**Die Unity-Renderpipeline enthält nicht die Renderhooks:**

Azure Remote Rendering erstellt Hooks in der Unity-Renderpipeline, um die Framezusammenstellung mit dem Video und die Neuprojektion durchzuführen. Um zu überprüfen, ob diese Hooks vorhanden sind, öffnen Sie das Menü *Window > Analysis > Frame debugger* (Fenster > Analyse > Framedebugger). Aktivieren Sie ihn, und stellen Sie sicher, dass zwei Einträge für `HolographicRemotingCallbackPass` in der Pipeline vorhanden sind:

![Unity-Framedebugger](./media/troubleshoot-unity-pipeline.png)

## <a name="unity-code-using-the-remote-rendering-api-doesnt-compile"></a>Unity-Code, der die Remote Rendering-API verwendet, wird nicht kompiliert.

Stellen Sie den *Buildtyp* der Unity-Lösung auf **Debug** um. Beim Testen von ARR im Unity-Editor ist die Definition `UNITY_EDITOR` nur in Debugbuilds verfügbar. Beachten Sie, dass dies nicht mit dem Buildtyp in Zusammenhang steht, der für [bereitgestellte Anwendungen](../quickstarts/deploy-to-hololens.md) verwendet wird. Hierbei sollten Sie Releasebuilds verwenden.

## <a name="unstable-holograms"></a>Instabile Hologramme

Wenn bei Kopfbewegungen auch gerenderte Objekte bewegt werden, liegen möglicherweise Probleme mit der *Late Stage Reprojection* (LSR) vor. Hinweise zur Vorgehensweise in einer solchen Situation finden Sie im Abschnitt zur [Late Stage Reprojection](../overview/features/late-stage-reprojection.md).

Ein weiterer Grund für instabile Hologramme (wabernde, verzerrte, zitternde oder springende Hologramme) kann eine schlechte Netzwerkkonnektivität sein, insbesondere unzureichende Netzwerkbandbreite oder zu hohe Latenz. Ein guter Indikator für die Qualität der Netzwerkverbindung ist der [Leistungsstatistikwert](../overview/features/performance-queries.md) `ARRServiceStats.VideoFramesReused`. Wiederverwendete Frames deuten auf Situationen hin, in denen ein altes Videoframe auf der Clientseite wiederverwendet werden musste, da kein neues Videoframe verfügbar war – beispielsweise aufgrund von Paketverlusten oder von Schwankungen der Netzwerklatenz. Wenn `ARRServiceStats.VideoFramesReused` häufig größer als null ist, deutet dies auf ein Netzwerkproblem hin.

Ein anderer Wert, der untersucht werden kann, ist `ARRServiceStats.LatencyPoseToReceiveAvg`. Er sollte konstant weniger als 100 ms betragen. Wenn höhere Werte angezeigt werden, bedeutet dies, dass Sie mit einem zu weit entfernten Rechenzentrum verbunden sind.

Eine Liste der möglichen Entschärfungen finden Sie in den [Richtlinien für die Netzwerkkonnektivität](../reference/network-requirements.md#guidelines-for-network-connectivity).

## <a name="next-steps"></a>Nächste Schritte

* [Systemanforderungen](../overview/system-requirements.md)
* [Netzwerkanforderungen](../reference/network-requirements.md)
