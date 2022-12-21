---
layout: post
title:  "Introducing the Microsoft Data Encryption SDK: Protecting CSV Files"
date:   2021-03-20 22:45:00 -0600
categories: azure encryption
---
About two months ago, Microsoft quietly published a new GitHub repo titled [Microsoft Data Encryption SDK][mde-sdk] (MDE SDK). It includes sample code for encrypting and decrypting Apache Parquet files using the same encryption algorithm implemented in [SQL Always Encrypted][sql-ae]. This is significant because it represents an expansion of Microsoft's vision for column level encryption beyond SQL server to popular data formats and, eventually, other data engines as well. As the repo describes, the idea recognizes that enterprise data needs to flow across various systems for processing, analytics, and a variety of other uses. For some organizations, this reality represents a significant challenge to data protection. Column level encryption offers an additional layer of assurance that sensitive data remains confidential.

At the heart of the MDE SDK are three NuGet packages: [Microsoft.Data.Encryption.Cryptography][mde-crypto], [Microsoft.Data.Encryption.FileEncryption][mde-file], and [
Microsoft.Data.Encryption.AzureKeyVaultProvider][mde-keyvault]. In this post, we'll walk through how all three packages can be used to encrypt sensitive columns in a CSV file, push them to Azure SQL Database without decrypting, and view the data from SQL as an authorized user. The following diagram illustrates the flow:

[![mde csv diagram](/media/2021-03-20/diagram-mde-csv.png "Data protection CSV example")](/media/2021-03-20/diagram-mde-csv.png)

## Configuration and Prerequisites

Everything needed demonstrate this is available in the following GitHub repo:

https://github.com/travisnielsen/column-encryption

It provides deployment scripts for all of the Azure components represented in the above diagram as well as sample code, which includes:

* A common library project that extends the MDE SDK with support for CSV files and externalized cryptographic metadata.
* A console application used for loading configuration metdata and initiating the desired encryption / decryption task
* An Azure Function that loads CSV files and imports records into SQL

Clone the repo locally and follow the [Azure Environment Configuration][ce-setup] steps to deploy the supporting services, configure SQL Always Encrypted, and generate the YAML metadata file. Once completed, encrypting data via an external process, moving it to SQL, and viewing the plaintext as an authorized user is straightforward.

## Walkthrough: Encrypt with External Process and Import to SQL

Open a terminal and navigate to `src\ColumnEncryptApp`. Confirm you have a valid `config.yaml` file for your environment in this folder. Run the following command to process `userdata.csv`:

```bash
dotnet build
.\bin\Debug\net5.0\ColumnEncryptApp.exe encrypt -m config.yaml -d userdata.csv
```

You will be prompted to authenticate to your Azure Active Directory tenant. This is necessary for the MDE SDK to access your Key Vault instance to unwrap the Column Encryption Key for  `SSN`. Once this is done, you should see the column encrypted in `userdata-encrypted.csv`.

![SSN column encrypted](/media/2021-03-20/csv-encrypted.png "SSN column encrypted")

Next, upload `userdata-encrypted.csv` to the `userdata` container in your storage account. The Function will then process the file and import the data to the SQL `userdata` table. During this process, the `SSN` column is never decrypted. In fact, the Function doesn't even have access to Key Vault to unwrap any encryption keys.

Once completed, you should now be able to connect to the SQL database using the tool of your choice. By default, you will only see the hexadecimal string ciphertext for the `SSN` column. This indicates the data is still protected within SQL.

![sql query with ciphertext](/media/2021-03-20/sql-query-ciphertext.png "Query with no admin access to SSN")

However, because the column was originally encrypted with the MDE SDK, it can be decrypted by authorized users via standard SQL Always Encrypted tools. Here is an example of the same query run from Azure Data Studio with the Column Encryption setting enabled.

![sql query with plaintext](/media/2021-03-20/sql-query-plaintext.png "Query with no admin access to SSN")

## Wrap up

Hopefully, this example illustrates how the MDE SDK can be used to unlock additional encryption scenarios with its built-in support for Parquet and, through extensibility, additional formats like CSV. Compatibility with data engines like Azure SQL Database allows for sensitivie columns to remain encrypted when moving across systems, thus reducing risk of key and data disclosure throughout.

[mde-sdk]: https://github.com/Azure/microsoft-data-encryption-sdk
[sql-ae]: https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine?view=azuresqldb-current
[sql-ae-crypto]: https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-cryptography?view=sql-server-ver15#data-encryption-algorithm
[mde-crypto]: https://www.nuget.org/packages/Microsoft.Data.Encryption.Cryptography
[mde-file]: https://www.nuget.org/packages/Microsoft.Data.Encryption.FileEncryption
[mde-keyvault]: https://www.nuget.org/packages/Microsoft.Data.Encryption.AzureKeyVaultProvider
[ce-setup]: https://github.com/travisnielsen/column-encryption/blob/main/docs/configure-azure.md
