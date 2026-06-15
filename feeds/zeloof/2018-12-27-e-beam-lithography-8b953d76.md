---
title: E-beam Lithography
url: https://sam.zeloof.xyz/e-beam-lithography/
published: "2018-12-27T13:20:39Z"
feed: zeloof
guid: http://sam.zeloof.xyz/?p=1899
---

# E-beam Lithography

I modified my [JEOL](http://sam.zeloof.xyz/i-got-an-electron-microscope-jeol-jsm-6300-sem/) scanning electron microscope to not only image tiny things, but make tiny things too.

[![DSC_8877](http://sam.zeloof.xyz/wp-content/uploads/2018/12/DSC_8877-1024x670.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2018/12/DSC_8877.jpg)

This technique is called Electron Beam Lithography. Normally, a SEM works by scanning a beam of electrons over the sample and detecting secondary electrons. In this case, I built a PC scan controller that “drives” the electron beam around and draws the image onto the sample which is coated with electron-sensitive resist.

[![Schematic-illustration-of-electron-beam-lithography-Electron-beam-is-focused-on-a-resist](http://sam.zeloof.xyz/wp-content/uploads/2018/12/Schematic-illustration-of-electron-beam-lithography-Electron-beam-is-focused-on-a-resist.png)](http://sam.zeloof.xyz/wp-content/uploads/2018/12/Schematic-illustration-of-electron-beam-lithography-Electron-beam-is-focused-on-a-resist.png) For resist, I used SU-8 although PMMA (Acrylic) dissolved in solvent will work as well.

The scan controller uses dual 12 bit DACs. They had current outputs so a transimpedance amplifier creates +/-10v proportional to that current to drive the microscope’s external XY inputs. The controller also contains an Arduino and Altera CPLD. To turn the beam off when needed, a high voltage is generated (+2kv) and applied to a “beam blanker” inside the microscope.

 [![](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_3734-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_3734.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_3736-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_3736.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_5350-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_5350.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_5351-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_5351.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_5320-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_5320.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/12/D7V8233-e1545917213293-1024x756.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/12/D7V8233-e1545917213293.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_5313-1024x1024.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_5313.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_5316-1024x1024.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_5316.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_5379-1024x1024.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_5379.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_3258-e1545917287574-1024x754.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_3258-e1545917287574.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_3261-1024x753.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_3261.jpg)

I learned that the 12 bit DACs do not nearly have enough resolution, so I will be looking at upgrading to at least 18 bits hopefully. Also, the geometry of the beam blanker is not correct to create a high electric field in its center, so the beam is only partially de-focused and never fully “turns off”.

Update (2020): New photos with improved beam blanker –

 [![](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_0019-e1591631057397-300x300.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_0019-e1591631057397.jpg) [![](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_9994-e1591631042344-300x265.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2018/12/IMG_9994-e1591631042344.jpg)
