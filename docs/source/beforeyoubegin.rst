Prerequisites
==============

Interferometric data
---------------------

The interferometric (continuum/spectral line) data from GMRT can be analysed using NRAO's 
Common Astronomy Software Applications (`CASA`_) or Astronomical Image Processing System (`AIPS`_).
We will focus on the analysis using CASA in these tutorials.

CASA
~~~~~

You need to download and install CASA. It is important to read the CASA `documentation`_ in order to understand the details of the CASA tasks and the overall functioning of CASA. It will be useful to get familiar with the `usage`_ at the ipython prompt before 
starting the tutorials. The specific CASA version in which the tutorial was prepared is provided at the beginning 
of the individual tutorials.

Pulsar/Beam data
-----------------

The pulsar/beam data are typically recorded using the phased-array (PA) and the
incoherent-array (IA) observing modes of the GMRT. The format of the native beam
data is same for both the PA and the IA observing modes. After a simple format
conversion using `SIGPROC`_/`RFIClean`_, the native beam data from GMRT can be
used with the publicly available softwares such as `SIGPROC`_, `PRESTO`_, `DSPSR`_,
etc. The tutorial will use data already converted to the SIGPROC filterbank format.
The primary softwares needed for the tutorial are `PRESTO`_, `tempo2`_, and their
dependencies.




.. _SIGPROC: https://github.com/SixByNine/sigproc.git
.. _PRESTO: https://github.com/scottransom/presto
.. _DSPSR: https://github.com/demorest/dspsr
.. _RFIClean: https://github.com/ymaan4/RFIClean
.. _TEMPO2: https://bitbucket.org/psrsoft/tempo2.git
.. _CASA: https://casadocs.readthedocs.io/en/stable/
.. _AIPS: http://www.aips.nrao.edu/index.shtml
.. _documentation: https://casadocs.readthedocs.io/en/stable/api/casatasks.html
.. _usage: https://casadocs.readthedocs.io/en/stable/notebooks/usingcasa.html
