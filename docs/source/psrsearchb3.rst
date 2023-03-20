.. _psrsearchb4:

Searching for new pulsars
--------------------------

This part of the tutorial aims to demostrate the process of pulsar search using observations from GMRT. We will use band-4 (550-750 MHz) data of a test pulsar.   


Introduction
~~~~~~~~~~~~~

Pulsars are fascinating extraterrestrial objects that produce pulse like emission with very precise periodicity. These objects are often detected in time domain radio observations. In this tutorial, we will learn how to search for these objects using the strengths of individual pulsed emission as well as their precise periodic behaviour. This tutorial will require PRESTO installed on your machine along with a filterbank file with pulsar in it.
 
Checking filterbank file
~~~~~~~~~~~~~~~~~~~~~~~~~

Filterbank file is a file-format for beam data from radio telescopes. This file has the metadata of the observation along with the raw beamdata recorded in the telescope. Use following syntax to check the metadata,

.. code-block::

   header <filterbank_file.fil>
   
.. figure:: /images/pulsar/header_tutorial.png
   :align: center
   
   *Example of a filterbank header*

Dedispersion
~~~~~~~~~~~~~

For an unknown pulsar, we don't know the dispersion caused by the intersteller medium. We try a lot of trial DM values and perform search in each dedispersed time series. We will use ``prepsubband`` to dedisperse the filterbank file at a range of trial DM values,

.. code-block:: 
   
   prepsubband -lodm <smallest_trial_dm> -dmstep <seperation_between_dms> -numdms <number_of_trial_dms> -numout <number_of_output_samples> <input_fil_name> -o <output_base_name>

Please note that number of output samples should be an even number (in order to get a straight-forwand FFT in later stage). This will create a number of dedispersed time series with names output_base_name_dm.dat. These are the time series that will be used for single pulse search as well as periodicity search.


Exploring the time series
~~~~~~~~~~~~~~~~~~~~~~~~~~

Now, since we have the dedispersed time series, we can try to visualize the pulses in the time series. Use ``exploredat`` to visualize the time series,

.. code-block::

   exploredat output_base_name.dat

This will generate an interactive plot,

.. figure:: /images/pulsar/J1534+5334_exploredat.png
   :alt: Importgmrt log
   :align: center
   :scale: 70% 
   
   *Output plot of exploredat. One can zoom in, zoom out, and move around in this plot.*

You can even try to estimate the period of the repeating signal from this plot.


Single pulse search
~~~~~~~~~~~~~~~~~~~~

Though we have seen the single pulses from the pulsar directly in the time series, there is a formal way to search for single pulses in the time series. 

.. code-block::

   single_pulse_search.py -t <significance_threshold> -m <maximum_width> <dedispersed_time_Series.dat>


This generates a text file with information of individual detected pulses for each time series. Now, we need to combine the results of each search in a single summary plot,

.. code-block::

   single_pulse_search.py -t <significance_threshold> <_dedispersed_time_Series.singlepulse>



.. figure:: /images/pulsar/single_pulse_tutorial.png
   :alt: Listobs log
   :align: center
   :scale: 70% 
   
   *Output of single pulse search. We get location of pulses, number of pulses, and single pulse S/Ns as a function of DM.*
   
You can note down timestamp of some strong pulses from the text file and try to varify it's presence in the time series using ``exploredat``. The candidate plot of a single pulse is contains various diagonstic plots,

.. figure:: /images/pulsar/single_pulse_cand.png
   :alt: Listobs log
   :align: center
   :scale: 20% 
   
   *Diagnostic plot of a single pulse candidate.*

   
.. figure:: /images/pulsar/single_pulse_noise.png
   :alt: Listobs log
   :align: center
   :scale: 20% 
   
   *False single pulse candidate.*
   
  
Search for faint pulsars using periodicity search
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are pulsars that are weak and can not be seen in single pulses. To recover such puslars, we use their periodic property to get a good indication of their presence in the time series. Here, we will see one such method called FFT based periodicity search. The process is to take Fourier transform of the time series and search for strong signal in the power spectra. 
First we will see how FFT of a time series with repeating signal looks like. To compute the FFT of the time series,

.. code-block::
   
   realfft -fwd <dedispersed_time_series.dat>
   
This will create a file with .fft extension. To see this file in a plot,

.. code-block::
   
   explorefft <dedispersed_time_series.fft>
   
You will see signal at fundamental and subsequent harmonics.

.. figure:: /images/pulsar/fft_tutorial.png
   :alt: Listobs log
   :align: center
   :scale: 70% 
   
   *Visualization of power-spectra of a time series. Strong signal can be seen at the fundamental and subsequent harmoics.*
   
An algorithm called harmonic summing is used to add up signal power in the harmonics to increase the detection significance. This algorithm is implemented in ``accelsearch``.
 We use ``accelsearch`` to search for periodic signal, after cleaning the FFT of timeseries using ``rednoise``
 
 .. code-block::

   rednoise <dedispersed_time_series.fft>
   aacelsearch -zmax 0 -numharm 16 <dedispersed_time_series_red.fft>
   
This will generate a text file containing the results of the search.

.. figure:: /images/pulsar/Accelsearch_result.png
   :alt: Listobs log
   :align: center
   :scale: 70% 
   
   *Result of accelsearch. First detection is the first harmonic of the pulsar signal.*

Folding filterbank file
~~~~~~~~~~~~~~~~~~~~~~~~

Once we know the correct period and DM of the pulsar, we can fold the filterbank file to generate characteristic plots of the pulsar. We use ``prepfold`` to fold a filterbank,

.. code-block::

   prepfold -p <period> -dm <DM> -nosearch -zerodm <filterbank_file.fil>
   
.. figure:: /images/pulsar/tutorial_profile.png
   :alt: Listobs log
   :align: center
   :scale: 70% 
   
   *Result of prepfold. Profile of the pulsar along with subintegration vs phase, frequency vs phase, S/N vs DM, S/N vs period plots.*
   

.. figure:: /images/pulsar/J1533_noise_cand.png
   :alt: Listobs log
   :align: center
   :scale: 70% 
   
   *A false candidate. The pulse seen in the profile is caused by RFI in the time series.*
   
There are lot of such plots for each trial DM in a blind search. All these plots are inspected either by eye or an machine learning based classifier to tell which one is a pulsar and which one is not. Once a pulsar is identified, one needs to check if it's a new pulsar or an already known pulsar.


