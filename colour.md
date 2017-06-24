### Colour

First, there was/is light.

Wait; first there is electromagnetic radiation, which is a fairly well defined physical phenomenon, and light is the _visible_ subset of that.
Or almost visible, in the case of ultraviolet and infrared "light".  Light (and all electromagnetic radiation) comes in wavelengths (see wikipedia).
Humans see wavelengths in the range 390 to 700 nm (on average; but not everyone sees the same).
Each wavelength of light is perceived as a _colour_ (see the rainbow).
But not every colour is a single wavelength.
Brown, for example, is normally considered a colour, but it is not a single wavelength of light.

Colour is _our perception of light_.

Our eyes contain three (very rarely four!) different types of _cone cells_, each with a different spectral sensitivity curve -
ie a curve mapping how sensitive the cell is to various wavelengths of light.
ie how strong of a signal or "measurement" that type of cell will send off to the brain for a given wavelength.
The brain combines these 3 measurements into a single "colour".

As you can imagine, not everyone has the exact same perception.
However, scientists have modeled the "average" person's wavelength response curves, as such:

![cone response](https://upload.wikimedia.org/wikipedia/commons/thumb/0/04/Cone-fundamentals-with-srgb-spectrum.svg/810px-Cone-fundamentals-with-srgb-spectrum.svg.png "Cone Response")

The curves are denoted as S,M,L for Short, Medium and Long wavelengths (scientists can be boring).
They roughly - very roughly - align with and peak near the colours red, green, and blue.
(Very roughly! The long wavelength curve ("red") actually peaks around greenish-yellow, for example.)
These cells create signals for our brain which then interprets it as "colour".
Note that the sensitivity curves overlap - a wavelength of 460mn (blue) is measured by all three types of cone cells,
but the short wavelength ("blue") cones are most sensitive in this range and supply (to the brain) the strongest signal.
Note that around 485nm (blueish-green) all 3 cone types are almost equally sensitive!

So a single wavelength will produce 3 measurements for the brain to combine to interpret as a colour.
2 simultaneous wavelengths would produce 2 values _per curve_ (ie 6 "measurements")
but the values per curve are added together to form just 3 measurements for the brain.
100 wavelengths combined will also produce just 3 measurements for the brain to interpret as colour.
Thus different combinations of wavelengths can form the same _tristimulus_ signals,
and thus the same perceived colours in the brain.
So different combinations of light appear to us as the same colour!
This is called _metamerism_. (It seems "sad" that our eyes trick us this way, but in fact we can use it to our advantage... later.)

If we could record these three measurements from the eye, we could write down a "colour".
For example, 3 numbers, from 0 to 1, representing the response/signal/measure that three cone types in the eye send to the brain.
3 numbers, thus, mathematically, a point in Cartesian 3 space.
Thus a "colour space".
This is the basis of the LMS colour space (LMS is long/medium/short, I don't know why the order was reversed!) -
3 numbers, from 0 to 1, that represent any colour that the average person can perceive.
Oh, but wait. In LMS a colour like (0,0,1) for example, CANNOT happen (typically) -
the L and M curves almost completely overlap the S curve, so if a strong S signal was measured,
L and M values should have been recorded as well.
So LMS is a colour space, but not a perfect or best mathematical colour space.

There are other colour spaces, each with varying good and bad properties, each useful for different purposes.
One important colour space is CIE's XYZ colour space.
Among other useful properties, CIE XYZ was designed such that Y maps to the luminance ("brightness") of a colour.
CIE XYZ is the "standard" colour space, that covers all perceived colours and all other colour spaces can be mapped to and from.

Now, as mentioned above, due to metamerism
(our eyes combining all wavelengths into 3 measures and thus different light combinations being perceived as the same colour),
we can use 3 wavelengths, typically a red, a green, and a blue (that closely match our tristimulus curves),
to stimulate our cones such as build whichever LMS eye-to-brain signals we'd like.
Thus using just red, green, and blue light, we can form all (well, most of) the colours we perceive.

So RGB is also a useful colour space.  However, be careful, which "red" "green" "blue" are we talking about?
And is it a single wavelength of red (like a red laser) or a range of wavelengths?
This is why RGB on one monitor does not match the RGB on another.
So to be specific, there is the sRGB colour space (s for 'standard'), which is precisely defined.

// To be continued
