---
title: Berechtigungen für Repositorys in Azure Container Registry
description: Erstellen eines Tokens mit Berechtigungen, die für bestimmte Repositorys in einer Registrierung gelten, um Images zu pullen oder zu pushen bzw. andere Aktionen auszuführen
ms.topic: article
ms.date: 02/13/2020
ms.openlocfilehash: 9004c45401833d3070266055dd7eb99a2bb43bde
ms.sourcegitcommit: bc738d2986f9d9601921baf9dded778853489b16
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/02/2020
ms.locfileid: "80618824"
---
# <a name="create-a-token-with-repository-scoped-permissions"></a>Erstellen eines Token mit repositorybezogenen Berechtigungen

In diesem Artikel wird beschrieben, wie Sie Token und Bereichszuordnungen zur Verwaltung von Berechtigungen in Ihrer Containerregistrierung erstellen, die sich auf das Repository beziehen. Durch die Erstellung von Token kann ein Registrierungsbesitzer Benutzern oder Diensten einen bereichsbezogenen, zeitlich begrenzten Zugriff auf Repositorys gewähren, um Images zu pullen oder zu pushen oder andere Aktionen durchzuführen. Ein Token bietet detailliertere Berechtigungen als andere [Authentifizierungsoptionen](container-registry-authentication.md) für Registrierungen, die Berechtigungen für eine gesamte Registrierung umfassen. 

Zu den Szenarien für die Erstellung eines Tokens gehören die folgenden:

* IoT-Geräten mit individuellen Token das Pullen eines Images aus einem Repository gestatten
* Bereitstellen einer externen Organisation mit Berechtigungen für ein bestimmtes Repository 
* Einschränken des Zugriffs auf verschiedene Benutzergruppen in Ihrer Organisation. Beispielsweise Bereitstellen von Schreib- und Lesezugriff für Entwickler, die Images für bestimmte Repositorys erstellen, und von Lesezugriff für Teams, die aus diesen Repositorys Bereitstellungen durchführen.

> [!IMPORTANT]
> Dieses Feature befindet sich derzeit in der Vorschauphase. Es gelten einige [Einschränkungen](#preview-limitations). Vorschauversionen werden Ihnen zur Verfügung gestellt, wenn Sie die [zusätzlichen Nutzungsbedingungen][terms-of-use] akzeptieren. Einige Aspekte dieses Features werden bis zur allgemeinen Verfügbarkeit unter Umständen noch geändert.

## <a name="preview-limitations"></a>Einschränkungen der Vorschau

* Dieses Feature ist nur in einer Containerregistrierung des Typs **Premium** verfügbar. Weitere Informationen zu den Tarifen des Registrierungsdiensts und zu den Einschränkungen finden Sie unter [Azure Container Registry-SKUs](container-registry-skus.md).
* Zurzeit können Sie einer Azure Active Directory-Identität (z. B. einem Dienstprinzipal oder einer verwalteten Identität) keine repositorybezogenen Berechtigungen zuweisen.
* Sie können keine Bereichszuordnung in einer Registrierung erstellen, die für [anonymen Pull-Zugriff](container-registry-faq.md#how-do-i-enable-anonymous-pull-access) aktiviert ist.

## <a name="concepts"></a>Konzepte

Zum Konfigurieren von repositorybezogenen Berechtigungen erstellen Sie ein *Token* mit einer zugehörigen *Bereichszuordnung*. 

* Ein **Token** zusammen mit einem generierten Kennwort ermöglicht dem Benutzer die Authentifizierung bei der Registrierung. Sie können ein Ablaufdatum für ein Tokenkennwort festlegen oder ein Token jederzeit deaktivieren.  

  Nach der Authentifizierung mit einem Token kann der Benutzer oder Dienst eine oder mehrere  *Aktionen* durchführen, die auf ein oder mehrere Repositorys verteilt sind.

  |Aktion  |BESCHREIBUNG  | Beispiel |
  |---------|---------|--------|
  |`content/delete`    | Entfernen von Daten aus dem Repository  | Löschen eines Repositorys oder eines Manifests |
  |`content/read`     |  Lesen von Daten aus dem Repository |  Pullen eines Artefakts |
  |`content/write`     |  Schreiben von Daten in das Repository     | Verwenden der Aktion zusammen mit `content/read` zum Pushen eines Artefakts |
  |`metadata/read`    | Lesen von Metadaten aus dem Repository   | Auflisten von Tags oder Manifesten |
  |`metadata/write`     |  Schreiben von Metadaten in das Repository  | Aktivieren oder Deaktivieren von Lese-, Schreib- oder Löschvorgängen |

* Eine **Gültigkeitsbereichszuordnung** gruppiert die Repositoryberechtigungen, die Sie auf ein Token anwenden oder auf andere Token erneut anwenden. Jedes Token ist einer einzelnen Gültigkeitsbereichszuordnung zugeordnet. 

   Mit einer Gültigkeitsbereichszuordnung haben Sie folgende Möglichkeiten:

    * Konfigurieren mehrerer Token mit identischen Berechtigungen für eine Gruppe von Repositorys
    * Aktualisieren von Tokenberechtigungen beim Hinzufügen oder Entfernen von Repositoryaktionen in der Gültigkeitsbereichszuordnung oder beim Anwenden einer anderen Gültigkeitsbereichszuordnung 

  Azure Container Registry bietet auch mehrere systemdefinierte Gültigkeitsbereichszuordnungen, die Sie anwenden können, mit festen Berechtigungen für alle Repositorys.

In der folgenden Abbildung wird die Beziehung zwischen Token und Gültigkeitsbereichszuordnungen veranschaulicht. 

![Registrierungstoken und Gültigkeitsbereichszuordnungen](media/container-registry-repository-scoped-permissions/token-scope-map-concepts.png)

## <a name="prerequisites"></a>Voraussetzungen

* **Azure CLI**: Azure CLI-Befehle zum Erstellen und Verwalten von Token sind in Azure CLI Version 2.0.76 oder höher verfügbar. Führen Sie `az --version` aus, um die Version zu ermitteln. Informationen zum Durchführen einer Installation oder eines Upgrades finden Sie bei Bedarf unter [Installieren der Azure CLI](/cli/azure/install-azure-cli).
* **Docker**: Um sich zum Pullen oder Pushen von Images bei der Registrierung zu authentifizieren, benötigen Sie eine lokale Docker-Installation. Docker-Installationsanleitungen stehen für Systeme unter [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) und [Linux](https://docs.docker.com/engine/installation/#supported-platforms) zur Verfügung.
* **Containerregistrierung**: Erstellen Sie in Ihrem Azure-Abonnement eine Premium-Containerregistrierung, wenn noch keine Registrierung vorhanden ist, oder führen Sie ein Upgrade für eine vorhandene Registrierung durch. Verwenden Sie beispielsweise das [Azure-Portal](container-registry-get-started-portal.md) oder die [Azure CLI](container-registry-get-started-azure-cli.md). 

## <a name="create-token---cli"></a>Token erstellen – Befehlszeilenschnittstelle

### <a name="create-token-and-specify-repositories"></a>Erstellen eines Tokens und Angeben von Repositorys

Erstellen Sie mit dem Befehl [az acr token create][az-acr-token-create] ein Token. Wenn Sie ein Token erstellen, können Sie mindestens ein Repository und zugehörige Aktionen für jedes Repository angeben. Die Repositorys müssen sich noch nicht in der Registrierung befinden. Um ein Token durch die Angabe einer vorhandenen Gültigkeitsbereichszuordnung zu erstellen, lesen Sie den nächsten Abschnitt.

Das folgende Beispiel erstellt ein Token in der Registrierung *myregistry* mit den folgenden Berechtigungen für das `samples/hello-world`-Repository: `content/write` und `content/read`. Standardmäßig legt der Befehl den standardmäßigen Tokenstatus auf `enabled` fest, aber Sie können den Status jederzeit auf `disabled` aktualisieren.

```azurecli
az acr token create --name MyToken --registry myregistry \
  --repository samples/hello-world \
  content/write content/read
```

In der Ausgabe werden Details zum Token angezeigt, einschließlich der zwei generierten Kennwörter. Es wird empfohlen, die Kennwörter an einem sicheren Ort zu speichern, um sie später für die Authentifizierung zu verwenden. Die Kennwörter können nicht erneut abgerufen werden, aber es können neue Kennwörter generiert werden.

```console
{
  "creationDate": "2020-01-18T00:15:34.066221+00:00",
  "credentials": {
    "certificates": [],
    "passwords": [
      {
        "creationTime": "2020-01-18T00:15:52.837651+00:00",
        "expiry": null,
        "name": "password1",
        "value": "uH54BxxxxK7KOxxxxRbr26dAs8JXxxxx"
      },
      {
        "creationTime": "2020-01-18T00:15:52.837651+00:00",
        "expiry": null,
        "name": "password2",
        "value": "kPX6Or/xxxxLXpqowxxxxkA0idwLtmxxxx"
      }
    ],
    "username": "MyToken"
  },
  "id": "/subscriptions/xxxxxxxx-adbd-4cb4-c864-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.ContainerRegistry/registries/myregistry/tokens/MyToken",
  "name": "MyToken",
  "objectId": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myresourcegroup",
  "scopeMapId": "/subscriptions/xxxxxxxx-adbd-4cb4-c864-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.ContainerRegistry/registries/myregistry/scopeMaps/MyToken-scope-map",
  "status": "enabled",
  "type": "Microsoft.ContainerRegistry/registries/tokens"
```

Die Ausgabe enthält Details über die vom Befehl erstellte Gültigkeitsbereichszuordnung. Mit der Gültigkeitsbereichszuordnung, hier unter der Bezeichnung `MyToken-scope-map`, können Sie dieselben Repositoryaktionen auf andere Token anwenden. Oder aktualisieren Sie die Gültigkeitsbereichszuordnung später, um die Berechtigungen der zugeordneten Token zu ändern.

### <a name="create-token-and-specify-scope-map"></a>Erstellen eines Tokens und Angeben der Gültigkeitsbereichszuordnung

Eine alternative Möglichkeit zum Erstellen eines Tokens besteht darin, eine vorhandene Gültigkeitsbereichszuordnung anzugeben. Wenn Sie nicht bereits über eine Gültigkeitsbereichszuordnung verfügen, erstellen Sie zunächst eine, indem Sie Repositorys und zugehörige Aktionen angeben. Geben Sie dann beim Erstellen eines Tokens die Gültigkeitsbereichszuordnung an. 

Um eine Gültigkeitsbereichszuordnung zu erstellen, verwenden Sie den Befehl [az acr scope-map create][az-acr-scope-map-create]. Der folgende Befehl erstellt eine Gültigkeitsbereichszuordnung mit denselben Berechtigungen für das zuvor verwendete `samples/hello-world`-Repository. 

```azurecli
az acr scope-map create --name MyScopeMap --registry myregistry \
  --repository samples/hello-world \
  content/write content/read \
  --description "Sample scope map"
```

Führen Sie [az acr token create][az-acr-token-create] aus, um ein Token zu erstellen, das die Gültigkeitsbereichszuordnung *MyScopeMap* angibt. Wie im vorherigen Beispiel legt der Befehl den standardmäßigen Tokenstatus auf `enabled` fest.

```azurecli
az acr token create --name MyToken \
  --registry myregistry \
  --scope-map MyScopeMap
```

In der Ausgabe werden Details zum Token angezeigt, einschließlich der zwei generierten Kennwörter. Es wird empfohlen, die Kennwörter an einem sicheren Ort zu speichern, um sie später für die Authentifizierung zu verwenden. Die Kennwörter können nicht erneut abgerufen werden, aber es können neue Kennwörter generiert werden.

## <a name="create-token---portal"></a>Token erstellen – Portal

Sie können das Azure-Portal zum Erstellen von Token und Gültigkeitsbereichszuordnungen verwenden. Wie beim CLI-Befehl `az acr token create` können Sie eine vorhandene Gültigkeitsbereichszuordnung anwenden oder eine Gültigkeitsbereichszuordnung erstellen, wenn Sie ein Token erstellen, indem Sie ein oder mehrere Repositorys und zugehörige Aktionen angeben. Die Repositorys müssen sich noch nicht in der Registrierung befinden. 

Das folgende Beispiel erstellt ein Token und erstellt eine Gültigkeitsbereichszuordnung mit den folgenden Berechtigungen im `samples/hello-world`-Repository: `content/write` und `content/read`.

1. Navigieren Sie im Azure-Portal zu Ihrer Containerregistrierung.
1. Wählen Sie unter **Dienste** die Option **Token (Vorschau) > +Hinzufügen** aus.
  ![Erstellen von Token im Portal](media/container-registry-repository-scoped-permissions/portal-token-add.png)
1. Geben Sie einen Tokennamen ein.
1. Wählen Sie unter **Gültigkeitsbereichszuordnung** die Option **Neu erstellen** aus.
1. Konfigurieren der Gültigkeitsbereichszuordnung:
    1. Geben Sie einen Namen und eine Beschreibung für die Gültigkeitsbereichszuordnung ein. 
    1. Unter **Repositorys** geben Sie `samples/hello-world` ein, und unter **Berechtigungen** wählen Sie `content/read` und `content/write` aus. Wählen Sie dann **+Hinzufügen** aus.  
    ![Erstellen einer Gültigkeitsbereichszuordnung im Portal](media/container-registry-repository-scoped-permissions/portal-scope-map-add.png)

    1. Nachdem Sie Repositorys und Berechtigungen hinzugefügt haben, wählen Sie **Hinzufügen** aus, um die Gültigkeitsbereichszuordnung hinzuzufügen.
1. Akzeptieren Sie den **Status** des Standardtokens von **Aktiviert**, und wählen Sie dann **Erstellen** aus.

Nachdem das Token überprüft und erstellt wurde, werden die Details des Tokens auf dem Bildschirm **Token** angezeigt.

### <a name="add-token-password"></a>Hinzufügen eines Tokenkennworts

Generieren Sie ein Kennwort, nachdem Sie ein Token erstellt haben. Das Token muss für die Authentifizierung bei der Registrierung aktiviert sein und über ein gültiges Kennwort verfügen.

Sie können ein oder zwei Kennwörter generieren und für jedes ein Ablaufdatum festlegen. 

1. Navigieren Sie im Azure-Portal zu Ihrer Containerregistrierung.
1. Wählen Sie unter **Dienste** die Option **Token (Vorschau)** und dann ein Token aus.
1. Wählen Sie in den Tokendetails die Option **password1** oder **password2** und dann das Symbol „Generieren“ aus.
1. Legen Sie auf dem Bildschirm für das Kennwort optional ein Ablaufdatum für das Kennwort fest, und wählen Sie **Generieren** aus.
1. Nachdem Sie ein Kennwort generiert haben, kopieren und speichern Sie es an einem sicheren Ort. Sie können ein generiertes Kennwort nach dem Schließen des Bildschirms nicht mehr abrufen, aber Sie können ein neues Kennwort generieren.

    ![Erstellen eines Tokenkennworts im Portal](media/container-registry-repository-scoped-permissions/portal-token-password.png)

## <a name="authenticate-with-token"></a>Authentifizieren mit einem Token

Wenn ein Benutzer oder Dienst ein Token zur Authentifizierung bei der Zielregistrierung verwendet, stellt er den Tokennamen als Benutzernamen und eines seiner generierten Kennwörter zur Verfügung. Die Authentifizierungsmethode hängt von der oder den konfigurierten Aktionen ab, die mit dem Token verbunden sind.

|Aktion  |Authentifizierung  |
  |---------|---------|
  |`content/delete`    | `az acr repository delete` in der Azure CLI |
  |`content/read`     |  `docker login`<br/><br/>`az acr login` in der Azure CLI  |
  |`content/write`     |  `docker login`<br/><br/>`az acr login` in der Azure CLI     |
  |`metadata/read`    | `az acr repository show`<br/><br/>`az acr repository show-tags`<br/><br/>`az acr repository show-manifests` in der Azure CLI   |
  |`metadata/write`     |  `az acr repository untag`<br/><br/>`az acr repository update` in der Azure CLI |

## <a name="examples-use-token"></a>Beispiele: Verwenden von Token

In den folgenden Beispielen wird das zuvor in diesem Artikel erstellte Token verwendet, um allgemeine Operationen für ein Repository durchzuführen: Images pushen und pullen, Images löschen und Repositorytags auflisten. Das Token wurde anfänglich mit Pushberechtigungen (`content/write`- und `content/read`-Aktionen) im `samples/hello-world`-Repository eingerichtet.

### <a name="pull-and-tag-test-images"></a>Pullen und Kennzeichnen von Testimages

Für die folgenden Beispiele pullen Sie die Images `hello-world` und `alpine` aus dem Docker Hub und kennzeichnen sie für Ihre Registrierung und Ihr Repository.

```bash
docker pull hello-world
docker pull alpine
docker tag hello-world myregistry.azurecr.io/samples/hello-world:v1
docker tag hello-world myregistry.azurecr.io/samples/alpine:v1
```

### <a name="authenticate-using-token"></a>Authentifizierung mithilfe des Tokens

Führen Sie `docker login` aus, um sich bei der Registrierung zu authentifizieren, stellen Sie den Tokennamen als Benutzernamen bereit, und geben Sie eines seiner Kennwörter an. Das Token muss den Status `Enabled` aufweisen.

Das folgende Beispiel ist für die Bash-Shell formatiert und stellt die Werte mithilfe von Umgebungsvariablen bereit.

```bash
TOKEN_NAME=MyToken
TOKEN_PWD=<token password>

echo $TOKEN_PWD | docker login --username $TOKEN_NAME --password-stdin myregistry.azurecr.io
```

Die Ausgabe sollte die erfolgreiche Authentifizierung anzeigen:

```console
Login Succeeded
```

### <a name="push-images-to-registry"></a>Übertragen von Images in die Registrierung per Push

Versuchen Sie nach erfolgreicher Anmeldung, die markierten Images in die Registrierung zu pushen. Da das Token über die Berechtigung verfügt, die Images in das `samples/hello-world`-Repository zu pushen, ist der folgende Pushvorgang erfolgreich:

```bash
docker push myregistry.azurecr.io/samples/hello-world:v1
```

Das Token hat keine Berechtigungen für das `samples/alpine`-Repository, sodass beim folgenden Pushversuch ein Fehler auftritt, der `requested access to the resource is denied` ähnelt:

```bash
docker push myregistry.azurecr.io/samples/alpine:v1
```

### <a name="change-pushpull-permissions"></a>Ändern der Push-/Pullberechtigungen

Um Berechtigungen eines Tokens zu aktualisieren, aktualisieren Sie die Berechtigungen in der zugehörigen Gültigkeitsbereichszuordnung. Die aktualisierte Gültigkeitsbereichszuordnung wird sofort auf alle zugeordneten Token angewendet. 

Aktualisieren Sie z. B. `MyToken-scope-map` mit den Aktionen `content/write` und `content/read` im `samples/alpine`-Repository, und entfernen Sie die Aktion `content/write` im `samples/hello-world`-Repository.  

Führen Sie [az acr scope-map update][az-acr-scope-map-update] zum Aktualisieren der Gültigkeitsbereichszuordnung aus, um die Azure CLI zu verwenden:

```azurecli
az acr scope-map update \
  --name MyScopeMap \
  --registry myregistry \
  --add samples/alpine content/write content/read \
  --remove samples/hello-world content/write 
```

Führen Sie im Azure-Portal die folgenden Schritte aus:

1. Navigieren Sie zu Ihrer Containerregistrierung.
1. Wählen Sie unter **Dienste** die Option **Gültigkeitsbereichszuordnungen (Vorschau)** aus, und wählen Sie die zu aktualisierende Gültigkeitsbereichszuordnung aus.
1. Unter **Repositorys** geben Sie `samples/alpine` ein, und unter **Berechtigungen** wählen Sie `content/read` und `content/write` aus. Wählen Sie dann **+Hinzufügen** aus.
1. Unter **Repositorys** wählen Sie `samples/hello-world` aus, und unter **Berechtigungen** deaktivieren Sie `content/write`. Klicken Sie dann auf **Speichern**.

Nach der Aktualisierung der Gültigkeitsbereichszuordnung wird der folgende Pushvorgang erfolgreich durchgeführt:

```bash
docker push myregistry.azurecr.io/samples/alpine:v1
```

Da die Gültigkeitsbereichszuordnung nur die `content/read`-Berechtigung für das `samples/hello-world`-Repository besitzt, tritt jetzt ein Fehler beim Pushversuch für das `samples/hello-world`-Repository auf:
 
```bash
docker push myregistry.azurecr.io/samples/hello-world:v1
```

Das Pullen von Images aus beiden Repositorys wird erfolgreich durchgeführt, da die Gültigkeitsbereichszuordnung `content/read`-Berechtigungen für beide Repositorys bereitstellt:

```bash
docker pull myregistry.azurecr.io/samples/alpine:v1
docker pull myregistry.azurecr.io/samples/hello-world:v1
```
### <a name="delete-images"></a>Löschen von Images

Aktualisieren Sie die Gültigkeitsbereichszuordnung durch Hinzufügen der Aktion `content/delete` zum `alpine`-Repository. Diese Aktion ermöglicht das Löschen von Images im Repository oder das Löschen des gesamten Repositorys.

Der Kürze halber zeigen wir nur den Befehl [az acr scope-map update][az-acr-scope-map-update] zur Aktualisierung der Gültigkeitsbereichszuordnung an:

```azurecli
az acr scope-map update \
  --name MyScopeMap \
  --registry myregistry \
  --add samples/alpine content/delete
``` 

Informationen zum Aktualisieren der Gültigkeitsbereichszuordnung über das Portal finden Sie im vorherigen Abschnitt.

Verwenden Sie den folgenden Befehl [az acr repository delete][az-acr-repository-delete], um das `samples/alpine`-Repository zu löschen. Um Images oder Repositorys zu löschen, wird das Token nicht durch `docker login` authentifiziert. Übergeben Sie stattdessen den Namen und das Kennwort des Tokens an den Befehl. Das folgende Beispiel verwendet die zuvor in diesem Artikel erstellten Umgebungsvariablen:

```azurecli
az acr repository delete \
  --name myregistry --repository samples/alpine \
  --username $TOKEN_NAME --password $TOKEN_PWD
```

### <a name="show-repo-tags"></a>Anzeigen von Repositorytags 

Aktualisieren Sie die Gültigkeitsbereichszuordnung durch Hinzufügen der Aktion `metadata/read` zum `hello-world`-Repository. Diese Aktion ermöglicht das Lesen von Manifest- und Tagdaten im Repository.

Der Kürze halber zeigen wir nur den Befehl [az acr scope-map update][az-acr-scope-map-update] zur Aktualisierung der Gültigkeitsbereichszuordnung an:

```azurecli
az acr scope-map update \
  --name MyScopeMap \
  --registry myregistry \
  --add samples/hello-world metadata/read 
```  

Informationen zum Aktualisieren der Gültigkeitsbereichszuordnung über das Portal finden Sie im vorherigen Abschnitt.

Um Metadaten im `samples/hello-world`-Repository zu lesen, führen Sie den Befehl [az acr repository show-manifests][az-acr-repository-show-manifests] oder [az acr repository show-tags][az-acr-repository-show-tags] aus. 

Um Metadaten zu lesen, wird das Token nicht durch `docker login` authentifiziert. Übergeben Sie stattdessen den Namen und das Kennwort des Tokens an beide Befehle. Das folgende Beispiel verwendet die zuvor in diesem Artikel erstellten Umgebungsvariablen:

```azurecli
az acr repository show-tags \
  --name myregistry --repository samples/hello-world \
  --username $TOKEN_NAME --password $TOKEN_PWD
```

Beispielausgabe:

```console
[
  "v1"
]
```
## <a name="manage-tokens-and-scope-maps"></a>Verwalten von Token und Gültigkeitsbereichszuordnungen

### <a name="list-scope-maps"></a>Auflisten von Gültigkeitsbereichszuordnungen

Verwenden Sie den Befehl [az acr scope-map list][az-acr-scope-map-list] oder den Bildschirm **Gültigkeitsbereichszuordnungen (Vorschau)** im Portal, um alle in einer Registrierung konfigurierten Gültigkeitsbereichszuordnungen aufzulisten. Beispiel:

```azurecli
az acr scope-map list \
  --registry myregistry --output table
```

Die Ausgabe zeigt die von Ihnen definierten Gültigkeitsbereichszuordnungen und mehrere systemdefinierte Gültigkeitsbereichszuordnungen an, die Sie zur Konfiguration von Token verwenden können:

```
NAME                 TYPE           CREATION DATE         DESCRIPTION
-------------------  -------------  --------------------  ------------------------------------------------------------
_repositories_admin  SystemDefined  2020-01-20T09:44:24Z  Can perform all read, write and delete operations on the ...
_repositories_pull   SystemDefined  2020-01-20T09:44:24Z  Can pull any repository of the registry
_repositories_push   SystemDefined  2020-01-20T09:44:24Z  Can push to any repository of the registry
MyScopeMap           UserDefined    2019-11-15T21:17:34Z  Sample scope map
```

### <a name="show-token-details"></a>Anzeigen von Tokendetails

Um die Details eines Tokens anzuzeigen, z. B. seinen Status und das Ablaufdatum des Kennworts, führen Sie den Befehl [az acr token show][az-acr-token-show] aus, oder wählen Sie das Token auf dem Bildschirm **Token (Vorschau)** im Portal aus. Beispiel:

```azurecli
az acr scope-map show \
  --name MyScopeMap --registry myregistry
```

Verwenden Sie den Befehl [az acr Tokenliste][az-acr-token-list] oder den Bildschirm **Token (Vorschau)** im Portal, um alle in einer Registrierung konfigurierten Token aufzulisten. Beispiel:

```azurecli
az acr token list --registry myregistry --output table
```

### <a name="generate-passwords-for-token"></a>Generieren von Kennwörtern für das Token

Wenn Sie nicht über ein Tokenkennwort verfügen oder neue Kennwörter generieren möchten, führen Sie den Befehl [az acr token credential generate][az-acr-token-credential-generate] aus. 

Das folgende Beispiel generiert einen neuen Wert für „password1“ für das Token *MyToken* mit einer Ablauffrist von 30 Tagen. Das Kennwort wird in der Umgebungsvariablen `TOKEN_PWD` gespeichert. Dieses Beispiel ist für die Bash-Shell formatiert.

```azurecli
TOKEN_PWD=$(az acr token credential generate \
  --name MyToken --registry myregistry --days 30 \
  --password1 --query 'passwords[0].value' --output tsv)
```

Informationen zur Verwendung des Azure-Portals zum Generieren eines Tokenkennworts finden Sie in den Schritten unter [Token erstellen – Portal](#create-token---portal) weiter oben in diesem Artikel.

### <a name="update-token-with-new-scope-map"></a>Aktualisieren von Token mit neuer Gültigkeitsbereichszuordnung

Wenn Sie ein Token mit einer anderen Gültigkeitsbereichszuordnung aktualisieren möchten, führen Sie [az acr token update][az-acr-token-update] aus, und geben Sie die neue Gültigkeitsbereichszuordnung an. Beispiel:

```azurecli
az acr token update --name MyToken --registry myregistry \
  --scope-map MyNewScopeMap
```

Wählen Sie im Portal auf dem Bildschirm **Token (Vorschau)** das Token und unter **Gültigkeitsbereichszuordnung** eine andere Gültigkeitsbereichszuordnung aus.

> [!TIP]
> Nach der Aktualisierung eines Tokens mit einer neuen Gültigkeitsbereichszuordnung möchten Sie vielleicht neue Tokenkennwörter generieren. Verwenden Sie den Befehl [az acr token credential generate][az-acr-token-credential-generate], oder generieren Sie ein neues Tokenkennwort im Azure-Portal.

## <a name="disable-or-delete-token"></a>Deaktivieren oder Löschen von Token

Möglicherweise müssen Sie die Verwendung der Tokenanmeldeinformationen für einen Benutzer oder Dienst vorübergehend deaktivieren. 

Führen Sie mit der Azure CLI den Befehl [az acr token update][az-acr-token-update] aus, um `status` auf `disabled` festzulegen:

```azurecli
az acr token update --name MyToken --registry myregistry \
  --status disabled
```

Wählen Sie im Portal das Token im Bildschirm **Token (Vorschau)** und dann **Deaktiviert** unter **Status** aus.

Um ein Token zum dauerhaften Annullieren des Zugriffs von Personen, die seine Anmeldeinformationen verwenden, zu löschen, führen Sie den Befehl [az acr token delete][az-acr-token-delete] aus. 

```azurecli
az acr token delete --name MyToken --registry myregistry
```

Wählen Sie im Portal das Token im Bildschirm **Token (Vorschau)** und dann **Verwerfen** aus.

## <a name="next-steps"></a>Nächste Schritte

* Um Gültigkeitsbereichszuordnungen und Token zu verwalten, verwenden Sie zusätzliche Befehle in den [az acr scope-map][az-acr-scope-map]- und [az acr token][az-acr-token]-Befehlsgruppen.
* In der [Authentifizierungsübersicht](container-registry-authentication.md) finden Sie weitere Optionen zur Authentifizierung mit einer Azure-Containerregistrierung, einschließlich der Verwendung einer Azure Active Directory-Identität, eines Dienstprinzipals oder eines Administratorkontos.


<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-repository]: /cli/azure/acr/repository/
[az-acr-repository-show-tags]: /cli/azure/acr/repository/#az-acr-repository-show-tags
[az-acr-repository-show-manifests]: /cli/azure/acr/repository/#az-acr-repository-show-manifests
[az-acr-repository-delete]: /cli/azure/acr/repository/#az-acr-repository-delete
[az-acr-scope-map]: /cli/azure/acr/scope-map/
[az-acr-scope-map-create]: /cli/azure/acr/scope-map/#az-acr-scope-map-create
[az-acr-scope-map-list]: /cli/azure/acr/scope-map/#az-acr-scope-map-show
[az-acr-scope-map-show]: /cli/azure/acr/scope-map/#az-acr-scope-map-list
[az-acr-scope-map-update]: /cli/azure/acr/scope-map/#az-acr-scope-map-update
[az-acr-scope-map-list]: /cli/azure/acr/scope-map/#az-acr-scope-map-list
[az-acr-token]: /cli/azure/acr/token/
[az-acr-token-show]: /cli/azure/acr/token/#az-acr-token-show
[az-acr-token-list]: /cli/azure/acr/token/#az-acr-token-list
[az-acr-token-delete]: /cli/azure/acr/token/#az-acr-token-delete
[az-acr-token-create]: /cli/azure/acr/token/#az-acr-token-create
[az-acr-token-update]: /cli/azure/acr/token/#az-acr-token-update
[az-acr-token-credential-generate]: /cli/azure/acr/token/credential/#az-acr-token-credential-generate
