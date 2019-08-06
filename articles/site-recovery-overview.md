<properties
	pageTitle="Site Recovery overview" 
	description="Azure Site Recovery coordinates the replication, failover and recovery of virtual machines and physical servers located on on-premises to Azure or to a secondary on-premises site." 
	services="site-recovery" 
	documentationCenter="" 
	authors="rayne-wiselman" 
	manager="jwhit" 
	editor=""/>

<tags 
	ms.service="site-recovery" 
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="storage-backup-recovery" 
	ms.date="05/10/2015" 
	ms.author="raynew"/>

#  Site Recovery Overview

The Site Recovery service contributes to a robust business continuity and disaster recovery (BCDR) solution that protects your on-premises physical servers and virtual machines by orchestrating and automating replication and failover to Azure, or to a secondary on-premises datacenter. 

- **Simplify**-Site Recovery helps simplify your BCDR strategy by making it easy to configure replication, and run failover and recovery for your on-premises workloads and applications.
- **Replication**-You can replicate on-premises workloads to Azure storage, or to a secondary datacenter. 
- **Vault**-To manage replication you set up a Site Recovery vault in an Azure region you select. All metadata remains within that region.
- **Metadata**-No application data is send to Azure. Site Recovery only needs metadata such as virtual machine and VMM cloud names, in order to orchestrate replication and failover. 
- **Connection to Azure**-Management servers communicate with Azure depending on your deployment scenario. For example if you're replicating virtual machines located in an on-premises VMM cloud, the VMM server communicates with Site Recovery over an encrypted outbound HTTPS connection. No connection is required from the virtual machines or Hyper-V hosts.
- **Hyper-V Replica**-Azure Site Recovery leverages Hyper-V Replica for the replication process, and can also use SAN replication if you're replicating between two on-premises VMM sites. Site Recovery uses smart replication, replicating only data blocks and not the entire VHD for the initial replication. For ongoing replication only delta changes are replicated. Site Recovery supports offline data transfer and works with WAN optimizers.
- **Pricing**-You can [Read more](pricing/details/site-recovery) about Site Recovery pricing.


## Deployment scenarios

This table summarizes the replication scenarios supported by Site Recovery.

**Replicate to** | **Replicate from (on-premises)** | **Details** | **Article**
---|---|---|---
Azure | Hyper-V site | Replicate virtual machine on one or more on-premises Hyper-V host servers that are defined as a Hyper-V site to Azure. No VMM server required. | [Read more](site-recovery-hyper-v-site-to-azure)
Azure| VMM server | Replicate virtual machines on one or more on-premises Hyper-V host servers located in a VMM cloud to Azure. | [Read more](site-recovery-vmm-to-azure) 
Azure | Physical Windows server | Replicate a physical Windows or Linux server to Azure | [Read more](site-recovery-vmware-to-azure)
Azure | VMware virtual machine | Replicate VMware virtual machines to Azure | [Read more](site-recovery-vmware-to-azure)
Secondary datacenter | VMM server | Replicate virtual machines on on-premises Hyper-V host servers located in a VMM cloud to a secondary VMM server in another datacenter | [Read more](site-recovery-vmm-to-vmm)
Secondary datacenter | VMM server with SAN | Replicate virtual machines on on-premises Hyper-V host servers located in a VMM cloud to a secondary VMM server in another datacenter using SAN replication| [Read more](site-recovery-vmm-san)
Secondary datacenter | Single VMM server | Replicate virtual machines on on-premises Hyper-V host servers located in a VMM cloud to a secondary cloud on the same VMM server | [Read more](site-recovery-single-vmm) 


## Supported workloads

This table summarizes the workload replication scenarios that have been tested by Microsoft. 

**Workload** | <p>**Replicate Hyper-V virtual machines**</p> <p>**(to secondary site)**</p> | <p>**Replicate Hyper-V virtual machines**</p> <p>**(to Azure)**</p> | <p>**Replicate VMware virtual machines**</p> <p>**(to secondary site)**</p> | <p>**Replicate VMware virtual machines**</p><p>**(to Azure)**</p>
---|---|---|---|---
Active Directory, DNS | Y | Y | Y | Coming soon 
Web apps (IIS, SQL) | Y | Y | Y | Coming soon
SCOM | Y | Y | Y | Coming soon
<p>SAP</p><p>Replicate SAP site to Azure for non cluster</p> | Y (tested by Microsoft) | Y (tested by Microsoft) | Y (tested by Microsoft) | Coming soon
Exchange (non-DAG) | Y | Y | Y | Coming soon
Remote Desktop/VDI | Y | Y | Y | Coming soon 
<p>Linux</p> <p>(operating system and apps)</p> | Y (tested by Microsoft) | Y (tested by Microsoft) | Y (tested by Microsoft) | Coming soon 
Dynamics AX | Y | Y | Y | Coming soon
Dynamics CRM | Coming soon | Coming soon | Y | Coming soon
Oracle | Coming soon | Coming soon | Y | Coming soon


## Features and requirements 

This table summarizes the main Site Recovery features and how they're handled during replication to Azure, replication to a secondary site using the default Hyper-V Replica replication, and using SAN.

<table border="1">
<thead>
<tr>
	<th>Feature</th><th>Replicate to Azure</th>
	<th>Replicate to a secondary site (Hyper-V Replica)</th>
	<th>Replicate to a secondary site (SAN)</th>
</tr>
</thead>

<tr>
<td>Data replication</td>
<td><p>Metadata about on-premises servers and virtual machines is stored in the Site Recovery vault.</p> <p>Replicated data is stored in Azure storage.</p></td>
<td><p>Metadata about on-premises servers and virtual machines is stored in the Site Recovery vault.</p> <p>Replicated data is stored in the location specified by the target Hyper-V server.</p></td>
<td><p>Metadata about on-premises servers and virtual machines is stored in the Site Recovery vault.</p> <p>Replicated data is stored in the target array storage.</p></td>
</tr>

<tr>
<td>Vault requirements</td>
<td><p>Azure account with the Site Recovery service</p></td>
<td><p>Azure account with the Site Recovery service</p>
</td><td><p>Azure account with the Site Recovery service</p></td>
</tr>

<tr>
<td>Replication</td>
<td>Replicate virtual machine from source Hyper-V host to Azure storage. Fail back to the source location.</td>
<td>Replicate virtual machine from source Hyper-V host to target Hyper-V host. Fail back to the source location.</td>
<td>Replicate virtual machines from source SAN storage device to target SAN device. Fail back to the source location.</td>
</tr>

<tr>
<td>Virtual machine</td>
<td>Virtual machine hard disk stored in Azure storage</td>
<td>Virtual machine hard disk stored on Hyper-V host</td>
<td>Virtual machine hard disk stored on SAN storage array</td>
</tr>

<tr>
<td>Azure storage</td>
<td>Required to store replicated virtual machine hard disks</td>
<td>Not applicable</td>
<td>Not applicable</td>
</tr>

<tr>
<td>SAN storage array</td>
<td><p>Not applicable</p></td>
<td>Not applicable</td>
<td>SAN storage array must be available in both the source and target sites and managed by VMM</td>
</tr>

<tr>
<td>VMM server</td>
<td>VMM server only in the source site only. </td>
<td>VMM servers in source and target sites are recommended. You can replicate between clouds on a single VMM server.</td>
<td>VMM server in source and target VMM sites. Clouds must contain at least one Hyper-V cluster.</td>
</tr>

<tr>
<td>VMM  version</td>
<td>System Center 2012 R2</td>
<td><p>System Center 2012 with SP1</p><p>System Center 2012 R2</p></td>
<td><p>System Center 2012 R2 with VMM Update Rollup 5.0</p></td>
</tr>

<tr>
<td>VMM configuration</td>
<td><p>Set up clouds in source and target sites</p><p>Set up VM networks in source and target site</p><p>Set up storage classifications in source and target sites <p>Install the Provider on source and target VMM servers</p></td>
<td><p>Set up clouds in source site</p><p>Set up SAN storage</p><p>Set up VM networks in source site</p><p>Install the Provider on source VMM server</p><p>Enable virtual machine protection</p></td>
<td><p>Set up clouds in source and target sites</p><p>Set up VM networks in source and target sites</p><p>Install the Provider on source and target VMM server</p><p>Enable virtual machine protection</p></td>
</tr>

<tr>
<td><p>Azure Site Recovery Provider</p><p>Used to connected over HTTPS to Site Recovery</p></td>
<td>Install on source VMM server</td>
<td>Install on source and target VMM servers</td>
<td>Install on source and target VMM servers</td>
</tr>

<tr>
<td><p>Azure Recovery Services Agent</p><p>Used to connected over HTTPS to Site Recovery</p></td>
<td>Install on Hyper-V host servers</td>
<td>Not required</td>
<td>Not required</td>
</tr>

<tr>
<td>Virtual machine recovery points</td>
<td><p>Set recovery points by time.</p> <p>Specifies how long a recovery point should be kept (0-24 hours)</p></td>
<td><p>Set recovery points by amount.</p> <p>Specifies how many additional recovery points should be kept (0-15). By default a recovery point is created every hour</p></td>
<td>Configured in array storage settings</td>
</tr>

<tr>
<td>Network mapping</td>
<td><p>Map VM networks to Azure networks.</p> <p>Network mapping ensures that all virtual machines that fail over in the same source VM network can connect after failover. In addition if there's a network gateway on the target Azure network then virtual machines can connect to on-premises virtual machines. </p><p>If mapping isn't enabled only virtual machines that fail over in the same recovery plan can connect to each other after failover to Azure.</p></td>
<td><p>Map source VM networks to target VM networks.</p> <p>Network mapping is used to place replicated virtual machines on optimal Hyper-V host servers, and ensures that virtual machines associated with the source VM network are associated with the mapped target network after failover. </p><p>If mapping isn't enabled replicated virtual machines won't be connected to a network.</p></td>
<td><p>Map source VM networks to target VM networks.</p> <p>Network mapping ensures that virtual machines associated with the source VM network are associated with the mapped target network after failover. </p><p>If mapping isn't enabled replicated virtual machines won't be connected to a network.</p></td>
</tr>

<tr>
<td>Storage mapping</td>
<td>Not applicable</td>
<td><p>Maps storage classifications on source VMM servers to storage classifications on target VMM servers.</p> <p>With mapping enable virtual machine hard disks in the source storage classification will be located in the target storage classification after failover.</p><p>If storage mapping isn't enabled replicated virtual hard disks will be stored in the default location on the target Hyper-V host server.</p></td>
<td><p>Maps between storage arrays and pools in the primary and secondary sites.</p></td>
</tr>

</table>

## Next steps

After you're finished this overview [read the best practices](site-recovery-best-practices) to help you get started with deployment planning. 
