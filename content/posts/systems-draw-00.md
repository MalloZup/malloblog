---
title: "Systems Design: 00-Terraform-libvirt"
date: 2019-08-08T18:25:23+02:00
---

The first 00 unconventional series of Software design.

# Meta:

- `Project`: https://github.com/dmacvicar/terraform-provider-libvirt
- `Description`:Terraform provider to provision infrastructure with Linux's KVM using libvirt)
- `Language`: golang

I'm attempting to draw Software architecture in a unconventional way, because I think describing software with predifined models is somehow restrictive.
Here is an humble tentative:

# Analysis:

![design](/terraform-libvirt.jpeg)

We have 3 layers of abstraction: 

### 1)  Number 1 in the picture:  the data model and flow in highlevel manner:

  * terraform-libvirt take as `input` an HCL file, and transform it in `XML` format which will then consumed by Libvirt. The terraform-libvirt is in the middle as translator of both worlds.

  * A terraform plugin follows a strict protocol for resource lifecycle mgmt.  See right side, (specification)

  * In Libvirt every resource ( a domain, disk, network etc) is defined by XML ( see left side)

  Imagine an user calling `terraform apply`, the flow would be from the right to `the left of the image 
   (applying terraform protocols of provider-right, the terraform-libvirt  implement this interface and call libvirt functions).

 This flow can be bi-directional. When you create the first time a resource, it is from right to left. But when updating existing infrastructure it is from left to right (reading existing resources).

 Here is simplified because in more complex scenarios the provider is reading, creating resources at same time. So it switch from left to right or from right to left.


### 2) Second perspective (number 2 in the picture) the internal structure of the provider, layout:

  We start describing from the smallest piece of abstraction, and going further to the higher one.

  *  [Domain_def.go](https://github.com/dmacvicar/terraform-provider-libvirt/blob/master/libvirt/domain_def.go) is  where  XML Schema/Description of Resource is stored and manipulated.
     
  * The [Domain.go](https://github.com/dmacvicar/terraform-provider-libvirt/blob/master/libvirt/domain.go) contains function which will be called by the terraform create/destroy function etc.. The function are mostly using either libvirt-xml or libvirt go libraries 

  * Finally the [resource_libvirt_domain.go]((https://github.com/dmacvicar/terraform-provider-libvirt/blob/master/libvirt/resource_libvirt_domain.go) implement the Create/Destroy/Update etc functions which are the common protocol which a Terraform Provider need to respect and implement.

  The `provider.go` file encapsulate all resources registration, so it is the "last" abstraction. https://github.com/dmacvicar/terraform-provider-libvirt/blob/master/libvirt/provider.go#L10 

  Note: the Domain is just an example resource, the same reasoning apply for other resource like network etc.


### 3) The last picture, see the on the bottom, show a quick summary of the usage of terraform-libvirt:

  * An user specify the infrastructure in HCL file format. 
  --> 
  * In the same machine where the HCL specification lives, the user execute terraform operation. ( both pluging should be installed)
  -->
  * KVM/Libvirt Infrastructure is up and running

# Conclusion:

Definitely since I'm co-maintaining the project I have some implicit knowledge which it might be shadowed by the fact that I know already the project.

If you want to know more about the project, feel free to check the webpage https://github.com/dmacvicar/terraform-provider-libvirt.

Also for terraform and libvirt  there is lot of documentation online! 

See you the next series where I will be the detective of a Clojure project!
