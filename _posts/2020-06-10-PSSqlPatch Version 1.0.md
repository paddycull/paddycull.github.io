---
layout: post
title: PSSqlPatch Version 1.0
date: 2020-06-10 19:32:39 +0100
categories: [PowerShell, SqlServer, Patching]
tags: [powershell, sqlserver, patching, pssqlpatch]
comments: true
imgpath: /assets/img/posts/2020-06-10-PSSqlPatch_Version_1.0
---
I'm excited to announce that I've released a major update to my PSSqlPatch module, which is available on my Github <a href="https://github.com/paddycull/PSSqlPatch" target="_blank">here</a>. This update brings a lot of new functions to the module. It now has functions which cover everything you need for managing SQL Server patching.
* **Downloading** - The module has functions to check for and download the latest SQL Server patches from Microsoft. 
* **Applying** - The module contains multiple functions to apply patches, either automatically applying the latest applicable patch available from a specified share, or via a full path to the patch file.
* **Reporting** - There's a function to report if a given list of servers have been updated to the latest available patch or not.

Full descriptions, examples and output of each of the functions are on the <a href="https://github.com/paddycull/PSSqlPatch" target="_blank">Github page</a>, so check it out if this sounds interesting. :slightly_smiling_face:

It's also available via the PowerShell gallery. To install it, all you need to do is run the following command in PowerShell;
```powershell
Install-Module PSSqlPatch
```

I've spent a while improving the module as much as I can before this release, and I'm really happy with the result. We've been using the module in my workplace for two patch cycles now, and it's saved us a huge amount of time. The function we use for scheduling our patching is <a href="https://github.com/paddycull/PSSqlPatch#install-spmultiplesqlpatches" target="_blank">Install-SPSqlMultiplePatches</a>. This function patches a given list of servers to the latest applicable patch , and does 5 servers concurrently (by default) as it iterates through the server list. You can click that link to read more information about. It can be scheduled out of hours using a Windows Scheduled Task. I will create a blog post about how to do this in the near future. 

Thanks for reading, and if you do try the module out or if you have any suggestions, I'd love to hear from you. :slightly_smiling_face: