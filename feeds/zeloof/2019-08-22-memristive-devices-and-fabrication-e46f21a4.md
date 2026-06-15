---
title: Memristive Devices and Fabrication
url: https://sam.zeloof.xyz/memristive-devices/
published: "2019-08-22T02:59:21Z"
feed: zeloof
guid: http://sam.zeloof.xyz/?p=2041
---

# Memristive Devices and Fabrication

_Homemade Memristors_

[Memristors](https://en.wikipedia.org/wiki/Memristor) are interesting devices first posed by Leon Chua in 1971 as the 4th basic circuit element, after the resistor, capacitor, and inductor. It would theoretically satisfy a relationship between charge and flux, and in simple terms is a 2-terminal device whose instantaneous resistance depends on the device’s electrical past (total charge or current that has flowed through the device). The possibility for existence of a memristor in our universe is [highly debated](https://arxiv.org/abs/1207.7319) and claims of “discovering” the missing memristor were circulated by HP Labs in 2008. The resistance-memory effect they noted was met with huge amounts of press and recognition, but in reality it was [first noted in the late 1960s](https://www.sciencedirect.com/science/article/pii/0038110168900920?via%3Dihub) in similar materials and was not valid under the original mathematical definitions for a memristor. Since then, the definition and requirements for a memristor have been relaxed by some and even Leon Chua himself has recognized HP’s memristor. As with most politics and similar situations in academia, you should read about this on your own and come to your own conclusions, but it is safest to describe such devices with a resistance-memory as “memristive devices” or “memristive systems”, because while they may not adhere to the original definition of a memristor, they are still worth exploring and have proved to be useful for machine learning and artificial neuron applications. Interestingly, an ideal memristor fits as a perfect electrical model for Axons/synapses.

 [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/The-three-basic-electrical-elements-and-their-memory-counterparts-memristor-e1566443215983-300x171.png)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/The-three-basic-electrical-elements-and-their-memory-counterparts-memristor-e1566443215983.png)  [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/Construction-of-Memristor-e1566443255509-300x174.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/Construction-of-Memristor-e1566443255509.jpg)

Most of these memristors are made with some metal oxide stack with top and bottom electrodes and rely on the electrical current moving oxygen vacancies within the metal oxide to create a sort of hysteresis in resistivity. The results in a unique pinched-figure-eight hysteresis type IV curve whose loop area decreases with increasing frequency:

![](https://ars.els-cdn.com/content/image/1-s2.0-S0026269216303779-gr4.jpg)

Basic memristive devices can also be made easily at home out of Copper Sulfates, as described [here](http://sparkbangbuzz.com/memristor/memristor.htm) and [here](https://www.youtube.com/watch?v=MlswP_qXbdA). Of course, it is advantageous for lots of reasons to be able to manufacture these on a parallelized, industry standard CMOS-type fab process, where thousands of these devices can be made at once using photolithography.

About 2 years ago when I was setting up my sputtering chamber, none of the coatings were conductive. This was frustrating but eventually I figured out the films were metal oxides, not pure conductive metal coatings, due to too much Oxygen being present during the sputtering. Recently, I read that memristive devices can be made out of materials such as Titanium and Copper oxides, so I dug out some old failed sputtering attempts of these materials, put them back in a chamber to deposit a top Copper electrode, and measured their IV characteristics.

 [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7559-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7559.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7560-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7560.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7561-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7561.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7558-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7558.jpg)

Both Titanium and Copper oxide based devices worked as memristive systems and showed similar characteristics. The bottom electrode in both cases was doped Silicon which causes the devices to be slightly rectifying (asymmetry in curve) and highly sensitive to light (built-in PN junction) but they still show a resistance-memory hysteresis effect. In the future, more devices will be built on insulating substrates and should yield much better results. Rectifying memristors made on Si substrates have applications in [high density 3D array stacks](https://www.nature.com/articles/ncomms15666), even though they deviate highly from an “ideal” memristor.

 [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7546-300x225.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7546.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7557-300x229.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7557.jpg)

Figure (a) above shows a typical curve from one of my devices and figure (b) shows a curve from a [paper published studying similar metal oxide-based memristors.](https://avs.scitation.org/doi/abs/10.1116/1.4828701?journalCode=jva) Scale on fig. (a) is 100uA and 5V per division.

[![IMG_7546](http://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7546-1-1024x669.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2019/08/IMG_7546-1.jpg)

Here, differences in slope (1 / resistance) for the on (blue) and off (red) regions can clearly be seen, as well as the switching region (yellow).

As previously mentioned, these devices are highly light-sensitive due to the doped Si substrate and a have very strange response:

[http://sam.zeloof.xyz/wp-content/uploads/2019/08/media.io\_IMG\_7529.m4v](http://sam.zeloof.xyz/wp-content/uploads/2019/08/media.io_IMG_7529.m4v)

After these first encouraging results, I made [Copper Sulfide based devices](http://sparkbangbuzz.com/memristor/memristor.htm) that actually exhibit more ideal characteristics and can be made for a few dollars. Excess pure Sulfur powder from Amazon is dissolved in Ethanol or Isopropyl alcohol and a bare Copper PCB was placed in solution for 12 hours. The experiment was in a closed vessel to avoid evaporation and a black CuₓSᵧ film was noted. Then, the device was washed in alcohol for 3 hours to remove any residual Sulfur particles before being probed with a function generator and oscilloscope wired as a curve tracer.

Homemade CuₓSᵧ memristors:

[![memristor curve](http://sam.zeloof.xyz/wp-content/uploads/2019/08/memristor-curve.png)](http://sam.zeloof.xyz/wp-content/uploads/2019/08/memristor-curve.png)

A nice figure-eight is seen to pass through the origin and have a loop area which is decreasing with increasing frequency, as expected by the equations that govern a memristor (time integral of charge or current).
