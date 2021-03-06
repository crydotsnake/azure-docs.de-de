---
title: Schützen von Azure AD Domain Services | Microsoft-Dokumentation
description: Erfahren Sie, wie Sie schwache Verschlüsselungen, alte Protokolle und die NTLM-Kennworthashsynchronisierung für eine verwaltete Azure AD DS-Domäne (Azure Active Directory Domain Services) deaktivieren.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.assetid: 6b4665b5-4324-42ab-82c5-d36c01192c2a
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 03/31/2020
ms.author: iainfou
ms.openlocfilehash: 581963c94129c36acbd8761d93e369281797fa9f
ms.sourcegitcommit: 62c5557ff3b2247dafc8bb482256fef58ab41c17
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/03/2020
ms.locfileid: "80654716"
---
# <a name="disable-weak-ciphers-and-password-hash-synchronization-to-secure-an-azure-ad-domain-services-managed-domain"></a>Deaktivieren von schwachen Verschlüsselungen und der Kennworthashsynchronisierung zum Schützen einer verwalteten Azure AD Domain Services-Domäne

Standardmäßig erlaubt Azure Active Directory Domain Services (Azure AD DS) die Verwendung von Verschlüsselungen wie NTLM v1 und TLS v1. Diese Verschlüsselungen sind möglicherweise für einige Legacyanwendungen erforderlich, gelten jedoch als schwach und können deaktiviert werden, wenn Sie sie nicht benötigen. Wenn Sie über eine lokale Hybridverbindung über Azure AD Connect verfügen, können Sie auch die NTLM-Kennworthashsynchronisierung deaktivieren.

In diesem Artikel erfahren Sie, wie Sie NTLM v1- und TLS v1-Verschlüsselungen sowie die NTLM-Kennworthashsynchronisierung deaktivieren.

## <a name="prerequisites"></a>Voraussetzungen

Damit Sie die Anweisungen in diesem Artikel ausführen können, benötigen Sie folgende Ressourcen:

* Ein aktives Azure-Abonnement.
    * Wenn Sie kein Azure-Abonnement besitzen, [erstellen Sie ein Konto](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Einen mit Ihrem Abonnement verknüpften Azure Active Directory-Mandanten, der entweder mit einem lokalen Verzeichnis synchronisiert oder ein reines Cloudverzeichnis ist.
    * [Erstellen Sie einen Azure Active Directory-Mandanten][create-azure-ad-tenant], oder [verknüpfen Sie ein Azure-Abonnement mit Ihrem Konto][associate-azure-ad-tenant], sofern erforderlich.
* Eine verwaltete Azure Active Directory Domain Services-Domäne, die in Ihrem Azure AD-Mandanten aktiviert und konfiguriert ist.
    * Bei Bedarf [erstellen und konfigurieren Sie eine Azure Active Directory Domain Services-Instanz][create-azure-ad-ds-instance].
* Installieren und konfigurieren Sie Azure PowerShell.
    * Führen Sie bei Bedarf die Anweisungen zum [Installieren des Azure PowerShell-Moduls und Verbinden mit Ihrem Azure-Abonnement](/powershell/azure/install-az-ps) aus.
    * Stellen Sie sicher, dass Sie sich mit dem Cmdlet [Connect-AzAccount][Connect-AzAccount] bei Ihrem Azure-Abonnement anmelden.
* Installieren und konfigurieren Sie Azure AD PowerShell.
    * Führen Sie bei Bedarf die Anweisungen zum [Installieren des Azure AD PowerShell-Moduls und Verbinden mit Azure AD](/powershell/azure/active-directory/install-adv2) aus.
    * Stellen Sie sicher, dass Sie sich mit dem Cmdlet [Connect-AzureAD][Connect-AzureAD] bei Ihrem Azure AD-Mandanten anmelden.

## <a name="disable-weak-ciphers-and-ntlm-password-hash-sync"></a>Deaktivieren von schwachen Verschlüsselungen und der NTLM-Kennworthashsynchronisierung

Wenn Sie schwache Verschlüsselungssammlungen und die NTLM-Kennworthashsynchronisierung deaktivieren möchten, melden Sie sich bei Ihrem Azure-Konto an, und rufen Sie dann mithilfe des Cmdlets [Get-AzResource][Get-AzResource] die Azure AD DS-Ressource ab:

> [!TIP]
> Wenn Sie beim Ausführen des Befehls [Get-AzResource][Get-AzResource] eine Fehlermeldung mit dem Hinweis erhalten, dass die Ressource *Microsoft.AAD/DomainServices* nicht vorhanden ist, [erhöhen Sie die Zugriffsrechte, um alle Azure-Abonnements und Verwaltungsgruppen verwalten zu können][global-admin].

```powershell
Login-AzAccount

$DomainServicesResource = Get-AzResource -ResourceType "Microsoft.AAD/DomainServices"
```

Definieren Sie als Nächstes *DomainSecuritySettings*, um die folgenden Sicherheitsoptionen zu konfigurieren:

1. Deaktivieren der NTLM v1-Unterstützung
2. Deaktivieren der Synchronisierung von NTLM-Kennworthashes aus Ihrer lokalen AD-Instanz
3. Deaktivieren von TLS v1

> [!IMPORTANT]
> Benutzer- und Dienstkonten können keine einfachen LDAP-Bindungen mehr ausführen, wenn Sie die NTLM-Kennworthashsynchronisierung für Ihre verwaltete Azure AD DS-Domäne deaktivieren. Wenn Sie einfache LDAP-Bindungen ausführen müssen, legen Sie im folgenden Befehl die Sicherheitskonfigurationsoption *"SyncNtlmPasswords"="Disabled"* nicht fest.

```powershell
$securitySettings = @{"DomainSecuritySettings"=@{"NtlmV1"="Disabled";"SyncNtlmPasswords"="Disabled";"TlsV1"="Disabled"}}
```

Wenden Sie schließlich mithilfe des Cmdlets [Set-AzResource][Set-AzResource] die definierten Sicherheitseinstellungen auf die verwaltete Azure AD DS-Domäne an. Geben Sie die Azure AD DS-Ressource aus dem ersten Schritt und die Sicherheitseinstellungen aus dem vorherigen Schritt an.

```powershell
Set-AzResource -Id $DomainServicesResource.ResourceId -Properties $securitySettings -Verbose -Force
```

Es dauert einen Moment, bis die Sicherheitseinstellungen auf die verwaltete Azure AD DS-Domäne angewendet werden.

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zum Synchronisierungsvorgang finden Sie unter [Synchronisieren von Objekten und Anmeldeinformationen in einer verwalteten Azure AD DS-Domäne][synchronization].

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[global-admin]: ../role-based-access-control/elevate-access-global-admin.md
[synchronization]: synchronization.md

<!-- EXTERNAL LINKS -->
[Get-AzResource]: /powershell/module/az.resources/Get-AzResource
[Set-AzResource]: /powershell/module/Az.Resources/Set-AzResource
[Connect-AzAccount]: /powershell/module/Az.Accounts/Connect-AzAccount
[Connect-AzureAD]: /powershell/module/AzureAD/Connect-AzureAD