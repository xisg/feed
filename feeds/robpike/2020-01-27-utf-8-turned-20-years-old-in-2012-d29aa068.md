---
title: UTF-8 turned 20 years old (in 2012)
url: https://commandcenter.blogspot.com/2020/01/utf-8-turned-20-years-old-in-2012.html
published: "2020-01-27T05:20:00Z"
feed: robpike
guid: tag:blogger.com,1999:blog-6983287.post-4319675480500536866
---

# UTF-8 turned 20 years old (in 2012)

(Another resurrected post from the my old Google+ stream, which appeared on or about Sep 4, 2012. Thirty years of age isn't too far off now.)

UTF-8 turned 20 years old yesterday.

It's been well documented elsewhere ( [http://www.cl.cam.ac.uk/~mgk25/ucs/utf-8-history.txt](http://www.cl.cam.ac.uk/~mgk25/ucs/utf-8-history.txt)) that one Wednesday night, after a phone call from X/Open, Ken Thompson and I were sitting in a New Jersey diner talking about how best to represent Unicode as a byte stream. Given the experience we had accumulated dealing with the original UTF, which had many problems, we knew what we wanted. X/Open had offered us a deal: implement something better than their proposal, called FSS/UTF (File System Safe UTF; the name tells you something on its own), and do so before Monday. In return, they'd push it as existing practice.

UTF was awful. It had modulo-192 arithmetic, if I remember correctly, and was all but impossible to implement efficiently on old SPARCs with no divide hardware. Strings like "/\*" could appear in the middle of a Cyrillic character, making your Russian text start a C comment. And more. It simply wasn't practical as an encoding: think what happens to that slash byte inside a Unix file name.

FSS/UTF addressed that problem, which was great. Big improvement though it was, however, FSS/UTF was more intricate than we liked and lacked one property we insisted on: If a byte is corrupted, it should be possible to re-synch the encoded stream without losing more than one character. When we claimed we wanted that property, and sensed we could press for a chance to design something _right_, X/Open gave us the green light to try.

The diner was the Corner Café in New Providence, New Jersey. We just called it Mom's, to honor the previous proprietor. I don't know if it's still the same, but we went there for dinner often, it being the closest place to the Murray Hill offices. Being a proper diner, it had paper placemats, and it was on one of those placemats that Ken sketched out the bit-packing for UTF-8. It was so easy once we saw it that there was no reason to keep the placemat for notes, and we left it behind. Or maybe we did bring it back to the lab; I'm not sure. But it's gone now.

I'll always regret that.

But that's my only regret in this story. UTF-8 has made the world a better place and I am delighted to have been a facilitator in its creation.

So tonight, please give a toast to a few quickly sketched boxes on a placemat in a New Jersey diner that today define how humans represent their text to be sent across international boundaries ( [http://googleblog.blogspot.com/2012/02/unicode-over-60-percent-of-web.html](http://googleblog.blogspot.com/2012/02/unicode-over-60-percent-of-web.html)), and especially to the X/Open folks for giving us the chance and the Unicode consortium and IETF for pushing it so successfully.

Signed,

-rob

U+263A '☺' white smiling face
