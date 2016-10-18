---
ms.assetid: 898d72f1-01e7-4b87-8eb3-a8e0e2e6e6da
title: Adding nodes or drives to Storage Spaces Direct
ms.prod: windows-server-threshold
ms.author: cosdar
ms.manager: dongill
ms.technology: storage-spaces
ms.topic: article
author: cosmosdarwin
ms.date: 09/30/2016
---
# Adding nodes or drives to Storage Spaces Direct

This topic describes how to add nodes or drives to Storage Spaces Direct. 

**Adding nodes** (also known as scaling out) adds storage capacity, unlocks greater storage efficiency, and improves storage performance. If you’re running a hyper-converged cluster, adding nodes also provides more compute resources for your workload. **Adding drives** (also known as scaling up) adds storage capacity and can also improve performance.

## Adding nodes

Typical deployments are simple to scale out by adding nodes: 

1. Run cluster validation by opening an elevated PowerShell session on the cluster and then running the following command, including the new node *\<NewNode>*:

   ```PowerShell
   Test-Cluster -Node <Node>, <Node>, <Node>, <NewNode> -Include "Storage Spaces Direct", Inventory, Network, "System Configuration"
   ```
    This confirms that the new node is running Windows Server 2016 Datacenter Edition, has joined the same Active Directory Domain Services domain as the existing nodes, has all the required roles and features, and has networking properly configured. 

   You can further verify that the new drives are ready for use by running **Get-PhysicalDisk** in PowerShell on the new node. Check that they are listed and marked **CanPool = True**. If they aren’t, you can check their **CannotPoolReason**, and if they contain old data or metadata, consider using **Clear-Disk**.

2. Run the following command on the cluster to finish adding the node:

   ```PowerShell
   Add-ClusterNode -Name NewNode 
   ```

Note that automatic pooling depends on you having only one pool. If you manually created multiple pools, add new drives to your preferred pool manually by using the **Add-PhysicalDisk** cmdlet.

### Special case: from 2 to 3 nodes

With two nodes, you can only create two-way mirrored volumes (compare to distributed RAID-1). Any of the following cmdlets in PowerShell will do so.

```PowerShell
New-Volume -FriendlyName "A" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -Size 100GB
New-Volume -FriendlyName "B" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -Size 100GB -ResiliencySettingName Mirror
New-Volume -FriendlyName "C" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -StorageTierFriendlyNames Capacity -StorageTierSizes 100GB
```

   >[!TIP]
   > Two-way mirroring only has **PhysicalDiskRedundancy = 1**, meaning that you are not protected against multiple simultaneous drive or node failures.

With three nodes, you can create three-way mirrored volumes, which are tolerant to multiple simultaneous failures. We recommend using three-way mirroring whenever possible. To begin creating three-way mirroed volumes, you have several options:

#### OPTION 1: Specify **PhysicalDiskRedundancy = 2** on each new volume upon creation.

```PowerShell
New-Volume -FriendlyName "D" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -Size 100GB -PhysicalDiskRedundancy 2
New-Volume -FriendlyName "E" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -Size 100GB -ResiliencySettingName Mirror -PhysicalDiskRedundancy 2 
New-Volume -FriendlyName "F" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -StorageTierFriendlyNames Capacity -StorageTierSizes 100GB -PhysicalDiskRedundancy 2
```

#### OPTION 2: Set **PhysicalDiskRedundancy = 2** on the pool’s **ResiliencySetting** object, if you won’t reference the tier when thereafter creating volumes.

```PowerShell
Get-StoragePool S* | Get-ResiliencySetting -Name Mirror | Set-ResiliencySetting -PhysicalDiskRedundancyDefault 2 
```

#### OPTION #: Set **PhysicalDiskRedundancy = 2** on the **StorageTier** called *Capacity*, and then create volumes by referencing the tier.

```PowerShell
Set-StorageTier -FriendlyName Capacity -PhysicalDiskRedundancy 2 
```

It is worth noting that with three nodes, you are also able to use single parity (compare to distributed RAID-5). This has the advantage of greater storage efficiency: 66.7%, compared to 50.0% with two-way mirroring or 33.3% with three-way mirroring.

```PowerShell
New-Volume -FriendlyName "P" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -Size 100GB -ResiliencySettingName Parity 
```

   >[!TIP]
   > Tip: Single parity only has **PhysicalDiskRedundancy = 1**, meaning that you are not protected against multiple simultaneous drive or node failures.

### Special case: from 3 to 4 nodes

With four nodes, you can use "dual" parity, also commonly called "erasure coding" (compare to distributed RAID-6). This provides the same dual fault tolerance as three-way mirroring, but with a surprising 50.0% efficiency. Here again, if you’re coming from a smaller configuration, you have several options:

#### OPTION 1: Specify **PhysicalDiskRedundancy = 2** on each new volume upon creation.

```PowerShell
New-Volume -FriendlyName "Q" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -Size 100GB -ResiliencySettingName Parity -PhysicalDiskRedundancy 2
```

#### OPTION 2: Set **PhysicalDiskRedundancy = 2** on the pool’s **ResiliencySetting** object, if you won’t reference the tier when thereafter creating volumes.

```PowerShell
Get-StoragePool S* | Get-ResiliencySetting -Name Parity | Set-ResiliencySetting -PhysicalDiskRedundancyDefault 2 
```

With four nodes, you can also begin using mixed resiliency, or "multi-resiliency", where an individual volume is part mirror, part parity.

For this, you will need to update your **StorageTier** configuration to have both *Performance* and *Capacity* tiers, as they would be created if you had first run **Enable-ClusterS2D** at four nodes. Specifically, both tiers should have the **MediaType** of your capacity devices (e.g. HDD) and **PhysicalDiskRedundancy = 2**. The *Performance* tier should be **ResiliencySettingName = Mirror**, and the *Capacity* tier should be **ResiliencySettingName = Parity**.

You may find it easiest to simply remove the existing tier, and create the two new ones.

#### OPTION 3: Modify the **StorageTier** definitions and then create volumes by referencing the tier.

```PowerShell
Remove-StorageTier -FriendlyName Capacity

New-StorageTier -StoragePoolFriendlyName S2D* -MediaType HDD -PhysicalDiskRedundancy 2 -ResiliencySettingName Mirror -FriendlyName Performance
New-StorageTier -StoragePoolFriendlyName S2D* -MediaType HDD -PhysicalDiskRedundancy 2 -ResiliencySettingName Parity -FriendlyName Capacity
```

That’s it! You are now ready to create mixed resiliency volumes!

Example:

```PowerShell
New-Volume -FriendlyName "M" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -StorageTierFriendlyNames Performance, Capacity -StorageTierSizes 50GB, 50GB 
```

### Greater parity efficiency beyond 4 nodes

As you scale beyond four nodes, new volumes can benefit from ever-greater parity encoding efficiency. For example, between six and seven nodes, efficiency improves from 50.0% to 66.7% as it becomes possible to use Reed-Solomon 4+2 (rather than 2+2). There are no steps you need to take to begin enjoying this new efficiency; the best possible encoding is determined automatically each time you create a volume.

However, any pre-existing volumes will *not* be "converted" to the new, wider encoding. One good reason is that to do so would require a massive calculation affecting literally *every single bit* in the entire deployment. If you would like pre-existing data to become encoded at the higher efficiency, you can migrate it to new volume(s).

### Adding nodes when using chassis or rack fault tolerance
If your deployment uses chassis or rack fault tolerance, you must specify the chassis or rack of new nodes before adding them to the cluster. This tells Storage Spaces Direct how best to distribute data to maximize fault tolerance.

1. Create a temporary fault domain for the node by opening an elevated PowerShell session and then using the following command, where *\<NewNode>* is the name of the new cluster node:

   ```PowerShell
   New-ClusterFaultDomain -Type Node -Name <NewNode> 
   ```

   >[!TIP]
   >Double-check that you enter the correct name of the new node.

2. Move this temporary fault-domain into the chassis or rack where the new node is located in the real world, as specified by *\<ParentName>*:

   ```PowerShell
   Set-ClusterFaultDomain -Name <NewNode> -Parent <ParentName> 
   ```

   For more information, see [Fault domain awareness in Windows Server 2016](../../failover-clustering/fault-domains.md).

3. Add the node to the cluster as described in [Adding nodes](#adding-nodes). <br>When the new node joins the cluster, it's automatically associated (using its name) with the placeholder fault domain.

## Adding drives

If you have available slots, you can add drives to each node to expand your storage capacity without adding nodes. You can add cache drives, or capacity drives, independently. 

   >[!IMPORTANT]
   >We strongly recommend configuring all nodes with identical storage configurations.


To scale up, connect the drives and verify that Windows discovers them. They should appear in the output of this PowerShell cmdlet (run as Administrator), on any cluster node, marked as **CanPool = True**.

```PowerShell
Get-PhysicalDisk 
```

Within a short time, eligible drives will automatically be claimed by Storage Spaces Direct, added to the storage pool, and volumes will automatically be redistributed evenly across all the drives. At this point - you're finished and ready to create more volumes.

If the drives don’t appear, manually scan for hardware changes. This can be done using **Device Manager**, under the **Action** menu. If they contain old data or metadata, consider reformatting them. This can be done using **Disk Management**.

   >[!TIP]
   > Automatic pooling depends on you having only one pool. If you’ve circumvented the standard configuration to create multiple pools, you will need to add new drives to your preferred pool yourself using **Add-PhysicalDisk**.