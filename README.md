Introduction
------------

These are my derivations of Fabien 'Ryg's rANS coder plus a
static arithmetic coder derived from Eugene Shelwein's work.
Therefore none of the really complex stuff is my own and the real
credit belongs elsewhere.

The originals can be found at:
-    https://github.com/rygorous/ryg_rans
-    http://ctxmodel.net (defunct?)

I thought about forking this, but I didn't do that way back when I
first created my variants and so it'd be a fork in name only, with no
actual history.

The arithmetic coder was designed to be a fast unrolled variant that
coded using multiple arithmetic coders simultaneously. I also
implemented a static order-1 model in addition to the usual order-0.

About the same time I'd finished that code the ANS implementations
started arriving so I took Ryg's original rANS (two way interleaved
variant) and produced a 4-way interleaved version.  Successive
optimisations lead version "4c".  Most recently I explored some newer
tweaks and ideas.

- arith_static.c
	The unrolled order-0/1 arithmetic coder.

- rANS_static4c.c
	A 4-way unrolled order-0/1 rANS coder.

- rANS_static4j.c
	Recent (Apr 2016) optimisations to 4c.c above.
	The output is interchangeable.

- rANS_static64c.c, rans64.h
	A 4 way unrolled version of Ryg's rans64.h.
	(This needs a mulhi cpu instruction to do 64-bit by 64-bit
	multiplication and take the top 64-bits of the result.
	Available on most(all?) x86_64 architectures, but I needed to
	consider 32-bit hosts too.)

- rANS_static4_16i.c
	A variant of the rANS_static4 code that uses 16-bit shifts.
	Kludgy and full of warnings. It may also not be robust!
	This is the same idea used in Ryg's SIMD variant; by keeping
	the rANS state as 16+16 instead of 24+8 we only ever have one
	normalisation step instead of two.

Sorry there's no makefile, but it's all self contained single .c files
basically.  I put these here in place of my old ftp site simply
because it's easier for people to find and discuss.

PS.
See http://encode.ru/threads/1867-Unrolling-arithmetic-coding
for the thread that started this particular ball rolling.


Usage
-----

Order 0 encoding and decoding

    rANS_static4j -o0 in in.rans0
    rANS_static4j -d in.rans0 in.decoded

Order 1 encoding and decoding

    rANS_static4j -o0 in in.rans1
    rANS_static4j -d in.rans1 in.decoded

Testing/benchmarking order 1

    rANS_static4j -o1 -t in


Benchmarks
----------

Q40 and Q8 are DNA sequencing quality values with approx 40 and 8
distinct values each, representing high and low complexity data.

    $ head -5 /tmp/Q40
    AAA7DDFFFHHHGFHIGHGIGGIJIGHCIGGHBGFCGGGEGGCGEG@F;FC;8@@F>@E@6)=?B66?@A;?(555;=((,55(((((+2(39(288<
    :?A=DA;?BDH?F<BGG>F>?;EF>8:8?E>FHGIIIEIHICH9@F(?BFEEGHEIIC<@DEEEEEC2?=A;;?@?;((5,5?9(9<0()9388(>AC
    AAA@ABFFFHHHGHGJIGEEIGHIGIIIIIJIHEGIGGIGGGIDGIIIJH>B<@DD;;B?E===;@FDGH3=@EAHA;?))7;6((.5(,;(,((5(,
    ?BABFABDFHHHHHHHIJJJJJIIJJJGHGIJDIE@DDGDF@GHGGGAEGGCEE>CDFDB;>>A@=;AC??BBDC(<A<?C<AB?<??AB8<(28?09
    2((2+&)&)&(+24++(((+22(:))&&0-))&&&)&3,,,((,,'',''-(/)).))))(((((0(()))3.--2))).))2)50-(((((((((((
    
    $ head -5 /tmp/Q8
    --A--JJ7-----<---7-
    7----<--<<-<7-7--FF
    --FAAA7-JFFFF<FFJ<7<FFFJFAJFFFJJAAF<A<JJ<FJJ7FJJFFJJ7F7FAAJJFFFFJJJJFJFJJFJFJFJJJJFJFJJFFJF<JJFJJJJJ7FJJJJF<7<FJJJAJJJFAFJFFFJF<JFJJJJJJFAJJFJJJJJFFFFF
    JJJ<JJJJ<JJJJJ7FFJFJJFJJF<AFAAJJJFJFJJ7FJFJJ7FFJJFJFFFFF<<AAFJJAFF
    FFFFFJFJJFJJJFJJJJJAJJJJAJJJFJJJJFJJJJFJJJJJJJJJJJJJJJFJFJFAJJJJJJJFJAJJJJ<AJJJJJJFJJJJFFJFJFJJFFJFJJFFJFJJAJJJJFJJ7JJJJAJJJ7J-FAJAJ7JJJAFFAFFFAFF-JFFA


Tested on an "Intel(R) Core(TM) i5-4570 CPU @ 3.20GHz" (from
/proc/cpuinfo on Ubuntu Trusty).


Q40 test file, order 0:

    arith_static            251.2 MB/s enc, 143.0 MB/s dec  94602182 bytes -> 53711390 bytes
    rANS_static4c           289.2 MB/s enc, 340.0 MB/s dec  94602182 bytes -> 53690171 bytes
    rANS_static4j           293.7 MB/s enc, 376.1 MB/s dec  94602182 bytes -> 53690159 bytes
    rANS_static4_16i        299.9 MB/s enc, 377.0 MB/s dec  94602182 bytes -> 53690048 bytes
    rANS_static64c          279.0 MB/s enc, 439.4 MB/s dec  94602182 bytes -> 53691108 bytes
   
Q8 test file, order 0:

    arith_static            239.1 MB/s enc, 145.4 MB/s dec  73124567 bytes -> 16854053 bytes
    rANS_static4c           291.7 MB/s enc, 349.5 MB/s dec  73124567 bytes -> 16847633 bytes
    rANS_static4j           290.5 MB/s enc, 354.2 MB/s dec  73124567 bytes -> 16847597 bytes
    rANS_static4_16i        371.0 MB/s enc, 453.6 MB/s dec  73124567 bytes -> 16847528 bytes
    rANS_static64c          351.0 MB/s enc, 549.4 MB/s dec  73124567 bytes -> 16848348 bytes
    
Q40 test file, order 1:

    arith_static            128.4 MB/s enc,  94.7 MB/s dec  94602182 bytes -> 43420823 bytes
    rANS_static4c           168.4 MB/s enc, 212.1 MB/s dec  94602182 bytes -> 43167683 bytes
    rANS_static4j           170.0 MB/s enc, 240.2 MB/s dec  94602182 bytes -> 43167683 bytes
    rANS_static4_16i        188.4 MB/s enc, 254.9 MB/s dec  94602182 bytes -> 43229063 bytes
    rANS_static64c          203.9 MB/s enc, 287.1 MB/s dec  94602182 bytes -> 43168614 bytes
    
Q8 test file, order 1:

    arith_static            189.2 MB/s enc, 130.4 MB/s dec  73124567 bytes -> 15860154 bytes
    rANS_static4c           208.6 MB/s enc, 269.2 MB/s dec  73124567 bytes -> 15849814 bytes
    rANS_static4j           210.5 MB/s enc, 305.1 MB/s dec  73124567 bytes -> 15849814 bytes
    rANS_static4_16i        240.4 MB/s enc, 345.4 MB/s dec  73124567 bytes -> 15869029 bytes
    rANS_static64c          239.2 MB/s enc, 397.6 MB/s dec  73124567 bytes -> 15850522 bytes