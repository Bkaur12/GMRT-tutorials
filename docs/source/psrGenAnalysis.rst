.. _psrGenAnalysis:

Pulsar General Data Analysis
----------------------------

..
The tutorial is intended to provide you with a basic introduction to the steps involved in the general analysis of pulsar data, including searching for giant pulses or transient events and timing a newly discovered pulsar. The filterbank data obtained from GMRT are converted to SIGPROC filterbank format using either the filterbank command from SIGPROC or the rficlean command from RFIClean. The tutorial will use data already converted to the SIGPROC filterbank format.

This part of the tutorial aims to demonstrate the process of pulsar detection and determining its properties using observations from GMRT. We will use band-4 (550-750 MHz) data of a test pulsar.

Introduction
~~~~~~~~~~~~~

You need to have PRESTO, SIGPROC, TEMPO2, TEMPO, and their dependencies installed on your machine. You also need the SIGPROC filterbank data to be available on your disk.

The header information of the data from a SIGPROC filterbank file can be inspected using the readfile command (shown below) from PRESTO or the header command from SIGPROC.
 

.. code-block::

   readfile B1133+16_b4_rficlean_digifil.fil
   
.. figure:: /images/pulsar/readfile.png
   :align: center
   
   *Example of a filterbank header*

Data inspection
~~~~~~~~~~~~~

A section of the raw data can be inspected by plotting the data as a function of time and frequency, e.g., using the waterfaller.py command from PRESTO. The command waterfaller.py has several provisions that enable inspecting the data in several ways (e.g., before and after dedispersion, partial and full averaging over the bandwidth, etc.)

.. code-block::

   waterfaller.py B1133+16_b4_rficlean_digifil.fil -T 126.4 -t 0.3 --show-ts
   
.. figure:: /images/pulsar/waterfaller_dispersed.png
   :align: center

The sweep across the frequencies is caused by the propagation effects of the medium. Due to the frequency-dependent refractive index, signal at higher frequencies reach earlier compared to its counterpart at the lower frequencies. The time delay depends on the electron column density along the line of sight (dispersion measure or DM), and can be corrected if the DM is known.

.. code-block::

   waterfaller.py B1133+16_b4_rficlean_digifil.fil -T 126.4 -t 0.3 -d 4.84 --show-ts
   
.. figure:: /images/pulsar/waerfaller_dm.png
   :align: center


Dedispered Time Series
~~~~~~~~~~~~~~~~~~~~~~~~

A small section of data can be dedispersed and viewed using waterfaller.py as seen above.
To get the dedispersed timeseries for the whole data we will use prepdata from PRESTO. For an unknown pulsar, prepsubband can be used to dedisperse the filterbank file at multiple trial DM values.

.. code-block::

   prepdata -nobary -o B1133+16_b4_rficlean_digifil -dm 4.840 B1133+16_b4_rficlean_digifil.fil


Exploring the time series
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now, since we have the dedispersed time series, we can try to visualize the pulses in the time series. Use exploredat to visualize the time series,

.. code-block::

   exploredat B1133+16_b4_rficlean_digifil.dat
   
This will generate an interactive plot,

.. figure:: /images/pulsar/explordat.png
   :alt: Importgmrt log
   :align: center
   :scale: 70% 
   
   *Output plot of exploredat. One can zoom in, zoom out, and move around in this plot.*


Single pulse search
~~~~~~~~~~~~~~~~~~~~

Though we have seen the single pulses from the pulsar directly in the time series, there is a formal way to search for single pulses in the time series. 

.. code-block::

   single_pulse_search.py -t <significance_threshold> -m <maximum_width> <dedispersed_time_Series.dat>


.. figure:: /images/pulsar/singlepulses.png
   :alt: Listobs log
   :align: center
   :scale: 70% 
   
   *Output of single pulse search. We get the location of pulses, number of pulses, and single pulse S/Ns as a function of DM.*
   
You can note down the timestamp of some strong pulses from the text file and try to verify its presence in the time series using ``exploredat``. 


Folding Process
~~~~~~~~~~~~~~~~

There are pulsars that are weak and can not be seen in single pulses. To recover such pulsars, we use their periodic property to get a good indication of their presence in the time series. Once the period, period derivative, and higher order period derivatives (if any) for the pulsar are known, the time stamp array (in mjd) is wrapped around that period (also accounting for any changes in the period with time due to its higher derivatives). This process of wrapping the time sample and adding the intensities at the corresponding spin-phases of the pulsar is known as folding the data. 

Folding filterbank file
~~~~~~~~~~~~~~~~~~~~~~~~

Once we know the correct period and DM of the pulsar, we can fold the filterbank file to generate characteristic plots of the pulsar. We use ``prepfold`` to fold a filterbank,

.. code-block::

   prepfold -p <period> -dm <DM> -nosearch -zerodm <filterbank_file.fil>
   
.. figure:: /images/pulsar/folded_profile.png
   :alt: Listobs log
   :align: center
   :scale: 70% 
   
   *Result of prepfold. Profile of the pulsar along with subintegration vs phase, frequency vs phase, S/N vs DM, S/N vs period plots.*



Once the pulsar is detected, one can find out other properties of the pulsar (duty cycle, flux density, scintillation, etc). as explained below.



Flux determination
~~~~~~~~~~~~~~~~~~~


From the telescope, we obtain a intensity time series (corresponding to the Electric field of radio emission from the source of interest from the sky) in arbitrary units. These arbitrary unit values are then converted to Jansky (Jy) units. For this, we need to know the conversion factor of noise fluctuation (of the blank sky) of the radio telescope. This is already done by the observatory and is given in the form of T_sys/G as a function of radio frequency.

The equation to be used is known as the radiometer equation.

Flux  (in Jansky) = SNR x RMS


Where RMS is the $\\frac{T_{\\text{sys}}}{G} \\times \\frac{1}{\\sqrt{\\text{number of polarizations} \\times \\text{bandwidth} \\times \\text{time interval} \\times \\text{antenna samplings}}} \\times \\sqrt{\\frac{W}{P-W}}$ ($W$ and $P$ are the width and the period of pulsar), which has units of Jansky and SNR is the ratio of signal to noise which makes it unitless (T_sys is the antenna temperature (Kelvin: K) and G is the gain of each antenna which has units of K Jy-1.

Antenna samplings in the above formulae depend on the type of beam used (IA: incoherent array or PA: phased array). For IA the value of antenna samplings would be the total number of antennas (N), in the case of PA the value is N(N-1)/2.


Scintillation
~~~~~~~~~~~~~~


The radio waves (EM waves) emitted from the source, pass through the interstellar medium (ISM) and earth’s ionosphere. The difference between the refractive indices of the medium between the source and observer causes the phases of the EM wave to modulate. This causes a scope of interference between the EM waves with slightly different relative phases (traveling through LOS very close to the source’s direction) and causes constructive and destructive interference patterns. Observationally, this interference pattern injects modulation of the observed flux density (in the form of dynamic spectra). This constructive and destructive interference is seen in timescales of a few seconds to a few hours, and this type of scintillation is called diffractive scintillation. 

The other type of scintillation which has timescales of few months to years, is called refractive scintillation. These are caused by changes in the large refractive index of the intervening medium, which in turn causes to focus/defocus the rays of light emitted from the source.


Radius to Frequency Mapping (RFM)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


Pulsars have a magnetosphere extended up to the light cylinder, which comprises highly magnetized plasma flowing outwards. Considering the dipolar magnetic field, the plasma generated in an electric gap near the surface, pair-cascades and flows along the open field lines. The profile of the pulsar at a given observing frequency represents emission from a corresponding range of emission heights. The plasma in the magnetosphere emits in the radio regime, by the process of coherent curvature radiation (CCR). As per the theory of CCR, different frequencies are emitted at different heights (distance from the surface of the neutron star). This coupled with the multi-component profile ultimately creates a shift in the relative position of different components of the profile. This phenomenon is called the Radius-to-Frequency Mapping.


