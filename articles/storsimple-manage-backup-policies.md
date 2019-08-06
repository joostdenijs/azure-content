<properties 
   pageTitle="Manage your StorSimple backup policies"
   description="Explains how you can use the StorSimple Manager service to create and manage backup jobs and backup schedules."
   services="storsimple"
   documentationCenter="NA"
   authors="SharS"
   manager="adinah"
   editor="tysonn" />
<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="05/14/2015"
   ms.author="v-sharos" />

# Manage your StorSimple backup policies

## Overview

This tutorial explains how to use the StorSimple Manager service **Backup Policies** page to control backup processes and backup retention for your StorSimple volumes. It also describes how to complete a manual backup.

The **Backup Policies** page allows you to manage backup policies and schedule local and cloud snapshots. (Backup policies are used to configure backup schedules and backup retention for a collection of volumes.) This page lists the backup policies, their types, the associated volumes, the number of backups retained, and the option to enable these policies.

The **Backup Policies** page also allows you to filter the existing backup policies by one or more of the following fields:

- **Policy name** – The name associated with the policy. The different types of policies include:

   - Scheduled policies, which are explicitly created by the user.
   - Automatic policies, which are created when the default backup for this volume option was enabled at the time of volume creation. These policies are named as VolumeName_Default where Volume name refers to the name of the StorSimple volume configured by the user in the Management Portal. The automatic policies result in daily cloud snapshots beginning at 22:30 device time.
   - Imported policies, which were originally created in the StorSimple Snapshot Manager. These have a tag that describes the StorSimple Snapshot Manager host that the policies were imported from.

- **Volumes** – The volumes associated with the policy. All the volumes associated with a backup policy are grouped together when backups are created.

- **Last successful backup** – The date and time of the last successful backup that was taken with this policy.

- **Next backup** – The date and time of the next scheduled backup that will be initiated by this policy.

- **Schedules** – The number of schedules associated with the backup policy.

The frequently used operations that you can perform from this page are:

- Add a backup policy 
- Add or modify a schedule 
- Delete a backup policy 
- Take a manual backup 
- Create a custom backup policy with multiple volumes and schedules 

## Add a backup policy

Add a backup policy to automatically schedule your backups. Perform the following steps in the Management Portal to add a backup policy for your StorSimple device. After you add the policy, you can define a schedule (see [Add or modify a schedule](#add-or-modify-a-schedule).

[AZURE.INCLUDE [storsimple-add-backup-policy](../includes/storsimple-add-backup-policy.md)]


## Add or modify a schedule

You can add or modify a schedule that is attached to an existing backup policy on your StorSimple device. Perform the following steps in the Management Portal to add or modify a schedule.

[AZURE.INCLUDE [storsimple-add-modify-backup-schedule](../includes/storsimple-add-modify-backup-schedule.md)]

## Delete a backup policy

Perform the following steps in the Management Portal to delete a backup policy on your StorSimple device.

[AZURE.INCLUDE [storsimple-delete-backup-policy](../includes/storsimple-delete-backup-policy.md)]


## Take a manual backup

Perform the following steps in the Management Portal to create an on-demand (manual) backup for a single volume.

[AZURE.INCLUDE [storsimple-create-manual-backup](../includes/storsimple-create-manual-backup.md)]

## Create a custom backup policy with multiple volumes and schedules

Perform the following steps in the Management Portal to create a custom backup policy that has multiple volumes and schedules.

[AZURE.INCLUDE [storsimple-create-custom-backup-policy](../includes/storsimple-create-custom-backup-policy.md)]


## Next steps

Learn how to manage StorSimple backups with the [StorSimple Snapshot Manager](https://msdn.microsoft.com/library/azure/dn772365.aspx).