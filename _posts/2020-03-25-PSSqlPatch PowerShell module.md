---
layout: post
title: PSSqlPatch PowerShell module
date: 2020-03-25 20:21:52
categories: [PowerShell, SqlServer, Patching]
tags: [powershell, sqlserver, patching]
comments: true
imgpath: /assets/img/posts/2020-03-25-PSSqlPatch_PowerShell_module
seo:
  date_modified: 2020-03-25 22:10:01 +0000
---
I've created a PowerShell module that can check for and download the latest SQL Server patch from Microsoft. The module currently consists of two functions, Get-SPSqlPatch and Save-SPSqlPatch. In this post I'll go through their usage and outputs. You can check and download the module from my Github <a href="https://github.com/paddycull/PSSqlPatch" target="_blank">here</a>.

# Get-SPSqlPatch
The Get-SPSqlPatch function accesses the Microsoft SQL patch page, which displays the latest patches available for each version of SQL Server. The website is <a href="https://technet.microsoft.com/en-us/library/ff803383.aspx" target="_blank">here</a>, so you can see what it looks like.

The function uses an Invoke-WebRequest to access the site, and then parses the resulting HTML. It gets the first table on the site, which is the patch table, and then converts it into a PSObject. It divides up the useful details like the download URL into a separate property.  

## Usage
The following will return the latest patches for all of the versions listed on the Microsoft website.
```powershell
Get-SPSqlPatch | Format-Table
```
The output of this command is;
<img src="{{ page.imgpath }}/Get-SPSqlPatch_All.png">
<span class="img-subheader">Get-SPSqlPatch result</span>


You can also specify a specific version of SQL to search for;
```powershell
Get-SPSqlPatch -SqlVersion "2017"
```
The output of this command is;
<img src="{{ page.imgpath }}/Get-SPSqlPatch_Single.png">
<span class="img-subheader">Get-SPSqlPatch singluar result</span>


# Save-SPSqlPatch
This function uses the Get-SPSqlPatch function to check for the available patches, and then uses a handy function called "Save-KBFile" to download the file. I speak about the Save-KBFile and how I modified it in my other blog post <a href="/posts/Issue-with-Save-KBFile-function/" target="_blank">here</a>. 

The way this works is using the download links returned by Get-SPSqlPatch, which contains the KB number. The KB number is then passed to the Save-KBFile function which downloads the file to the given directory. 

## Folder structure
The function downloads the files in a structured folder strucure. The structure is;
* For versions that don't have Service Packs (2017 and newer) 
   * **$DownloadDirectory\\SQL $SqlVersion\Patches\\$CUNumber**
* For versions that have Service Packs (2016 and older)
    * **$DownloadDirectory\\SQL $SqlVersion\Patches\\$SPNumber\\$SPandCUName**

## Usage
Basic usage for this function is;
```powershell
Save-SPSqlPatch -SqlVersion "2017" -DownloadDirectory "C:\test\patches"
```

This will create the folder structure specifed above and place the latest patch it finds on the Microsoft page in that folder. The output and resulting folder structure is;
<img src="{{ page.imgpath }}/DownloadSQL2017CU19.png">
<span class="img-subheader">Downloading CU19 for SQL Server 2017</span>

<img src="{{ page.imgpath }}/2017FolderStructure.png">
<span class="img-subheader">Folder structure for SQL Server 2017</span>

The function also returns the result in a PowerShell object, with attributes for version, CU and SP number, and download location. The SP attributes are blank as SQL Server 2017 does not have Service Packs.  
<img src="{{ page.imgpath }}/ObjectSQL2017CU19.png ">
<span class="img-subheader">2017 Patch PSObject</span>



If a SQL version which has Service packs is specifed, it will download the latest SP available as well as the latest CU. SP subfolders will be created as well. Example using 2016 is below;
```powershell
Save-SPSqlPatch -SqlVersion "2016" -DownloadDirectory "C:\test\patches"
```
Here it downloads the latest SP first and then the latest CU;
<img src="{{ page.imgpath }}/2016SP2Download.png">
<span class="img-subheader">Downloading SP2 for SQL Server 2016</span>

<img src="{{ page.imgpath }}/2016SP2CUDownload.png">
<span class="img-subheader">Downloading CU12 for SQL Server 2016</span>

and the PSObject it returns;

<img src="{{ page.imgpath }}/2016DownloadObjectResult.png">
<span class="img-subheader">2016 Patches PSObject</span>

And finally, the folder structure it creates. The SP is stored at the root of the SP folder, and a new folder to contain the CU is created within. 

<img src="{{ page.imgpath }}/2016FolderStructure.png">
<span class="img-subheader">2016 Folder Structure</span>

### PatchAge paramter
There is also a PatchAge parameter for the function, which allows you to specify the minimum age of the patch before downloading it. This defaults to 0. I don't have screenshots for this, as the usage is very simple and I don't think they're needed.

# End
I have other functions to install the patches as well, which I will be adding to this module once I've tidied them up a bit. I'll update my Github repo and create a new blog post when I do. :slightly_smiling_face:

