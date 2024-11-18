.. _HIabs:

Spectral line  data reduction using CASA (source 1543+480)
========================================================

The tutorial is intended to provide you an introduction to the basic steps involved in 
the analysis of spectral line data from the GMRT. 
We will be using a band-4 (550 - 900 MHz) dataset. 
In a targeted spectral line observation, one is already aware of the location of the line 
in terms of frequency and the corresponding spectral channels. 

CASA version used for the tutorial: 6.5.2

Introduction
-------------

You need to have CASA installed on your machine. You also need the data to be 
available on your disk.

From the GMRT online archive, you can download data in "lta" or "FITS" format. If you downloaded the data in lta format then you will need to do the following steps to convert it to FITS format. You can download the pre-compiled binary files "listscan" and "gvfits" from the observatory. The data can also be in "UVDATA" format, which is a "FITS" format and hence the following steps can be used for this data type too.
Note that the steps mentioned for file vis='example.ms' are more general, and the file mentioned for vis='1543+480.ms' is the particular dataset provided. 

LTA to FITS conversion
+++++++++++++++++++++++

The GMRT data is available in raw telescope format, called LTA, which stands for long term accumulation. For the data to be converted to FITS format, listscan and gvfits are used. This binary executable can be downloaded from GMRT portal, and to be made an executable, the command "chmod +x" can be used as follows:

.. code-block:: 
   
   chmod +x listscan
   chmod +x gvfits

For the LTA file name test.lta, the conversion is done using following command:

.. code-block:: 
         
   ./listscan test.lta


At the end of this, a file with extension .log is created. Using an editor like vi or gedit one can add the file name for the fits file. The next step is to run gvfits on this file.

.. code-block:: 
   
   ./gvfits test.log 

The file TEST.FITS contains your visibilities in FITS format.

FITS to MS conversion
++++++++++++++++++++++

At this stage, start CASA using the command ``casa`` on your terminal. You will be on the Ipython prompt, and a logger window will appear. 
The remaining analysis will be done at the CASA Ipython prompt. We use the CASA task importgmrt to convert 
data from FITS to MS format. In our case, the FITS file is already provided, so we proceed with it.

.. code-block::

   casa
   tget importgmrt
   inp
   fitsfile='1543+480.FITS'
   vis='1543+480.ms'
   go 

The output file *1543+480.ms* file will contain your visibilities in MS format.


An alternative way to run the task is as follows:

.. code-block::

   importgmrt(fitsfile='1543+480.FITS',vis='1543+480.ms')

For the tutorial, we will follow the first method. Note that instead of "tget importgmrt", the command "inp importgmrt" would also work based on the version of CASA you are using. For the latter case, to run the command, instead of "go", the command "go importgmrt" should be used.

The MS format stores the visibility data in the form of a table. The *data* column contains the data. Operations like calibration and deconvolution will produce additional columns, such as the *corrected* and *model* data columns.

**Bonus example** 
There can be cases where the data file contains multiple observations with two or more targets. In this case, we may wish to split the dataset containing only the target we are interested in, along with the calibrators related to it. For example, if we would like to split the fields ids 0,1,2 and 7 with channels from 1403 to 3450, it is done as follows:

.. code-block::

   tget split
   vis=’example.ms’
   outputvis='examplesplit.ms’
   field=’0,1,2,7’
   spw=’0:1403∼3450’
   go 

Data inspection
----------------

You will first want to know about the contents of the MS file. 
The task listobs will provide a summary of the contents of your dataset in the logger window. 

.. code-block::

   tget listobs
   vis='1543+480.ms'
   go 

.. figure:: /images/abs_line/listobs_log.png
   :alt: Listobs log 
   :align: center
   :scale: 70% 
   
   *Output of the task listobs in the logger.*

You can also choose to save the output to a text file so that you can refer to it when needed without having to run the task.

.. code-block::

   tget listobs
   vis='1543+480.ms'
   listfile='listobs-out.txt' 
   go 

Note the scans, field IDs, source names, number of channels, total bandwidth, channel width and central frequency for your observations. Identify the science target, its corresponding flux calibrators and the phase calibrator. In the tutorial dataset, there are 512 channels in the band from 608 MHz to 641 MHz, giving a spectral resolution of 65.1 KHz.  
Field IDs can be used in subsequent tasks to choose sources instead of their names (e.g., 3C48, 0311+430, etc.). In the tutorial dataset presented, a flux calibrator (3C286), phase calibrator (1602+334), and target (1543+480) are present, with field id 0, 1 and 2, respectively. 

Using online databases like NASA NED or SIMBAD, we learn more about the target, such as its type, redshift, etc. From the redshift, we can determine the frequency at which we expect the spectral line to be present. In the tutorial dataset given, the target 1543+480, also known as WISEA J154508.52+475154.6 (can be found from NED), is a Quasar (QSO) at a redshift of z=1.277. From this, using f' = fo/(1+z), where fo is the rest frequency of the line, 1420 MHz, we get the frequency at which the line should be, which comes out to be about 623.62 MHz. Note that this is a case where the absorbing is close to the background source. If the gas is present somewhere between us and the source/target, we won't be able to locate the frequency of the line in this way, as the redshift of the gas would be unknown.

The task ``plotms`` is used to plot the data. It opens a GUI in which you can choose to display portions of your data.
Go through the help for plotms GUI in CASA documentation for more details on its usage (https://casadocs.readthedocs.io/en/v6.2.0/api/tt/casatasks.visualization.plotms.html).
It is important to make a good choice of parameters to plot so that you do not end up asking to plot too much data simultaneously. Our aim is to inspect the data for non-working antennas. A good choice would be to limit the fields to calibrators, choose a single channel plotting Amp vs. time, and iterate over antennas. 
Another good plot for inspection is to choose a single antenna, select all the channels and plot Amp vs. channel while iterating 
over baselines.

.. admonition:: Note

   For spectral line analysis, usually, the targets are point sources, and we do not require the use of data from closely spaced central square baselines of    
   uGMRT. This is because these are mostly relevant for imaging extended objects and are also prone to have higher RFIs (Radio frequency 
   interferences). Hence, they are omitted from the entire process by setting the condition uvrange='>1.5km' in the functions.

In plotms, to view the raw data as a function of time for a particular frequency, say channel 400, set spw as 0:400, uvrange as >1.5km and corr as rr. From the Axes tab, choose x-axis as time and data as amp. One can also iterate over antennas in the Page tab seen on the left of the plotms window should be selected. 
It is good to set the inputs for a task to default before running it.  

.. code-block::

   default(plotms)
   plotms

.. figure:: /images/abs_line/plotms_timerawdata.png
   :alt: Plotms screenshot amp vs time
   :align: center
   :scale: 70% 
   
   *Screenshot of plotms. Fields 0 and 1 for channel 400 and correlation rr are plotted. Left is the data using all uv plane, and right excludes the short baselines uvrange < 1.5km. Note the cleaner data and lower RFI in the latter plot.*


Flagging
---------

Editing out bad data (e.g., non-working antennas, RFI-affected channels, etc.) is termed flagging. In our MS file, 
the bad data will be marked with flags and not actually removed as such - thus the term *flagging*.
The task ``flagdata`` will be used to flag the data. See the detailed CASA documentation on flagging using the 
task ``flagdata``.

Here, some typical steps for flagging are outlined to get you started.

Usually, the first spectral channel is saturated. Thus, it is a good idea to flag the first spectral channel.

.. code-block::

   tget flagdata
   default
   inp 
   vis = '1543+480.ms'
   mode = 'manual'
   spw = '0:0'
   savepars = True
   cmdreason = 'badchan'
   go 

Usually, it is wise to flag the first and last records of scan data, which is done as follows:

.. code-block::

   default(flagdata)
   inp flagdata
   vis = '1543+48.ms'
   mode = 'quack'
   quackmode = 'beg'
   quackinterval = 10
   savepars = True
   cmdreason ='quackbeg'
   go

.. code-block::

   default(flagdata)
   inp flagdata
   vis = '1543+48.ms'
   mode = 'quack'
   quackmode = 'endb'
   quackinterval = 10
   savepars = True
   cmdreason ='quackend'
   go

   
In the next step, we would like to flag antennas that were not working.
Using ``plotms``, plot the freq vs amp(data) with the iteration of antenna, with uvrange>1.5 km, and note the behaviour for all the scans. The condition of uvrange>1.5 km ensures we do not use the central square baselines for spectral line imaging.
Find out which antennas were not working. Non-working antennas *generally* show up as those having a very small relative amplitude, even on bright calibrators, show no relative change of amplitude for calibrators and target sources and the phases towards calibrator sources on any given baseline will be randomly distributed between -180 to 180 degrees. If such antennas are found in the data, those can be flagged using 
the task ``flagdata``. 
**Only an example is provided here - you need to locate the bad antennas in the tutorial data and flag those.** Remember also that some antennas may not be bad at all times. However, if an antenna stops working while on the target source, it can be difficult to find out. Thus, a decision should be made based on the secondary calibrator scans. Depending on when such antennas stopped working, you can choose to flag them for that duration. Check the two polarizations separately.

Although ``plotms`` provides options for flagging data interactively, at this stage, we will choose to just locate the bad data and flag it 
 using the task ``flagdata``.

The following command is an example where the three antennas, namely E02, S02 and W06, are non-functioning and are flagged. **For the dataset given to you, this may not be the case and hence check for bad antennas.** If all antennas are functioning, **skip** this step.


.. code-block::

   tget flagdata
   default
   inp 
   vis = 'example.ms'
   mode = 'manual'
   antenna = 'E02, S02, W06'
   savepars = True
   cmdreason = 'badant'
   inp
   go 

It is a good idea to review the inputs to the task using (``inp``) before running it.

Radio Frequency Interferences (RFI) are man-made radio band signals that enter the data and are unwanted. Signals such as 
those produced by FM radio, mobile, satellite and aircraft communications. They are confined to narrow bands in frequency and will appear in 
frequency channels with very high amplitudes. It is not easy to remove the RFI from such channels and recover our astronomical 
signal. Thus, we will flag the affected channels (individual or a group of channels). There are many ways to flag RFI, such as manually inspecting the spectra or using automated flaggers that look for outliers based on thresholds.

For the tutorial dataset given, upon plotting field ID 0 with freq vs amp(data), we see that there are a few RFI spikes. Select a few data points on the spike (see figure), and look up on the casa log. 

.. figure:: /images/abs_line/rfi_spikes.png
   :alt: Plotms screenshot rfi spike 1
   :align: center
   :scale: 70% 
   
   *Screenshot of RFI spikes. From the panel below in plotms, choose 'mark regions' and select a few points in spike.*

.. figure:: /images/abs_line/rfi_spikes2.png
   :alt: Plotms screenshot rfi spike 2
   :align: center
   :scale: 70% 
   
   *After selection, choose the option 'locate' from panel below and check the log file.*

.. figure:: /images/abs_line/rfi_spikes3.png
   :alt: Log screenshot rfi spike 3
   :align: center
   :scale: 70% 
   
   *Screenshot of casa log. Note down the antenna baselines, scan number, channels, etc in which the RFI is present. We need to flag it.*

Flag the corresponding channels/ baselines containing the RFI spikes individually. An example to flag a particular spike present in **all fields** at channel # 302 is shown below: 

.. code-block::

   tget flagdata
   default
   inp
   mode='manual'
   vis='1543+480.ms'
   spw='0:302'
   savepars = True
   go
   

Similarly, flag the other persistent RFI spikes. The RFI spikes need to be carefully looked at, and only the essential faulty channels/baselines need to be flagged. There may be cases where the RFI is transient, not present throughout the observation, and may not be present in all fields. These factors need to be carefully examined before flagging.

.. code-block::

   tget flagdata
   default
   inp
   spw='0:111,0:210,0:234,0:357,0:480'
   go

Tick the reload option on plotms and plot again on the plotms to verify if the flagging is reflected.

.. figure:: /images/abs_line/rfi_spikes_removed.png
   :alt: Plotms screenshot rfi spike removed
   :align: center
   :scale: 70% 
   
   *Screenshot of plotms after flagging RFI spikes. Note that the spikes are no longer present, and the selected region can be unselected using the 'clear region' from the panel below.*


**Bonus example** 
If, for any reason, you flag the wrong data and want to reverse the flag, the command "flag manager" is used. 

.. code-block::

   tget flagmanager
   default
   inp
   vis='example.ms'
   mode='list'
   go

This displays the list of all flag operations performed. Note the flag version name from this list, and say the latest flag that you performed has the name flagdata_4. To unflag this latest flag operation, the following command is used:

.. code-block::

   tget flagmanager
   default
   inp
   vis='example.ms'
   mode='restore'
   versionname='flagdata_4'
   go



Initial Gain calibration before flagging unwanted data
---------------------------------------------------------

Pick a clean line-free channel (or if many gain solutions fail due to low SNR, a bunch of channels that do not have any RFI and do not contain the target spectral line). This would be a reference, which sets its gain amp as 1 and gain phase as zero, and the gain calibration is done relative to it and later applied to all channels. The number of channels to be selected for averaging depends on SNR we require (if too many solutions fail and get flagged in gaincal for minsnr=5, average more channels to increase SNR). If this fails as well, reduce the minsnr threshold. Typically, a single channel is chosen for this, say channel 100, hence the command spw='0:100'. 
Create a directory for the solution tables and also one for images as follows (use "!" mark at the beginning if the commands are written within the casa ipython prompt):

.. code-block::

   !mkdir caltables
   !mkdir images

The field ID of the flux calibrator is 0, and that of the phase calibrator is 1. Hence, the first round of initial gain calibration is done only on calibrators (and **not on target**) as follows:

.. code-block::

   tget gaincal
   inp
   vis='1543+480.ms'
   caltable='caltables/gainsol.apcal'
   solint='int'
   uvrange='>1.5km'
   minsnr=5.0
   field='0,1'
   spw='0:100'
   go


Note that since the source would be a point source, we have excluded the short baselines by uvrange='>1.5km'. This is followed by an ``applycal``, applying the calibration to all the channels of the calibrators only.

.. code-block::

   tget applycal
   inp
   vis='1543+480.ms'
   field='0,1'
   gaintable=['caltables/gainsol.apcal']
   calwt=[False]
   go
   

It is wise to keep track of the flagging percentage in the data. If too much data gets flagged, there won't be much useful data left. The task ``flagdata`` in the mode of 'summary' allows us to keep track of this. Use the following commands:

.. code-block::

   tget flagdata
   inp
   default
   vis='1543+480.ms'
   mode='summary'
   go



In the plotms, plot amp vs uvdist with the corrected data column for the entire channel, and check the calibrator data starting with field 0. Inspect and flag the baselines that jump around too much from the pack. Ideally, the pack must be centred around an amp of 1, with the baselines staying in and around that value. If the entire line jumps from this median by a large amount, it can be flagged.



.. figure:: /images/abs_line/uvdistvsamp_before1.png
   :alt: Plotms screenshot before flag calibration
   :align: center
   :scale: 70% 
   
   *Screenshot of plotms for uvdist vs amp (corrected). Note that there is a lot of bad data, and baselines are jumping.*

As there are not many obvious visible bad data, we can run a round of automated flagger ``rflag`` on the calibrator fields.

.. code-block::

   tget flagdata
   vis='1543+480.ms'
   mode='rflag'
   field='0,1'
   datacolumn='corrected'
   go


The plot is shown below:   

.. figure:: /images/abs_line/uvdistvsamp_after1.png
   :alt: Plotms screenshot after flag calibration
   :align: center
   :scale: 70% 
   
   *Screenshot of plotms for uvdist vs amp (corrected). Note that most of the baselines are packed around amp = 1 with almost no outliers.*

We need to check if amp and/or phase plotted w.r.to uvdist is flat because these are point sources at phase centre, so amp should not depend on uvdist, and phase should also not depend on uvdist. To summarize, check uvdist vs amp corrected plots, with antenna iteration and baseline colourization or baseline iteration and antenna1/corr colourization, if required channels averaged, field by field with uvrange>1.5km. 


Absolute flux density calibration
----------------------------------

We use the task ``setjy`` to set the flux densities of the standard flux calibrators in the data here before redoing the ``gaincal``. Following are the commands for the task 'setjy', which is to be done for all flux calibrator fields present:

.. code-block::

   tget setjy
   default
   inp
   vis='1543+480.ms' 
   field='0'
   usescratch=True
   go   

The flux values assigned can be verified using the VLA calibrator manual, and the obtained value must be close to the wavelength band value from the manual where the spectral line is expected. For the calibrator in the tutorial dataset, we find that the setjy flux level is 21.71 Jy, which is close to the reference level in the calibrator manual. Now, we can perform the gain calibration on calibrators using the single channel (or a bunch of channels if used, as explained earlier) and apply it to all the channels and fields except the target source. 

.. code-block::

   tget gaincal
   field='0,1'
   caltable='caltables/gainsol_1.apcal'
   uvrange='>1.5km'
   spw='0:100'
   solint='int'
   go

   tget applycal
   field='0,1'
   gaintable=['caltables/gainsol_1.apcal']
   go


As we have completed the setjy, the flux of flux calibrators, which was centred about 1, will now be centred on their respective flux values. Note that the standard, 'Perley-Butler 2017' identifies most of the flux calibrators uGMRT uses. Some calibrators may not be recognized, in which case the standard 'Stevens-Reynolds 2016' can be used. If the calibrator is still not recognized by these standards, the flux values need to be entered manually for the calibrator.

.. figure:: /images/abs_line/vla_cal_manual_setjy.png
   :alt: Log screenshot after setjy
   :align: center
   :scale: 70% 
   
   *VLA calibrator manual flux densities for calibrator 3C286. Note that the assigned flux for the calibrator 3C286 is 21.71 Jy. Since the central frequency of our dataset is about 623 MHz, which is about 48.1 cm wavelength, from the VLA calibrator manual, we see that the flux value lies between the 20cm band and 90cm band.*

We would want to transfer the flux calibration solutions to the phase calibrator so that its flux can be calibrated and scaled. If the data has two or more flux calibrators, we may choose the brightest one having cleaner and lower flagged data to use as a reference to transfer the solutions from. To transfer the solution from flux calibrator field 0 to phase calibrator field 1:

.. code-block::

   tget fluxscale
   inp
   vis='1543+480.ms'
   caltable='caltables/gainsol_1.apcal'
   fluxtable='caltables/gainsol_1.fcal'
   reference=['0']
   transfer=['1']
   go

After the task ``fluxscale``, the reported flux density of the phase calibrator must be compared with the standard flux density from VLA calibrator manual. 

.. figure:: /images/abs_line/fluxscale_phasecal_vla_cali.png
   :alt: Log screenshot after setjy
   :align: center
   :scale: 70% 
   *VLA calibrator manual flux densities for phase calibrator 1602+334.*


Initial Bandpass calibration
----------------------------

In this step, an initial bandpass calibration is done on flux calibrators. We can also use the phase calibrator for this purpose if it is bright enough, more precisely if the relation tcal > tobj(Sobj/Scal)^2 holds true, where tcal is the total time spent observing the calibrator, tobj is time spent observing the target, Sobj and Scal are the flux densities of the target and calibrator respectively. The observation time values can be found from ``listobs``; Sobj can be found in databases like NVSS survey by inputting the coordinates of the target, and Scal is found from fluxscale. We also need to choose a reference antenna for bandpass calibration, where we select the best-behaving antenna with ideally the least data flagged.

.. admonition:: Note
   For flux values of target: https://www.cv.nrao.edu/nvss/NVSSlist.shtml 

By working out this math, we find that the phase cal is bright enough to be used in bandpass calibration. We included it in bandpass calibration along with flux calibrator as:

.. code-block::

   tget bandpass
   default
   inp
   vis='1543+480.ms'
   caltable='caltables/bpass_1.bcal'
   refant='C03'
   gaintable=['caltables/gainsol_1.apcal']
   field='0,1'
   minsnr=5.0
   uvrange='>1.5km'
   go

The solutions are first applied to the flux calibrator field by applycal, and a round of automated flagger rflag can be used if required. 

.. code-block::

   tget applycal
   inp
   vis='1543+480.ms'
   field='0'
   gaintable=['caltables/gainsol_1.apcal','caltables/bpass_1.bcal']
   go


   tget flagdata
   mode='rflag'
   spw=' '
   field='0'
   datacolumn='corrected'
   timedevscale=4.5
   freqdevscale=4.5
   go

After this, the amp(corrected) vs frequency plot would look like the figure below, where the flux is peaked and centred around the limit set by setjy, and we see a band.


.. figure:: /images/abs_line/field0_post_inibpass_rflag.png
   :alt: Screenshot of the plotms after initial bpass and rflag
   :align: center
   :scale: 80% 
   
   *Screenshot of amp(corrected) vs frequency on plotms for field 0.*

Examine the bandpass table using ``plotms``. Choose the bandpass table bpass_1.bcal in data and check the plots Amp Vs Channels and Phase Vs Channels  iterated over antennas. Note that solution tables do not take uvrange or corr inputs on plotms.

.. figure:: /images/abs_line/initialbpass_ampvsfreq.png
   :alt: Screenshot of the plotms for bandpass table
   :align: center
   :scale: 80% 
   
   *Screenshot of amp(data) vs frequency for the initial bandpass solution table on plotms.*

Note the shape of the band across the frequencies.


Delay calibration and final Bandpass calibration
------------------------------------------------

In the delay calibration as well a reference antenna is required. Here "C03" is only taken as an example. You may use any antenna that is working for the whole duration of the observation. We perform delay calibration only with flux calibrator field used for fluxscale and not with all calibrators.


.. code-block::

   !cp  gaincal.last gaincal.last.bk
   tget gaincal
   default
   inp
   vis='1543+480.ms'                                                    
   field='0'
   gaintype='K'                                                        
   caltable='caltables/delay.kcal'                                     
   refant='C03'
   go

Copying the solutions to a new table, we do a round of amp-phase gaincal with all calibrator fields and solution types of ’int’ or different interval sizes like ’2min’ can be explored.

.. code-block::

   !cp gaincal.last gaincal.last.kcal
   !cp gaincal.last.bk gaincal.last
   tget gaincal
   default
   inp
   vis='1543+480.ms'
   spw='0:100'
   solint='int'
   minsnr=5.0
   uvrange>'1.5km'
   field='0,1'
   gaintable=['caltables/delay.kcal']
   caltable='caltables/gainsol_int.apcal'
   go

   solint='2min'
   caltable='caltables/gainsol_2m.apcal'
   go

The task ``fluxscale`` is performed again on both the solutions with the same parameters and flux calibrator field used earlier in fluxscale and save the solutions which will be used to transfer the final bandpass solutions to all fields, including the target field. 


.. code-block::

   tget fluxscale
   caltable='caltables/gainsol_int.apcal'
   fluxtable='caltables/gainsol_int.fcal'
   go
   caltable='caltables/gainsol_2m.apcal'
   fluxtable='caltables/gainsol_2m.fcal'
   go


The bandpass calibration solutions are found using all (if phase calibrator was also used in initial bandpass, else only flux calibrators are used) the calibrator fields :


.. code-block::

   tget bandpass
   inp
   vis='1543+480.ms'
   field='0,1'
   combine=''
   refant='C03'
   minsnr=5.0
   gaintable=['caltables/delay.kcal','caltables/gainsol_int.apcal']
   caltable='caltables/bandpass_finalint.bcal' 
   go

The solutions are applied to all fields, including the target:

.. code-block::

   tget applycal
   gaintable=['caltables/delay.kcal','caltables/bandpass_finalint.bcal','caltables/gainsol_int.fcal'] 
   field=''
   go

The bandpass solution tables in plotms look like the following, where amp vs freq and gain phase vs freq are plotted for the final bandpass solution table:
 
.. figure:: /images/abs_line/finalbpass_ampvsfreq.png
   :alt: Screenshot of the plotms after final bpass amp vs freq
   :align: center
   :scale: 80% 
   
   *Screenshot of amp(data) vs frequency for the final bandpass solution table on plotms.*

.. figure:: /images/abs_line/finalbpass_gainphasevsfreq.png
   :alt: Screenshot of the plotms after final bpass gain phase vs freq
   :align: center
   :scale: 80% 
   
   *Screenshot of gain phase(data) vs frequency for the final bandpass solution table on plotms.*


At this point, we should be able to see the spectral line features in plotms in the visibility domain upon plotting the target field amp (corrected) vs channel and averaging in time, scan and baselines. This helps us determine the channel number where the line is present and to choose a bunch of channels containing the entire line width to be used later in self-calibration to avoid cleaning these channels and potentially erasing the line features.

.. figure:: /images/abs_line/postbpass_field2_avg.png
   :alt: Screenshot of the plotms after final bpass amp (corrected) vs chan with time and baseline averaging
   :align: center
   :scale: 80%
   
   *Screenshot of amp(corrected) vs frequency for the calibrated ms file with time and baseline averaging on plotms. Note the parameters set for the said averaging.*

We run a round of automated 'rflag' on the source field to remove bad data.

.. code-block::

   tget flagdata
   vis='1543+480.ms'
   mode='rflag'
   field='2'
   datacolumn='corrected'
   timedevscale=4.5
   freqdevscale=4.5
   go



Splitting the calibrated target source data
--------------------------------------------

We will split the calibrated target source data into a new file and do the subsequent analysis on that file.
Create a new directory named 'source'. We will split the target and save the new MS file in this directory. In the tutorial dataset, the target has field ID of 2, and is used in the "split" task as follows:

.. code-block::

   !mkdir source
   tget split 
   default
   inp
   field='2'                                                        
   vis='1543+480.ms'                                                
   outputvis='source/source.ms' 
   go


If the data set is too large and has many channels of data, for instance, 2048 channels (standard uGMRT GWB data have a channel width of 12.207 kHz, giving a bandwidth of 25MHz for 2048 channels), to save on computation load and time, the file is can be further split into a lower resolution, the channel averaged coarse MS file upon which self-calibration task can be performed. For example, a 2048-channel source MS file can be split by channel averaging of 20 channels chosen arbitrarily, giving a low-resolution coarse file of about 101 channels.  For this, width = 20 must be given in task ``split``.
Since our tutorial dataset contains 512 channels, we can skip this step. Following is an example depicting the splitting of an ms file into a coarser resolution file. Please skip this step for tutorial data.

.. code-block::

   cd source
   tget split
   default
   inp
   vis='example.ms'
   outputvis='example_coarse.ms'
   width=20
   datacolumn='data'
   go 


It is easier and faster to self-calibrate on a coarse file and later transfer the solutions to a higher resolution split file to proceed with imaging.

.. admonition: Note
   We have not taken any special note of the spectral line in steps till now. The channels 
   containing the line must not be treated special and usual steps of flagging and initial calibration must be performed. The important 
   deviation arrives during self-calibration, where we have to exclude the channel range where line features are present or expected to 
   occur.


Self-calibration process
------------------------

This is an iterative process. The model from the first ``tclean`` is used to calibrate the data and the corrected data are then imaged to make a better model and the process is repeated. The order of the tasks is tclean, gaincal, applycal, tclean. In this section, we perform self-calibration on the source file (if a coarse file is created, these steps need to be done on that file and later transferred to the source file). In the following example, we perform it on the source file. A test dirty image can be created before the self-cal run to ensure the parameters are used in cleaning the image using the task "clean" and for self-cal cycles. The parameter uvtaper is found by plotting 'uvwave' vs amp in plotms for the visibility source.ms file and noting the distance where the tapering must be smoothed from, which would be some distance before the amp starts going to zero. 

.. figure:: /images/abs_line/uvtaper.png
   :alt: Screenshot of the plotms Amp Vs uvwave for uvtaper
   :align: center
   :scale: 80% 
   
   *Screenshot of amp(data) vs uvwave for ms file to determine the uvtaper parameter on plotms.*

Inputs to make a dirty image are given as follows, where the first two lines are to create new directories for images and calibration tables:

.. code-block::

   !mkdir images
   !mkdir caltables   
   tget tclean
   inp
   vis='source.ms'  
   cell=['0.14 arcsec']
   imsize = [3000]
   imagename='images/testimage'
   gridder='wproject'
   wprojplanes=-1
   weighting='briggs'
   robust=-0.5
   uvtaper=['40klambda'] 
   uvrange='>1.5km'
   go


The imsize is chosen such that it covers and images the FWHM of the primary beam. The cell size is chosen to be at least a third or more of the expected synthesized beam size. These can be determined from the antenna aperture and wavelength of observation and the longest baseline of uGMRT array, respectively.



**Self-cal cycles:** We start by cleaning the image (deconvolving) by only selecting the channels that do not contain the line. This is done in the ``tclean`` by selecting spw range suitably. 

The cleaning is done interactively by first masking the sources visible in the dialog view, and running the process again using the green arrow button (continue deconvolving with current clean regions) which continues the deconvolution with current clean channels in viewer GUI. We keep adding masks to any new source visible in each step and keep deconvolving until the target source noise level is reached, i.e. until the entire image looks like a uniform noise. The deconvolution is stopped at this point by clicking the red cross button. Then a round of phase-only cal is performed while selecting the same spw range and applying it to all channels. With the same parameters to task ``tclean``, following parameters are updated and subsequently the phase-only cal is done:

.. figure:: /images/abs_line/intcleandialogbox.png
   :alt: Screenshot of the viewer dialog box GUI
   :align: center
   :scale: 80%
   
   *Screenshot of casa viewer interactive windoow dialog menu.*

Note that the spectral line of interest lies near channel 230 in the full resolution source file, so we exclude the line and nearby continuum channels, picking a spectral window of spw='0:0~209,0:271~511' for the self-cal steps.

.. code-block::

   tget tclean
   inp
   imsize=[3000]
   cell=['0.14 arcsec']
   niter=1000000
   interactive=True
   imagename='images/selfcal_0'
   pblimit=-0.01
   savemodel='modelcolumn'
   spw='0:0~209,0:271~511'
   go


The viewer GUI opens automatically, and we will see the following window. Here, the masking of sources is done by checking the 'add' option, drawing contours around the visible source, and double-clicking inside the region to save the mask. To delete a mask, check the 'erase' option, create the boundary around the mask you wish to remove and double-click inside the region.

.. figure:: /images/abs_line/intcleangui.png
   :alt: Screenshot of the viewer dialog GUI
   :align: center
   :scale: 80%
   
   *Screenshot of CASA viewer interactive window.*

The phase-only cal is performed once the viewer GUI closes automatically after you stop the deconvolution when the image noise level is reached as follows:

.. code-block::

   tget gaincal  
   inp
   vis='source.ms'
   caltable='caltables/selfcal_0.pcal'
   calmode='p'
   solint='int'
   spw='0:0~209,0:271~511'
   uvrange='>1.5km'
   minsnr=5.0
   go
   
   tget applycal 
   inp
   vis='source.ms'
   gaintable=['caltables/selfcal_0.pcal']
   calwt=[False] 
   go


This process of interactive tclean and phase-only calibration is done until there seems to be no improvement in noise levels of background RMS, which is found by drawing a rectangular region far from the source and looking at the RMS value of the background noise in the statistics tab. At this point, 4 times RMS is chosen as the threshold and a run of tclean is made with this threshold. This can be done either by setting interactive as False and specifically writing the threshold value as a command in tclean, or it can be set in the interactive mode and the central blue button can be pressed for automatic deconvolution until the set threshold level is reached. Finally, an amplitude and phase calibration (apcal) is performed before creating the final selfcal image. At each step, we just need to change the image name and update the mask for tclean, and for gaincal and applycal, change the gaintable and caltable names. Observe the background noise rms of the image using review, and take four times this value to set the threshold for ``clean ``.

For example, the cycles can be continued in the following manner:


.. code-block::

   tget tclean
   inp
   imagename='images/selfcal_1' 
   go
   
   tget gaincal 
   caltable = 'caltables/selfcal_1.pcal' 
   go 
   
   tget applycal
   inp
   gaintable=['caltables/selfcal_1.pcal']
   go
   
   tget tclean
   inp
   imagename='images/selfcal_2'
   mask = 'images/selfcal_1.mask' 
   go
   
   tget gaincal
   inp
   caltable = 'caltables/selfcal_2.pcal'
   go
   
   tget applycal 
   gaintable=['caltables/selfcal_2.pcal']
   go

Typically, 4 such rounds needs to be done. After this, we do an apcal (amplitude and phase cal) with the same spw parameters and then final tclean. Make sure to enter the latest selfcal image name and caltables properly.

.. code-block::

   tget gaincal
   inp
   calmode='ap'                                                       
   solnorm=True                                                       
   normtype='median'
   caltable = 'caltables/selfcal_4.apcal'
   go
   
   tget applycal
   inp
   gaintable=['caltables/selfcal_4.apcal'] 
   go

Create the final image using ``tclean`` task, either with interactive cleaning or without it. For the tutorial dataset, 4 rounds of phase-only selfcal and 2 rounds of amplitude and phase selfcal will suffice, taking us to an RMS noise levels close to 0.6 mJy.

.. figure:: /images/abs_line/continuum_img.png
   :alt: Screenshot of the viewer dialog GUI
   :align: center
   :scale: 80%
   
   *Final continuum image.*




Apply the calibration and fill the model column of source file
--------------------------------------------------------------

If a coarse resolution file was used in selfcal. the final calibration table of the last selfcal run is applied to source.ms file. For example if the latest selfcal caltables is selfcal_5.apcal, then this is done as: (skip this step for tutorial dataset)


.. code-block::

   tget applycal
   gaintable=['caltables/selfcal_5.apcal'] 
   vis='source.ms'  
   go

Essentially, we use exactly the same applycal command as used during the last round of selfcal but with vis='source.ms', instead of vis='source_coarse.ms'.

The next task is to fill the model column of 'source.ms'. We use the same tclean command as used to create the final image but with the following changes: 

.. code-block::

   tget tclean
   inp
   niter=0 
   spw=''
   uvrange=''
   vis='source.ms'
   mask='' 
   imagenam ='images/savemodelrun'
   startmodel='images/selfcal_6.model' 
   go

Where, in startmodel, use the last selfcal run model. 

Subtraction of continuum
-------------------------

Perform uvsub on source.ms file, which does the table operation
corrected = corrected - model column, 
subtracting the model solutions (which are essentially a model for the continuum sources) from the corrected data visibilities column.


.. code-block::

   tget uvsub      
   inp
   vis='source.ms'
   go

At this point, the data can be checked by plotting amp (corrected) vs frequency for the source.ms file.



Perform continuum subtraction using uvcontsub
---------------------------------------------

The continuum is subtracted from the visibilities of source.ms making sure to exclude HI channels.

Note that the old task will be depreciated. If using the old task, follow the steps:

.. code-block::

   tget uvcontsub_old
   inp
   vis='source.ms'
   fitorder=1
   fitspw='0:0~209,0:271~511'
   want_cont=True
   excludechans = False
   go

For using the new task, follow:

.. code-block::

   tget uvcontsub
   inp
   vis='source.ms'
   outputvis='source_contsub.ms'
   fitorder=1
   datacolumn='corrected'
   fitspec='0:0~209,0:271~511'
   fitmethod='casacore'
   writemodel=True
   go


For the tutorial dataset, please use the task uvcontsub_old.

Excluding the HI channels from uvcontsub, which in this file lies between channel range 210 to 270. A fitorder of 1 is selected, which fits a straight line to the baseline and subtracts it out. After this, we have a new visibility file named source.ms.contsub (if you have used the old task), which is the subtracted visibility. 

We are all set and can make the cube from this file and extract the spectrum. But before that, further fine flagging can be done on these subtracted visibilities could be done. If the selfcal process used coarser resolution file, the same set of flagging process done during the selfcal process on source coarse.ms file should be repeated, for which one can follow the task created by Aditya Chowdhury, NCRA (https://github.com/chowdhuryaditya/repeatflag).
The command to use is repeatflag(visfrom=’source coarse.ms’,visto=’source.ms’). We skip this for the tutorial dataset.

Another essential step is to perform flagging by averaging, i.e. average over time (by large arbitrary value, say 1e8 s) and with iteration of baseline, browse through the amp (corrected) vs frequency for the source.ms.contsub visibilities. Flag the channels in baselines with unusually high amp, ideally the amplitudes must be close to 0 as they are subtracted visibilities. Next average channels (say 40) and browse through time vs amp (corrected) data with baseline iteration and flag faulty timestamps. This is also the standard procedure to reduce the ripples in baseline in the final spectra extracted from image cube.



Make the image cube and extract the spectra
-------------------------------------------

We need to run ``tclean`` with vis='source.ms', specmode='cube', niter=0. We also need to put in all the usual parameters like cell, imsize, weighting, uvrange, uvtaper, as well as spectral-cube-related parameters such as start, nchan, width; one can leave the spectral line-related parameters to their default values if you want to image every single channel and at the highest possible spectral resolution. Also, it is typical to start by using natural weighting and then try other weighting schemes to see if the noise improves.


.. code-block::

   tget tclean 
   inp
   vis='source.ms.contsub'
   weighting='natural'
   imsize=[720]
   cell=['0.14 arcsec']
   outframe='bary'
   imagename = 'images/cube_1'
   gridder='standard'
   savemodel='none'
   uvrange='>1.5km'
   startmodel=''
   specmode='cube' 
   mask=''
   spw=''
   niter=0
   go


.. figure:: /images/abs_line/cube_img.png
   :alt: Screenshot of the viewer dialog GUI
   :align: center
   :scale: 80%
   
   *Cube at the channel where we expect the absorption line.*


Parameters like rest frequency can be given as well, which is the expected frequency of the line. The spectrum is extracted for the location where the target source lies using CASA ``imview``. This is done by first opening the cube image and then opening the final selfcal continuum image simultaneously in one imview window, and then extract the spectrum across a single point at the brightest pixel of the source in the continuum image, using the "collapse" icon above.

.. figure:: /images/abs_line/abs_line1.png
   :alt: Screenshot of the viewer dialog GUI
   :align: center
   :scale: 80%
   
   *Spectrum extracted from the cube along the bright pixel of the source.*

Acknowledgement: We thank Nissim Kanekar for providing the dataset used for this tutorial. We thank Narendra S. for preparing the tutorial and Balpreet Kaur, Aditya Chowdhury and Ruta Kale for editing it further. 


