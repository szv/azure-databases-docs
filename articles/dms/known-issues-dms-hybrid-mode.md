---
title: Known issues/migration limitations with using Hybrid mode
description: Learn about known issues/migration limitations with using Azure Database Migration Service in hybrid mode.
author: abhims14
ms.author: abhishekum
ms.reviewer: randolphwest
ms.date: 09/18/2024
ms.service: azure-database-migration-service
ms.topic: troubleshooting
ms.custom:
  - mvc
  - sql-migration-content
---

# Known issues/migration limitations with using hybrid mode

Known issues and limitations associated with using Azure Database Migration Service in hybrid mode are described in the following sections.

## Installer fails to authenticate

After uploading the certificate to your AdApp, there's a delay of up to a couple of minutes before it can authenticate with Azure. The installer will attempt to retry with some delay, but it's possible for the propagation delay to be longer than the retry, and you'll see a **FailedToGetAccessTokenException** message. If the certificate was uploaded to the correct AdApp and the correct AppId was provided in dmsSettings.json, try running the install command again.

## Service "offline" after successful installation

If the service shows as offline after the installation process completes successfully, try using the following steps.

1. In the Azure portal, in your instance of Azure Database Migration Service, navigate to the **Hybrid** settings tab, and then verify that the worker is registered by checking the grid of registered workers.

   The status of this worker should be **Online**, but it might show as **Offline** if there's a problem.

1. On the worker computer, check the status of the service by running the following PowerShell command:

   ```powershell
   Get-Service Scenario*
   ```

   This command gives you the status of the Windows service running the worker. There should only be a single result. If the worker is stopped, you can attempt to restart it by using the following PowerShell command:

   ```powershell
   Start-Service Scenario*
   ```

   You can also check the service in the Windows Services UI.

1. If the Windows service cycles between Running and Stopped, then the worker encountered problems starting up. Check the Azure Database Migration Service hybrid worker logs to determine the problem.

   - Installation process logs are stored in the "logs" folder within the folder from which the installer executable was run.

   - Azure Database Migration Service hybrid worker logs are stored in the **WorkerLogs** folder, in the folder in which worker is installed. The default location for the hybrid worker log files is **C:\Program Files\DatabaseMigrationServiceHybrid\WorkerLogs**.

<a id="using-your-own-signed-certificate"></a>

## Use your own signed certificate

The certificate generated by the action GenerateCert is a self-signed certificate, which might not be acceptable based on your internal security policies. Instead of using this certificate, you can provide your own certificate and provide the thumbprint in dmsSettings.json. This certificate will need to be uploaded to your AdApp and installed on the computer on which you're installing the Azure Database Migration Service hybrid worker. Then, install this certificate with the private key into the Local Machine certificate store.

<a id="running-the-worker-service-as-a-low-privilege-account"></a>

## Run the worker service as a low-privilege account

By default, the Azure Database Migration Service hybrid worker service runs as the Local System account. You can change the account used for this service as long as the account you use has network permissions. To change the service 'run as' account, use the following process.

1. Stop the service, either through Windows Services or by using the Stop-Service command in PowerShell.

1. Update the service to use a different logon account.

1. In certmgr for Local Computer certificates, give private key permissions to the new account for the **DMS Hybrid App Key** and **DMS Scenario Engine Key Pair** certificates.

   1. Open certmgr to view the following keys:

   - DMS Hybrid App Key
   - DMS Hybrid Worker Setup Key
   - DMS Scenario Engine Key Pair

   1. Right-click the **DMS Hybrid App Key** entry, point to **All Tasks**, and then select **Manage Private Keys**.

   1. On the **Security** tab, select **Add**, and then enter the name of the account.

   1. Use the same steps to grant private key permission for the new account to the **DMS Scenario Engine Key Pair** certificate.

<a id="unregistering-the-worker-manually"></a>

## Unregister the worker manually

If you no longer have access to the worker computer, you can unregister the worker and reuse your Azure Database Migration Service instance by performing the following steps:

1. In the Azure portal, got to your Azure Database Migration Service instance, and then navigate to the **Hybrid** settings page.

   Your worker entry appears in the list, with the status showing as **Offline**.

1. To the far right of the worker entry listing, select the ellipses, and then select **Unregister**.

<a id="addressing-issues-for-specific-migration-scenarios"></a>

## Address issues for specific migration scenarios

The sections below describe scenario-specific issues related to using Azure Database Migration Service hybrid mode to perform an online migration.

### Online migrations to Azure SQL Managed Instance

**High CPU usage**

**Issue**: For online migrations to SQL Managed Instance, the computer running the hybrid worker will encounter high CPU usage if there are too many backups or if the backups are too large.

**Mitigation**: To mitigate this issue, use compressed backups, split the migration so that it uses multiple shares, or scale up the computer running the hybrid worker.
