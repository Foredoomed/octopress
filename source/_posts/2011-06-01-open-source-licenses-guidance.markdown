---
layout: post
title: "开源许可协议比较"
date: 2011-06-01 13:43
comments: true
categories: opensource
---
原文地址：[http://kkovacs.eu/businessmens-guide-to-open-source-licenses](http://kkovacs.eu/businessmens-guide-to-open-source-licenses "Open Source Licences") 


"Is it OK to take program code under the ______ license, add our own functionality, and then ______?"

Here is your answer:  
<table width="100%">
<tr>
  <th>License (in order of friendliness)</th>
  <th>...sell (license) it as a product?</th>
  <th>...provide it as software-as-a-service?</th>
</tr>
<tr>
  <td>[BSD](http://www.freebsd.org/copyright/freebsd-license.html "BSD") / [MIT](http://dev.jquery.com/browser/trunk/jquery/MIT-LICENSE.txt "MIT") License</td>
  <td>OK</td>
  <td>OK</td>
</tr>
<tr>
  <td>[Apache](http://www.apache.org/licenses/LICENSE-2.0 "Apache") License</td>
  <td>OK</td>
  <td>OK</td>
</tr>
<tr>
  <td>[MPL](http://www.mozilla.org/MPL/MPL-1.1.html "MPL") (Mozilla Public License)</td>
  <td>If done right *</td>
  <td>OK</td>
</tr>
<tr>
  <td>[LGPL](http://www.gnu.org/licenses/lgpl.txt "LGPL") (Lesser GNU Public License)</td>
  <td class="usually">As a larger work **</td>
  <td class="ok">OK</td>
</tr>
<tr>
  <td>[GPL](http://www.gnu.org/licenses/gpl.txt "GPL") (Lesser GNU Public License) (GNU Public License)</td>
  <td>your code must be given away for free</td>
  <td>OK</td>
</tr>
<tr>
  <td>[AGPL](http://www.gnu.org/licenses/agpl.txt "AGPL") (Lesser GNU Public License) (Affero GNU Public License)</td>
  <td>your code must be given away for free</td>
  <td>anyone has to be able to download and use freely</td>
</tr>
</table>

<!--more-->

**Notes:**

(1) MPL covers only the original files, and new ones that contain code from the originals. *Your new files are yours*, and yours only.

(2) It's okay if your product is NOT the modified version of the original code, but a program that just uses ("links") the code under the LGPL.

**TIP:** Interestingly, you can also legally modify any code under any of these licenses (even AGPL), and use it *internally* in your organization, *without* having to give your modifications to anybody.

**TIP 2:** For all these licenses, it's legal to contract for software development work on an open-source software for a client, who then will use the resulting product internally.

**There are some things to watch out for...**

GPL cares about where the code runs. Even if you provide something as SaaS over the Internet, some parts of your software (JavaScript) *actually is* distributed to your costumers. Just as an example, [ExtJS](http://www.extjs.com/ "ExtJS") is under GPL, but you MUST NOT base your SaaS on it, since the code gets transferred to the visitor and then run there, requiring you to distribute your code for free under the GPL too.

GPL and AGPL are picky about what kind of code gets linked to it. Not even all open-source licenses qualify.

In what exact way parts of your code interact with each other can be important. For example, you can make GPL and non-GPL code communicate through a database, web services, or APIs. But the moment you call GPL code directly, your code must be put under the GPL.

Some of these licenses may require you to include notices, source code of the original, or other things to include with your product distribution.

Make your project manager and lawyer talk to each other!

**IMPORTANT:** You may be providing a service over the Internet now, but also think about the time when you might want to sell your assets to another company; either because you are selling the service to somebody, or because you're redistributing your assets. I'm just saying, think ahead!

**What about GPL v2 versus GPL v3?**

Earlier, when GPL was v2, there was talk of putting an anti-SaaS clause into v3, but later the Affero GPL was created for that purpose. (This causes lots of confusion.)

Actually, the differences between GPL v2 and v3 are not really important to you, unless:

- you own related trademarks,  
- you are developing something that enforces DMCA,  
- you are disallowing modified code to run on your hardware.

**The financial benefits of giving code back for free**

Besides being a socially responsible businessman, you should also know that giving the code you have had developed to the open source community for free *can* also be the right thing to do financially. If you are not profiting directly from that code, it's rarely smart to hang onto it.

The community will go forward with the development anyway. Their next release will have nice new features, bug fixes, and better security. If you have kept your code secret, *you* will be the one who pays (in money or in time) for the integration of that secret code into their new releases. Or integrating their security fixes into your code. Both get messy quick, even with the help of state-of-the-art version control systems (that you should be using for *every project*, but that's another article.)

**Useful links**

[Gnu Public License FAQ](http://www.gnu.org/licenses/gpl-faq.html "Gnu Public License FAQ")  
[Mozilla Public License FAQ](http://www.mozilla.org/MPL/mpl-faq.html "Mozilla Public License FAQ")  
[Apache License FAQ](http://www.apache.org/foundation/licence-FAQ.html "Apache License FAQ")  

**Disclaimer**

I'm not a lawyer and this article should NOT be considered legal advice. Always consult your own lawyer before betting the company on somebody else's licensing terms (this applies both for open-source and non-open-source licenses).

Don't take legal, medical, or financial advice from strangers on the Internet, ok?
