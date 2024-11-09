.. _continuumb4:

Timing of Pulsars
------------------

The tutorial is intended to provide you an introduction to the basic steps related to the timing analysis of pulsar using GMRT beam data. We will be using the band-3 (300 - 500 MHz) datasets for this purpose. We will only consider the case of an isolated pulsar for the exercise.

We will use PSRCHIVE and TEMPO2 Softwares.

Introduction
~~~~~~~~~~~~~

The GMRT beam data is in binary format and is converted to "filterbank" format (same binary file with a header). The "filterbank" is folded based on pulsar ephemeris, yielding three-dimensional (time, frequency, phase) folded pulsar data ("pfd"). For this tutorial, we will begin with "pfd" files.

You need to have PSRCHIVE and TEMPO2 installed on your machine. You also need the "pfd" files to be 
available on your disk.

Prediction of phases of pulsar using TEMPO2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Right Ascension (RA) and Declination (DEC) provide the source's two-dimensional position in the sky, which is typically defined by the J2000 system. The pulsar's spin frequency (F0) is defined as the reciprocal of its spining period. F1 is the first derivative of spin frequency with respect to time. Since the pulsar slows down over time, F1 has a negative value. The dispersion measure (DM) quantifies the line of sight column density of electrons. All of these parameters are listed in a file called the pulsar's parameter file. Let's start with a parameter file of pulsar J1646-2142, which contains the pulsar's name, RA, DEC, F0, F1, and DM values.

.. code-block::

   cat J1646-2142.par
   
.. figure:: /images/pulsar/RAS_IM1.png


.. code-block:: 
 
   tempo2 -gr fake -f J1646-2142.par -nobsd 1 -ndobs 7 -randha y -ha 8 -start 59000 -end 60000 -rms 1e-3

This will generate time of arrivals (ToAs) for this pulsar ranging from MJD 59000 to 60000. ToAs are measured for weekly observations (i.e. one observation per seven days). A single ToA is generated for each observation. Each ToA will have a 1e-3 millisecond error (i.e. 1 microsecond). The observation hour angle is chosen at random over an 8-hour period.

At the end of this a file with name J1646-2142.simulate is created. The file J1646-2142.simulate contains the predicted ToAs of the pulsar.

.. code-block:: 
   
   head J1646-2142.simulate 

.. figure:: /images/pulsar/RAS_IM2.png

Normally, the first column represents the file name; in this case, it is the same here because it was predicted using tempo2's fake plugin. The frequency is represented in megahertz in the second column. The third and fourth columns contain the ToAs for the observation and the associated error. Note that we maintained the same error during prediction for all observations. The the last column contains the observatory site code.


The impact of parameter errors on timing data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

First check the phases of the pulsar are derived accurately by running the command:

.. code-block:: 

	tempo2 -nofit -f J1646-2142.par  J1646-2142.simulate -gr plk

The command above displays the difference between the ToAs predicted by the parameter file J1646-2142.par and the ToAs listed in the timing file J1646-2142.simulate. The residuals must be randomly distributed around the zero line if the ToAs listed in the J1646-2142.simulate file matched the prediction from the J1646-2142.par file. The figure below demonstrates this.

.. figure:: /images/pulsar/RAS_IM3.png

Now let's introduce a small error (say around an arc-sec) in postion (RA and DEC) in the parameter file to see the effect of wrong position on the timing data. After putting the offset in the parameter file run the above command again which will show that an error in position has yearly periodic variation in the residuals. This is demonstrated in figure below.
	
.. figure:: /images/pulsar/RAS_IM4.png

If you fit for the RA/DEC for which the error is introduced, you will again get the randomly sampled residuals around the zero line in the post-fit residuals.

Now, reset the parameter file to its default values and try introducing a small error in F0. The residuals for the F0 error show a straight line pattern with no periodicity, as shown below.

.. figure:: /images/pulsar/RAS_IM5.png

Again, if you fit for the F0, you will get the randomly sampled residuals around the zero line in the post-fit residuals.

Try the same thing again with error in F1, and this time you'll notice a parabolic long-term variation like the one in the image below.

.. figure:: /images/pulsar/RAS_IM6.png

Understanding the pattern in the residuals caused by parameter error will assist us in solving a newly discovered pulsar. Once accurate (phase-coherent) solutions for a pulsar are obtained, one can establish its clock-like behaviour and predict its phases at any point in time.
 
Time to solve a Newly Discovered Pulsar
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This time we will start with the real GMRT data set for a newly discovered pulsar J1244-4708 which is discovered in the GMRT High Resolution Southern Sky Survey. 

This pulsar was localised through imaging of a single observational epoch which provides us with its RA and DEC which are accurate to a few arc-seconds. The F0 and DM values of the pulsar were estimated using data from an epoch observation in which the pulsar was strongly detected. We used the reciprocal of the period value (in seconds) given in the waterfall plot of that observation to calculate F0. Similarly, from the same plot, we noted the DM and epoch of the observation. Using these values, we created a parameter file for this pulsar (in this case, J1244-4708.par) that contains an initial guess of its parameters.

.. code-block::

   cat J1244-4708.par
   
.. figure:: /images/pulsar/RAS_IM10.png

In this exercise, we will use GMRT's incoherent array (IA) beam data. This pulsar has been observed 13 times over a baseline of 9 months. The GMRT IA beam data is first converted to "filterbank" format, then dedispersed and folded to "pfd" format. We'll start with the "pfd" files right away.

Time of arrival estimation
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The "pfd.ps" files contain waterfall plots for all "pfd" files. The first step is to go through all of the profiles and find the cleanest one with the sufficiently bright integrated pulse. We will use that profile as our reference or template profile, assuming it is the true profile of the pulsar. Use the following command to view all the profiles.

.. code-block::

	evince *ps
	
After selecting the template profile, the next step is to estimate how many ToAs should be extracted from each observation. We will derive a single ToA for full observation for weak detections, two ToAs for mildly detectect profiles, and four ToAs for bright detections. This approach will be adequate to find the solution in this specific case. To determine the number of ToAs for individual observations, check all the profiles again using the command above.

Let's say you've selected k ToAs for a particular observation A.pfd and B.pfd is your chosen epoch for the template. Then run the following command to get the ToAs for observation A.pfd.

.. code-block::

	get_TOAs.py -n k -t B.bestprof A.pfd >> J1244-4708.tim

This command will find the ToAs for observation A.pfd by cross-correlating it with template B.bestprof. The ToAs will be saved in the J1244-4708.tim timing file.

The value of k varies depending on the detection strength of the pulsar for each observation. So, for each observation, use the same command, but change the file names A.pfd and the corresponding k value while keeping the template same (i.e., B.bestprof). Once all observations' ToAs have been generated, run the command below.

.. code-block::

	tempo2 -nofit -f J1244-4708.par J1244-4708.tim -gr plk
	
This command displays the difference (or residuals) between the observed ToAs (in the J1244-4708.tim file) and the predicted ToAs (predicted using J1244-4708.par file). Because the ToAs for a newly discovered pulsar are not phase-connected, we will not see any general systematic pattern in the residuals. This is illustrated in the figure below.

.. figure:: /images/pulsar/RAS_IM7.png

Now, take a sample of densely sampled points and start fitting from F0. Once you've identified a pattern in the ToAs, try fitting RA and DEC. Finally, you can try fitting F1, but keep in mind that the value of F1 for 9 months of data will be highly unreliable. Once phase coherent solutions are obtained, ToAs will exhibit near-random behaviour around the zero line, as shown in the figure below.

.. figure:: /images/pulsar/RAS_IM8.png

Examine the F0, F1, RA, and DEC values. Since RA and DEC are obtained through imaging, the post-fit RA and DEC should be within a few arc-seconds of the initial guess (i.e., pre-fit values). The F1 value after fitting should be negative and small (compared to error in F0). Below are the fitted values for this exercise.

.. figure:: /images/pulsar/RAS_IM9.png

Finally, on the graphical interface, select "new par" to create the new parameter file. Now that the pulsar's phase-coherent solution has been found, you will be able to predict the ToAs in future observations. The prediction accuracy improves as the number of observations and the timing baseline increase.


