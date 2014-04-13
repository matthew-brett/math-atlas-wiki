#############################################
Some notes on the ``configure`` step in ATLAS
#############################################

*******
Summary
*******

* ATLAS configure is not an `autoconf`_ configure script
* It is a ``sh`` script that compiles and runs binaries from C code called
  *probes*. Configure therefore needs a working C compiler and a working
  ``make`` command.
* The probes do most of the work of getting configuration values from the
  hardware.

After the configure step comes [[the build step|ATLAS build step]].

************
Introduction
************

You'll find the main docs for ``configure`` at `ATLAS configure step`_.

As you will see from these docs, the basic procedure is::

    cd BLDdir ; SRCdir/configure [flags]

where ``BLDdir`` is an empty directory to contain the configuration and build
files, and ``SRCdir`` is the path to the ATLAS source code.

*********************
Architecture overview
*********************

The first thing to remember is: **ATLAS configure is not an autoconf configure
command**. ATLAS doesn't use autoconf_ at all.

See `Clint's email on ATLAS configure / build`_ for Clint Whaley's summary of
configure.  Some of that email appears in this document.

But ``configure`` *is* a ``sh`` script.  At the top of this script you will
find the following comment::

    # BFI configure-like script to bootstrap ATLAS's C-based config scripts
    # dependencies: sed, pwd
    # shell built-in deps: echo, test

The script compiles a bunch of C scripts and probes which are found in
``SRCdir/CONFIG``.

The configure script:

* Creates the universal ``Make.inc``, which defines all Makefile macros for all
  of ATLAS's makefiles.  In the case of new systems where things must be
  kludged, ``Make.inc`` allows the user to edit only one file, rather than
  the standard autoconf configure method of generating the individual makefiles,
  all of which have to be hand-edited in the case of problems.
* Moves all needed Makefiles into ``BLDdir``.
* Probes the system using simple test routines (in ``CONFIG/``) to figure out
  very detailed information (what ISA, what ISA extension/vector operation, what
  assembly dialect, what type of threading works, etc).  Most of this probe info
  is reflected in the produced ``Make.inc``.

The configure step is therefore analogous to the normal autoconf configure step,
though ATLAS is looking for much more detailed information than a typical
autoconf configuration run.

There is more detail on the architecture of ATLAS configure in `information on
atlconf`_.

***********************
Running ATLAS configure
***********************

There is some useful help on ``flags`` from::

    SRCdir/configure --help

Note the double hyphen for ``--help``.

Here's the output from ``SRCdir/configure --help``::

    ATLAS config includes this script, and probes written in C.
    Therefore, configure flags are union of script and probe flags.
    This configure script accepts the following flags:
    --cc=<C compiler> : compiler to compile configure probes
    --cflags='<flags>' : flags for above
    --prefix=<dirname> : Toplevel installation directory.
                            Default: /usr/local/atlas
    --incdir=<dirname> : Installation dir for include files
                            Default: /usr/local/atlas/include
    --libdir=<dirname> : Installation dir for libraries
                            Default: /usr/local/atlas/lib
    --rtarg=<mach> : remote cross-compile target machine
    --accel=[0/1/2] : build for accelerator:
        0: Don't build for accelerator
        1: Build for TI_C66_BM
        2: Build for Xeon Phi
    --shared : same as --dylibs
    --dylibs : build dynamic/shared libs in addition to static libs
    --force-tids="<#> threadIDlist"
    --nof77 : You have no Fortran compiler installed.  Note that
                this will disallow building the F77 interface, and
                some of the tests (eg, standard BLAS testers)
    --with-netlib-lapack-tarfile=<path to lapack tarfile>
    --gcc3pass=<gcc for frontend>,<assembler>:<linker>
        Provide full paths for all compilers/assemblers
    Attempting to build xconfig to get probe flags:
    ../ATLAS/configure: line 113: ./xconfig: No such file or directory

ATLAS tried to compile a file ``xconfig``, but failed.  We can get it to compile
and run by passing a bad argument into configure, e.g. ``SRCdir/configure
bad-arg``.  Then we get this::

    gcc -I/Users/mb312/dev_trees/math-atlas/TEST/build_test/../ATLAS//CONFIG/include  -g -w -c /Users/mb312/dev_trees/math-atlas/TEST/build_test/../ATLAS//CONFIG/src/atlconf_misc.c
    gcc -I/Users/mb312/dev_trees/math-atlas/TEST/build_test/../ATLAS//CONFIG/include  -g -w -o xconfig /Users/mb312/dev_trees/math-atlas/TEST/build_test/../ATLAS//CONFIG/src/config.c atlconf_misc.o 
    ./xconfig -d s /Users/mb312/dev_trees/math-atlas/TEST/build_test/../ATLAS/ -d b /Users/mb312/dev_trees/math-atlas/TEST/build_test  bad-arg

    ERROR around arg 7 (bad-arg).
    USAGE: ./xconfig [flags] where flags are:
    -v <verb> : verbosity level
    -O <enum OSTYPE #>  : set OS type
    -s <enum ASMDIA #>  : set assembly dialect
    -A <enum MACHTYPE #> : set machine/architecture
    -V #    # = ((1<<vecISA1) | (1<<vecISA2) | ... | (1<<vecISAN))
    -b <32/64> : set pointer bitwidth
    -o <outfile>
    -C [xc,ic,if,sk,dk,sm,dm,al,ac] <compiler>
    -F [xc,ic,if,sk,dk,sm,dm,al,ac,gc] '<comp flags>'
    -Fa [xc,ic,if,sk,dk,sm,dm,al,ac,gc] '<comp flags to append>'
            al: append flags to all compilers
            ac: append flags to all C compilers
            gc: append flags to gcc compiler used in user-contributed index files.
            acg: append to all C compilers & the index gcc
            alg: append to all compilers & the index gcc
    -T <targ> : ssh target for cross-compilation (probably broken)
    -D [c,f] -D<mac>=<rep> : cpp #define to add to [CDEFS,F2CDEFS]
        eg. -D c -DL2SIZE=8388604 -D f -DADD__ -D f -DStringSunStyle
    -d [s,b]  : set source/build directory
    -f <#> : size (in KB) to flush before timing
    -t <#> : set # of threads (-1: autodect; 0: no threading)
    -tl <#> <list> : set # of threads, use list of affinity IDs
    -r <#>: set the number of floating point registers to #
    -m <mhz> : set clock rate
    -S[i/s] <handle> <val>  : special int/string arg
        -Si bozol1 <0/1> : supress/enable bozo L1 defaults
        -Si archdef <1/0> : enable/supress arch default use
        -Si ieee <1/0> : dis/allow optimizations that break IEEE FP standard
            (eg., NEON, 3DNow!)
        -Si latune <1/0> : do/don't tune F77 LAPACK routines
        -Si nof77 <0/1> : Have/don't have fortran compiler
        -Si nocygwin <0/1> : Do/don't depend on GPL cygwin library
                            (Windows compiler/cygwin install only)
        -Si omp <0/1> : don'tuse/use OpenMP for threading
        -Si antthr <0/1/2> : nobuild/build/use Antoine's code for threading
        -Si lapackref <0/1>: Netlib lapack is not/is unpacked
                            to $BLDdir/src/lapack/ref
        -Ss kern <path/to/comp> : use comp for all kernel compilers
        -Ss ADdir <path/to/archdefs> : Get archdefs frm custom path
        -Ss pmake <parallel make invocation (eg '$(MAKE) -j 4')>
        -Ss f77lib <path to f77 lib needed by C compiler>
        -Ss flapack <path to netlib lapack>: used to build full lapack lib
        -Ss [s,d]maflags 'flags'
    NOTE: enum #s can be found by : make xprint_enums ; ./xprint_enums
    xconfig exited with 7


This output tells us about another aspect of ``configure`` - which is
**configure needs a working C compiler**.  Configure commpiles many small
binaries, called "probes" which it will use to get configuration binaries.  To
do this, it uses ``make`` and a C compiler.  By default the C compiler command
is ``gcc``, but you can change this with the ``--cc=`` input argument to
``configure``.

Another thing we notice is that many arguments are integers which are enumerated
values.  We can get the integers we need by following the instructions above:
``make xprint_enums ; ./xprint_enums``::

    Architectural enums (Config's enum MACHTYPE):
        0 = 'UNKNOWN'
        1 = 'POWER3'
        2 = 'POWER4'
        3 = 'POWER5'
        4 = 'PPCG4'
        5 = 'PPCG5'
        6 = 'POWER6'
        7 = 'POWER7'
        8 = 'POWERe6500'
        9 = 'IBMz9'
        10 = 'IBMz10'
        11 = 'IBMz196'
        12 = 'x86x87'
        13 = 'x86SSE1'
        14 = 'x86SSE2'
        15 = 'x86SSE3'
        16 = 'P5'
        17 = 'P5MMX'
        18 = 'PPRO'
        19 = 'PII'
        20 = 'PIII'
        21 = 'PM'
        22 = 'CoreSolo'
        23 = 'CoreDuo'
        24 = 'Core2Solo'
        25 = 'Core2'
        26 = 'Corei1'
        27 = 'Corei2'
        28 = 'Corei3'
        29 = 'Atom'
        30 = 'P4'
        31 = 'P4E'
        32 = 'Efficeon'
        33 = 'K7'
        34 = 'HAMMER'
        35 = 'AMD64K10h'
        36 = 'AMDDOZER'
        37 = 'AMDDRIVER'
        38 = 'UNKNOWNx86'
        39 = 'IA64Itan'
        40 = 'IA64Itan2'
        41 = 'USI'
        42 = 'USII'
        43 = 'USIII'
        44 = 'USIV'
        45 = 'UST2'
        46 = 'UnknownUS'
        47 = 'MIPSR1xK'
        48 = 'MIPSICE9'
        49 = 'ARMv7'
        50 = 'TI_C66_BM'
        51 = 'XeonPHI'

    Operating System enums (Config's enum OSTYPE):
        0 = 'UNKNOWN'
        1 = 'Linux'
        2 = 'SunOS'
        3 = 'SunOS4'
        4 = 'OSF1'
        5 = 'IRIX'
        6 = 'AIX'
        7 = 'Win9x'
        8 = 'WinNT'
        9 = 'Win64'
        10 = 'HPUX'
        11 = 'FreeBSD'
        12 = 'OSX'

    Compiler integer defines:
        0 = 'ICC'
        1 = 'SMC'
        2 = 'DMC'
        3 = 'SKC'
        4 = 'DKC'
        5 = 'XCC'
        6 = 'GCC'
        7 = 'F77'


    ISA extensions are combined by adding their values together (bitvector):
            none: 1
            VSX: 2
        AltiVec: 4
            AVXZ: 8
        AVXMAC: 16
        AVXFMA4: 32
            AVX: 64
            SSE3: 128
            SSE2: 256
            SSE1: 512
            3DNow: 1024
    FPV3D2MACNEON: 2048
    FPV3D16MACNEON: 4096
    FPV3D32MAC: 8192
    FPV3D16MAC: 16384

This is more on probes in `ATLAS probe overview`_


.. _autoconf: http://www.gnu.org/software/autoconf
.. _Clint's email on ATLAS configure / build:
   http://sourceforge.net/p/math-atlas/mailman/message/32177779/
.. _ATLAS configure step:
   http://math-atlas.sourceforge.net/atlas_install/node7.html
.. _Information on atlconf:
   http://math-atlas.sourceforge.net/devel/atlas_devel/node45.html
.. _ATLAS probe overview:
   http://math-atlas.sourceforge.net/devel/atlas_devel/node47.html
