---
layout: post
title: Issue with Save-KBFile function
date: 2020-03-20 15:19:29
categories: [PowerShell, SqlServer]
tags: [powershell, sqlserver, save-kbfile, issue]
comments: true
imgpath: /assets/img/posts/2020-03-20-Issue_with_Save-KBFile_function
seo:
  date_modified: 2020-03-21 20:36:39 +0000
---

***Update 2022-02-06** - I ran into a new issue with this function, separate from the one detailed below. Microsoft have changed the URL format, so Save-KBFile no longer worked. I've fixed the URL string search and uploaded it to https://github.com/paddycull/PSSqlPatch/blob/master/Private/Save-KBFile.ps1 as part of my PSSqlPatch module*

I was recently working on a way to use PowerShell to download SQL Server patches automatically. I came across a great function by Chrissy LeMaire (creator of the excellent <a href="https://dbatools.io/">dbatools</a> module), called "Save-KBFile", on Github <a href="https://gist.github.com/potatoqualitee/b5ed9d584c79f4b662ec38bd63e70a2d">here</a>.

The function worked great it seemed, you simply give it the KB number , and optionally a download location, and it downloads the file from Microsoft. Exactly what I needed!

I came across an issue while using it though. I would an error almost every other time I ran the function. "No file found for $name" (**$name** being the KB number I wanted to download), as shown below;

<img src="{{ page.imgpath }}/ErrorMessage.png">

I done some digging, and it seems that the function fails sometimes at the following section of code, lines 85-88 from 
<a target="_blank" href="https://gist.github.com/potatoqualitee/b5ed9d584c79f4b662ec38bd63e70a2d">this gist</a>

```powershell
$links = Invoke-WebRequest -Uri 'http://www.catalog.update.microsoft.com/DownloadDialog.aspx' -Method Post -Body $body |
    Select-Object -ExpandProperty Content |
    Select-String -AllMatches -Pattern "(http[s]?\://download\.windowsupdate\.com\/[^\'\""]*)" |
    Select-Object -Unique
```

The function works reliably up to this point, and then fails here intermittently. So I went to the Microsoft Catalog website, which the function downloads the file from, and tried to do it manually. The KB I was downloading is KB4535007, CU19 for SQL Server 2017. The search worked fine, and I clicked download here;
<img src="{{ page.imgpath }}/SearchKb.png">

A popup window is then supposed to show you the download link which is what the Save-KBFile function looks for and uses to download the file. But I found that sometimes this would work as expected and show me the download file;

<img src="{{ page.imgpath }}/ExpectedResult.png">

But sometimes it wouldn't show the download link at all;
<img src="{{ page.imgpath }}/ErrorResult.png">

And this is why the function was failing sometimes. I can't think of any way to fix this issue, as it seems to be server side. I looked at other options to download KB files, but couldn't find any that are as simple as this function, so I thought of a way to workaround the issue. 

I added a retry loop to the code. If the download file isn't returned by the Microsoft catalog request, it sleeps for 1 second and tries the request again, up to a maximum of 10 retries. 

```powershell
# Added loop, as sometimes the DownloadDialog below doesn't work on the Microsoft website itself, so we retry up to 10 times.
$RetryCount = 0
while(!$links -and $RetryCount -ne 10) {
    $links = Invoke-WebRequest -Uri 'http://www.catalog.update.microsoft.com/DownloadDialog.aspx' -Method Post -Body $body |
        Select-Object -ExpandProperty Content |
        Select-String -AllMatches -Pattern "(http[s]?\://download\.windowsupdate\.com\/[^\'\""]*)" |
        Select-Object -Unique

    Write-Verbose "DownloadDialog did not return the download file, retrying."
    $RetryCount ++
    Start-Sleep 1
}
```

This works well, even though it may take a few tries. It has consistently returned the download file within the 10 allowed in my testing. You can see the retries in the Verbose output below, and it has started to download the file requested;

<img src="{{ page.imgpath }}/RetryDownload.PNG">


I'll be uploading my modified version of the Save-KBFile, along with the SQL Patches download function I use it for, soon! :slightly_smiling_face:


