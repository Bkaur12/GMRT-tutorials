.. _HIabs:

Spectral line  data reduction using CASA
=========================================

The tutorial is intended to provide you an introduction to the basic steps involved in 
the analysis of spectral line data from the GMRT. 
We will be using a band-4 (550 - 750 MHz) dataset. 
In a targeted spectral line observation, we know the line frequency and the corresponding spectral channels. 

CASA version used for the tutorial: 6.5.2

Introduction
-------------

You need to have CASA installed on your machine. You also need the data to be 
available on your disk.

From the GMRT online archive you can download data in "lta" or "FITS" format. If you downloaded the data in lta format then you will need to do the following steps to convert it to the FITS format. You can download the pre-compiled binary files "listscan" and "gvfits" from the observatory. The data can also be in "UVDATA" format, which is a "FITS" format and hence the following steps can also be used for this data type.
Note that the steps mentioned for file vis='example.ms' are more general. For this tutorial, we will be using the dataset '0311.ms'. 

LTA to FITS conversion
+++++++++++++++++++++++

The GMRT data is available in raw telescope format, called LTA, which stands for long term accumulation. For the data to be converted to FITS format, listscan and gvfits are used. These binary executables can be downloaded from GMRT portal, and make them executable, use the command "chmod +x" as follows:

.. code-block:: 
   
   chmod +x listscan
   chmod +x gvfits

For the LTA file name test.lta, the conversion is done using following command:

.. code-block:: 
         
   ./listscan test.lta


This command will create a file with extension .log. Using an editor like vi or gedit one can add the file name for the fits file. The next step is to run gvfits on this file.

.. code-block:: 
   
   ./gvfits test.log 

The file TEST.FITS contains your visibilities in FITS format.

FITS to MS conversion
++++++++++++++++++++++

At this stage start CASA using the command ``casa`` on your terminal. You will be on the Ipython prompt and a logger window will appear. 
The remaining analysis will be done at the CASA Ipython prompt. We use the CASA task importgmrt to convert 
data from FITS to MS format. In our case, the FITS file is already provided as "UVDATA" file so we proceed with that FITS file.

.. code-block::

   casa
   tget importgmrt
   inp
   fitsfile='0311.UVDATA'
   vis='0311.ms'
   go 

The output file *0311.ms* file will contain your visibilities in MS format.

.. figure:: /images/specline/importgmrtlog.png
   :alt: Importgmrt log
   :align: center
   :scale: 70% 
   
   *Output of the task importgmrt in the logger and view of the terminal. We will ignore the warnings in the logger for this tutorial.*

An alternative way to run the task is as follows:

.. code-block::

   importgmrt(fitsfile='0311.UVDATA',vis='0311.ms')

For the tutorial, we will follow the first method. Note that instead of "tget importgmrt", the command "inp importgmrt" would also work, based on the version of CASA you are using.

The MS format stores the visibility data in the form of a table. The *data* column contains the data. Operations 
like calibration and deconvolution will produce additional columns such as the *corrected* and *model* data columns.


Data inspection
----------------

You will first want to know about the contents of the MS file. 
The task listobs will provide a summary of the contents of your dataset in the logger window. 

.. code-block::

   tget listobs
   vis='0311.ms'
   go 

.. figure:: /images/specline/listobslog.png
   :alt: Listobs log 
   :align: center
   :scale: 70% 
   
   *Output of the task listobs in the logger.*

You can also choose to save the output to a text file so that you can refer to it when needed without having to run the task.

.. code-block::

   tget listobs
   vis='0311.ms'
   listfile='listobs-out.txt' 
   go 

Note the scans, field ids, source names, number of channels, total bandwidth, channel width and central frequency for your observations. Identify the science target, flux calibrators(s) and the phase calibrator(s).
Both field IDs (e.g. 0,1,2) and names (e.g. 3C48, 0311+430, etc.) can be used in subsequent tasks to select on the corresponding sources. In the tutorial dataset presented, no phase calibrator was used, as the target itself is bright. Hence only a flux calibrator and the target is present, with field id 0 and 1 respectively. Also note that in this tutorial the steps are shown for data where a phase calibrator is also present. **Hence the steps related to phase calibrator operation should be skipped while reducing the sample data provided in the school.**

There can be cases where the data file contains multiple observations with two or more targets. In this case, we may wish to split the dataset containing only the target we are interested in along with the calibrators related to it. For example, if we would like to split the field ids 0,1,2 and 7 with channels from 1403 to 3450, it is done as follows:

.. code-block::

   tget split
   vis=’example.ms’
   outputvis='examplesplit.ms’
   field=’0,1,2,7’
   spw=’0:1403∼3450’
   go 


Using online database, like NASA NED or SIMBAD, we can learn more about the target e.g., type, redshift, etc. Our science target is a Damped Lyman alpha absorber (DLA) at z=2.289, with a background quasar at z=2.87. We will be looking for HI 21cm absorption of the background radiation from the quasar, at the redshift of the intervening absorber. Using the DLA redshift in the relation f' = fo/(1+z), where fo is the rest frequecny of HI 21cm transition fo= 1420.4057 MHz, we get the frequency of the expected HI 21cm absorption line, f'=431.86 MHz. 

The task ``plotms`` is used to plot the data. It opens a GUI where you can select a variety of ways to visualize your data.
Go through the help for plotms GUI in CASA documentation for more details on its usage **link needed**.
It is important to make a good choice of parameters to plot so that you do not end up asking to plot too much data at the same 
time. Our aim is to inspect the data for non-working antennas. A good choice would be to limit the fields to 
calibrators and choosing a single channel and plot Amp Vs Time and iterating over antennas. 
Another good plot for inspection is to choose a single antenna, choose all the channels and plot Amp Vs Channel while iterating 
over baselines.

.. admonition:: Note

   For spectral line analysis, usually the targets are point sources and we do not require the use of data from central square baselines of    
   uGMRT. This is because these are mostly relevant for imaging extended objects and also are prone to have higher RFIs (Radio frequency 
   interferences). Hence they are omitted from the entire process, by setting the condition uvrange='>1.5km' in the functions.

Hence in plotms, to view the data as shown in the following image, set spw as 0:400, uvrange as >1.5km and corr as rr. Iteration over anntennas in the Page tab seen on the left of the plotms window should be selected. From the Axes tab, choose x-axis as time and data as amp.
It is good to set the inputs for a task to default before running it. 

.. code-block::

   default(plotms)
   plotms

.. figure:: /images/specline/plotmsampvstime.png
   :alt: Plotms screenshot amp vs time
   :align: center
   :scale: 70% 
   
   *Screenshot of plotms. Fields 0 and 1 for the channel 400 and correlation rr are plotted for antenna C00.*


Flagging
---------

Editing out bad data (e. g. non-working antennas, RFI affected channels, etc.) is termed as flagging. In our MS file, 
the bad data will be marked with flags and not actually removed as such - thus the term *flagging*.
The task ``flagdata`` will be used to flag the data. See the detailed CASA documentation on flagging using the 
task ``flagdata``.

Here some typical steps of flagging are outlined to get you started.

Usually the first spectral channel is saturated. Thus it is a good idea to flag the first spectral channel.

.. code-block::

   tget flagdata
   default
   inp 
   vis = '0311.ms'
   mode = 'manual'
   spw = '0:0'
   savepars = True
   cmdreason = 'badchan'
   go 

   
In the next step we would like to flag data on antennas that were not working.
Using ``plotms``, plot the freq vs amp(data) with iteration of antenna with uvrange>1.5 km, and note the behaviour for all the scans. The condition of uvrange>1.5 km is given so as to not use the central square baselines for spectral line imaging.
Find out which antennas were not working. Non-working antennas *generally* show up as those having very small amplitude even on bright calibrators, show no relative change of amplitude for calibrators and target sources and the phases towards calibrator sources on any given baseline will be randomly distributed between -180 to 180 degreees. If such antennas are found in the data, those can be flagged using 
the task ``flagdata``. 
**Only an example is provided here - you need to locate the bad antennas in the tutorial data and flag those.** Remember also that some antennas may not be bad at all times. However if an antennas stops working while on the target source, it can be difficult to find out. Thus make a decision based on the secondary calibrator scans. Depending on when such antennas stopped working, you can choose to flag them for that duration. Check the two polarizations separately.

Although ``plotms`` provides options for flagging data interactively, at this stage, we will choose to just locate the bad data and flag it 
 using the task ``flagdata``.

The following command is an example where the three antennas namely E02, S02 and W06 are non functioning and are flagged. **For the dataset given to you, this may not be the case and hence check for bad antennas.** If all antennas are functioning, skip this step.


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

Radio Frequency Interference (RFI) are the manmade radio band signals that enter the data and are unwanted. Signals such as 
those produced by satellites, aircraft communications are confined to narrow bands in the frequency and will appear as 
frequency channels that have very high amplitudes. It is not easy to remove the RFI from such channels and recover our astronomical 
signal. Thus we will flag the affected channels (may be individual or groups of channels). There are many ways to flag RFI - could be done manually after inspecting the spectra or using automated flaggers that look for outliers.

For the dataset given, upon plotting field id 0 with freq vs amp(data), we see that there is a RFI spike. Selecting the data points on the spike (see figure), and look up on the casa log. 

.. figure:: /images/specline/flagrfispike_1.png
   :alt: Plotms screenshot rfi spike 1
   :align: center
   :scale: 70% 
   
   *Screenshot of rfi spike. From the panel below in plotms, choose 'mark regions' and select a few points in spike.*

.. figure:: /images/specline/flagrfispike_2.png
   :alt: Plotms screenshot rfi spike 2
   :align: center
   :scale: 70% 
   
   *Screenshot of rfi spike. After selection, choose the option 'locate' from panel below.*

.. figure:: /images/specline/flagrfispike_3log.png
   :alt: Log screenshot rfi spike 3
   :align: center
   :scale: 70% 
   
   *Screenshot of casa log. Note down the antenna baselines, scan number, channels, etc in which the RFI is present. We need to flag it.*

We see that the RFI is present in baselines of particular channel numbers 126-127. We carefully look at the bad baselines present in the rfi spike and flag only required baselines, as follows:
Note: first flag the channel 127 (i.e. till first "go") and then continue with flagging others. Also, the following is conservative way of flagging a spike, to save time on expense of accuracy, one can flag the entire channels 126 to 128, 180 and 311 completely. 

.. code-block::

   tget flagdata
   default
   inp
   mode='manual'
   vis='0311.ms'
   spw='0:127'
   savepars = True
   field='0'
   antenna='W04&W05;W05&W06;E05&E06'
   go
   spw='0:311'
   antenna='W06'
   go
   spw='0:126~128'
   scan='1'
   antenna='W04,W05,W06'
   go
   spw='0:127'
   go
   antenna='S01,E06,C02'
   go
   antenna='C10&S04'
   scan='3'
   antenna='W06,S01,C02,E06,W05'
   go
   antenna='E06&W05;C10&W05'
   go
   field='0'
   spw='0:180'
   scan='1'
   antenna='W01,S01,S02,S03,S04,C13'
   go
   scan='3'
   antenna='C12,C13,S03,S02,S06'
   go

After flagging on field 0, repeat the same for other fields in data. The RFI spikes need to be carefully looked at, and only flag the essential fault baselines. For field 1, entire channels with RFI spikes are flagged as:

.. code-block::

   tget flagdata
   default
   inp
   mode='manual'
   vis='0311.ms'
    savepars=True
   field='1'
   spw='0:123'
   go
   spw='0:126~131'
   go
   spw='180'
   go
   spw='0:300~304'
   antenna='S02&S04'
   go

Tick the reload option on plotms and plot again on the plotms to verify if the flagging is reflected.

.. figure:: /images/specline/flagrfispike_4done.png
   :alt: Plotms screenshot rfi spike removed
   :align: center
   :scale: 70% 
   
   *Screenshot of plotms after flagging. Note that the spike is no more present, and the selected region can be unselected using the 'clear region' from below panel.*


If for some reason you flag a wrong data and want to reverse the flag, the command "flagmanager" is used. 

.. code-block::

   tget flagmanager
   default
   inp
   vis='example.ms'
   mode='list'
   go

This displays the list of all flag operations performed. Note the flag version name from this list, and say the latest flag that you performed has the name flagdata_4. To unflag this latest flag operation, following command is used:

.. code-block::

   tget flagmanager
   default
   inp
   vis='example.ms'
   mode='restore'
   versionname='flagdata_4'
   go



Intital Gain calibration before flagging of unwanted data
---------------------------------------------------------

Pick a clean line free channel (or a bunch of channels which does not have any RFI and does not contain the target spectral line). This would act as a reference upon which gain calibration is done, and later applied to all channels. Number of channels to be selected for averaging depends on SNR we require (if too many solutions fail and get flagged in gaincal for minsnr=5, average more channels to increase SNR). Typically, a single channel is chosen for this, however in the example below, 40 channels from channel number 300 to 339 are averaged, hence the command spw='0:300~339'. If however only a single channel, say channel number 300 were to be chosen, it would be written as spw='0:300'.
Create a directory for the solution tables, and also one for images as follows (use "!" mark at the beginning if commands are written at the casa ipython prompt):

.. code-block::

   mkdir caltables
   mkdir images

Say for example the field ids of flux calibrator are 0 and 3, and that of phase calibrator is 1. A first round of initial gain calibration is done only on calibrators (and not on target) as follows:

.. code-block::

   tget gaincal
   inp
   vis='example.ms'
   caltable='caltables/gainsol.apcal'
   solint='int'
   uvrange='>1.5km'
   minsnr=5.0
   field='0,1,3'
   spw='0:500∼539'
   go


Followed by an ``applycal``, applying the calibration to all the channels of calibrators.

.. code-block::

   tget applycal
   inp
   vis='example.ms'
   field='0,1,3'
   gaintable=['caltables/gainsol.apcal']
   calwt=[False]
   go
   
For the tutorial dataset given, we do not have a phase calibrator, and there is a single flux calibrator with field id 0. This step is implimented as follows:

.. code-block::

   tget gaincal
   inp
   defalut
   vis='0311.ms'
   caltable='caltables/gainsol.apcal'
   solint='int'
   uvrange='>1.5km'
   minsnr=5.0
   field='0'
   spw='0:360~399'
   go

Followed by ``applycal``:

.. code-block::

   tget applycal
   inp
   default
   vis='0311.ms'
   field='0'
   gaintable=['caltables/gainsol.apcal']
   calwt=[False]
   go

It is wise to keep a track of flagging percentage in the data. If too much of data gets flagged, there won't be much useful data left. The task ``flagdata`` in mode of 'summary' allows us to keep track of this. Use the following commands:

.. code-block::

   tget flagdata
   inp
   default
   vis='0311.ms'
   mode='summary'
   go

In the following figure, we can see the flag percentage for each field.

.. figure:: /images/specline/flagpercent.png
   :alt: Log screenshot flagmanager
   :align: center
   :scale: 70% 
   
   *Screenshot of casa log file for noting flagging percentage.*

In the plotms, plot amp vs uvdist with corrected data column for the entire channel, check field by field the calibrator data starting with field 0. Inspect and flag the baselines which jump around too much from the pack. Ideally the pack must be centered around amp of 1, with the baselines staying in and around that value. If the entire line jumps from this median by a large amount, it can be flagged.

In the following figure, we can see the flag percentage for each field.

.. figure:: /images/specline/uvdistvsamp_before1.png
   :alt: Plotms screenshot before flag calibration
   :align: center
   :scale: 70% 
   
   *Screenshot of plotms for uvdist vs amp (corrected). Note that a few baselines are jumping.*

It can be seen again by selection and from casa log that the lines belong to scan 1 are from baselines 'W04&W05', 'W05&W06', 'C05&S01', 'C10&S01' and 'C11&S01' and that from scan 3 are 'C11&S01' and 'S01&S04'. These are repectively flagged as follows:

.. code-block::

   tget flagdata
   inp
   default
   vis='0311.ms'
   scan='1'
   field='0' 
   antenna='C11&S01;C05&S01
   go

   scan='3'
   antenna='C11&S01;S01&C09'
   go

The plot shows as below:   

.. figure:: /images/specline/uvdistvsamp_after1.png
   :alt: Plotms screenshot after flag calibration
   :align: center
   :scale: 70% 
   
   *Screenshot of plotms for uvdist vs amp (corrected). Note that most of the baselines are packed around amp = 1 with almost no outliers.*

We need to check if amp and/or phase plotted w.r.to uvdist is flat because these are point sources at phase center so amp should not depend on uvdist and phase should also not depend on uvdist. To summarize, check uvdist vs amp corrected plots, with antenna iteration and baseline colorization; or baseline iteration and antenna1/corr colorization, if required channels averaged, field by field with uvrange>1.5km.


Absolute flux density calibration
----------------------------------

We use the task ``setjy`` to set the flux densities of the standard flux calibrators in the data here before redoing the ``gaincal``. Following are the commands for setjy, which is to be done for all flux calibrator fields present:

.. code-block::

   get setjy
   default
   inp
   vis='0311.ms'
   field='0'
   usescratch=True
   go   

The flux values assigned can be verified using the VLA calibrator manual, and the obtained value must be close to the wavelength band value from the manual where the spectral line is expected. Now, we can perform the gain calibration on calibrators using averaged bunch of channels and apply it to all the channels and fields except the target source. As we have completed setjy, the flux of flux calibrators which was centered about 1 will now be centered about their respective values. Note that the standard, 'Perley-Butler 2017' identifies most of the flux calibrators used by uGMRT. Some calibrators may not be recognized, for which the standard 'Stevens-Reynolds 2016' can be used. If the calibrator is still not recognized by these standards, the flux values need to entered manually for the calibrator.

.. figure:: /images/specline/setjy_3c48.png
   :alt: Log screenshot after setjy
   :align: center
   :scale: 70% 
   
   *Screenshot of casa log for task setjy. Note that assigned flux for the calibrator 3C48 is 38.43 Jy. Since the central frequency of our dataset is 431.7 MHz, which is about 69.4 cm wavelength, from VLA calibrator manual we see that the flux value lies between 20cm band and 90cm band.*

We would want to transfer the flux calibration solutions to the phase calibrator, so that its flux can be calibrated and scaled. If the data has two or more flux calibrators, we may choose the brightest one having cleaner and lower flagged data to use as reference to transfer the solutions from. To transfer the solution from flux calibrator field 3 to phase calibrator field 1:

.. code-block::

   tget fluxscale
   inp
   vis='example.ms'
   caltable='caltables/gainsol 1.apcal'
   fluxtable='caltables/gainsol 1.fcal'
   reference=['3']
   transfer=['1']
   go

After the task ``fluxscale``, the reported flux density of the phase calibrator must be compared with standard flux density from VLA manual. Since there is no phase calibrator present in tutorial data, ``fluxscale`` part is not needed.
A round of ``gaincal`` and ``applycal`` is to be done before the inital bandpass calibration with same paramters as before:

.. code-block::

   tget gaincal
   inp
   field='0'
   caltable='caltables/gainsol_1.apcal'
   go

   tget applycal
   inp
   field='0'
   gaintable=['caltables/gainsol_1.apcal']
   go


Initial Bandpass calibration
----------------------------

In this step, initial bandpass calibration is done on flux calibrators. We can also use the phase calibrator for this purpose if it is bright enough, more precisely if the relation tcal > tobj(Sobj/Scal)^2 holds true, where tcal is the total time spent observing the calibrator, tobj is time spent oberving the target, Sobj and Scal are the flux densities of the target and calibrator respectively. The observation time values can be found from ``listobs``; Sobj can be found in database like NVSS survey by inputting the coordinates of target and Scal is found from fluxscale.

.. admonition:: Note
   For flux values of target: https://www.cv.nrao.edu/nvss/NVSSlist.shtml 

For the example data, if the phase calibrator is bright enough, it is included in bandpass calibration along with flux calibratior fields of 0 and 3:

.. code-block::

   tget bandpass
   default
   inp
   vis='example.ms'
   caltable='caltables/bpass 0.bcal'
   uvrane='>1.5km'
   refant='C00'
   gaintable=['caltables/gainsol_1.apcal']
   field='0,1,3'
   minsnr=5.0
   uvrange='>1.5km'
   go

The solutions are first applied to the flux calibrator field by applycal and a round of automated flagger rflag is used. After this, the amp(corrected) vs frequency plot would look like the figure below, where the flux is peaked and centred around the limit set by setjy and we see a band.

.. code-block::

   tget applycal
   inp
   vis='example.ms'
   field='0,3'
   gaintable=['caltables/gainsol_1.apcal','caltables/bpass_0.bcal']
   go

   tget flagdata
   mode='rflag'
   spw=' '
   field='0,3'
   datacolumn='corrected'
   timedevscale=5.0
   freqdevscale=5.0
   go

For the tutorial dataset, this entire set of tasks are shown below:

.. code-block::

   tget bandpass
   inp
   vis='0311.ms'
   caltable='caltables/bpass_0.bcal'
   gaintable=['caltables/gainsol_1.apcal'] 
   field='0'
   uvrange='>1.5km'
   refant='C00'
   minsnr=5.0
   go

   tget applycal
   inp
   field='0'
   gaintable=['caltables/gainsol_1.apcal','caltables/bpass_0.bcal']
   go

   tget flagdata
   mode='rflag'
   spw=' '
   field='0'
   datacolumn='corrected'
   timedevscale=5.0
   freqdevscale=5.0
   go


Following is the amp (corrected) vs freq plot for 0311.ms field 0 of tutorial dataset post initial bandpass calibration and automated flagging by rflag.

.. figure:: /images/specline/field0_postinibpass_postrflag.png
   :alt: Screenshot of the plotms after initial bpass and rflag
   :align: center
   :scale: 80% 

Examine the bandpass table using ``plotms``. Choose the bandpass table bpass_0.bcal in data and check the plots Amp Vs Channels and Phase Vs Channels  iterated over antennas.

.. figure:: /images/specline/initialbpass_ampvsfreq.png
   :alt: Screenshot of the plotms for bandpass table
   :align: center
   :scale: 80% 

Note the shape of the band across the frequencies.


Delay calibration and final Bandpass calibration
------------------------------------------------

In delay calibration, a reference antenna is required. Here "C00" is only taken as an example. You may use any antenna that is working for the whole duration of the observation. We perform delay calibration only with flux calibrator field used for fluxscale and not with all calibrators.


.. code-block::

   !cp  gaincal.last gaincal.last.bk
   tget gaincal
   default
   inp
   vis='0311.ms'                                                    
   field='0'
   gaintype='K'                                                        
   caltable='caltables/delay.kcal'                                     
   refant='C00'
   go

Copying the soultions to a new table, we do a round of amp-phase gaincal with all calibrator fields and solution types of ’int’ and ’2min’. The ’int’ solutions are used for bandpass calibration and the ’2min’ solutions are used for the actual calibration.

.. code-block::

   !cp gaincal.last gaincal.last.kcal
   !cp gaincal.last.bk gaincal.last
   tget gaincal
   default
   inp
   vis='0311.ms'
   spw='0:360~399'
   solint='int'
   minsnr=5.0
   uvrange>'1.5km'
   field='0'
   gaintable=['caltables/delay.kcal']
   caltable='caltables/gainsol_int.apcal'
   go

   solint='2min'
   caltable='caltables/gainsol_2m.apcal'
   go

The task ``fluxscale`` is performed again on both the solutions with the same parameters and flux calibrator field used earlier in fluxscale and save the solutions which will be used to transfer the final bandpass solutions to all fields, including the target field. 
Note that this step is skipped for tutorial data set as there are no phase calibrators.

.. code-block::

   tget fluxscale
   caltable='caltables/gainsol int.apcal'
   fluxtable='caltables/gainsol int.fcal'
   go
   caltable='caltables/gainsol 2m.apcal'
   fluxtable='caltables/gainsol 2m.fcal'
   go


The bandpass calibration solutions are found using all (if phase calibrator was also used in inital bandpass, else only flux calibrators are used) the calibrator fields :


.. code-block::

   tget bandpass
   inp
   vis='0311.ms'
   field='0'
   combine=''
   refant='C00'
   minsnr=5.0
   gaintable=['caltables/delay.kcal','caltables/gainsol_int.apcal']
   caltable='caltables/bandpass_final.bcal' 
   go

The solutions are applied to all fields, including the target:

.. code-block::

   tget applycal
   gaintable=['caltables/delay.kcal','caltables/bandpass_final.bcal'] 
   field=''
   go

The bandpass solution tables in plotms looks like the following, where amp vs freq and gain phase vs freq is plotted for the final bandpass solution table:
 
.. figure:: /images/specline/finalbpass_ampvsfreq.png
   :alt: Screenshot of the plotms after final bpass amp vs freq
   :align: center
   :scale: 80% 

.. figure:: /images/specline/finalbpass_gainphasevsfreq.png
   :alt: Screenshot of the plotms after final bpass gain phase vs freq
   :align: center
   :scale: 80% 


At this point, we should be able to see the line features in plotms upon plotting the target field amp (corrected) vs channel and averaging in time, scan and baselines. This helps us determine the channel number where line is present and to choose a bunch of channels containing the entire line width to be used later in self calibration to avoid cleaning of these channels.

.. figure:: /images/specline/postbpassavgtimebl.png
   :alt: Screenshot of the plotms after final bpass amp (corrected) vs chan with time and baseline averaging
   :align: center
   :scale: 80%


Splitting the calibrated target source data
--------------------------------------------

We will split the calibrated target source data to a new file and do the subsequent analysis on that file.
Create a new directory named 'source'. We will split the target and save the new MS file in this directory. In the tutorial dataset, the target has field id of 1, and is used in "split" task as follows:

.. code-block::

   !mkdir source
   tget split 
   default
   inp
   field='1'                                                        
   vis='0311.ms'                                                 
   outputvis='source/source.ms' 
   go

A round of automated flagger "rflag" can be run on this MS file.

.. code-block::

   tget flagdata
   vis='source.ms'
   mode='rflag' 
   savepars=True
   go

When the data set is too large, and has many channels of data, like 2048 channels (standard uGMRT GWB data have a channel width of 12.207 kHz, giving a bandwidth of 25MHz for 2048 channels), to save on computation load and time, the file is can be further split into a lower resolution, channel averaged coarse MS file upon which self calibration task can be performed. For example, a 2048 channel source MS file is split by channel averaging of 20 channels chosen arbitrarily, giving a low resolution coarse file of about 101 channels.  For this, width = 20 must be given in task ``split``.
Since our tutorial dataset contains 512 channels, we can average by about 4 channels, if required, else this step can be skipped.

.. code-block::

   cd source
   tget split
   default
   inp
   vis='source.ms'
   outputvis='source_coarse.ms'
   width=4
   datacolumn='data'
   go 


It is easier and faster to do self calibration on coarse file and later transfer the solutions to higher resolution split file to proceed for imaging.

.. admonition: Note
   We have not taken any special note of the spectral line in steps till now. The channels 
   containing the line must not be treated special and usual steps of flagging and initial calibration must be performed. The important 
   deviation arrives during self calibration, where we have to exclude the channel range where line features are present or expected to 
   occur.


Self calibration process
------------------------

This is an iterative process. The model from the first ``tclean`` is used to calibrate the data and the corrected data are then imaged to make a better model and the process is repeated. The order of the tasks is tclean, gaincal, applycal, tclean. In this section we perform self calibration on the coarse file (if created, else it is performed on source file). In following example, we perform it on source file. A test image can be created before the self cal run, to be sure of the parameters to be used in cleaning the image using the task "tclean" and for selfcal cycles. Inputs are given as follows, where first two lines are to create new directories for images and calibration tables:

.. code-block::

   !mkdir images
   !mkdir caltables   
   tget tclean
   inp
   vis='source_coarse.ms'  
   cell=['0.3 arcsec']
   imsize=[256]
   imagename='images/testimage'
   gridder='wproject'
   wprojplanes=-1
   weighting='briggs'
   robust=-0.5
   uvtaper=['30klambda'] 
   uvrange='>1.5km'
   go


The imsize is chosen to cover a size of the field at least covering FWHM of the primary beam. The cellsize is chosen to be at least a third or more of the expected synthesized beam size.
Here, uvtaper parameter is found by plotting 'uvwave' vs amp in plotms for the visibility source.ms file and noting the distance where the tapering must be smoothed from, which would be some distance before the amp starts going to zero. 

.. figure:: /images/specline/uvtaper.png
   :alt: Screenshot of the plotms Amp Vs uvwave for uvtaper
   :align: center
   :scale: 80% 


Self-cal cycles: We start by cleaning the image (deconvolving) only selecting the channels which do not contain the line. This is done in the ``tclean`` by selecting spw range suitably. 

The cleaning is done interactively by first masking the sources visible in the dialogue view, and running the process again using the green arrow button (continue deconvolving with current clean regions) which continues the deconvolution with current clean channels in viewer GUI. We keep adding masks to any new source visible in each step and keep deconvolving until the target source noise level is reached, i.e. until the entire image looks like noise. The deconvolution is stopped at this point by clicking the red cross button. Then a round of phase only cal is performed while selecting the same spw range and applying it to all channels. With the same parameters to task ``tclean``, folowing paramters are updated and subsequestly the phase only cal is done:


.. code-block::

   tget tclean
   inp
   imsize=[4096]
   cell=['0.4 arcsec']
   niter=1000000
   interactive=True
   imagename='images/selfcal_0'
   pblimit=-0.01
   savemodel='modelcolumn'
   spw='0:0~53,0:73~127'
   go


Where we have noted that the line features are within the channels 230 to 290 for source file, and hence for coarse file it would be about 57th to 73rd channels to be excluded. The viewer GUI opens automatically and we see the following window. Here, the masking of sources is done by checking the 'add' option and drawing contours around the visible source and double clicking inside the region to save the mask. To delete a mask, check the 'erase' option, create the boundary around the mask you wish to remove and double click inside the region1.

.. figure:: /images/specline/intcleangui.png
   :alt: Screenshot of the viewer dialog GUI
   :align: center
   :scale: 80%

The phase only cal is performed once the viewer GUI closes automatically as follows:

.. code-block::

   tget gaincal  
   inp
   vis='source_coarse.ms'
   caltable='caltables/selfcal_0.pcal'
   calmode='p'
   solint='2min'
   spw='0:0~53,0:73~127'
   uvrange='>1.5km'
   minsnr=5.0
   go
   
   tget applycal 
   inp
   vis='source_coarse.ms'
   gaintable=['caltables/selfcal_0.pcal']
   calwt=[False] 
   go


This process of interactive tclean and phase only calibration is done until there seems to be no improvement in noise levels of background, which is found by drawing a rectangular region far from source and looking at the rms value of the background noise. At this point, 4 times the rms is chosen as the threshold and a run of tclean is made with this threshold. This can be done either by setting interactive as False and specifically wrtiting the threshold value as command in tclean, or can be set in the interactive mode and the central blue button can be pressed for automatic deconvolution until the set threshold level is reached. Finally an amplitude and phase calibration (ap cal) is performed, before creating the final selfcal image. Everytime, we just need to change the image name and update the mask for tclean, and for gaincal and applycal, change the gaintable and caltable names. Observe the background noise rms of the image using imview, and take four times this value to set the threshold for ``tclean``.

For example, the cycles can be continued in following manner:


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

Typically, 4 such rounds needs to be done. After this, we move ahead to do an ap cal with same spw parameters and then final tclean. Make sure to enter the latest selfcal image name and caltables properly.

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

Create the final image using ``tclean`` task, either with interactive cleaning or without it.


Subtraction of continuum
-------------------------

Perform uvsub on source coarse.ms file, which does 
corrected = corrected - model column, 
subtracting the model solutions (which are essentially other sources in the field of view) from the corrected data visibilities column.


.. code-block::

   tget uvsub      
   inp
   vis='source_coarse.ms'
   go


Apply the calibration and fill the model column of source file
--------------------------------------------------------------

If required, a round of automated 'rflag' can be run on 'source_coarse.ms' file followed by a gainacal and applycal, after which create the final image using 'tclean'. The final calibration table of the last selfcal run is applied to source.ms file. For example if the latest selfcal caltables is selfcal_5.apcal, then this is done as: 


.. code-block::

   tget applycal
   gaintable=['caltables/selfcal_5.apcal'] 
   vis='source.ms'  
   go

Essentially, we use exactly the same applycal command as used during the last round of selfcal but with vis='source.ms', instead of vis='source coarse.ms'.
Next task is to fill the model column of 'source.ms'. We use the same tclean command as used to create the final image but with the following changes: 

.. code-block::

   tget tclean
   inp
   niter=0 
   spw=''
   uvrange=''
   vis='source.ms'
   mask='' 
   imagenam ='images/savemodelrun'
   startmodel='images/selfcal_5.model' 
   go

Where, in startmodel, use the last selfcal run model. The tasks of uvsub and flagging on the target field is repeated for the source.ms file. A few rounds of rflags with higher constraints like timedev and freqdevscales of 4.5 can be applied if necessary. At this point, the data can be checked by plotting amp (corrected) vs frequency in source.ms file.

.. code-block::

   tget uvsub 
   inp
   vis='source.ms' 
   go



Perform continuum subtraction using uvcontsub
---------------------------------------------

The continuum is subtracted from the visibilities of source.ms making sure to exclude HI channels.

.. code-block::

   tget uvcontsub
   inp
   vis='source.ms'
   fitorder=1
   fitspw='0:0~230,0:290~511'
   excludechans = False
   go

Excluding the HI channels from uvcontsub, which in this file lies between channel range 230 to 290. A fitorder of 1 is selected. After this, we have a new visibility file named source.ms.contsub, which is the subtracted visibilities. Generally we can make the cube from this file and extract the spectrum. But before that, flagging on this subtracted visibilities could be done. Ideally the same set of flagging process done during the selfcal process on source coarse.ms file should be repeated, for which one can follow the task created by Aditya Chowdhury, NCRA (https://github.com/chowdhuryaditya/repeatflag).
The command to use is repeatflag(visfrom=’source coarse.ms’,visto=’source.ms’).

Other way is to perform flagging by averaging, i.e. first average over all time (by large arbitrary value, say 1e8 s) and with iteration of baseline, browse through the amp (corrected) vs frequency for the source.ms.contsub visibilities. Flag the channels in baselines with unusually high amp, ideally the amplitudes must be close to 0 as they are subtracted visibilities. Next average channels (say 40) and browse through time vs amp (corrected) data with baseline iteration and flag faulty timestamps. This is also the standard procedure to reduce the ripples in baseline in the final spectra extratced from image cube


Make the image cube and extract the spectra
-------------------------------------------

We need to run ``tclean`` with vis='source.ms', specmode='cube', niter=0. We also need to put in all the usual parameters like cell, imsize, weighting, uvrange, uvtaper, as well as spectral-cube-related parameters such as start, nchan, width; one can leave the spectral line-related parameters to their default values if you want to image every single channel and at the highest possible spectral resolution. Also, it is typical to start by using natural weighting and then try other weighting schemes to see if the noise improves.


.. code-block::

   tget tclean 
   inp
   vis='source.ms.contsub'
   weighting='natural'
   imsize=[256]
   outframe='bary'
   imagename = 'images/cube_1'
   gridder='standard'
   savemodel='none'
   uvrange='>1.5km'
   startmodel=''
   specmode='cube' 
   go


Parameters like rest frequency can be given as well, with  it being the expected frequecy of the line. The spectrum is extracted for the location where the target source lies using CASA ``imview``. This is done by first opening the cube images and then opening the final selfcal continuum image simultaneously in one imview window, and then place a dot right at the center of the source in the continuum image, and extract the spectrum at this point, using the "collapse" icon above.



Acknowledgement: We thank Nissim Kanekar for providing the dataset used for this tutorial. We thank the Narendra S. for preparing the tutorial and Balpreet Kaur, Aditya Chowdhury and Ruta Kale for editing it further. 


