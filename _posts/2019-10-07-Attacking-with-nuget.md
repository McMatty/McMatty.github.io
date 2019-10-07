---
layout: post
category: publish
title: Attacking with nuget
---

<p>
Supply chain attacks through package management is nothing new but I have been looking into how it could be used
to attack an organization recently. I wanted to see how easy it would be to gain a foothold.
</p>

<h3>What is nuget?</h3>
<p>
Going from the first google result - its a package manager developed by Microsoft much like NPM.
</p>

<h3>Nuget packages</h3>
<p>
Nuget packages are basically a zip file with some artefacts to be used in a .Net project. These are normally dlls to be used in project.
Microsoft analyzers also come in this format and add themselves to projects. 
The question is how do they add themselves to projects - well it turns out when these projects are pulled down by a developer or build a PowerShell script in the package is executed.
</p>
<h5>Methods of attack</h5>
<p>  
So we have a few avenues here. 
Create a malicious dll to be attached to a users project - perhaps maintaining the expected funtionality.
Create a malicious PowerShell script and add it to the package so it executes and adds the expected artefact to the project.
</p>
<h5>How to weaponize</h5>
<p>  

</p>
  
<h3>Nuget feeds</h3>
<p>
Nuget.org is the central feed most developers will be familar with. It allows any developer to create, upload an maintain packages for use in projects. The creation part is the import part here. Feeds can be created on servers with ease or hosted in the cloud. Azure or Myget are some alternative locations feeds are defined with MyGet.org being quite popular. These feeds can be marked private so they are only accessible within an organization or authorized users.
</p>
<h5>Package name squating</h5>
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
<p>

</p>
