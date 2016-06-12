---
title: ksetup:delkdc
ms.custom: na
ms.prod: windows-server-2012
ms.reviewer: na
ms.suite: na
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: 7d6ec389-094c-4a7b-a78b-605497ddc289
---
# ksetup:delkdc
deletes instances of Key Distribution Center \(KDC\) names for the Kerberos realm. for examples of how this command can be used, see [Examples](#BKMK_Examples).

## Syntax

```
ksetup /delkdc <RealmName> <KDCName>
```

### Parameters

|Parameter|Description|
|-------------|---------------|
|<RealmName>|The realm name is stated as an uppercase DNS name, such as CORP.CONTOSO.COM, and it is listed as the default realm when **ksetup** is run. It is to this realm from which you are attempting to delete the other KDC.|
|<KDCName>|The KDC name is stated as a case\-insensitive, fully qualified domain name, such as mitkdc.contoso.com.|

## remarks
These mappings are stored in the registry in **HKEY\_LOCAL\_MACHINE\\System\\CurrentControlSet\\Control\\LSA\\Kerberos\\Domains**. To remove realm configuration data from multiple computers, use the Security Configuration Template snap\-in and policy distribution instead of using **ksetup** explicitly on individual computers.

On computers running Windows 2000 Server with Service Pack 1 \(SP1\) and earlier, the computer must be restarted before the changed realm setting configuration will be used.

To verify the default realm name for the computer, or to verify that this command worked as intended, run **ksetup** at the command prompt and verify that the KDC that was removed does not exist in the list.

## <a name="BKMK_Examples"></a>Examples
The security requirements for this computer have changed so the link between the Windows realm and the non\-Windows realm must be removed. First, determine which association to remove and produce the output of existing associations:

```
ksetup
```

remove the association by using the following command:

```
ksetup /delkdc CORP.CONTOSO.COM mitkdc.contoso.com
```

## additional references

-   [ksetup](../ksetup.md)

-   [Command-Line Syntax Key](../commandline-syntax-key.md)

