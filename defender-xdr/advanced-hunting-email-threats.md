---
title: Advanced hunting example for Microsoft Defender for Office 365
description: Get started searching for email threats using advanced hunting
search.appverid: met150
ms.service: defender-xdr
ms.subservice: adv-hunting
f1.keywords: 
  - NOCSH
ms.author: maccruz
author: schmurky
ms.localizationpriority: medium
manager: dansimp
audience: ITPro
ms.collection: 
- m365-security
- tier3
ms.custom:
- cx-ti
- cx-ah
ms.topic: how-to
ms.date: 04/22/2024
---

# Advanced hunting example for Microsoft Defender for Office 365

[!INCLUDE [Microsoft Defender XDR rebranding](../includes/microsoft-defender.md)]

**Applies to:**
- Microsoft Defender XDR

Want to get started searching for email threats using advanced hunting? Try these steps:

The [Microsoft Defender for Office 365 deployment guide](/defender-office-365/mdo-deployment-guide) explains how to jump right in and get configuration going on Day 1.

Depending on your [preset security policy vs. custom policy choices](/defender-office-365/mdo-deployment-guide#determine-your-protection-policy-strategy), **Zero-Hour auto purge** (ZAP) settings are important to know if a malicious message was removed from a mailbox after delivery.

Quickly navigating to Kusto query language to hunt for issues is an advantage of converging these two security centers. Security teams can monitor ZAP misses by taking their next steps in the Microsoft Defender portal at <https://security.microsoft.com> \> **Hunting** \> **Advanced Hunting**.

1. On the **Advanced Hunting page** at <https://security.microsoft.com/v2/advanced-hunting>, verify the **New Query** tab is selected.
1. Copy the following query into the **Query** box:

   ```kusto
   EmailPostDeliveryEvents 
   | where Timestamp > ago(7d)
   //List malicious emails that were not zapped successfully
   | where ActionType has "ZAP" and ActionResult == "Error"
   | project ZapTime = Timestamp, ActionType, NetworkMessageId , RecipientEmailAddress 
   //Get logon activity of recipients using RecipientEmailAddress and AccountUpn
   | join kind=inner IdentityLogonEvents on $left.RecipientEmailAddress == $right.AccountUpn
   | where Timestamp between ((ZapTime-24h) .. (ZapTime+24h))
   //Show only pertinent info, such as account name, the app or service, protocol, the target device, and type of logon
   | project ZapTime, ActionType, NetworkMessageId , RecipientEmailAddress, AccountUpn, 
   LogonTime = Timestamp, AccountDisplayName, Application, Protocol, DeviceName, LogonType
   ```

1. Select **Run query**.

    :::image type="content" source="/defender/media/ah-query-example-new.png" alt-text="The Advanced hunting page (under Hunting) with Query selected at the top of the query panel, and running a Kusto query to capture ZAP actions over the last seven days." lightbox="/defender/media/ah-query-example-new.png":::

    The data from this query appears in the **Results** panel below the query itself. Results include information like `DeviceName`, `AccountDisplayName`, and `ZapTime` in a customizable result set. Results can also be exported for your records. To save the query for reuse, select **Save** \> **Save As** to add the query to your list of queries, shared, or community queries.

## Related information

- [Advanced hunting best practices](advanced-hunting-best-practices.md)
- [Overview - Advanced hunting](advanced-hunting-overview.md)
[!INCLUDE [Microsoft Defender XDR rebranding](../includes/defender-m3d-techcommunity.md)]
