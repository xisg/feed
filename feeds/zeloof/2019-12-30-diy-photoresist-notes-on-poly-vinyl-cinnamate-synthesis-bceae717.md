---
title: DIY Photoresist – Notes on Poly(vinyl cinnamate) synthesis
url: https://sam.zeloof.xyz/diy-pr/
published: "2019-12-30T02:41:38Z"
feed: zeloof
guid: http://sam.zeloof.xyz/?p=2139
---

# DIY Photoresist – Notes on Poly(vinyl cinnamate) synthesis

**\*\*\*DISCLAIMER\*\*\*** this project is incomplete and the write-up is intended only to save someone a little  time who is attempting to explore this topic on their own. I only spent about 2 weeks working on this during winter break from school and also I did very poorly in the organic chem unit in high school which is the extent of my chemistry knowledge. Major YMMV on this one

* * *

[![IMG_8817](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8817-e1578195267113-300x219.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8817-e1578195267113.jpg)

## **Why DIY Photoresist?**

1) commercial photoresist is expensive – modern photoresist can be upwards of $8,000 per gallon. And you thought gasoline prices were bad?

2) commercial photoresist is hard to get – even if you can afford it, no [actual photoresist supplier](https://www.microchemicals.com/support/photolithografy_book_2017.html) will ship (even samples) to an individual or residential address.

So, unless you make friends in industry/academic lab so that your refrigerator looks like mine then it may be impossible for you to do photolithography at home. The good news, however, for making chips at home is that photoresist is the only major chemical that is difficult to source. Everything else can be found in the local supermarket or on sites like eBay/Amazon (rust stain remover HF, boric acid roach killer, etc.) This means that if a reliable synthesis method for photoresist is found, making ICs at home will become one step more accessible.

[![My fridge](http://sam.zeloof.xyz/wp-content/uploads/2019/12/IMG_8618-300x225.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2019/12/IMG_8618.jpg) My fridge

* * *

## Plan of Attack

In the 1950s, [Dichromated gelatin](https://holowiki.org/wiki/Dichromated_Gelatin) was a contender for the first “photoresist” for early microelectronics, but it couldn’t withstand more than ~1% HF etching solution. Also, it had a “dark reaction” in which it would decompose readily without use. So, early IC pioneers at Bell Labs teamed up with the smart people at The Eastman Kodak Company and came up with poly(vinyl cinnamate), an alternative which has no “dark reaction”, excellent chemical stability, and relative ease of synthesis. Also, most of the chemistry smells very good for this because it is based on cinnamon-like compounds. However, poly(vinyl cinnamate) was quickly replaced by cyclized rubber resists because of its superior adhesion to Silicon. For this project, I have chosen to try synthesizing poly(vinyl cinnamate) because it is the easiest and requires the least precursors with above a 3 on the NFPA health diamond. Poly(vinyl cinnamate) is a negative acting resist in which the exposed regions become crosslinked an are no longer soluble in the developer solution.

[![IMG_8748](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8748-300x225.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8748.jpg)

For the synthesis, I used <$50 of glassware from Amazon including a Buchner funnel, filtering flask, and assorted small beakers. A hotplate with stirrer is nice. All of the following chemistry should be done in subdued room light, without any natural light from the Sun. Most interior LED lighting will not expose these materials for a matter of hours and are safe to work under, it definitely does not need to be carried out in darkness.

**Supplemental readings on Poly(vinyl cinnamate) synthesis:**

[Photochemical properties of cinnamic acid epoxy esters](http://sam.zeloof.xyz/wp-content/uploads/2019/12/sierocka1984.pdf)

[Original Kodak patent on its synthesis and use](http://sam.zeloof.xyz/wp-content/uploads/2019/12/US2690966.pdf)

 [![](https://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8919-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8919.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8918-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8918.jpg)

[![IMG_8925](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8925-1024x768.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8925.jpg)

Directly after spin coating, all samples are soft baked at 50ºC on a hotplate for 1 minute. Then, the poly(vinyl cinnamate) films were exposed using an unfiltered Mercury vapor arc lamp (sunlight works too!) in the form of a Loctite UV epoxy curing lamp. For order-of-magnitude exposure dosage calculations, this lamp has an approximate intensity of ~80mW/cm^2 at the 365nm emission line. Proper poly(vinyl cinnamate) films can be exposed in a matter of seconds with this. To yield a visible image, the samples are developed in MEK for about 1 minute which concludes the photolithography process.

* * *

## Synthesis

I explored three main routes to get to our desired end product of poly(vinyl cinnamate) solution. The first starts with simple and readily available chemicals which be sourced wholly from tier 1 suppliers (see appendix A at the bottom for definition of chem. supplier tiers) like Amazon and eBay. I had encouraging but sub-optimal results with this method, this is certainly more work to be done. The second method starts with vinyl cinnamate monomer and attempts to polymerize it, which yields better results but requires tier 2 or 3 chemical supplier account access. This means commercial shipping addresses. And the third method relies on directly purchasing poly(vinyl cinnamate) powder from a tier 2 or 3 source (this is guaranteed to have the best results) and adding it to a solvent solution. Methods 2 and 3 gave basically the same results and showed the most promise for the least amount of work.

#### Method 1:

Here we attempt an esterification reaction between substituted cinnamic acid derivatives and poly(vinyl alcohol) in a pyridine or dimethylformamide solution. The synthesis is given in the [original Kodak patent](http://sam.zeloof.xyz/wp-content/uploads/2019/12/US2690966.pdf) and is as follows:

[![synthesis](http://sam.zeloof.xyz/wp-content/uploads/2019/12/synthesis-300x248.png)](http://sam.zeloof.xyz/wp-content/uploads/2019/12/synthesis.png)

For testing, I used all the same proportions of materials at about a 10x reduction in size (1g of PVA, etc.). It is unclear if the pyridine is actually required here, I used dimethylformamide from Amazon but in retrospect I probably should have waited the few extra days it takes to get pyridine from eBay for the solvent to eliminate a variable in the reaction.

 [![](https://sam.zeloof.xyz/wp-content/uploads/2019/12/IMG_8712-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/12/IMG_8712.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8805-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8805.jpg)

This reaction uses a substituted cinnamic acid derivative such as cinnamoyl chloride or 4-Nitrocinnamic acid, of which I had neither. The cinnamoyl chloride can be purchased on Alibaba (or synthesized from nasty things like thionyl chloride) and I also tried making 4-Nitrocinnamic acid via nitration of cinnamaldehyde (nitrate salt and sulfuric acid) and subsiquent oxidation to yield the acid but was left with a brown sludge that didn’t have the correct properties. Thus, I attempted the synthesis with other cinnam- compounds, listed below. I would definitely recommend getting the actual cinnamoyl chloride, although it is very moisture sensitive so care must be taken during the reaction.

1) Cinnamic acid as precursor

2) Cinnamaldehyde as precursor

3) Cinnamon bark essential oil as precursor

4) Cinnamyl alcohol as precursor

After trying these 4, I noted that 1, 2, and 3 all had similar results. The “photoresist” they produced had incorrect properties so it should not be called poly(vinyl cinnamate) but is likely some other unknown photo-sensitive compound with about 100x lower light sensitivity than proper poly(vinyl cinnamate) (required ~1hr exposure). However, these films did provide a “latent” image in the resist after exposure but would have little to no differential solubility in the developer chemical, i.e. the exposed and non-exposed regions would fade away at the same rate. Chemical 4 as the precursor had slightly better results during developing, but still did not have the required characteristics to be called poly(vinyl cinnamate). This synthesis should be much more successful with actual cinnamoyl chloride. It was also noted in the patent that certain dyes can be added to the solution to increase its sensitively. I added crystal violet from Amazon in concentrations from 1 – 10% wt. and noted to change here.

Exposure is carried out using a perf board as a quick and easy mask

[![IMG_8717](http://sam.zeloof.xyz/wp-content/uploads/2019/12/IMG_8717-1024x768.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2019/12/IMG_8717.jpg)

Without using a spectrometer, it should be fairly easy to identify a proper poly(vinyl cinnamate) sample. First, it should be a white to yellow-white powder and insoluble in water and moderately soluble in warm alcohols. Also, it should have an exceedingly high melting point (higher than any of the chemicals used to synthesize it) above 275ºC. None of my results from method 1 pass all of these tests, however the resists from method 2 do.

 [![](https://sam.zeloof.xyz/wp-content/uploads/2019/12/IMG_8718-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/12/IMG_8718.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2019/12/IMG_8699-1024x768.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2019/12/IMG_8699.jpg)

#### Method 2:

Here we attempt polymerization of vinyl cinnamate monomer to yield poly(vinyl cinnamate).

[![IMG_8869](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8869-300x225.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8869.jpg)

The required vinyl cinnamate can be purchased from most of the tier 2 or 3 chemical suppliers listed in appendix A and should be kept refrigerated.

Similar polymerizations are usually carried out on a lab scale by heating the monomer in a solvent in the presence of an initiator, usually azobisisobutyronitrile. I couldn’t get my hands on any of this but read that Methyl ethyl ketone peroxide (MEKP) can be used as a polymerization initiator. This can be synthesized from MEK (buy anywhere, eBay or Home Depot) and 50% Hydrogen peroxide. I decided to use Bondo brand “fiberglass resin liquid hardener” which can be bought in home improvement stores (or search for “NOROX 925” on Amazon). MEKP is a high explosive, so these products usually also contain 50% dimethyl phthalate to reduce MEKP’s shock sensitivity and should not play a large role in our reaction.

[![IMG_8872](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8872-300x225.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8872.jpg)

Theoretically, only about 2 – 5% by mass of the MEKP should be needed in a a 20% solution of vinyl cinnamate and solvent (dimethylformamide from Amazon) for complete polymerization of the monomer after about 6 hours at 80ºC. Then, the solution should be added to cold water with lots of stirring to precipitate the desired polymer. I tried this and obtained a milky white solution and very little white precipitate which is presumably our poly(vinyl cinnamate).

[![I have seen this described as "evil yield" in old papers](http://sam.zeloof.xyz/wp-content/uploads/2019/12/IMG_8636-300x225.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2019/12/IMG_8636.jpg) I have seen this described as “evil yield” in old papers

My procedure which actually yielded results is same as above (in a capped amber bottle on hot plate) but with no solvent and ~20% or higher fiberglass resin hardener. This is a huge excess of MEKP and it still did not precipitate an appreciable amount of polymer in cold water or methanol but I was able to filter it to a slightly yellow viscous soup through a coffee filter (gravity, ~5min). This viscous soup dried as a hard solid and did not have the expected physical properties (not a white powder) but had an appropriate melting point of over 250ºC (polymer has much higher melting point than monomer) and correct solubility properties (not soluble in water and slightly soluble in warm DMF, acetone, toluene, etc.). After dissolving the product in one of the spin coating solvents mentioned in appendix B to achieve about a 2.5% solution, I exposed it with UV and got promising results with the same edge definition and UV sensitivity as method 3, further indicating that we have actually synthesized some of the polymer (polymer is MUCH more photo-sensitive than the monomer or any of the cinnamic acid derivatives because it can cross-link effectively). The exposed film is developed in MEK for about 1 minute.

[![IMG_8921](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8921-1024x768.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8921.jpg)

#### Method 3:

Here we directly use poly(vinyl cinnamate) powder as the photoresist.

The required poly(vinyl cinnamate)can be purchased from most of the tier 2 or 3 chemical suppliers listed in appendix A.

[![IMG_8870](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8870-1024x768.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8870.jpg)

I prepared a 2.5% solution of the purchased poly(vinyl cinnamate) in spin coating solvent mixture #2 listed in appendix B. The solution was mixed until all the solids had dissolved, filtered and spun onto the wafer, baked at 50ºC, exposed, and developed just as all the above trials. The results were similar to those of method 2 and yielded clearly visible patterns in the developed film which had poor definition and definitely needs some improvement to the process before it can be used to manufacture ICs. However, I did an etch trial in 2% HF solution and the resist held up to the acid (very difficult task, most modern photoresists cannot) which was promising for home IC manufacture.

[![IMG_8828](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8828-1024x887.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8828.jpg)

* * *

## Appendix A: Notes on Chemical Sourcing

Sourcing the required chemicals for an amateur project is usually half the battle. Making friends in industry or in an academic lab is always an easy way out but is not reproducible for other amateurs so the goal should be to synthesis these photosensitive compounds from chemicals which are readily available to any individual and can be shipped to a residential address.

Sigma Aldrich is everyone’s worst nightmare, it shows up as the #1 result for [almost any chemical](https://www.sigmaaldrich.com/catalog/product/cerillian/l001?lang=en&region=US) just to taunt us hobbyists who do not have the required commercial shipping address, DUNS Number, Federal Tax ID, etc. Sigma is especially mean to us this time, as you can see below they sell our end product, ready to go:

[![Screen Shot 2020-01-05 at 10.40.44 PM](http://sam.zeloof.xyz/wp-content/uploads/2019/12/Screen-Shot-2020-01-05-at-10.40.44-PM-1024x584.png)](http://sam.zeloof.xyz/wp-content/uploads/2019/12/Screen-Shot-2020-01-05-at-10.40.44-PM.png)

For this project, I will define three tiers of chemical suppliers. It is our goal to make the poly(vinyl cinnamate) solely from tier 1 suppliers, which seems to be possible.

**Tier 1:** Websites such as Amazon, eBay, and Alibaba that will, with rare exception, ship their items to just about anyone. No commercial shipping address required. They are not reliable, however, and depend on what sellers are currently active. For example, years ago you were able to purchase concentrated Sulfuric acid on Amazon, but now you cannot.

**Tier 2:** Websites such as [Santa Cruz Biotechnology](https://www.scbt.com/home), [Polysciences](https://www.polysciences.com/default/), and [Ak Scientific](https://aksci.com/) that have a superset of the items sold on tier 1 sites and will ship to anyone, no questions asked, as long as there is a commercial shipping address. In most cases, an amateur scientist can make this happen in at least one way such as shipping to their employer or having one of their friends order it to work. For this project, 100% of the required precursors can reliably be sourced from tier 2 sites.

**Tier 3:** Websites such as Sigma Aldrich, Millipore Sigma, Alfa Aesar Thermo Fisher Scientific which have every substance known to humankind. These are likely run by cruel wealthy people who like to tease amateur scientists. If you have an account on any of these websites, please consider starting a black market website or forum for hobbyists.

[![](https://sam.zeloof.xyz/wp-content/uploads/2019/12/Screen-Shot-2020-01-05-at-10.57.22-PM-300x157.png)](https://sam.zeloof.xyz/wp-content/uploads/2019/12/Screen-Shot-2020-01-05-at-10.57.22-PM.png)[![](https://sam.zeloof.xyz/wp-content/uploads/2019/12/Screen-Shot-2020-01-05-at-10.57.44-PM-300x133.png)](https://sam.zeloof.xyz/wp-content/uploads/2019/12/Screen-Shot-2020-01-05-at-10.57.44-PM.png)

??? thank you Alibaba / Vicky Fan ???, imagine if this website existed in the 50s when they were working out early photoresists

* * *

## Appendix B: Notes on Spin Coating

Spin coating homemade materials can be very difficult, there are many parameters such as viscosity, solvent evaporation rate, spin RPM max, and acceleration that can combine to give poor films. I use a PC fan with reduced input voltage to spin ~3500RPM and double sided tape to hold the wafer fragment. Commercial photoresists are guaranteed to give nice films for some acceleration profile and max speed on the spinner, but we do not have such a guaranteed for homemade materials. At worst, these homemade resits can completely fly off the wafer during coating and leave no film behind.

 [![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/spin-on-diffusant-interference-patterns-4-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2016/12/spin-on-diffusant-interference-patterns-4.jpg)  [![](https://sam.zeloof.xyz/wp-content/uploads/2016/12/spin-on-diffusant-interference-patterns-3-300x199.jpg)](https://sam.zeloof.xyz/wp-content/uploads/2016/12/spin-on-diffusant-interference-patterns-3.jpg)

One tip to combat this is to change the surface energy states of the wafer to make it either hydrophilic or hydrophobic. Commercial photoresists are best coated on a hydrophobic wafer (may be counter-intuitive) which can be accomplished via thorough cleaning and a dehydration bake.  Other materials are best coated on a hydrophilic wafer which can be achieved by a quick dip in [piranha solution](https://en.wikipedia.org/wiki/Piranha_solution). You may need to do some experimentation. In our case, the photosensitive polymer is a solid so we need to suspend it in some solvent for coating. I have found that the following solvents work quite nicely, in descending order of performance:

1) Propylene glycol methyl ether acetate (PGMEA)

2) Butyl acetate

3) 40% Xylene, 40% Toluene, 20% Isopropyl alcohol solution

4) Trichloromethane

One final point on spin coating these homemade materials is the need to filter resist before coating to remove particulate. These will wreak havoc on the spin coating process and leave you with terrible films full of comet-like shapes.  To eliminate the possibility of this headache, I recommend filtering all materials through a .45µm syringe filter prior to coating (can be found on Amazon) or at least a coffee filter. Below is a comparison showing different filter methods and the film they produce of homemade resist.

[![IMG_8889](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8889-1024x768.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8889.jpg)

From left to right: unfiltered, coffee filter gravity filtration, coffee filter + subsequent .45µm filter.

If you made it this far, thanks for reading about my failures and mild successes in making photoresist. I hope it was interesting and that you may learn from it, let me know if you have questions or comments!

[![IMG_8891](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8891-1024x768.jpg)](http://sam.zeloof.xyz/wp-content/uploads/2020/01/IMG_8891.jpg)
