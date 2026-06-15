---
title: SiO2 Patterning
url: https://sam.zeloof.xyz/sio2-patterning/
published: "2018-04-23T13:32:37Z"
feed: zeloof
guid: http://sam.zeloof.xyz/?p=1689
---

# SiO2 Patterning

In IC fabrication it is necessary to deposit/grow and etch insulating layers of Silicon Dioxide. This presents a few problems for standard photolithographic patterning because SiO2 is hydrophilic which can cause photoresist adhesion issues and also the HF etchant attacks most photoresists. These issues combine to leave you with poor pattern definition and often complete photoresist lifting during etch.

 [![](https://sam.zeloof.xyz/wp-content/uploads/2018/02/1active-300x169.png)](https://sam.zeloof.xyz/wp-content/uploads/2018/02/1active.png)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8200-e1524491593156-300x169.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8200-e1524491593156.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_4078-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_4078.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_4077-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_4077.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_4726-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_4726.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_4725-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_4725.jpg)

The steps I have found to mitigate these issues are (in order): **dehydration bake, HMDS vapor prime, thick resist coating, hard bake, and buffered oxide etch.**

 [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V7881-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V7881.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V7882-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V7882.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V7877-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V7877.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8176-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8176.jpg)

First, SiO2 is thermally grown on a test wafer using a water vapor source on a nearby hotplate to fill the furnace with steam during oxidation. The first step to ensure good resist adhesion is a dehydration bake which creates a hydrophobic wafer surface. This does not need to be done if the wafer recently came out of the furnace but if it has been in storage, then a bake of up to 700C may be necessary to restore the dehydrated surface. The next step is HMDS vapor priming:

[![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8283-1024x678.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8283.jpg) [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/FullSizeRender-8-e1524492825775-1024x682.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/FullSizeRender-8-e1524492825775.jpg)

Here, the wafer is heated to around 200C in the presence of Hexamethyldisilazane (HMDS) vapor forming a surface monolayer on the wafer that further increases resist adhesion. HMDS can also be spin coated but this often yields a far too thick layer and can lead to incomplete photoresist development.

The final steps before etch are to spin the resist and to hard bake it. Naturally, a thicker resist film allows for a longer etch time. For maximum chemical stability, the hard bake should be conducted for extended periods of time close to the resist softening point which is usually around 145C. This can make the photoresist difficult to remove, so an ultrasonic acetone bath may be necessary unless you have proper stripping chemicals. If difficulty persists, then it is likely that the top layer of resist has cross-linked and you may be unable to remove it. One may try high power Oxygen RIE followed by [Piranha solution](https://en.wikipedia.org/wiki/Piranha_solution) and N-Methylpyrrolidone (NMP) stripper as is used commercially to remove resists after hard ion implantation.

 [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/w_D7V8200-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/w_D7V8200.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8201-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8201.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8202-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8202.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8203-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8203.jpg)

Instead of a standard HF etch, a buffered oxide etch of NH4F (Ammonium Fluoride) in HF can be used to control the etch rate and photoresist lifting. I use approximately 20-30g of 100% NH4F per 50mL of HF (stock whink rust remover) and etch time for 6000Å SiO2 is 20min at 20C. A couple drops of Triton X-100 nonionic surfactant may be added to the BOE to improve etch uniformity, wetting, and ensure consistency through a thicker resist. A good BOE recipe can be found [here](http://www.northeastern.edu/wanunu/WebsiteMSDSandSOPs/Protocols/SOP_BOE2.htm) but assumes industrial-strength HF.

[![d](http://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8211-1024x678.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8211.jpg) Etch failures [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8204-1024x678.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8204.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8279-1024x678.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8279.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8206-1024x678.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/D7V8206.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3203-e1524492332810-1024x678.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3203-e1524492332810.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/THIN-HMDS-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/THIN-HMDS.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/THICK-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/THICK.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3482-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3482.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3483-1024x767.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3483.jpg)

[![IMG_3480](http://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3480-1024x768.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2018/04/IMG_3480.jpg)

Etch trials
