####################
The ATLAS build step
####################

This is slightly edited from `Clint's email on ATLAS configure / build`_.

The first person "I" here is Clint - from the email.

See also ATLAS docs `information on atlconf`_.

The build step follows the [[configure step|ATLAS configure step]].

Call the ATLAS source directory ``SRCdir``.

*********************
Architecture overview
*********************

* started by ``make`` (after ``configure``)
* ``make`` compiles large C wrapper routine ``SRCdir/bin/atlas_install.c``.
  ``atlas_install.c`` is a very old file that could be rewritten for much
  greater beauty, but it doesn't really do much.  It's just a wrapper that
  invokes all the parts of the ATLAS build system in a proper order.  It is ugly
  because it contains a bunch of commented out cruft from old installs, and it
  creates all the individual log files about the build.
* ``atlas_install`` invokes a series of searches to tune the L1BLAS (see
  ``SRCdir/tune/blas/level1``), and the L2BLAS (``SRCdir/tune/blas`` ``gemv`` &
  ``ger``), and the L3BLAS (``SRCdir/tune/blas/gemm``).
* Once searches are finished, ``atlas_install`` will invoke some basic TRSM
  tuning in ``SRCdir/tune/blas/level3``, then do a series of build steps.  The
  build steps use unix make, a C compiler, and a few unix commands like mv, cp,
  ln, etc.

For most tuning, there are various search drivers all written in ANSI C.  Some
searches just try a bunch of hand-tuned routines, while others also invoke
various code generators.  Code generators are usually either ANSI C programs, or
`extract`_ basefiles (``extract`` is written in ANSI C).

The search routines could conceivably be written in any language, including
bash.  Perhaps perl would be the easiest to write in given what they do (though
obviously not the easiest to read).

The searches are written in C because this does not add any additional language
dependencies beyond what ATLAS needs anyway (basic unix environment, make, C
compiler, weak dep on gcc).  In iFKO (`iterative Floating Point Kernel
Optimizer`_), I tried writing all the searches in Python, but the virtual
machine seems to malform all the timings, even though they are invoked as
separate C programs (not timed directly by Python).

ATLAS has to be very portable, so I do not plan on searches ever using
virtual machine languages, which can be a barrier for embedded use, even
if we didn't care about the performance interference from the virtual
machines, which I do.

In my opinion all the search routines are ugly to varying degrees.
However, they typically have to deal with such real-world vagaries that
my every elegant design has always decayed to ugliness.  Not just over
time, but by the time it actually works across most of ATLAS's target
platforms, it is already dialed to 11 on the repugnance scale.  Some
very simple searches that don't do much (eg., L1BLAS) retain enough
simplicity not to be so horrific.

There is a lot of scope for improvements for these search drivers, but there is
equal opportunity to mess things up.  They tend to get rewritten when I
fundamentally change how tuning is done (like in the recent AMM rewrite, and
possibly again when we go to threading-first tuning), and each time it is done,
there is a *long* painful process of getting them fairly robust again.

.. _Clint's email on ATLAS configure / build:
   http://sourceforge.net/p/math-atlas/mailman/message/32177779/
.. _Information on atlconf:
   http://math-atlas.sourceforge.net/devel/atlas_devel/node45.html
.. _Extract: http://web.eecs.utk.edu/~rwhaley/extract/Extract.html
.. _Iterative Floating Point Kernel optimizer:
   http://dx.doi.org/10.1109/ICPP.2005.77
