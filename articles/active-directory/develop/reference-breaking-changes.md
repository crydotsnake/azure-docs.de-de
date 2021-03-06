---
title: Referenz zu wichtigen Änderungen in Azure Active Directory
description: Hier erhalten Sie Informationen zu Änderungen an den Azure AD-Protokollen, die sich auf Ihre Anwendung auswirken können.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 3/13/2020
ms.author: ryanwi
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 52ede7d66bd657b5002272e34673b4b01c9ab1aa
ms.sourcegitcommit: d187fe0143d7dbaf8d775150453bd3c188087411
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/08/2020
ms.locfileid: "80883456"
---
# <a name="whats-new-for-authentication"></a>Neuerungen bei der Authentifizierung 

>Erhalten Sie Benachrichtigungen über Aktualisierungen dieser Seite. Fügen Sie Ihrem RSS-Feedreader einfach [diese URL](https://docs.microsoft.com/api/search/rss?search=%22whats%20new%20for%20authentication%22&locale=en-us) hinzu.

Für das Authentifizierungssystem werden fortlaufend Änderungen vorgenommen und Features hinzugefügt, um die Sicherheit und Einhaltung von Standards zu verbessern. Damit Sie auf dem neuesten Stand der aktuellen Entwicklungen bleiben, bietet dieser Artikel Informationen zu Folgendem:

- Neueste Features
- Bekannte Probleme
- Protokolländerungen
- Veraltete Funktionen

> [!TIP] 
> Diese Seite wird regelmäßig aktualisiert. Daher sollten Sie die Seite oft besuchen. Sofern nicht anders angegeben, werden diese Änderungen nur für neu registrierte Anwendungen umgesetzt.  

## <a name="upcoming-changes"></a>Bevorstehende Änderungen

Zurzeit sind keine geplant.  Nachfolgend finden Sie weitere Informationen zu den Änderungen, die in der Produktionsumgebung bestehen oder eingeführt werden. 

## <a name="march-2020"></a>März 2020 

### <a name="user-passwords-will-be-restricted-to-256-characters"></a>Benutzerkennwörter werden auf 256 Zeichen beschränkt.

**Gültigkeitsdatum:** 13. März 2020

**Betroffene Endpunkte:** v1.0 und v2.0

**Betroffenes Protokoll:** Alle Benutzerflows 

Benutzer, die sich mit Kennwörtern mit mehr als 256 Zeichen direkt bei Azure AD (im Gegensatz zu einem Verbundidentitätsanbieter wie AD FS) anmelden, können sich ab dem 13. März 2020 nicht mehr anmelden und werden stattdessen aufgefordert, ihr Kennwort zurückzusetzen.  Administratoren erhalten möglicherweise Anforderungen zum Zurücksetzen der Kennwörter der Benutzer. 

Der Fehler in den Anmeldeprotokollen lautet: AADSTS 50052: InvalidPasswordExceedsMaxLength

Meldung: `The password entered exceeds the maximum length of 256. Please reach out to your admin to reset the password.`

Abhilfe:

Die Benutzer können sich nicht anmelden, da das Kennwort die zulässige maximale Länge überschreitet. Benutzer sollten sich an den Administrator wenden, damit das Kennwort zurückgesetzt wird. Wenn SSPR für den zugehörigen Mandanten aktiviert ist, können Benutzer das Kennwort über den Link „Kennwort vergessen“ zurücksetzen.



## <a name="february-2020"></a>Februar 2020 

### <a name="empty-fragments-will-be-appended-to-every-http-redirect-from-the-login-endpoint"></a>An jede HTTP-Umleitung vom Endpunkt der Anmeldung werden leere Fragmente angehängt. 

**Gültigkeitsdatum:** 8. Februar 2020

**Betroffene Endpunkte:** v1.0 und v2.0

**Betroffenes Protokoll:** OAuth-und OIDC-Flows, die „response_type=query“ verwenden. Dies betrifft in einigen Fällen den [Autorisierungscodeflow](v2-oauth2-auth-code-flow.md) und den [impliziten Flow](v2-oauth2-implicit-grant-flow.md). 

Wenn eine Authentifizierungsantwort von login.microsoftonline.com über die HTTP-Umleitung an eine Anwendung gesendet wird, hängt der Dienst ein leeres Fragment an die Antwort-URL an.  Dadurch wird eine Klasse von Umleitungsangriffen verhindert, indem sichergestellt wird, dass der Browser jedes vorhandene Fragment in der Authentifizierungsanforderung bereinigt.  Keine App sollte eine Abhängigkeit von diesem Verhalten aufweisen. 


## <a name="august-2019"></a>August 2019

### <a name="post-form-semantics-will-be-enforced-more-strictly---spaces-and-quotes-will-be-ignored"></a>POST-Formularsemantik wird strenger erzwungen – Leerzeichen und Anführungszeichen werden ignoriert.

**Gültigkeitsdatum:** 2. September 2019

**Betroffene Endpunkte:** v1.0 und v2.0

**Betroffenes Protokoll:** Überall dort, wo POST verwendet wird ([Clientanmeldeinformationen](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow), [Einlösung von Autorisierungscodes](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow), [ROPC](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth-ropc), [OBO](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) und [Einlösung von Aktualisierungstoken](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow#refresh-the-access-token))

Ab der Woche vom 2. September werden Authentifizierungsanforderungen, die die POST-Methode verwenden, mit strengeren HTTP-Standards überprüft.  Insbesondere werden Leerzeichen und doppelte Anführungszeichen (") nicht mehr aus den Anforderungsformularwerten entfernt. Es wird nicht erwartet, dass diese Änderungen vorhandene Clients unterbrechen, und durch diese Änderungen wird sichergestellt, dass an Azure AD gesendete Anforderungen jedes Mal zuverlässig verarbeitet werden. In Zukunft (siehe oben) planen wir, doppelte Parameter zusätzlich abzulehnen und die BOM innerhalb von Anforderungen zu ignorieren. 

Beispiel:

Heute wird `?e=    "f"&g=h` wie `?e=f&g=h` analysiert, also `e` == `f`.  Durch diese Änderung würde die Analyse jetzt so durchgeführt, dass `e` == `    "f"`. Es ist unwahrscheinlich, dass es sich dabei um ein gültiges Argument handelt, und die Anforderung würde jetzt fehlschlagen. 


## <a name="july-2019"></a>Juli 2019

### <a name="app-only-tokens-for-single-tenant-applications-are-only-issued-if-the-client-app-exists-in-the-resource-tenant"></a>Reine App-Token für Anwendungen mit einem einzigen Mandanten werden nur ausgegeben, wenn die Client-App im Ressourcenmandanten vorhanden ist.

**Gültigkeitsdatum:** 26. Juli 2019

**Betroffene Endpunkte:** [v1.0](https://docs.microsoft.com/azure/active-directory/develop/v1-oauth2-client-creds-grant-flow) und [v2.0](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)

**Betroffenes Protokoll:** [Clientanmeldeinformationen (reine App-Token)](https://docs.microsoft.com/azure/active-directory/develop/v1-oauth2-client-creds-grant-flow)

Ab dem 26. Juli gilt eine Sicherheitsänderung, mit der die Ausgabe von reinen App-Token (über die Zuweisung von Clientanmeldeinformationen) geändert wird. Bisher konnten Anwendungen Token zum Aufruf einer beliebigen anderen App abrufen, unabhängig vom Vorhandensein im Mandanten oder von genehmigten Rollen für diese Anwendung.  Dieses Verhalten wurde aktualisiert, sodass für Ressourcen (gelegentlich auch Web-APIs genannt) mit einem einzigen Mandanten (Standard) die Clientanwendung jetzt innerhalb des Ressourcenmandanten vorliegen muss.  Beachten Sie, dass eine vorhandene Zustimmung zwischen Client und API weiterhin nicht erforderlich ist. Ferner müssen Apps weiterhin eigene Autorisierungsüberprüfungen durchführen, um sicherzustellen, dass ein `roles`-Anspruch vorhanden ist und den erwarteten Wert für die API enthält.

Die Fehlermeldung für dieses Szenario lautet aktuell folgendermaßen: 

`The service principal named <appName> was not found in the tenant named <tenant_name>. This can happen if the application has not been installed by the administrator of the tenant.`

Um dieses Problem zu beseitigen, verwenden Sie die Benutzeroberfläche für die Administratorzustimmung, um den Dienstprinzipal der Clientanwendung in Ihrem Mandanten zu erstellen, oder erstellen Sie diesen manuell.  Durch diese Anforderung wird sichergestellt, dass der Mandant der App die Berechtigung zum Betrieb innerhalb des Mandanten erteilt hat.  

#### <a name="example-request"></a>Beispielanforderung

`https://login.microsoftonline.com/contoso.com/oauth2/authorize?resource=https://gateway.contoso.com/api&response_type=token&client_id=14c88eee-b3e2-4bb0-9233-f5e3053b3a28&...` In diesem Beispiel lautet der Ressourcenmandant (Autorität) contoso.com, die Ressourcen-App ist eine Einzelmandanten-App namens `gateway.contoso.com/api` für den Contoso-Mandanten, und die Client-App ist `14c88eee-b3e2-4bb0-9233-f5e3053b3a28`.  Wenn die Client-App über einen Dienstprinzipal innerhalb von contoso.com verfügt, kann diese Anforderung fortgesetzt werden.  Falls nicht, ist die Anforderung nicht erfolgreich, und es wird der oben gezeigte Fehler gemeldet.  

Wenn es sich bei der Contoso-Gateway-App um eine mehrinstanzenfähige App handeln würde, könnte die Anforderung unabhängig davon fortgesetzt werden, ob die Client-App über einen Dienstprinzipal innerhalb von contoso.com verfügt.  

### <a name="redirect-uris-can-now-contain-query-string-parameters"></a>Umleitungs-URIs können jetzt Abfragezeichenfolgenparameter enthalten

**Gültigkeitsdatum:** 22. Juli 2019

**Betroffene Endpunkte:** v1.0 und v2.0

**Betroffenes Protokoll:** Alle Flows

Gemäß [RFC 6749](https://tools.ietf.org/html/rfc6749#section-3.1.2) können in Azure AD-Anwendungen ab sofort Umleitungs-URIs (Antwort-URIs) mit statischen Abfrageparametern (z. B. `https://contoso.com/oauth2?idp=microsoft`) für OAuth 2.0-Anforderungen registriert und verwendet werden.  Dynamische Umleitungs-URIs sind weiterhin untersagt, weil sie ein Sicherheitsrisiko darstellen und nicht dazu verwendet werden können, statische Informationen über eine Authentifizierungsanforderung hinweg beizubehalten. Verwenden Sie in diesem Fall den `state`-Parameter.

Der statische Abfrageparameter wird, ebenso wie jeder andere Bestandteil des Umleitungs-URI, einem Zeichenfolgenabgleich unterzogen. Wenn keine registrierte Zeichenfolge vorhanden ist, die dem URI-decodierten Umleitungs-URI entspricht, wird die Anforderung abgelehnt.  Wird der Antwort-URI in der App-Registrierung gefunden, wird die gesamte Zeichenfolge – einschließlich des statischen Abfrageparameters – zum Umleiten des Benutzers verwendet. 

Beachten Sie, dass die Benutzeroberfläche zur App-Registrierung im Azure-Portal zum aktuellen Zeitpunkt (Ende Juli 2019) Abfrageparameter weiterhin blockiert.  Sie können das Anwendungsmanifest jedoch manuell bearbeiten, um Abfrageparameter in Ihre App einzufügen und diese zu testen.  


## <a name="march-2019"></a>März 2019

### <a name="looping-clients-will-be-interrupted"></a>In der Schleife befindliche Clients werden unterbrochen

**Gültigkeitsdatum:** 25. März 2019

**Betroffene Endpunkte:** v1.0 und v2.0

**Betroffenes Protokoll:** Alle Flows

Clientanwendungen können manchmal ein unerwünschtes Verhalten aufweisen und über einen kurzen Zeitraum Hunderte derselben Anmeldeanforderung ausgeben.  Diese Anforderungen können erfolgreich oder nicht erfolgreich sein, aber sie alle tragen zu einer schlechten Benutzererfahrung und erhöhten Workloads für den IDP bei, was zu einer höheren Latenz für alle Benutzer und einer geringeren Verfügbarkeit des IDP führt.  Die Ausführung dieser Anwendungen erfolgt außerhalb der Grenzen der normalen Nutzung. Sie sollten aktualisiert werden, um das ordnungsgemäße Verhalten aufzuweisen.  

Clients, die doppelte Anforderungen ausgeben, erhalten einen `invalid_grant`-Fehler: `AADSTS50196: The server terminated an operation because it encountered a loop while processing a request`. 

Das Verhalten der meisten Clients muss nicht geändert werden, um diesen Fehler zu vermeiden.  Von diesem Fehler sind nur falsch konfigurierte Clients betroffen, also Clients ohne Zwischenspeicherung von Token oder Clients, die sich bereits in Prompt-Schleifen befinden.  Clients werden pro Instanz lokal (über Cookie) anhand der folgenden Faktoren nachverfolgt:

* Benutzerhinweis, falls vorhanden

* Angeforderte Bereiche oder Ressourcen

* Client-ID

* Umleitungs-URI

* Antworttyp und -modus

Apps, die innerhalb kurzer Zeit (5 Minuten) mehrere Anforderungen (15 und mehr) senden, erhalten einen `invalid_grant`-Fehler, der erklärt, dass sie sich in einer Schleife befinden.  Die angeforderten Token haben eine ausreichend lange Lebensdauer (mindestens 10 Minuten, standardmäßig 60 Minuten), sodass in diesem Zeitraum keine wiederholten Anforderungen erforderlich sind.  

Alle Apps sollten `invalid_grant` verarbeiten, indem sie eine interaktive Eingabeaufforderung anzeigen, statt automatisch ein Token anzufordern.  Um diesen Fehler zu vermeiden, sollte für die Clients sichergestellt werden, dass sie die empfangenen Token ordnungsgemäß zwischenspeichern.


## <a name="october-2018"></a>Oktober 2018

### <a name="authorization-codes-can-no-longer-be-reused"></a>Autorisierungscodes können nicht mehr wiederverwendet werden.

**Gültigkeitsdatum:** 15. November 2018

**Betroffene Endpunkte:** v1.0 und v2.0

**Betroffenes Protokoll:** [Codeflow](v2-oauth2-auth-code-flow.md)

Ab dem 15. November 2018 akzeptiert Azure AD bereits zuvor verwendete Authentifizierungscodes für Apps nicht mehr. Diese Sicherheitsänderung trägt dazu bei, Azure AD an die OAuth-Spezifikation anzupassen. Sie wird auf v1- und v2-Endpunkten erzwungen.

Wenn Ihre App Autorisierungscodes zum Abrufen von Token für mehrere Ressourcen wiederverwendet, sollten Sie den Code für das Abrufen eines Aktualisierungstokens verwenden. Verwenden Sie dieses Aktualisierungstoken anschließend, um die zusätzlichen Token für andere Ressourcen abzurufen. Autorisierungscodes können nur einmal verwendet werden, Aktualisierungstoken hingegen können mehrere Male und für mehrere Ressourcen verwendet werden. Für jede neue App, die einen Authentifizierungscode im OAuth-Codefluss erneut verwenden möchte, wird ein Fehler vom Typ „invalid_grant“ zurückgegeben.

Weitere Informationen zu Aktualisierungstoken finden Sie unter [Aktualisieren der Zugriffstoken](v2-oauth2-auth-code-flow.md#refresh-the-access-token).  Wenn Sie ADAL oder MSAL verwenden, wird dies automatisch durch die Bibliothek durchgeführt – ersetzen Sie die zweite Instanz von AcquireTokenByAuthorizationCodeAsync durch AcquireTokenSilentAsync. 

## <a name="may-2018"></a>Mai 2018

### <a name="id-tokens-cannot-be-used-for-the-obo-flow"></a>ID-Token können nicht für den OBO-Fluss verwendet werden.

**Datum:** 1. Mai 2018

**Betroffene Endpunkte:** v1.0 und v2.0

**Betroffene Protokolle:** Impliziter Ablauf und [„im Auftrag von“-Ablauf](v2-oauth2-on-behalf-of-flow.md)

Seit dem 1. Mai 2018 können ID-Token nicht mehr als Assertion in einem OBO-Fluss für neue Anwendungen verwendet werden. Stattdessen müssen Zugriffstoken zum Schützen von APIs genutzt werden (auch zwischen einem Client und der mittleren Ebene der gleichen Anwendung). Vor dem 1. Mai 2018 registrierte Apps funktionieren weiterhin und können ID-Token gegen Zugriffstoken austauschen. Dieses Muster wird jedoch nicht empfohlen.

Sie können die folgenden Schritte ausführen, um diese Änderung zu umgehen:

1. Erstellen Sie eine Web-API für Ihre Anwendung mit einem oder mehreren Bereichen. Dieser explizite Einstiegspunkt ermöglicht eine differenziertere Kontrolle und höhere Sicherheit.
1. Vergewissern Sie sich im App-Manifest, im [Azure-Portal](https://portal.azure.com) oder im [App-Registrierungsportal](https://apps.dev.microsoft.com), dass die App Zugriffstoken über den impliziten Fluss ausgeben darf. Dies wird durch den Schlüssel `oauth2AllowImplicitFlow` gesteuert.
1. Wenn Ihre Clientanwendung ein ID-Token über `response_type=id_token` anfordert, fordern Sie zusätzlich ein Zugriffstoken (`response_type=token`) für die oben erstellte Web-API an. Daher sollte bei Verwendung des v2.0-Endpunkts der `scope`-Parameter etwa wie folgt aussehen: `api://GUID/SCOPE`. Auf dem v1.0-Endpunkt sollte der `resource`-Parameter der App-URI der Web-API sein.
1. Übergen Sie anstelle des ID-Tokens dieses Zugriffstoken an die mittlere Ebene.  
