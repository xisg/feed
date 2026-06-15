---
title: Microelectromechanical Systems (MEMS) Fabrication
url: https://sam.zeloof.xyz/microelectromechanical-systems-mems/
published: "2019-08-22T13:16:28Z"
feed: zeloof
guid: http://sam.zeloof.xyz/?p=2062
---

# Microelectromechanical Systems (MEMS) Fabrication

_Homemade Silicon MEMS_

The term MEMS was first coined in the 1980s and has since brought tons of innovative/insane fabrication techniques and incredible devices. Among my favorite examples of this is the DLP technique invented by TI and a DNA nanoinjector for cells made at BYU (pictured below). Obviously, I won’t be making anything this complicated but had some decent results on my first attempts that are worth sharing and continuing to work on.

[![image-asset](http://sam.zeloof.xyz/wp-content/uploads/2019/08/image-asset-1024x772.jpeg)](http://sam.zeloof.xyz/wp-content/uploads/2019/08/image-asset.jpeg)

All of my MEMS devices are fabricated with chemical etching and [maskless photolithogrpahy](http://sam.zeloof.xyz/maskless-photolithography/). The most basic free-standing mechanical device is the cantilever, which can be made using a sacrificial underlayer made of oxide or photoresist (SiO2 in my case) and a harder upper layer (SU-8 epoxy resist in my case) which remains when the sacrificial layer is removed with chemical etching (HF for SiO2). These can be made into oscillators, nano balances, any many other useful instruments.

[![IMG_7564](http://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7564.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7564.jpg)

There are many variables here to optimize, so it is best to run many experiments at once.

 [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7264-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7264.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7270-e1566480609951-300x230.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7270-e1566480609951.jpg)

The next MEMS technique to demonstrate is bulk Silicon Micromachining which leverages the high level of order in a monocrystalline Silicon wafers to make controlled and often self-terminating etches. On a standard <100> orientation wafer, a solution of KOH (KOH pellets from amazon/ebay dissolved in water) can anisotropically etch Silicon to expose the <111> crystal face at at an angle of 54.7º from the wafer surface.

 [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7566-e1566481122353.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7566-e1566481122353.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7565.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7565.jpg)

The setup for this is quite simple, 60% KOH solution with a dash of isopropyl alcohol is brought to 80°C in a hotplate with stirring and etches about 90um in 1hr, or about one-third the thickness of a Silicon wafer. Nature’s fume hood is used for this experiment.

 [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/DSC_8766-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/DSC_8766.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/DSC_8772-300x198.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/DSC_8772.jpg)

A top layer of SiO2 is used as a hard mask to etch the underlaying Silicon. Before KOH etching, the SiO2 is thermally grown and [patterned in a standard process by maskless photolithography and HF etching](http://sam.zeloof.xyz/sio2-patterning/). The active layer mask for my [Z1 amplifier chip](http://sam.zeloof.xyz/first-ic/) was used here as a quick and convenient anisotropic etch test. Wafers are then inspected under [SEM](http://sam.zeloof.xyz/category/electron-microscope/).

 [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7567-1024x819.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7567.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/KOH_1-1024x819.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/KOH_1.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/KOH_4-1024x819.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/KOH_4.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/KOH_6-1024x819.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/KOH_6.jpg)

Suspended cantilever-like structures can also be made here. The first KOH etching attempt shows some undercutting, but in the future stirring conditions will be optimized to provide more undercutting. The circled areas in the high-tilt angle SEM image below show 500nm thick SiO2 overhanging 90um of airgap above the Silicon surface.

 [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/KOH_1-1.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/KOH_1-1.jpg) [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7568.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7568.jpg)
