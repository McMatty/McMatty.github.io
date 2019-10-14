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

    So there are a number of conditions. These will generally not exist on open source smaller projects. It tends to be larger
    enterprise projects where company specific libraries with reuse that you will find a private feed as well as the public nuget.org
    feed.
    Nuget feeds defined in configuration do not have an observed order - instead feeds resolve by fastest response. So if one feed
    has a mirror closer to a developer thats where packages will be checked for first. This means if a company hosts its own packages
    outside its network on a private feed there is a chance to hijack the internal package by name on nuget.org.
</p>

<h3>Typo squating</h3>
<p>
This is where it gets interesting, package names can clash between feeds. For example the popular package Newtonsoft.Json is found in   nuget.org but I could create my own package with the same name. So unless a certificate or has is being checked a call to either feed   from a project will consider both packages valid if the names and versions match.
</p>

<h5>Feed resolve order</h5>
<p>  
Package feeds in nuget are resolved based on which feed responds first. Visual studio provides the illusion that feed orders can be     controlled and resolved in defined order. As the feeds are resolved in order of response if packages are only in a single feed and the  not the other the first feed request will fail and the attempt another.
</p>
<p>
So nuget feeds are vulnerable to name collisions where the closest source will always be checked first. If this source is nuget.org an attacker can create a package on this public feed and have it pulled before a private feed as long as the attacker knows the package    name and versions. This package will also be signed by nuget.org.
</p>

<h5>All of this together..</h5>
<p>
A condition for this is a project will need to be using two feeds nuget.org and another slower feed. So any feed hosted locally or on the internal network should not be suspectiable to this issue.
</p>
<p>
So a real world example of this is MyGet.org hosting a private package - given I know the version and name I can create a match package on nuget.org given it doesn't already exist. Now I am relying on the order of feeds resolving - nuget.org is hosted on Azure and mirrored in the different regions MyGet.org is not - nuget.org wins the race and my package is pulled first.
</p>


<h3>Finding vulnerable projects</h3>
<h5>Non targeted</h5>
<p>
Use the github API to search repositories written in C# or another .Net language, ones that have recent activity such as a push or create.
Then search the repository for a nuget.config. Instead the configuration file check for two feeds in my example nuget.org and myget.org.
Then search the repository for packages.config and extract all the packages listed. Remove all well known references (Anything from Microsoft)
Now search nuget.org for any package name and version that was found that <b>isn't</b> hosted there. 
You now have a potentially way to gain access - if certificates or hashes are validated on packages this may not work.
</p>
<h5>Targeted</h5>
<p>
</p>
