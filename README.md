Molecular Geometry Retrieval Using Bayesian Inferencing
=======================================================

This package approximates the geometric probability of molecules for ultrafast diffraction experiments. We apply Bayesian Inferencing and ensemble anisotropy to access the molecular frame. This method is fully described in ... . This repository holds to code used for this work.

Prerequisites
-------------
Below are the packages and versions used, one may install more recent version
* Python 3.9.7
* Numpy 1.21.4
* Scipy 1.6.2
* Matplotlib 3.4.1
* h5py 3.3.0
* argparse 1.1
* ctypes 1.1.0

* emcee 3.0.2
* corner 2.2.1

> Folder structure options and naming conventions for software projects

### A typical top-level directory layout

    .
    ├── ...
    ├── test                    # Test files (alternatively `spec` or `tests`)
    │   ├── benchmarks          # Load and stress tests
    │   ├── integration         # End-to-end, integration tests (alternatively `e2e`)
    │   └── unit                # Unit tests
    └── ...



Project Organization
--------------------
> Folder Tree
   .
   ├── README.md          <- The top-level README for developers using this project.
   │
   ├── setup.py           <- Makes folders and soft links needed for the analysis
   ├── parameters.py      <- File containing the runtime parameters needed to run the analysis
   ├── build_posterior_template.py <- Example script to retrieve the marginal probability distribution P(Theta, C)
   ├── mode_search_template.py     <- N-dimensional search algorithm of P(Theta, C) for the optimal model parameters
   │
   ├── plots              <- Folder containing plots generated by the analysis
   │
   ├── XYZ                <- Folder containing the ground state molecular geometry in XYZ format
   │
   ├── scattering_amplitudes       <- Folder containing the scattering amplitudes ()
   │
   ├── modules            <- Folder containing the python modules and C++ source files and libraries 
   │   ├── density_extraction.py   <- Python module containing the density_extraction class that retrieves the
   │   │                              marginal probability P(Theta, C)
   │   ├── mode_search.py          <- Python module containing the functions used in the mode search algorithm 
   │   ├── plot_functions.py       <- Plotting functions for the NO2 data set
   │   ├── spherical_j_cpp.py      <- Python module that ports the C++ functions into python functions
   │   ├── NO2.py                  <- Python module containing the import functions, probability calculations,
   │   │                              and molecular ensemble generators for the NO2 dataset 
   │   └── module_template.py      <- Template python module containing the import functions, probability 
   │                                  calculations, and molecular ensemble generators
   │
   │
   ├── cpp_extensions     <- C++ packages used to evaluate functions faster than Numpy/Scipy
   │   ├── include        <- Folder containing header files
   │   ├── lib            <- Folder containing the resulting libraries we import into pythong
   │   └── src            <- Folder containing the C++ source code
   │
   ├── NO2                <- A copy of this current folder with the functions and parameters filled for 
   │   │                     the NO2 simulation 
   │   ├── parameters.py  <- Runtime parameters for to retrieve the marginal posterior P(Theta,C) and
   │   │                     its mode for the NO2 simulation 
   │   ├── parameters_data.py      <- Runtime parameters for to retrieve the marginal posterior P(Theta,C)   
   │   │                              and the mode for the N2O data
   │   ├── build_posterior.py      <- Script to retrieve the marginal probability distribution P(Theta,C)
   │   │                              for the NO2 simulation
   │   ├── mode_search.py <- Script to retrieve the optimal theta (model) parameters by finding the mode 
   │   │                     of the marginal probability distribution P(Theta,C) for the NO2 simulation
   │   ├── plots          <- Folder containing plots generated by this analysis on the NO2 simulation
   │   ├── modules        <- Softlink to '../modules'
   │   ├── cpp_extensions <- Softlink to '../cpp_extensions'
   │   ├── scattering_amplitudes   <- Softlink to '../scattering_amplitudes'
   │   └── XYZ                     <- Folder containing the ground state geometry
   │        ├── N2O.xyz            <- The N2O ground state geometry
   │        ├── CF2IBr.xyz         <- The CF2IBr ground state geometry
   │        ├── NO2.xyz            <- The NO2 ground state geometry
   │        └── NO2_symbreak.xyz   <- The stretched NO2 ground state geometry



Parameters
----------
The runtime parameters that define the analysis procedures, function behaviours, and the addresses to output and input files and folders are stored in 'parameters.py'. This dictionary of parameters becomes a member of the density extraction class and is passed to all functions. Because of this one can use this dictionary as a means to import their own variables into the ensemble generators and log probability calculations. Below is a list of parameters and a description

*  molecule - string | The name of the molecule used in the experiment (will make a folder)
*  experiment - string | A label to name the experiment or different set of parameters (will moke a folder)
*  density_model - string | Used to distringuish between the Delta, Normal, or any other chosen posterior distribution that one implements. Use this in the main program to choose between different log probability and ensemble calculations that one implements. (will make a folder)
*  calc_type - int | Choose the spherical bessel fucntion calculation implementation. If one is having difficulty compiling the libraries in 'cpp_extensions' they can still run the analysis with option 1.
          0 <- C++ implementation (Recommended, but cannot include very low q)
          1 <- Scipy implementation (Slowest but correct for all q values)
          2 <- Optimized Python implementation (Slower than 0 with the same errors)
*  multiprocessing - int | The number of cores to split the C calculations between
*  plot_setup - boolean | If True then plot the imported data, fit to I0, and other setup calculations for sanity checks
*  plot_progress - boolean | If True then plot the chain history and corner plot after every run_limit sampling steps
*  labels (optional) - list of strings | A list of labels for the theta parameters used to label the corner plots. Set to None or do not set at all to use standard labels.

*  Nwalkers - int | The number of walkers (chains) to simultaneosly run. This correspondes to the number of random initializations for the MCMC.
*  run_limit - int | The number of MCMC sampling steps before the algorithm saves the results and checks for convergence
*  min_acTime_steps (optional) - int | The minimum number of autocorrelations time steps needed before the MCMC can converge. Without this parameter the MCMC will run until it has reached at least 100 autocorrelations steps and the change in the autocorrelation time is less than 1%. 
*  init_thetas - np.array of type float | The expected theta (model) parameters of the initial (grount) state distribution. This is used to fit I0 and can be used in 'initialize_walkers'
*  init_thetas_std_scale (optional) - np.array of type float | The width of the distribution from which the walkers (initial theta parameters) are choosen from. This is can be used in 'initialize_walkers'
*  output_dir - string | The address for the MCMC output folder
*  save_sim_dir (optional) - string | The address for the folder where the siulated C coefficients and errors are saved to save time from calculating them for each run. If specified and one changes the ensemble generators, make sure to delete this folder in order to rerun the updated ensemble generator.

*  simulated_data - boolean | If False, the C coefficients are imported, if True they are calculated along with the error bars
*  simulated_error - tuple (name, params) | Determines the type of error bar calculation when smulated_data is True. The options are
          Poissonian (Recommended) : ("StoN", (SNR, range))
              Poissonian error is added to simulated diffraction images
              and propagated into the C coefficients. They are normalized
              such that the C200 signal to noise ratio calculated over
              range (list) is SNR. 
          Constant standard deviation : ("constant_sigma", sigma)
              Sigma will be the standard deviation for all C coefficients
          Constant background : ("constant_background", sigma)
              Will apply a constant error to the C coefficient errors such
              that the C200 signal to noise is given by sigma
*  sim_thetas - np.array of type float | This array specifies the parameters used to build the molecular ensemble that will generate the simulated C coefficients based on the 'ensemble_generator' function. 
*  dom - np.array of type float | The "dimension of measurement" the experiment probed. For x-ray and electron diffraction this corresponds to q and s, respectively, measured in inverse angstroms. One may also consider applying this package to photoelectron spectroscopy where the dom would be the electron energy measured in eV. One should set this to None when importing C coefficients. When simulating data, dom will specify the points to calculate the C coefficients.
*  fit_bases - list of np.array of type int | A list of the LMK coefficients to use in this analysis. If this parameter is not in the dictionary all the imported C_{LMK} coefficients will be used.
*  isMS - boolean | Set to True if the imported data is already diveded by the atomic scattering, otherwise set to False.
*  fit_range - list | The upper and lower end of the dom where the MCMC will calculated the log likelihood over.
*  ADM_params - dictionary | The parameters passed to 'get_ADMs' used to import the ADMs and sample them at the points in time corresponding to the measurement when using them to simulated the "StoN" error bars.


Analysis Templates
------------------
The files `build_posterior_template.py`, `mode_search_template.py`, and `modules/module_template.py` are templates for this analysis containing empty functions tha need to be implemented. After implementing these functions one can retrieve the marginal posterior P(theta|C) and find the optimal theta (model) parameters (theta*) for the posterior P(r,theta*|C). Theta* corresponds to the mode of the marginal posterior P(theta|C)

NO2 Analysis
------------
The NO2 folder contains the parameters and scripts used to both simulate the NO2 C coefficients and retrieve the molecular frame nuclear probability distribution |Psi(r)|^2 as outlined in ...... .

