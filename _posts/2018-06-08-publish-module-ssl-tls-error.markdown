---
layout: post
title:  "Publish-Module: Could not create SSl/TLS secure channel error"
date:   2018-06-08 9:00:51 -0500
categories: powershell powershellget
---
Ran into an issue this week with the Publish-Module cmdlet. I have recently setup a internal Nuget repo server, in my case a Proget server.
I began my work integrating a CI/CD pipeline for taking Powershell modules and uploading them to Proget. Unfortunately I quickly ran into an issue when attempting to upload
a module to the Proget feed. I would attempt to run {% highlight powershell %}Publish-Module -Path C:\pathtomodule\ -NuGetApiKey MyApiKey -Repository MyInternalRepo{% endhighlight %}
and it would error out with:

{% highlight powershell %}
writeErrorStream      : True
PSMessageDetails      :
Exception             : Microsoft.PowerShell.Commands.WriteErrorException: Failed to publish module 'MyModule': 'The request was aborted: Could not create SSL/TLS secure
                        channel.
                        '.
TargetObject          :
CategoryInfo          : InvalidOperation: (:) [Write-Error], WriteErrorException
FullyQualifiedErrorId : FailedToPublishTheModule,Publish-PSArtifactUtility
ErrorDetails          :
InvocationInfo        : System.Management.Automation.InvocationInfo
ScriptStackTrace      : at Publish-PSArtifactUtility, C:\Program Files\WindowsPowerShell\Modules\PowerShellGet\1.6.5\PSModule.psm1: line 5881
                        at Publish-Module<Process>, C:\Program Files\WindowsPowerShell\Modules\PowerShellGet\1.6.5\PSModule.psm1: line 10691
                        at <ScriptBlock>, <No file>: line 1
PipelineIterationInfo : {0, 1}
{% endhighlight %}

Initially I thought this was related to my cipher suite setup on my Server 2016 box. I kept trying to force Powershell to use different ciphers, ensuring it was using modern versions of TLS etc.

Turned out it was due to the outdated NuGet.exe that the PowerShellGet module will automatically download for you. It will download a version 2.8 of the Nuget CLI tool, which does not have support for modern
TLS. Manually replacing the [NuGet.exe with a more modern flavor (4.6.2.5055 for example)][nuget-cli-download] at C:\ProgramData\Microsoft\Windows\Powershell\PowerShellGet\ resolved the issue for me and I am now happily on my way.

This should be getting fixed rather soon, as I opened [an issue on the PowershellGet][github-issue] repo and it is already being worked on.

[github-issue]: https://github.com/PowerShell/PowerShellGet/issues/288
[nuget-cli-download]: https://dist.nuget.org/win-x86-commandline/v4.6.2/nuget.exe