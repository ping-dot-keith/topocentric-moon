# Topocentric Moon Monthly Calendar

Compile using...

cc tmoon.c -lm -o tmoon

## Quick start

<pre>
  Usage: tmoon date[yyyymm] timz[+/-h.hh] long[+/-dddmm] lat[+/-ddmm]
example: tmoon 200009 0 -00155 5230
</pre>

## How to use in detail

The program takes a command line with four purely numerical
arguments,

- the year and month in the format yyyymm,
- your time zone as a decimal number of hours before or after
  Greenwich, before Greenwich taken as negative
- your longitude as a signed integer in the format +/-dddmm with
  West longitudes taken as negative,
- your latitude as a signed integer in the format +/-ddmm with
  North latitudes taken as positive.

For example, the command line

$ tmoon 200009 0 -00155 5230 > sept.txt

will calculate a monthly Moon calendar for September 2000, in
time zone 0 (UT) for longitude 1 deg 55 minutes West, and 52 deg
30 minutes North. In this example, the symbol '>' tells the
operating system to 're-direct' the program output to a file
called 'sept.txt'. Redirection of output depends on the compiler
and operating system you are using - most support this feature.
Error messages are printed to the console using fprintf(stderr,
stuff).

## Background

This program calculates the approximate values of various numbers
of interest to lunar observers for a month at a time. I have
written the program using 'standard' C code and libraries. See 
'references' for the sources of the formulas.

For each day in the month, the program calculates the time of
Moon transit, and then calculates for that time the optical
libration of the Moon in selenographical longitude and latitude,
the position angle of the Moon's rotation axis, the selenographic
colongitude and latitude of the sub-Solar point, i.e. the point
on the Moon's surface where the Sun is overhead, the position
angle of the bright limb of the Moon (just add and subtract 90
degrees from this to get the position angles of the 'horns' of
the crescent), and finally the percentage illumination of the
Moon. The program also adds a character after the transit time
indicating the state of the Sun, blank for night, 'a', 'n', 'c'
for astronomical, nautical or civil twilight, and '*' if the Sun
is above the horizon at the time of lunar transit. A further
character is printed after the colongitude, an 'r' shows that the
Sun is rising on the Moon, and an 's' indicates a waning Moon.

Topocentric positions are calculated to your position on the Earth.
Geocentric positions are calculated to the centre of the Earth. There
is an appreciable difference for objects that are near to us, so that
your displacement from the centre of the Earth changes your angle to
the object by a significant amount.

The quantities here are calculated using topocentric positions for the
Moon and geocentric positions for the Sun, so that the resulting
values should represent the Moon's face that you will see in your
eyepiece fairly well. As the Moon is roughly 60 Earth radii away,
the topocentric correction has a large effect on the Moon's
position in your sky (sometimes up to one degree). The Sun is
roughly 140,000 Earth radii away, so I assume that the
topocentric correction to the Sun's position is so small as to
have no appreciable effect on the illumination of the Moon's
face.

Your location moves round the Earth's rotation axis in a 'small
circle' once a day, and the radius of the small circle depends on
your latitude. The geocentric optical libration changes on a
scale of the lunar month, as it arises in part from the
elliptical nature of the Moon's orbit around the Earth. The size
of the difference between the topocentric and geocentric
librations will therefore change during a day as you move round
your small circle. There will also be a 'constant' or more slowly
changing topocentric correction depending on your latitude and
the declination of the Moon. I imagine a smooth slow cycle with a
small amplitude fast 'wiggle' superimposed on it from the
topocentric librations. I have a strong suspicion that the
diurnal component of the topocentric correction to the Moon's
position is actually zero when the Moon transits, as at that
time the centre of the Moon, you and the centre of the Earth are
in a plane that includes your meridian.

Most published books and articles about Moon observation will
record the geocentric librations of the Moon, and these will be
different from the figures produced by this program.

## Accuracy

I checked a modified version of this program calculating
quantities for 0h UT each day against a topocentric ephemeris
generated using the JPL Horizons online ephemeris generator.
Quantities agreed to within 0.03 degrees for optical librations
(as might be expected as that is the level at which the physical
librations become important) and to within better than 0.005 for
the colongitude. I chose only one month to check, and a graph of
the colongitude errors looked as if there might be a periodic
term of amplitude about 0.02 degrees or so. All output has been rounded
to one decimal place for the selenographic coordinates, and whole
degrees for the position angles.

An earlier version of the program used the D46 formulas (0.3
degrees error in position) for the Moon coordinates - and I found
the errors in librations were three times as bad. The current moonpos()
function is good to 2 arcmin 'most of the time', and 5 arcmin
worst case (odd spikes in the error signal over a complete
saros).

## Algorithm and programming choices

The 'logic' of the main() function looks a bit like this:

<pre>
Get inputs
Do some error checking
    print title and column headings
    for each day in the month
        find the time of lunar transit using an iteration loop
	    if no transit, print message and skip to next day
	    find the days since J2000.0 corresponding to this time
	    calculate the ecliptic coordinates of the Sun
	    convert to equatorial coordinates
	    find the sun's altitude
	    code sun's altitude using a character ' anc*'
	    find the ecliptic coordinates of the Moon
	    convert to equatorial coordinates
	    find altitude of Moon
	    if Moon below horizon at transit print message and skip to next day
        apply topocentric correction to Moon equatorial coords
	    convert back to get topo ecliptic coords
	    find librations and pa of pole axis
	    find approximate heliocentric coordinates of Moon
	    use libration procedure to find coords of sub-solar point
	    find if Sun rising 'r' or setting 's' over Moon
	    find pa of bright limb using topocentric Moon coords
	    find % illumination
	    print line of output for that day
    next day
print key to table
end
</pre>

I have used 'pass by reference' a lot using pointers in the
functions to get round the limitation that C functions can only
return a single argument. Perhaps some structures for collections
of coordinates would be more elegant.

## References

The formulas for the Moon's geocentric coordinates, the siderial
time, days since J2000.0, coordinate transformations and all the
formulas for calculating the output quantities were modified and
simplified from Jean Meeus' book 'Astronomical Algorithms' (1st
Edition) published by Willmann-Bell, ISBN 0-943396-35-2. Chapter
51 was especially useful, but I have neglected the physical
libration (or 'free space motion') of the Moon and the nutation,
resulting in a simplification of the formulas.

The formulas for the Sun's geocentric coordinates, and the common
sense topocentric correction for the Moon's position (ignoring
the polar flattening of Earth), were taken from the Astronomical
Almanac pages C24 and D46. The iterative scheme to find the time
of Moon transit each day was adapted from the 'Explanatory
Supplement to the Astronomical Almanac', 1994, section 9.31 and
9.32.
