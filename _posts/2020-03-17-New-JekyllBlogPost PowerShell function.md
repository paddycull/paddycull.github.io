---
layout: post
title: New-JekyllBlogPost PowerShell function
date: 2020-03-17 19:50:24
categories: [PowerShell, Jekyll]
tags: [automation]
---
This is my first blog post on this site. I thought it would be appropriate to use it to share a small, simple PowerShell script I created to automatically initialise a Jekyll post .md file with the required parameters. :slightly_smiling_face:

The code is available on my GitHub [here]('https://github.com/paddycull/New-JekyllBlogPost')

## Why?
When playing around with Jekyll, I found it tiresome setting the filename with the correct date format, and then also adding it to the post parameters. So I thought it would be fun and useful to create a small simple function that automates the initialization of the md file :smiley:

## Features
The script initialises the Jekyll post md file with the following parameters;
* **layout** - set to "post" which is the type of layout this function will be used for.
* **title** - Set with the -PostTitle function parameter
* **date** - Defaults to the current date formatted correctly for the Jekyll file
* **tags** - Set by the user with the -Tags parameter of the function.

It creates the file in the directory that LocalPostDirectory variable is set to, which should be set to the _posts directory of the Jekyll site on the users machine.

It then opens the file in the default md file editor on the users machine. 

## Usage
A simple example is;
```powershell
New-JekyllBlogPost -PostTitle "New-JekyllBlogPost PowerShell function" -Categories "PowerShell", "Jekyll" -Tags "automation"
```

This creates a new file called '2020-03-17-New-JekyllBlogPost PowerShell function.md', in whatever directory the $LocalPostDirectory variable of teh function is set to, and sets the category and tags. It also sets the date in the Jekyll file to the current date time. Code snippets and explanations are below.

## The Code
The **datetime** parameter is set to the current date and time in the correct format for Jekyll, within the function declaration itself;
```powershell
[string] $datetime = (Get-Date -Format "yyyy-MM-dd HH:mm:ss"),
```

The function joins the **tags** and **categories** parameters with a comma and space, so they're in the correct format for the Jekyll file. It also converts the tag array to lower, which is best practice. 
```powershell
$TagsJoined = ($Tags -join ', ').ToLower()
$CategoriesJoined = $Categories -join ', '
```

It then sets the string that will be used to initialise the Jekyll md file. Here you can see how the variables are used.
```powershell
$CreateString = @"
---
layout: post
title: $PostTitle
date: $datetime
categories: [$CategoriesJoined]
tags: [$TagsJoined]
---
"@
```

The function then sets the filename of the post, using the current date along with the post title, and creates it in the directory specified with the **LocalPostDirectory** parameter.

```powershell
$PostDateForFile = (Get-Date -Format "yyyy-MM-dd").ToString() + "-"
$BlogFilePath = "$LocalPostDirectory\${PostDateForFile}${PostTitle}.md"
```

Before creating the file, we should check if the file already exists, and warn the user if it is, and get them to confirm the overwrite;
```powershell
if(Test-Path $BlogFilePath) {
    $ConfirmOverwrite = Read-Host "$BlogFilePath already exists. Do you want to overwrite it? (y/n)"

    if($ConfirmOverwrite -ne "y") {
        Throw "File already exists."
    }
}
```
Finally, we create the file with the string from above. It is important to use **-Encoding ascii** , as without it PowerShell default encoding does not work with Jekyll.
```powershell
$CreateString | $BlogFilePath -Encoding ascii
```

I also added a **DoNotOpen** switch to the function, in case the user does not want to open the file immediately after it's created. By default, it is opened in the users default md file editor;
```powershell
if(!$DoNotOpen) {
    Invoke-Item $BlogFilePath
}
```

