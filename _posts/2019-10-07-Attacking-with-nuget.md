---
layout: post
category: publish
title: Attacking with nuget
---

<p>
Supply chain attacks through package management systems is not new however there does not seem to be much written about nuget. 
</p>

<h3>What is nuget?</h3>
<p>
<pre>
    <code>
        <i>
        For .NET (including .NET Core), the Microsoft-supported mechanism for sharing code is NuGet, 
        which defines how packages for .NET are created, hosted, and consumed, and provides the tools for each of those roles.
        </i>
    </code>
  </pre>
</p>

<h3>Nuget packages</h3>
<p>
  <pre>
    <code>
        <i>
        Put simply, a NuGet package is a single ZIP file with the .nupkg extension that contains compiled code (DLLs), 
        other files related to that code, and a descriptive manifest that includes information like the package's version number.
        </i>
    </code>
  </pre>
</p>

<h3>Methods of attack</h3>
<p>    
    <ul>
        <li>Package hijacking on nuget or public feed</li>          
        <li>Tamper with existing packages in a feed</li>
        <li>Typo squat popular packages on nuget or public feed</li>
    </ul>
</p>
  
<h3>Package hijacking</h3>
<p>
<i>
    I have not seen this previously written about so I may be the first. Its not reliable to use to target organisations due to the         requirements. But it may be usuable in some cases which I hope will help some poor red teamer one day.
    </i>

Like dll hijacking it is possible to hijack nuget packages if there have been some misconfigurations.
To be able to do this several conditions are required:
    <ol>
        <li>The nuget.config must use two seperate feeds.</li>          
        <li>The package name not exist in all feeds</li>
        <li>The feed that does not have package is controllable by an attacker such as a public repository</li>
        <li>The feed that the attacker can use responds faster than the other feed</li>   
        <li>The attacker knows the name and version of the package used by the developer / build / project</li>  
        <li>The target developer / build / project does not have a local cache of the nuget project</li> 
        <li>The target developer / build / project does not validate hashes or package specific certificates</li> 
    </ol>   

    So there are a number of conditions. These will generally not exist on open source or smaller projects where just the nuget.org feed     is used. It tends to be larger enterprise projects where company specific libraries with reuse that you will find a private feed as     well as the public nuget.org feed.
    Nuget feeds defined in configuration do not have an observed order - instead feeds resolve by fastest response. So if one feed
    has a mirror closer to a developer thats where packages will be checked for first. This means if a company hosts its own packages
    outside its network on a private feed there is a chance to hijack the internal package by name on nuget.org.
    
    Mitigated with certificate validation and hash checking.
    Mitigated by reserving the package name on the public repository.
    Mitigated with an internal hosted nuget feed and removing the public feed reference.
</p>

<h3>Package tampering</h3>
<p>
    This is straight forward - if a server hosting the nuget packages for an organisation is breached the content within the projects
    can be alter. Open them up as a zip file and replace the dll. Additionally nuget installs using powershell so scripts can be
    altered. A tampered package will essentially become viral in a company travelling to developer machines and builds. 
    A malicious dll in one of those packages can end up in production if the original functionality of the dll remains.
    
    Mitigated with certificate validation and hash checking.
</p>

<h5>Typo squating</h5>
<p>  
    Simply create malicious packages on nuget.org with common typos of popular packages. Much like all typo squating.
</p>


<h5>Weaponizing a nuget package</h5>
<p>
A condition for this is a project will need to be using two feeds nuget.org and another slower feed. So any feed hosted locally or on the internal network should not be suspectiable to this issue.

</p>


<h3>Finding vulnerable projects</h3>
<h5>Non targeted</h5>
<p>
    <ol>
        <li>
            Use the github API to search repositories written in C# or another .Net language, ones that have recent activity such as a               push or create.
            </li>
        <li>
            Then search the repository for a nuget.config. Instead the configuration file check for two feeds in my example nuget.org               and myget.org.
        </li>
        <li>
            Then search the repository for packages.config and extract all the packages listed. 
        </li>
        <li>
            Now search nuget.org for any package name and version that was found that <b>isn't</b> hosted there.
        </li> 
    </ol>
</p>
<h5>Targeted</h5>
<p>
</p>
