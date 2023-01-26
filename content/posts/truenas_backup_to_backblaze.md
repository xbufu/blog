+++ 
draft = false
date = 2023-01-26T18:37:40+01:00
title = "TrueNAS Backup to Backblaze"
description = "Backing up TrueNAS datasets to Backblaze without storage limit"
slug = ""
author = "Bufu"
tags = ["truenas", "backblaze", "backup"]
categories = []
externalLink = ""
series = []
+++

# Backing up TrueNAS datasets with Backblaze

Are you tired of being limited by storage restrictions when backing up your TrueNAS datasets? One solution is to use Backblaze, a popular cloud storage provider. However, by using a personal version of Backblaze in combination with virtual drives, you can bypass these limitations and create unlimited backups of your important data. In this blog post, we will walk you through the process of setting up this backup solution, including creating a backup user in TrueNAS, setting permissions on the dataset, creating an SMB share, and mounting the share with Dokany. We will also show you how to create a scheduled task for automatic remounting and how to add the virtual drive to your Backblaze account. By following these steps, you can ensure that your data is safe and secure, even with large amounts of storage space needed.

## 1. Create backup user in TrueNAS

Create a new user in TrueNAS to use for backups. We can't use the root user account, since it is not allowed to authenticate to SMB shares.

## 2. Set ACL on dataset to back up

We need to give our new backup user permission to access the dataset we want to back up.

1. Select 3 dots on dataset and click `View Permissions`
2. Click `Edit permissions` in the top right corner of the new pane
3. Click `Set ACL` to open the ACL editor
4. Select the `POSIX_RESTRICTED` default ACL
5. Add a new item of type `mask` with read, write and execute permissions. The mask defines the maximum permissions users other than the owner of the dataset can have.
6. Add another item for the backup user. Set the type to `User` and enter the username. Set permissions to read, write and execute.

## 3. Create the SMB share

Go into shares in TrueNAS and create a new SMB share. Point it to the dataset you want to backup and give it a name.

## 4. Mount the SMB share with Dokany

We will mount the SMB share as a drive using the `mirror` utility from `Dokany`. This way Backblaze will think it is an actual hard drive and we will be able to back it up without storage limitations!

1. Download the latest release of dokany from [Github](https://github.com/dokan-dev/dokany/releases)
2. Install it while selecting all components
3. Open PowerShell as Administrator and go into the directory where `mirror.exe` is located. For me it was in `C:\Program Files\Dokan\DokanLibrary-2.0.6\sample\mirror\`
4. Test mount the SMB share as the `Z` drive using the following command: `.\mirror.exe /r \\192.168.0.9\share /l z`
5. Verify the share was mounted with `ls Z:` or by browsing to it via the explorer

## 5. Create scheduled task to mount share on boot

Since we don't want to have to manually remount the share each time the computer restarts, we will create a scheduled task to do it automatically.

1. Press the Windows key and type `Task Scheduler`
2. Click on `Task Scheduler Library`
3. On the right, select `Create Task...`
4. In the `General` tab, select
  1. a name
  2. Set `Run wether user is logged on or not`
  3. Set `Run with highest privileges`
5. In `Triggers` tab, create a new trigger and select `At startup`
6. In the `Actions` tab, create a new action
7. Set the action to `Start a program`
8. Set the program to the path of the `mirror.exe` binary, i.e. `C:\Program Files\Dokan\DokanLibrary-2.0.6\sample\mirror\mirror.exe`
9. Set the arguments to `/r \\192.168.0.9\share /l z`
10. In the `Conditions` tab
  1. Make sure both `Start the task only if computer is idle` and `Stop if computer ceases to be idle` are **NOT** selected
  2. Deselected both options in the `Power` category as well
11. In the `Settings` tab
  1. Select `Run task as soon as possible after a scheduled start is missed`
  2. Set `If the task fails, restart` and set it to restart every minute for 60 times
  3. Deselect `Stop the task if it runs longer than`

## 6. Add new drive to Backblaze

We are basically finished now. Just restart the computer and the new drive should show up as expected. Now you simply need to add it in the settings in the Backblaze Control Panel.

In case Backblaze gives you an error about not being able to create a the `.bzvol` folder on the drive, check your share permissions. If your permissions on the share are fine, check if the folder has been created already. If it has, just run `chmod 777 .bzvol` on it and it should work.
