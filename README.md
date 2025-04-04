[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.10532811.svg)](https://doi.org/10.5281/zenodo.10532811)


# Manual for the augmented orbit9 integrator

The [OrbFit](http://adams.dm.unipi.it/orbfit/) software was originally developed by Prof.
[Andrea Milani Comparetti](http://copernico.dm.unipi.it/~milani/) (University of Pisa), and it is currently maintained and
developed by the OrbFit consortium. OrbFit offers many tools to the user, including
programs for orbit determination, orbit propagation, and ephemeris computation.

This version of the [OrbFit](http://adams.dm.unipi.it/orbfit/) package contains a modified
version of the orbit9 integrator, a program that permits to propagate the dynamics of
asteroids. The new features included in this package are specifically designed for the
long-term propagation of small solar system objects, i.e. asteroids.

More specifically, the Yarkovsky effect is added to the gravitational vector field, and
the spin-axis evolution due to the YORP effect is integrated together with the orbital
dynamics. Technical details of the equations and of the implementation can be found in the
paper

M. Fenucci and B. Novaković: 2022. [*Mercury and OrbFit packages for numerical integration of planetary systems:
implementation of the Yarkovsky and YORP effects*](https://ui.adsabs.harvard.edu/abs/2022SerAJ.204...51F/abstract), Serbian Astronomical Journal 204, pp. 51-63

If you publish results using this version of the integrator, please refer to the package
using the above paper. 

In this readme you can find instructions on how to use the additional routines provided in
this version of the integrator.


## Authors 
- [Marco Fenucci](http://adams.dm.unipi.it/~fenucci/index.html), Department of Astronomy, Faculty of Mathematics, University of Belgrade (<marco_fenucci@matf.bg.ac.rs>) 
- [Bojan Novaković](http://poincare.matf.bg.ac.rs/~bojan/index_e.html), Department of Astronomy, Faculty of Mathematics, University of Belgrade (<bojan@matf.bg.ac.rs>) 

## Compilation

The compilation works the same as for the OrbFit version distributed by the OrbFit
Consortium. The distribution comes with a bash script called config, that permits to
choose the compilation settings for several FORTRAN compilers. By running the script
without any option, the user will receive a help message. The most popular FORTRAN
compilers are the GNU gfortran, and the INTEL ifort, that can be chosen with the script. 

We suggest the final user to choose the optimization flags, by

        ./config -O gfortran
        
At this point, the code can be compiled by typing 

        make

**Note.** This operation may take several minutes to complete.

## Files preparation

To run simulations that include the Yarkovsky/YORP effects in the model, some additional input files are needed. 
   1. **yorp_f.txt**, **yorp_g.txt**: these are files containing a discretization of the mean torques shown in Fig. 1 of the reference paper. They are supposed to be placed in a subfolder called *input*. A copy of these files can be found in the *dat* folder of the distribution.
   2. **yorp.in**: this is a file containing parameters for the integration of the spin-axis
     dynamics. This file is also supposed to be contained in a subfolder called
     *input*. An example of this input file can be found in the *tests/orbit9_yorp/input* folder.
     Here you have to provide:
         - if you want to include the YORP effect in the model
         - if you want to use a stochastic YORP model (see reference paper)
         - if you want to choose the stepsize or if you want to use the automatic
            selection
         - in case you want to specify the stepsize, write the stepsize in years
         - the peak of the Maxwellian distribution for the period resetting
         - if you want to enable the output for the spin-axis dynamics
         - the stepsize for the output    
         - the value of the parameters c_YORP, c_REOR, and c_STOC
 
     
   3. **yarkovsky.in**: this file contains physical and thermal parameters of the asteroids.
     This file is supposed to be contained in the folder where the mercury integrator is
     running. Here you have to provide, on each row:
         - the name of the asteroid
         - the density &rho; (kg/m^3)
         - the thermal conductivity K (W/m/K)
         - the heat capacity C (J/kg/K)
         - the diameter    D (meters)
         - the obliquity  &gamma; (degrees)
         - the rotation period  P (hours)
         - the absorption coefficient &alpha; (usually set to 1)
         - the emissivity &epsilon; (usually set to 1)


**NOTE 1.** To add the Yarkovsky effect to the model, make sure that the option iyark in the orb9.opt file is set to 3.

**NOTE 2.** When writing real numbers, please make sure they are written with at least a decimal digit, or by using the d0 FORTRAN notation.

## How to run a simulation and tests
We suggest the user to run each simulation in a separate folder, that can be placed in the *tests* directory. This directory contains also some test runs that you can use as a guide for the file preparation. To run a simulation, we suggest to follow these steps:
1. Make sure the code is compiled
2. Move in the *tests* folder
3. Create a directory for your own simulation

            mkdir myInteg

4. Move in your folder and create links to the binaries

            cd myInteg
            ln -s ../../src/orb9/orbit9.x
            ln -s ../../src/orb9/conv9.x
            
5. Create the basic files 
   - orb9.opt
   - filter.100, filted.d5, filter.d20, filter.d50
   - rk.coe

6. Create the initial conditions for the planets. This step can be performed with the programs inbaric.x and inbarmerc.x
    
7. Create the yarkovsky.in file

8. Create the folder for input

            mkdir input
            cd input
            
   and copy here the files needed for the YORP effect integration
   
            cp ../../../dat/yorp_f.txt .
            cp ../../../dat/yorp_g.txt .
            
   Create here also the file yorp.in with options for the spin dynamics integration.

9. Once everything is ready, you can go back to the directory myInteg, and run the code with

            ./orbit9.x
            
**NOTE.** You may want to run the program in background for long-term integrations.


### Output files
The output files for the orbital integration of the asteroids are the same as the one provided by the 
original orbit9 code.

In addition to the orbital integration, the modified orbit9 code provides the output files for the spin-axis 
dynamics when the flag *enable_out* is set to 1. The output files are called 

         <astname>.yorp

and it provides 4 columns, representing:
1. the output time (in years)
2. the rotation period (in hours)
3. the obliquity (in degrees)
4. the semi-major axis drift due to Yarkovsky (in au/My).
   
### Run the test simulation
The folder *tests/orbit9_yorp* contains the files to test a simulation that includes
the Yarkovsky and YORP effects in the model. To run the simulation, move into the 
*tests/orbit9_yorp* folder, and run orbit9 with

      ./orbit9.x

The simulation is set up for the integration of 6 asteroids over a timespan of 1 My, using
the static YORP model. Once the simulation has completed, you can convert the output files
by executing

      ./conv9.x

and choose 0 as input.

## Refereces
- M. Fenucci and B. Novaković: 2022. [*Mercury and OrbFit packages for numerical integration of planetary systems:
implementation of the Yarkovsky and YORP effects*](https://ui.adsabs.harvard.edu/abs/2022SerAJ.204...51F/abstract), Serbian Astronomical Journal 204, pp. 51-63

