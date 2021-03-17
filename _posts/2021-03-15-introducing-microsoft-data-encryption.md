---
layout: post
title:  "Introducing the Microsoft Data Encryption SDK: Protecting CSV Files"
date:   2021-03-15 22:16:25 -0600
categories: azure encryption
---
About two months ago, Microsoft quietly published a new GitHub repo titled [Microsoft Data Encryption SDK][mde-sdk] (MDE SDK). It includes sample code for encrypting and decrypting Apache Parquet files using the same encryption algorithm implemented in [SQL Always Encrypted][sql-ae]. This is significant because it represents an expansion of Microsoft's vision for column level encryption beyond SQL server to popular data formats and, eventually, other data engines as well. As the repo describes, the idea recognizes that enterprise data needs to flow across various systems for processing, analytics, and a variety of other uses. For some organizations, this reality represents a significant challenge to data protection. Column level encryption offers an additional layer of assurance that sensitive data remains confidential.

At the heart of the MDE SDK are three NuGet packages: [Microsoft.Data.Encryption.Cryptography][mde-crypto], [Microsoft.Data.Encryption.FileEncryption][mde-file], and [
Microsoft.Data.Encryption.AzureKeyVaultProvider][mde-keyvault]. In this post, we'll walk through how to all three packages can be used to encrypt senstive columns in a CSV file, push them to Azure SQL Database without decrypting, and view the data from SQL as an authorized user.

## Configuring Encryption Metadata




[mde-sdk]: https://github.com/Azure/microsoft-data-encryption-sdk
[sql-ae]: https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine?view=azuresqldb-current
[mde-crypto]: https://www.nuget.org/packages/Microsoft.Data.Encryption.Cryptography
[mde-file]: https://www.nuget.org/packages/Microsoft.Data.Encryption.FileEncryption
[mde-keyvault]: https://www.nuget.org/packages/Microsoft.Data.Encryption.AzureKeyVaultProvider
