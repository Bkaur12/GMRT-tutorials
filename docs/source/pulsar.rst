Pulsar data analysis
=====================

The tutorial is intended to provide you a basic introduction to the steps involved in
the analysis of pulsar data, including searching for new pulsars/transients and timing
a newly discovered pulsar. The filterbank data obtained from GMRT are
converted to SIGPROC filterbank format using either the ``filterbank`` command from
SIGPROC or the ``rficlean`` command from RFIClean. The tutorial will use data already
converted to the SIGPROC filterbank format.

Introduction
-------------

You need to have PRESTO, SIGPROC, TEMPO2, TEMPO, and their dependencies installed on
your machine. You also need the SIGPROC filterbank data to be available on your disk.

The header information of the data from a SIGPROC filterbank file can be inspected using
the ``readfile`` command from PRESTO (an example using the ``header`` command from
SIGPROC will be shown later).

.. code-block::

   readfile inputFile.fil

.. figure:: /images/pulsar/headerintro.png
   :alt: readfile output
   :align: center
   :scale: 70% 

Data-inspection
~~~~~~~~~~~~~~~~

A section of the raw data can be inspected by plotting the data as a function of time
and frequency, e.g., using the ``waterfaller.py`` command from PRESTO. The command
``waterfaller.py`` several provisions that enable inspecting the data in several ways
(e.g., before and after dedispersion, partial and full averaging over the bandwidth,
partial averaging over time, etc.)

.. code-block::

   waterfaller.py inputFile.fil -d 0 --subdm 0 --show-ts -s 64 -T 0 -t 5

.. figure:: /images/pulsar/wfaller1.png
   :alt: waterfall 0-DM
   :align: center
   :scale: 70% 

The single pulses from a reasonably strong pulsars might be visible after dedispersing
the data at correct dispersion measure (DM).

.. code-block::

   waterfaller.py inputFile.fil -d <DM> --subdm <DM> --show-ts -s 64 -T 0 -t 5

.. figure:: /images/pulsar/wfaller2.png
   :alt: waterfall correct-DM
   :align: center
   :scale: 70% 

Dedispersion and Folding
~~~~~~~~~~~~~~~~~~~~~~~~~

A small section of the raw data can be dedispersed and viewed using ``waterfaller.py``
as seen above. To dedisperse the whole data and also fold it over the rotation period
of the pulsar, we can use ``prepdata`` from PRESTO.

.. code-block::

   prepfold -p 3.7452943 -dm 50.9 -topo -n 128 -nosearch  introTestd4.fil

``prepfold`` is a powerful command, it can search for the most optimum parameters around
the given fiducial parameters.

.. figure:: /images/pulsar/prepout.png
   :alt: prepfold output
   :align: center
   :scale: 70% 



.. include:: psrsearchb3.rst

.. include:: psrtiming_tutorial.rst


Acknowledgements and Credits
-----------------------------

The **Searching for new pulsars** and **Timing of pulsars** sections of
the tutorial were entirely contributed by Shubham Singh and Shyam Sunder,
respectively. The data sets used in these parts were kindly provided by
Dr. Jayanta Roy and Dr. Bhaswati Bhattacharyya. Yogesh Maan is responsible
for the overall pulsar tutorial coordination as well as for making available
the data used in the **Introduction** section.
