# ConfigPowerSpectra

# ADD LATEX and LINKS
# Add note on R0

Code to compute configuration-space power spectra for arbitrary survey geometries, based on the work of Philcox & Eisenstein (2019, submitted). This computes the Legendre multipoles of the power spectrum, <img src="/tex/a939b8abbd34a6a7097130a860c9ebc2.svg?invert_in_darkmode&sanitize=true" align=middle width=38.738704949999985pt height=24.65753399999998pt/> by computing weighted pair counts over the survey, truncated at some maximum radius <img src="/tex/12d208b4b5de7762e00b1b8fb5c66641.svg?invert_in_darkmode&sanitize=true" align=middle width=19.034022149999988pt height=22.465723500000017pt/>. This fully accounts for window function effects, does not include shot-noise, and is optimized for small-scale power spectrum computation in real- or redshift-space.

# Code Requirements and Acknowledgements

- [C compiler](https://gcc.gnu.org/): Tested with gcc 5.4.O
- [Gnu Scientific Library (GSL)](https://www.gnu.org/software/gsl/doc/html/index.html): Any recent version
- [Corrfunc](https://corrfunc.readthedocs.io): 2.0 or later (required for aperiodic surveys to compute geometry correction)
- [OpenMP](https://www.openmp.org/): Any recent version (optional, but required for parallelization)
- [Python](https://www.python.org/): 2.7 or later, 3.4 or later (required for pre- and post-processing)

Corrfunc can be installed using ``pip install corrfunc`` and is used for efficient pair counting.

Note that many of the code modules and convenience functions are based on those of [RascalC](https://RascalC.readthedocs.io), developed by Oliver Philcox, Daniel Eisenstein, Ross O'Connell and Alexander Wiegand.

# Inputs

## Galaxy and Random Particle Files

The main inputs to the C++ code are files containing the locations and weights of galaxies and random particles (whose distribution matches that of unclustered galaxies in the same survey). These are the standard 'Data' and 'Random' files used in pair count analyses and are here required to be lists of particle positions in space-separated (x,y,z,weight) format, with the co-ordinates given in comoving <img src="/tex/76ea517ab34b24ca8d15761ef9722416.svg?invert_in_darkmode&sanitize=true" align=middle width=59.083140599999986pt height=26.76175259999998pt/> units.

We provide a convenience function to convert galaxy/random files in (RA,Dec,z,weight) co-ordinates to the required format;

    python python/convert_to_xyz.py {INFILE} {OUTFILE} {OMEGA_M} {OMEGA_K} {W_DARK_ENERGY}

where {INFILE} and {OUTFILE} are the filenames for the (RA,Dec,redshift,weight) and the remaining parameters specify the (present-day) cosmology. If these are not specified a cosmology of <img src="/tex/e938af78a11b5411aabfaa8d49257dd4.svg?invert_in_darkmode&sanitize=true" align=middle width=221.15576999999996pt height=24.65753399999998pt/> is assumed by default.

For pair-count analysis, we usually require a random particle files with <img src="/tex/0351561a31ea476c55b65a9781f5e173.svg?invert_in_darkmode&sanitize=true" align=middle width=108.63024974999998pt height=22.465723500000017pt/> for DR counts and <img src="/tex/360cb6fce6faca1aa801523bd1dd27e1.svg?invert_in_darkmode&sanitize=true" align=middle width=108.63024974999998pt height=22.465723500000017pt/> for RR counts to minimize random noise. For this reason we provide a further convenience function to draw a random subset of a given particle file (in (x,y,z,weight) co-ordinates). This is run as follows (where {N_PARTICLES} specifies the size of the output file)::

    python python/take_subset_of_particles/py {INPUT_FILE} {OUTPUT_FILE} {N_PARTICLES}

## Computing Binning Functions

In addition to the sets of galaxy/random positions, we require a file to specify the desired <img src="/tex/63bb9849783d01d91403bc9a5fea12a2.svg?invert_in_darkmode&sanitize=true" align=middle width=9.075367949999992pt height=22.831056599999986pt/>-space binning of the output power spectra. Two Python routines are provided to produce the relevant files in linear or logarithmic (base <img src="/tex/8cd34385ed61aca950a6b06d09fb50ac.svg?invert_in_darkmode&sanitize=true" align=middle width=7.654137149999991pt height=14.15524440000002pt/>) binning and are run as::

        python python/compute_binning_file_linear.py {N_LOG_BINS} {MIN_K} {MAX_K} {OUTPUT_FILE}
        python python/compute_binning_file_log.py {N_LINEAR_BINS} {MIN_K} {MAX_K} {OUTPUT_FILE}

with the output file saved to the specified destination. The binning file can also be manually specified as an ASCII file with each line specifying the upper and lower coordinates of each <img src="/tex/63bb9849783d01d91403bc9a5fea12a2.svg?invert_in_darkmode&sanitize=true" align=middle width=9.075367949999992pt height=22.831056599999986pt/>-bin (in <img src="/tex/f6246daf26f12d891bc642e0bffad22e.svg?invert_in_darkmode&sanitize=true" align=middle width=61.00088444999999pt height=29.168960700000017pt/> units). Note that the bins are required to be contiguous (i.e. the upper limit of the <img src="/tex/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode&sanitize=true" align=middle width=9.86687624999999pt height=14.15524440000002pt/>th bin should equal the lower limit of the <img src="/tex/949707b3bc37b3be0f8b25742664879e.svg?invert_in_darkmode&sanitize=true" align=middle width=50.962710149999985pt height=24.65753399999998pt/>th bin.

# Wrapper
Note that the code must be compiled in the correct manner before the wrapper is used.

# Computing the Survey Correction Function

An important ingredient in the weighted power spectrum pair counts is the 'survey correction function' <img src="/tex/5e16cba094787c1a10e568c61c63a5fe.svg?invert_in_darkmode&sanitize=true" align=middle width=11.87217899999999pt height=22.465723500000017pt/>, defined as the ratio of ideal and true (unweighted) RR pair counts. For periodic data, this is simply unity. In this package, we compute <img src="/tex/5e16cba094787c1a10e568c61c63a5fe.svg?invert_in_darkmode&sanitize=true" align=middle width=11.87217899999999pt height=22.465723500000017pt/> using the [Corrfunc](https://Corrfunc.readthedocs.io) pair counting routines (for aperiodic data) and store the results as quadratic fits to the first few multipoles of <img src="/tex/0f493c5b181cf5121a9d1c56ab850859.svg?invert_in_darkmode&sanitize=true" align=middle width=28.698746999999987pt height=26.76175259999998pt/>. Note that a normalization is also performed for later convenience.

This can be computed using the ``compute_correction_function.py`` script::

    python python/compute_correction_function.py {RANDOM_PARTICLE_FILE} {OUTFILE} {PERIODIC} [{R_MAX} {N_R_BINS} {N_MU_BINS} {NTHREADS}]

with inputs:

- {RANDOM_PARTICLE_FILE}: File containing random particles in the survey geometry. Since the correction function is being fit to a smooth function, we can use a relatively small random catalog here.
- {OUTFILE}: Location of output ASCII file. This is automatically read in by the C++ code
- {PERIODIC}: Whether to assume periodic boundary conditions
- *(if aperiodic)* {R_MAX}: Radius (in <img src="/tex/76ea517ab34b24ca8d15761ef9722416.svg?invert_in_darkmode&sanitize=true" align=middle width=59.083140599999986pt height=26.76175259999998pt/>) up to which to count pairs and fit the correction function. This should be at least as large as the truncation radius (<img src="/tex/12d208b4b5de7762e00b1b8fb5c66641.svg?invert_in_darkmode&sanitize=true" align=middle width=19.034022149999988pt height=22.465723500000017pt/>) used for the power spectrum computation. Note that the computation time scales as <img src="/tex/20fb92c8f59e080ecf6a1347812d43c6.svg?invert_in_darkmode&sanitize=true" align=middle width=36.739601249999986pt height=26.76175259999998pt/>.
- *(if aperiodic)* {N_R_BINS}: Number of radial bins in the pair counting. We recommend using <img src="/tex/321d72d7e01147d35ccbef3d7e9dddbe.svg?invert_in_darkmode&sanitize=true" align=middle width=87.39354029999998pt height=26.76175259999998pt/> radial binning here.
- *{if aperiodic}* {N_MU_BINS}: Number of angular bins used in the pair counting. We recommend 100 <img src="/tex/07617f9d8fe48b4a7b3f523d6730eef0.svg?invert_in_darkmode&sanitize=true" align=middle width=9.90492359999999pt height=14.15524440000002pt/> bins.
- *{if aperiodic}* {N_THREADS}: Number of CPU threads on which to perform pair counts.

For an aperiodic survey, this may take some time to compute, but only needs to be computed once for a given survey. Whilst the user can specify the number of radial and angular bins used by the pair counts, this does not have a significant affect on the output function, assuming a moderately fine binning is used.

# Note on choice of R0
Add time scaling and errors

# Computing the Power Spectrum

## Code Compilation

To compile the C++ code, simply run the following in the installation directory::

    bash clean
    make

This creates the ``./power`` executable in the working directory.

The Makefile may need to be modified depending on the specific computational system. In addition, by specifying the ``-DPERIODIC`` flag in the Makefile, the code will assume periodic boundary conditions (as appropriate for periodic box simulations), measuring the angle <img src="/tex/07617f9d8fe48b4a7b3f523d6730eef0.svg?invert_in_darkmode&sanitize=true" align=middle width=9.90492359999999pt height=14.15524440000002pt/> from the z-axis. Furthermore, by removing the ``-DOPENMP`` flag, the code will run single threaded, without use of OPENMP.


## Compute Weighted Pair Counts

Configuration space power spectra are computed by estimating RR, DR and DD pair counts (analogous to 2PCF computation) weighted by a <img src="/tex/63bb9849783d01d91403bc9a5fea12a2.svg?invert_in_darkmode&sanitize=true" align=middle width=9.075367949999992pt height=22.831056599999986pt/>-bin-dependent kernel. To compute pair counts for two input fields we simply run the ``./power`` executable, specifying the input parameters on the command line. (Parameters can also be altered in the ``power_mod/ppower_parameters.h`` file, but this requires the code to be re-compiled after each modification.). As an example let's compute a set of data-random (DR) counts from input files ``galaxy_positions.txt`` and ``random_positions.txt``.

    ./power -in galaxy_positions.txt -in2 random_positions.txt -binfile binfile.csv -output output/ -out_string DR -max_l 2 -R0 100 -inv_phi_file inv_phi_coefficients.txt -nthread 10

This runs in a few minutes to hours (depending on the catalog size and computational resources available) and uses the following main arguments:

    -``-in``: First input ASCII file containing space separated (x,y,z,weight) positions of particles in comoving Mpc/h units.
    -``-in2``: Second input ASCII file, with format as above.
    -``-binfile``: $k$-space ASCII binning file, as described above.
    -``-output``: Directory in which to house output products. This will be created if not already in existence.
    -``-out_string``: String to include in output filename for identification (e.g. RR, DR or DD)
    -``-max_l``: Maximum Legendre multipole required (must be even). Currently, only multipoles up to the hexadecapole ($\ell = 6$) have been implemented, but more can be added if required.
    -``-R0``: Truncation radius in Mpc/h units (default: $100\,h^{-1}\mathrm{Mpc}$). See note above.
    -``-inv_phi_file``: Location of survey geometry correction file, as produced above.
    -``-nthread``: Number of CPU threads to use for the computation.
    -``-periodic``: This flag must be set if we require the code to be run with *periodic* boundary conditions, measuring $\mu$ from the z-axis.

Note that a full list of command line options to the executable can be shown by running ``./power`` without any arguments. The code creates the output file ``{OUT_STRING}_power_counts_n{N_BINS}_m{MAX_L}_full.txt``, specifying the ``out_string`` parameter, the number of radial bins and the maximum Legendre multipole. Each line of the output file has the (weighted) pair count with the column specifying the Legendre multipole.

To compute the full power spectra, the data-data (DD), data-random (DR) and random-random (RR) pair counts must be computed. We do *not* have to use the same sized random catalogs for the DR and RR counts. It is usually preferable to use a larger random catalog for the DR pair counts to reduce noise. We recommend <img src="/tex/39336a2ffd276833bc2af414ed460bfa.svg?invert_in_darkmode&sanitize=true" align=middle width=12.785434199999989pt height=14.15524440000002pt/>50x randoms for DR counts and <img src="/tex/39336a2ffd276833bc2af414ed460bfa.svg?invert_in_darkmode&sanitize=true" align=middle width=12.785434199999989pt height=14.15524440000002pt/>10x randoms for the DD counts. Note that the RR counts are the most computationally intensive procedure, but they only need be computed for each survey once (i.e. when analyzing mock data, the RR pair counts are the same for each mock).

## Reconstruct Power Spectrum

Once the pair counts have been computed, it is straightforward to reconstruct the power spectrum. This can be done via a simple Python script;

    python python/reconstruct_power.py {DD_FILE} {DR_FILE} {RR_FILE} {GAL_FILE} {N_RAND_RR} {N_RAND_DR} {PERIODIC} {OUTFILE}

where {DD_FILE}, {DR_FILE} and {RR_FILE} give the locations of the DD, DR and RR weighted pair counts, {GAL_FILE} gives the input galaxy file (needed for normalization), {N_RAND_RR} and {N_RAND_DR} give the number of random particles used for RR and DR counts. {PERIODIC} is unity if the code is computed with periodic boundary conditions and zero else. The output power spectrum is given in ASCII format in the specified {OUTFILE}, with the power spectrum estimates for each <img src="/tex/63bb9849783d01d91403bc9a5fea12a2.svg?invert_in_darkmode&sanitize=true" align=middle width=9.075367949999992pt height=22.831056599999986pt/>-bin on a separate line, with the column indicating the (even) Legendre multipole.
