---
layout: post
title: "Configuring TeamCity 7 with Git revision short hash, Uploading to Amazon S3 and sending email with release link and Developer commit details"
date: 2013-06-05 20:37
categories: [git, TeamCity, amazon, Powershell, shell]
keywords: Git, TeamCity, Amzon S3, Powershell, send email, teamcity git short hash revision, shell
description: Configuring TeamCity 7 with Git Revision Hashes, Uploading to Amazon S3 and sending email with release link and Developer commit details.
---

Here is a short tutorial (with extra long title!) how to improve the [JetBrains](http://www.jetbrains.com/) 
[TeamCity](http://www.jetbrains.com/teamcity/) Build Server for _CI_ (Continuos Integration).

This is what wiill do to improve _TeamCity 7_:

* Change the Build version number to include Git short hash (7 chars) as Revison e.g. _1.0.1.53e7986_.
* Upload file to [Amazon S3](http://aws.amazon.com/s3/).
* Send email via Gmail with: 
	* Latest downloadable link of release from _Amazon S3_.
	* Developer commit message details.

> Note:
>
> This is not blog post how to set up _TeamCity_, this is usually really straight forward thing to do.
> So in this blog post I assume that there is _TeamCity_ project and at least one _Build Configuration_
> already created!


## Setting Git Short Hash as Revision of TeamCity Build Number

* In _TeamCity_ go to desired named "_Build Configuration_" in our case "_aeon_".
* Click somewhere on right side on _Edit Configuration Settings_ to enter _aeon Configuration Steps_.
* In _(1) General Settings_ set _Build format number_ to something like 1.0.{0} and _Save_.
* In _(3) Build Steps_ click _Add build step_ choose _Powershell_ in listbox.
* Set _Name_ if you want like "_Get Build Number with short Git Hash_".
* For _Script_ choose "_Source code_" from drop down box.
* In _Script Source_ add following _Powershell_ code and click _Save_:

```powershell
$BuildNumber = "%build.number%"
$Hash = "%build.vcs.number%"
$ShortHash = $Hash.substring(0,7)
Write-Host "##teamcity[buildNumber '$BuildNumber.$ShortHash']"
```
* _Script execution mode_ should not matter, mine is "Execute .ps1 script...".
* After save **make sure that this _Build Step_ is first**!!!!
* Use _Reorder build steps_ link, if you have existing build setps.

> Note:
>
> When build starts by Agent, it will be 1.0.1 and at the end it will change to
> 1.0.1.53e7986!

### Shell version (_Linux_ or _Mac_)

* All same as _Powershell_ until _(3) Build Steps_.
* In _(3) Build Steps_ click _Add build step_ choose _Command line_ in listbox.
* In _Custom Script_ add following _shell_ script and click _Save_:

```bash
BUILD_NUMBER=%build.number%
GIT_HASH=%build.vcs.number%
GIT_HASH_SHORT=${GIT_HASH:0:7}
echo "##teamcity[buildNumber '$BUILD_NUMBER.${GIT_HASH_SHORT}']"
```

> Note:
>
> I didn't test _shell_ version, it should work, but maybe some tweaks are necessary!

## Powershell script for sending file to Amazon S3 and sending email via Gmail

* Download and install [AWS Tools for Windows Powershell](http://aws.amazon.com/powershell).
* With this SDK is really easy to upload file to Amazon S3.
* Amazon S3 account is needed and also _Access Key_ and _Secret Key_.
* Assumes that Amazon S3 _bucket_ name is "_aeon_" and region is "_eu-west-1_".
* This will produce _https://s3-eu-west-1.amazonaws.com/aeon/<release-file>"_.
* The script is lousy written, it should be better, especially when it is needed to be reused for multiple Builded apps.
* The thing needed to be entered will be inside "&lt;something-to-do&gt;".
* In my case the uploaded file is ".exe" file so it is harcoded in Amazon S3 URL.
* Make sure that file name is not really crazy and there are no spaces in it (it is always easier this way)!

```powershell
$bucketName = 'aeon'
$hasCredentials = $false;
$filePath = '<absolute-path-to-folder-where-file-for-upload-is>'


function GetLastGitLog {
    $sourcePath = '<absolute-path-to-where-teamcity-pulls-repo>'
    $isSourcePathExist = Test-Path $sourcePath

    if (!$isSourcePathExist) {
        "source folder ($sourcePath) for file is not existent! Exiting..."
        exit
    }

    Set-Location $sourcePath

    git log -n 1

}

function MessageBody($version) {    
    $url = "https://s3-eu-west-1.amazonaws.com/{0}/{1}.exe" -f $bucketName, $version

    $commitMessage = GetLastGitLog | Out-String
    
    "<h2>My Fabulos Product - New Version</h2>

     <p><strong>Version:</strong> {0}</p>
     <p><strong>Download Link:</strong> <a href='{1}'>{2}.exe</a></p>

     <p></p>
     <p><strong>Developer Commit Details:</strong></p>
     <pre>{3}</pre>

     <p></p>
     <p>Regards,</p>
     <p>My Fabulous Team City Build Server (example.com)</p>
     " -f $version, $url, $version, $commitMessage
}

function SendEmail($binaryName) {    
    $version = $binaryName.TrimEnd('.exe')
    $mailTo = "<first-email>,<second-email>"
    $subject = "[New Version]: $version"

    $smtp = new-object Net.Mail.SmtpClient('smtp.gmail.com', 587) 
    $smtp.EnableSsl = $true
    $smtp.Credentials = New-Object System.Net.NetworkCredential("<my-gmail-username>", "<my-gmail-password>")

    $message = New-Object Net.Mail.MailMessage
    $message.From = "<my-gmail-username>@gmail.com"
    $message.IsBodyHtml = $true
    $message.Subject = $subject
    $message.Body = MessageBody($version)
    $message.To.Add($mailTo)

    $smtp.Send($message)
}

$credentials = Get-AWSCredentials -ListStoredCredentials

if ($currentCredentials -and $currentCredentials.Count -gt 0) {
    $hasCredentials = $true
}

if (!$hasCredentials) {
     Initialize-AWSDefaults -AccessKey <your-amazon-access-key> -SecretKey <your-amazon-secret-key> -Region eu-west-1
}

$isFilePathExist = Test-Path $filePath

if (!$isFilePathExist) {
    "Folder ($filePath) for file is not existent! Exiting..."
    exit
}

# Filename and extension of file.
$key = (Get-ChildItem $filePath)[0].Name;

$file = (Get-ChildItem $filePath)[0].FullName

if ($key -and $file) {
    Write-S3Object -BucketName $bucketName -File $file -Key $key -PublicReadOnly

    SendEmail($key)
} 
```
* Save this script to &lt;whatever-you-want-to-call-it&gt;.ps1.
* This _Powershell_ script can be added to _TeamCity_ as a final _Build Step_.
* In _(3) Build Steps_ click _Add build step_ choose _Powershell_ in listbox.
* For _Script_ choose _File_ from drop down box.
* For _Script File_ add absolute path to saved _Powershell_ script saved as _ps1_.

> Note:
>
> Sorry, no _Linux_ or _Mac_ _shell_ version of this script.

## Conclusion

Having short _Git_ hash for a _Revision_ in your versioning is very helpful and seven chars
are mostly always enough to uniquely identify this release build with commit in your _Git_ repository.

The amazon S3 upload + send email with developer commit deatails is not state of art 
_Powershell_ script, it is very buggy and not that much flexible, but it can help a 
lot in communication with QA (or outside beta testers) and having release build version binary, 
right after Developer commit!
