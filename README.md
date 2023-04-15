# 1. Download the Docker Desktop Application

This download will likely take 2-3 minutes and can be accessed at: https://docs.docker.com/get-docker/ 
You can learn more about containers here: https://www.docker.com/resources/what-container/

# 2. Install Docker Desktop on Your Computer

- Open the Docker Desktop Installer that you downloaded. Note that this will require having administrative privileges on your computer. 
Windows users: When prompted, uncheck the box for "Use WSL 2 instead of hyper-V (recommended)"
- There will be a lot of code whizzing by as files are unpacked and the like
- Close the installation box when it is done
- This last step is for Windows users only. You will need to add your computer to the docker-users group. Do this by going to the search bar on your bottom taskbar and search/select "Windows PowerShell." Then select this app by right-clicking and choosing "Run as administrator." In the terminal window that opens up, you will type the following code To check what the structure for your specific PC is, click on the windows button in the bottom right hand corner. Select the gear symbol for settings. In the window that opens, select the "Your info" tab on the left-hand menu. You should see the information as part of your user profile. Don't forget to hit enter after you have put in the modified code in your terminal window!
   `net localgroup docker-users ornl\<user> /ADD`    where you replace <user> with your username.
 

# 3. Open Docker Desktop and Modify Settings

- Open Docker Desktop 
Windows users: you will need to do this by right-clicking on the docker app and selecting "Run as administrator" 
- Skip the Docker tutorial. 
- Go to Settings by opening the gear symbol in the upper right hand corner of the Docker app 
- Select the "Resources" tab on the left-hand menu 
- Select "Filesharing" tab on the left-hand menu under "Advanced" 
- Select the "+" symbol and add your computer's main drive here so Docker can access and write to your computer. 
Windows users: this will probably look like "C:\Users\username" where "username" is your local username on the computer 
Mac users: this will probably look like "/Users/username" where "username" is your local username on the computer 
All users: If you don't want to share your whole drive, you can also add only specific folders, e.g. "/Users/username/scratch"
- Hit the button "Apply and restart" in the bottom right hand corner of the window
- When you are done, the Resources file sharing table should look like one of the images below 
Windows users: your username & address should show up instead of vsi
Mac users: your username & address should show up instead of sserbin 


_Note: If a directory containing your user directory is already in the list, there is no need to add the additional directory, and in fact Docker will ignore the new addition. For example, if /Users is in the list, then there is no need to add /Users/username because in that case Docker would have already been given access to directories within /Users._

- Hit the "X" on the top bar across from Settings to go back to the main Docker screen 


Troubleshooting notes: If Docker Desktop fails to start, you may need to uninstall and reinstall the program. A very common mistake for Windows users to is to forget to run with administrative privileges! This is an easy one to forget and even doing it once will muck things up for future runs. If this happens to you, the easiest thing to do is uninstall and reinstall Docker Desktop.
 
# 4. Download ELM Docker Container
- Open a terminal window on your computer. 
Windows users: do this by going to the search bar on your bottom taskbar, searching and then selecting "Windows PowerShell" 
Mac users: click the magnifying glass on the right side of the menu bar on top of the screen (to open Spotlight search) and type "Terminal", then open the Terminal program 
- In the terminal window, type or copy and paste the code below and hit enter

`docker pull fasstsimulation/elm-builds:elm_v2-for-ngee_multiarch`
- The download can take around 10 minutes or more to complete. If it is going well, your terminal window will show progress with a series of arrows that grow from left to right during each file's download.


- When this download is done, you can check that the container made it into your docker app. Select the "Images" tab in the left-hand menu in Docker. There should be an item that has an name `"fasstsimulation/elm-builds"` (NAME column) and `"elm_v2-for-ngee_multiarch"` (TAG column).

# 5. Defining Docker Volumes (2 options)
**Option 1: Graphically**

- In the Docker desktop app, select the "Volumes" tab on the left-hand menu
- Click the "Create (+)" button the upper right-hand corner
- In the popup window, name your volume elm_inputdata and hit "Create"
- Make another volume called elm_output

**Option 2: Command Line**

- Open a terminal window on your computer 
Windows users: do this by going to the search bar on your bottom taskbar and search/select "Windows PowerShell"
Mac users: click the magnifying glass on the right side of the menu bar on top of the screen (to open Spotlight search) and type "Terminal", then open the Terminal program.
- In the terminal window type or copy and paste the following code into your terminal window and hit enter

`docker volume create elm_inputdata`
- Now create the following additional volume:

`docker volume create elm_output`
- You can check that this was successful by opening the docker app and navigating to the "Volumes" tab on the left hand menu. There should be 2 new volumes visible in the table that are called elmdata and elmoutput. Regardless of whether you followed option 1 or 2, your Docker desktop volumes should now look like this: 


# 6. Download Input data

Open up a terminal window and type the following:

`docker run -t -i --hostname=docker --user modeluser -v elm_inputdata:/inputdata -v elm_output:/output fasstsimulation/elm-builds:elm_v2-for-ngee_multiarch /bin/bash`

This will start the docker container in the terminal window with a command line interface, mounting the newly created volumes.  Note that changes made to files in the volumes (/input and /output) will persist, but changes made elsewhere will not.  Next, we will download the input data to the volume.
Note that you may also use local directories instead of volumes (e.g. replace “elm_inputdata” with a local directory on the host that will contain the input data), however this will likely degrade performance.

`cd /inputdata`

First, we will download a script that will get the inputdata:

`wget https://web.lcrc.anl.gov/public/e3sm/ac.ricciuto/scripts/download_inputdata_2023hackathon.sh`

Next, change script permissions and run it:

`chmod u+x *.sh`

`./download_inputdata_2023hackathon.sh`

This will build a directory structure containing a number of different input files needed for ELM simulations. It includes atmospheric data (meteorological and aerosol forcing, and lightning input for the fire model), and land parameter, surface, and nutrient deposition data. Total size is about 2GB.

# 7. Download code

Note we do not need to download E3SM. The E3SM code is already included in the container in /E3SM  - this is the NGEE Arctic version used in the October MODEX workshop that has also been and tested for this hackathon. If you want to use docker as a testing platform for other branches/versions, you can obtain E3SM from the git repository.  For more information, see https://e3sm.org/model/running-e3sm/e3sm-quick-start/    Note if you want to develop code and have changes persist, you will need to put the E3SM code in a volume (e.g. /inputdata/) rather than in the container.
The Offline Land Model Testbed (OLMT) is a set of python scripts that enable performing site, regional and global offline simulations easily. There is an existing version of OLMT used for the MODEX workshop.  However, we will clone a new version for this hackathon.  Note because we are putting this in the container, changes will not persist and you will have to do the following every time you start a new container.

`cd /tools`

`mv OLMT OLMT_old`

`git clone https://github.com/dmricciuto/OLMT.git`

OLMT contains python scripts to run ELM in a number of different configurations.  We have prepared some shell scripts that already contain the correct python commands for running ARW simulations.  There are scripts here for three different model configurations:  Satellite phenology (SP), full CNP biogeochemistry (BGC) including spinup, and a BGC configuration that starts in the year 1980.  The BGC spinup version will take a long time to run, so you may choose to run one of these:

# 8. Run the simulation

`cd OLMT/examples/2023_hackathon`

`./runARW_BGC_1980-2014_docker.sh`

This will launch a BGC simulation initialized at the year 1980 with a restart file from a previous simulation that was run on NERSC. Before you run this command, you may open up this shell script (`vi runARW_BGC_1980-2014_docker.sh`) to take a look at how exactly we use OLMT and the options that are available. The full 35-year simulation will likely take between 6-12 hours, but we will do some basic analysis after 1 year is complete.

# 9. Plot the simulations

LMT has basic plotting capabilities for timeseries plots. You will learn more about ILAMB capabilities and more advanced analysis in the next session. While the simulation is running, you may open up another container in a different terminal window:

`docker run -t -i --hostname=docker --user modeluser -v elm_inputdata:/inputdata -v elm_output:/output fasstsimulation/elm-builds:elm_v2-for-ngee_multiarch /bin/bash`

We will first need to install matplotlib:

`pip3 install matplotlib`

Since we’re in another container, we’ll need to grab the latest version of OLMT again:
`cd /tools`

`mv OLMT OLMT_old`

`git clone https://github.com/dmricciuto/OLMT.git`

Next, we will download a previously run simulation that used the satellite phenology (SP) version of ELM.

`cd /output/cime_run_dirs`

`wget http://web.lcrc.anl.gov/public/e3sm/ac.ricciuto/model_output/20230403_hcru_hcru_ICBELMBC.tar.gz`

`tar -xzvf 20230403_hcru_hcru_ICBELMBC.tar.gz`

This contains monthly output from the years 1980-2014.

We may plot several variables from this case:

`cd /tools/OLMT`

To plot the SP case for the first 10 years: 

`python plotcase.py --runnames 20230403_hcru_hcru_ICBELMBC --ystart 1980 --yend 1990 --csmdir /output/cime_run_dirs/ --vars QRUNOFF,QVEGT,TLAI --model_name elm --png --index=-1`

The output will be produced in: `/output/cime_run_dirs/20230403_hcru_hcru_ICBELMBC/plots`

You may navigate to this folder in docker desktop and download it. When you have a full year of output from the running case (after 10-30 minutes), you may plot this and compare it to the SP case:

`python plotcase.py --runnames 20230403_hcru_hcru_ICBELMBC,20230413_hcru_hcru_ICB20TRCNPRDCTCBC --ystart 1980 --yend 1980 --csmdir /output/cime_run_dirs/ --vars QRUNOFF,QVEGT,TLAI --model_name elm --png --index=-1`

When you are plotting multiple cases (runnames), the output will go into the last case specified: `/output/cime_run_dirs/20230413_hcru_hcru_ICB20TRCNPRDCTCBC/plots`

When calling plotcase.py, the variable names, year start and end (ystart and yend) and runnames can be modified as appropriate.  The “index” argument tells it which gridcell to plot.  “-1” means average over all gridcells in the domain. You may see the full list of available output variables and metadata by performing an “ncdump -h” on one of the model history output files.

# 10. Additional simulations (optional)

**Changing model parameters:**

The simulations above ran with the default model parameters, which are located in `/inputdata/lnd/clm2/paramdata/clm_params_c180524.nc`
You can make changes by editing this parameter file.  You may also create a .txt file with the parameter changes in it. See `OLMT/examples/2023_hackathon/example_parm_file`

Here, we list the name of the parameter, the PFT to apply it to and the new value.
A modified run script that uses this file is in the same directory: `./runARW_BGC_1980-2014_parmfile_docker.sh`

**Running a full simulation at a single gridcell:**

This script will extract a single gridcell from the surface and domain data files, and perform a full simulation for that gridcell. This is much faster than running the full watershed. This includes 3 phases, each with separate model cases:  ad_spinup (200yr), final_spinup (200 yr) and transient (165 yr).

`./runARW_BGC_singlept_docker.sh`

A list of parameters and model output variables is available here:

https://docs.google.com/spreadsheets/d/1LAzaHAkZ_mydxyohxUiV5JQ3nheV49tJ/edit?usp=share_link&ouid=109005022604278426505&rtpof=true&sd=true


# 11. Follow-on activities

For the ILAMB-focused session in two weeks, you will use the model output (`elm.h0` and `elm.h1` files) you generated in this hackathon. Because the full 35-year simulation takes 6-12 hours, you may let it run overnight. For comparison purposes you may also run a second simulation with adjusted parameter values. When the runs are complete, you can transfer the outputs from the docker volume to your disk. In the ILAMB hackathon, you will not need to use the docker container.

For those interested, there will also be a breakout session at the ESS PI meeting focused on running single site simulations with updated workflows.

