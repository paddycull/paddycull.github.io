---
layout: post
title: New-JekyllBlogPost PowerShell function
date: 2020-03-17 19:50:24
categories: [PowerShell, Jekyll]
tags: [automation,powershell]
comments: true
seo:
  date_modified: 2020-03-31 13:39:56 +0100
---
This is my first blog post on this site. I thought it would be appropriate to use it to share a small, simple PowerShell script I created to automatically initialise a Jekyll post .md file with the required parameters. :slightly_smiling_face:

The code is available on my GitHub <a target="_blank" href='https://github.com/paddycull/New-JekyllBlogPost'>here</a> 

*Update 2020-03-31 - The function now automatically creates an image directory and variable for all new posts. Details below.*

## Why?
When playing around with Jekyll, I found it tiresome setting the filename with the correct date format, and then also adding it to the post parameters. So I thought it would be fun and useful to create a small simple function that automates the initialization of the md file and file name. :slightly_smiling_face:

## Features
The script initialises the Jekyll post md file with the following parameters;
* **layout** - set to "post" which is the type of layout this function will be used for.
* **title** - Set with the -PostTitle function parameter.
* **date** - Defaults to the current date formatted correctly for the Jekyll file.
* **categories** - Set by the user with the -Categories function parameter.
* **tags** - Set by the user with the -Tags function parameter.
* **comments** - Set to true.
* **imgpath** - Custom parameter that contains the relative path to an image directory for the current post.


It creates the file in the directory that LocalPostDirectory variable is set to, which should be set to the _posts directory of the Jekyll site on the users machine.

It then opens the file in the default md file editor on the users machine. 

## Usage
A simple example is;
```powershell
New-JekyllBlogPost -PostTitle "New-JekyllBlogPost PowerShell function" -Categories "PowerShell", "Jekyll" -Tags "automation"
```

This command will do the following; 
* Creates a new file called '2020-03-17-New-JekyllBlogPost PowerShell function.md', in within the posts directory of the **$LocalSiteDirectory**
* Sets all parameters outlined above.
* Sets the date parameter in the Jekyll file to the current date time in the required format. 
* Sets up an image directory using the post name, within the /assets/img/ directory of the site, and sets the imgpath parameter to this value.

Code snippets and explanations are below.

## The Code
First we need to initialise the local image and post directories, based on what the **$LocalSiteDirectory** parameter;
```powershell
$LocalPostDirectory = "$LocalSiteDirectory\_posts"
$LocalImageRootDirectory = "$LocalSiteDirectory\assets\img\posts\"
```

The **datetime** parameter is set to the current date and time in the correct format for Jekyll, within the function declaration itself;
```powershell
[string] $datetime = (Get-Date -Format "yyyy-MM-dd HH:mm:ss"),
```

The function joins the **tags** and **categories** parameters with a comma and space, so they're in the correct format for the Jekyll file. It also converts the tag array to lower, which is best practice. 
```powershell
$TagsJoined = ($Tags -join ', ').ToLower()
$CategoriesJoined = $Categories -join ', '
```

Setup the filename of the post, using the current date along with the post title, and creates it in the directory specified with the **LocalPostDirectory** parameter.

```powershell
$PostDateForFile = (Get-Date -Format "yyyy-MM-dd").ToString() + "-"
$BlogFilePath = "$LocalPostDirectory\${PostDateForFile}${PostTitle}.md"
```

Initialise a new image directory, which name is based on the post title. RelativeImageDirectory is what the md file will use.
```powershell
    $JoinedPostTitle = ("${PostDateForFile}${PostTitle}" -replace ' ', '_')
    $LocalPostImageDirectory = $LocalImageRootDirectory + $JoinedPostTitle
    New-Item $LocalPostImageDirectory -ItemType Directory -Force | Out-Null
    $RelativeImageDirectory = "/assets/img/posts/$JoinedPostTitle"
```

Setup the string that will be used to initialise the Jekyll md file. Here you can see how the variables are used.
```powershell
$CreateString = @"
---
layout: post
title: $PostTitle
date: $datetime
categories: [$CategoriesJoined]
tags: [$TagsJoined]
comments: true
imgpath: $RelativeImageDirectory
---
"@
```

Before creating the file, check if the file already exists, and warn the user if it is, and get them to confirm the overwrite;
```powershell
if(Test-Path $BlogFilePath) {
    $ConfirmOverwrite = Read-Host "$BlogFilePath already exists. Do you want to overwrite it? (y/n)"

    if($ConfirmOverwrite -ne "y") {
        Throw "File already exists."
    }
}
```
Finally, create the file with the string from above. It is important to use **-Encoding ascii** , as without it PowerShell default encoding does not work with Jekyll.
```powershell
$CreateString | Out-File $BlogFilePath -Encoding ascii
```

Images can then be accessed within the md file using;
```html
{% raw %}<img src="{{ page.imgpath }}/imagename.png">{% endraw %}
```

I also added a **DoNotOpen** switch to the function, in case the user does not want to open the file immediately after it's created. By default, it is opened in the users default md file editor;
```powershell
if(!$DoNotOpen) {
    Invoke-Item $BlogFilePath
}
```
