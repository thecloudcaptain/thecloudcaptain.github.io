---
layout: post
title: "Azure Files Migration POC: End-to-End Implementation Runbook"
description: "A practical Azure Files migration proof of concept using Azure File Sync, private endpoints, AD DS authentication, Kerberos and preserved NTFS permissions."
date: 2026-07-22
author: The Cloud Captain
categories:
  - Azure
  - Migration
tags:
  - Azure Files
  - Azure File Sync
  - Private Endpoints
  - Active Directory
  - Kerberos
  - NTFS ACLs
permalink: /articles/azure-files-migration-poc/
---
<div class="article-hero">

<span class="article-category">AZURE · MIGRATION · FILE SERVICES</span>

# Azure Files Migration POC

A practical end-to-end approach for validating the migration of an
on-premises Windows file server to Azure Files using Azure File Sync.

</div>

> **Article status:** This runbook is currently being prepared for publication.

## What this runbook covers

This implementation validates:

- Synchronisation of files, folders and NTFS permissions through Azure File Sync
- Private connectivity for Azure Files and Storage Sync Service
- On-premises Active Directory Domain Services authentication
- Kerberos-based SMB access without storage account keys
- Azure share-level RBAC
- Per-user folder isolation through NTFS ACLs
- Synchronisation health, connectivity and access validation
- Production cutover considerations

## Implementation phases

1. Environment cleanup and reset
2. Active Directory users and security groups
3. Source folders, test data and NTFS ACLs
4. Azure storage account and file share
5. Azure Files private endpoint and DNS
6. Storage Sync Service and private endpoint
7. Azure File Sync agent installation
8. Sync group and endpoint configuration
9. Synchronisation validation
10. On-premises AD DS authentication
11. Share-level Azure RBAC
12. Kerberos and ACL-isolation testing
13. Troubleshooting and evidence collection
14. Production cutover considerations

The completed article will include portal configuration steps,
PowerShell commands, expected results and troubleshooting guidance.
