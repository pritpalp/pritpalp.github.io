---
title: "Updating an ASE SSL Certificate"
date: 2021-08-30 20:00:00 +0100
categories:
  - Azure
tags:
  - Blog
  - ASE
  - SSL Certificate
excerpt: How to update your ASE SSL certificate
header: 
  overlay_image: /assets/images/post-header.jpg
  caption: "Photo by [**Irvan Smith**](https://unsplash.com/@mr_vero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [**Unsplash**](https://unsplash.com)"
  overlay_filter: 0.5
---
This should be a straight forward task, but do things in the wrong order and things can get weird.

This applies to ASEv2.

The ASE adds a hidden certificate to the resource group that it is deployed to. You'll be able to see the certificate in the resource group by selecting "Show hidden types". It will be of type "microsoft.web/certificate".

![Azure resource setting](/assets/images/hidden-resources.jpg)

Updating the certificate on the web apps first will update the "hidden" certificate...which will stop you doing the update within the ASE. Trying to update the certificate in the ASE will look like its applying, but will fail:
```json
'Another certificate exists with same thumbprint {0} at location {1} in the Resource Group {2}.'
```
In my case, my webapps were in a different resource group to my ASE. So the error didn't make much sense. Until you find the certificate that is hidden.

I ended up having to remove the certificate from the webapps (and the hidden certificate) and applying it to the ASE first. Once that was done, everything worked like it should.

## Steps to Perform
- Update the SSL Certificate on the ASE
- Update the SSL Certificate everywhere else you need it
