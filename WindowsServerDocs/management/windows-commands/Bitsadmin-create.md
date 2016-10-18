---
title: Bitsadmin create
description: "Windows Commands topic for **Bitsadmin create** -- Creates a transfer job with the given display name."
ms.custom: na
ms.prod: windows-server-threshold
ms.reviewer: na
ms.suite: na
ms.technology: manage-windows-commands
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: 9a8c53af-900b-4c24-9265-5b8b08213fac
author: coreyp-at-msft
ms.author: coreyp
manager: dongill
ms.date: 10/12/2016
---

# Bitsadmin create

>Applies To: Windows Server&reg; 2016, Windows Server&reg; 2012 R2, Windows Server&reg; 2012

Creates a transfer job with the given display name.
## Syntax
```
bitsadmin /create [Type] DisplayName
```
## Parameters
|Parameter|Description|
|-------------|---------------|
|Type|-   **/Download**—transfers data from a server to a local file.<br />-   **/Upload**—transfers data from a local file to a server.<br />-   **/Upload-Reply**—transfers data from a local file to a server and receives a reply file from the server.<br />-   This parameter defaults to **/Download** when not specified on the command line.|
|DisplayName|The display name assigned to the newly created job.|
Use the [Bitsadmin resume](Bitsadmin-resume.md) switch to activate the job in the transfer queue.
## <a name="BKMK_examples"></a>Examples
Creates a download job named *myDownloadJob*.
```
C:\>bitsadmin /create myDownloadJob
```
## Additional references
[Command-Line Syntax Key](Command-Line-Syntax-Key.md)