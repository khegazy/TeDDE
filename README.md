Molecular Geometry Retrieval Using Bayesian Inferencing
=======================================================

This package approximates the probability distribution |Psi(r,t)|^2 of time dependent molecular geometries measured by ultrafast diffraction experiments. We apply Bayesian Inferencing and ensemble anisotropy (althought not required) to access the molecular frame via a system of integral equations. This package sets up those molecular frame dependent equations and solves for the given posterior P(r,t|Theta) which approximates |Psi(r,t)|^2. Using the Metropolis-Hastings algorithm, implemented by the [emcee](https://emcee.readthedocs.io/en/stable/) package, we solve retrieve the marginalized posterior P(Theta,t). Using the `mode_search.py` method one can find the optimal Theta parameters that produces a posterior that best describes the observed data (C coefficients). This method is fully described in this [publication](https://arxiv.org/abs/2207.09600).


Setup and Prerequisites
-----------------------

**Setup**</br>
run the setup script `bash setup.sh`

**Prerequisites**</br>
Below is the minimum required version of Python, Python packages, scattering amplitudes, and simulation scripts:
- `Python >= 3.9.7`
- `Numpy >= 1.21.4`
- `Scipy >= 1.6.2`
- `Matplotlib >= 3.4.1`
- `h5py >= 3.3.0`
- `argparse >= 1.1`
- `ctypes >= 1.1.0`
- `emcee >= 3.0.2`
- `corner >= 2.2.1`


Optional modules and scripts to download for full functionality
- Scattering amplitude
  - One can calculate scattering amplitudes themselves
  - One can use the [electron scattering amplitudes](https://github.com/khegazy/physics_simulations/tree/a1f3e56153f738e8be8f71f5d87907f99bcc5237/scatteringAmplitudes/3.7MeV) for this analysis
- Diffraction Simulations
  - Only needed when running functions that require `diffraction.py`
  - One can use their own simulation method
  - One can use this analytical [diffraction Simulation](https://github.com/khegazy/physics_simulations/blob/42a2a0ef68e18f75f8ab8b3836672fa502ae1164/diffractionSimulation/diffraction.py) that must be saved in this directory.
    - One must also download this [script](https://github.com/khegazy/physics_simulations/blob/42a2a0ef68e18f75f8ab8b3836672fa502ae1164/diffractionSimulation/modules/script_setup.py) to the modules folder.



Project Organization
--------------------

    .
    ├── README.md                   <- The top-level README for developers
    │
    ├── setup.sh                    <- Makes folders and soft links needed for the analysis
    ├── parameters_template.py      <- Common starting runtime parameters needed to run the analysis
    ├── build_posterior_template.py <- Script template to retrieve the marginal posterior distribution
    │                                  P(Theta, C)
    ├── mode_search_template.py     <- Script template to retrieve optimal model (Theta) parameters 
    │                                  by finding the mean of the N-dimensional P(Theta, C) 
    │
    ├── plots                       <- Folder of plots generated by the analysis
    │
    ├── XYZ                         <- Folder of ground state molecular geometries in XYZ format
    │
    ├── scattering_amplitudes       <- Folder of the scattering amplitudes
    │
    ├── modules                     <- Folder of python modules and C++ source files and libraries 
    │   ├── density_extraction.py   <- Module of the density_extraction class used to retrieve the
    │   │                              marginal probability P(Theta, C)
    │   ├── mode_search.py          <- Module of the functions used in the mode search algorithm 
    │   ├── plot_functions.py       <- Plotting functions for the NO2 data set
    │   ├── c_calc_extensions.py    <- Python module porting the C++ functions into python functions
    │   ├── NO2.py                  <- Python module containing the probability calculations, ensemble
    │   │                              generators, scattering amplitude and ADM import functions, and
    │   │                              MCMC setup functions unique to the NO2 simulation. 
    │   └── module_template.py      <- Template module of the probability calculations, ensemble
    │                                  generators, scattering amplitude and ADM import functions, and
    │                                  MCMC setup functions unique to one's experiment 
    │
    ├── cpp_extensions              <- C++ packages used to evaluate functions faster than Numpy/Scipy
    │   ├── include                 <- Folder of the C++ header files
    │   ├── lib                     <- The compiled C++ libraries ported into python functions
    │   └── src                     <- Folder of the C++ source code
    │
    ├── NO2                         <- This folder contains the implemented modules to simulate and 
    │   │                              and analyze the NO2 signal as an example 
    │   ├── parameters.py           <- Runtime parameters to retrieve the marginal posterior
    │   │                              P(Theta,C) and its mode for the NO2 simulation 
    │   ├── parameters_N2O data.py  <- Runtime parameters for to retrieve the marginal posterior   
    │   │                              P(Theta,C) and the mode for the N2O data
    │   ├── build_posterior.py      <- Retrieves the marginal probability distribution P(Theta,C) for
    │   │                              the NO2 simulation
    │   ├── mode_search.py          <- Retrieves the optimal model (Theta) parameters by finding the 
    │   │                              mode of the marginal probability distribution P(Theta,C)
    │   ├── plots                   <- Plots generated by this analysis on the NO2 simulation
    │   ├── modules                 <- Softlink to `../modules`
    │   ├── cpp_extensions          <- Softlink to `../cpp_extensions`
    │   ├── scattering_amplitudes   <- Softlink to `../scattering_amplitudes`
    │   └── XYZ                     <- Folder containing the ground state geometry
    │        ├── N2O.xyz            <- The N2O ground state geometry
    │        ├── CF2IBr.xyz         <- The CF2IBr ground state geometry
    │        ├── NO2.xyz            <- The NO2 ground state geometry
    │        └── NO2_symbreak.xyz   <- The stretched NO2 ground state geometry



Parameters
----------
The runtime parameters that define the analysis procedures, function behaviours, and the addresses to output and input files and folders are stored in 'parameters.py'. This dictionary of parameters becomes a member of the density extraction class and is passed to all functions. Because of this one can use this dictionary as a means to import their own variables into the ensemble generators and log probability calculations. Below is a list of parameters and a description

  - **molecule** - string | The name of the molecule used in the experiment (will make a folder)
  - **experiment** - string | A label to name the experiment or different set of parameters (will moke a folder)
  - **density_model** - string | Used to distringuish between the Delta, Normal, or any other chosen posterior distribution that one implements. Use this in the main program to choose between different log probability and ensemble calculations that one implements. (will make a folder)
  - **calc_type** - int | Choose the spherical bessel fucntion calculation implementation. If one is having difficulty compiling the libraries in 'cpp_extensions' they can still run the analysis with option 1.
    - 0 <- C++ implementation (Recommended, but cannot include very low q)
    - 1 <- Scipy implementation (Slowest but correct for all q values)
    - 2 <- Optimized Python implementation (Slower than 0 with the same errors)
  - **multiprocessing** - int | The number of cores to split the C calculations between
  - **plot_setup** - boolean | If True then plot the imported data, fit to I0, and other setup calculations for sanity checks
  - **plot_progress** - boolean | If True then plot the chain history and corner plot after every run_limit sampling steps
  - **labels** (optional) - list of strings | A list of labels for the theta parameters used to label the corner plots. Set to None or do not set at all to use standard labels.

  - **Nwalkers** - int | The number of walkers (chains) to simultaneosly run. This correspondes to the number of random initializations for the MCMC.
  - **run_limit** - int | The number of MCMC sampling steps before the algorithm saves the results and checks for convergence
  - **min_acTime_steps** (optional) - int | The minimum number of autocorrelations time steps needed before the MCMC can converge. Without this parameter the MCMC will run until it has reached at least 100 autocorrelations steps and the change in the autocorrelation time is less than 1%. 
  - **init_thetas** - np.array of type float | The expected theta (model) parameters of the initial (grount) state distribution. This is used to fit I0 and can be used in 'initialize_walkers'
  - **init_thetas_std_scale** (optional) - np.array of type float | The width of the distribution from which the walkers (initial theta parameters) are choosen from. This is can be used in 'initialize_walkers'
  - **output_dir** - string | The address for the MCMC output folder
  - **save_sim_dir** (optional) - string | The address for the folder where the siulated C coefficients and errors are saved to save time from calculating them for each run. If specified and one changes the ensemble generators, make sure to delete this folder in order to rerun the updated ensemble generator.

  - **simulated_data** - boolean | If False, the C coefficients are imported, if True they are calculated along with the error bars
  - **simulated_error** - tuple (name, params) | Determines the type of error bar calculation when smulated_data is True. The options are
    - Poissonian (Recommended) : ("StoN", (SNR, range))
          Poissonian error is added to simulated diffraction images
          and propagated into the C coefficients. They are normalized
          such that the C200 signal to noise ratio calculated over
          range (list) is SNR. 
    - Constant standard deviation : ("constant_sigma", sigma)
          Sigma will be the standard deviation for all C coefficients
    - Constant background : ("constant_background", sigma)
          Will apply a constant error to the C coefficient errors such
          that the C200 signal to noise is given by sigma
  - **sim_thetas** - np.array of type float | This array specifies the parameters used to build the molecular ensemble that will generate the simulated C coefficients based on the 'ensemble_generator' function. 
  - **dom** - np.array of type float | The "dimension of measurement" the experiment probed. For x-ray and electron diffraction this corresponds to q and s, respectively, measured in inverse angstroms. One may also consider applying this package to photoelectron spectroscopy where the dom would be the electron energy measured in eV. One should set this to None when importing C coefficients. When simulating data, dom will specify the points to calculate the C coefficients.
  - **fit_bases** - list of np.array of type int | A list of the LMK coefficients to use in this analysis. If this parameter is not in the dictionary all the imported C_{LMK} coefficients will be used.
  - **isMS** - boolean | Set to True if the imported data is already diveded by the atomic scattering, otherwise set to False.
  - **fit_range** - list | The upper and lower end of the dom where the MCMC will calculated the log likelihood over.
  - **ADM_params** - dictionary | The parameters passed to 'get_ADMs' used to import the ADMs and sample them at the points in time corresponding to the measurement when using them to simulated the "StoN" error bars.


C++ versus Python calculation of spherical bessel functions
-----------------------------------------------------------
Calculating the spherical bessel functions for all the geometries the MCMC chooses is the most computationally intensive step that limits this methods capabilities. Using the `calc_type` runtime parameter one can specify whether to use the Python or C++ implementation. If the Python implementation is chosen, one will most likely be limited to the delta posterior due to it's limited Theta parameters. To retrieve width information of |Psi(r,t)|^2 one will likely need the C++ implementation of the spherical bessel functions. With the C++ implementation one must check the check_jl{i}_calculation.png plots to make sure the calculation is stable in the recipricol space region specified. The C++ implementation is unstable at small values, less than 0.5 inverse Angstroms, due to the 1/x contributions. This region of data is likely blocked by the detector in x-ray diffraction or may be contaminated by inelastic scattering in electron diffraction and should be removed anyway.


Analysis Templates
------------------
The files `build_posterior_template.py`, `mode_search_template.py`, and `modules/module_template.py` are templates for this analysis containing empty functions tha need to be implemented. After implementing these functions one can retrieve the marginal posterior P(theta|C) and find the optimal theta (model) parameters (theta*) for the posterior P(r,theta*|C). Theta* corresponds to the mode of the marginal posterior P(theta|C)


NO2 Analysis
------------
The NO2 folder contains the parameters and scripts used to both simulate the NO2 C coefficients and retrieve the molecular frame nuclear probability distribution |Psi(r)|^2 as outlined in 
[Bayesian inferencing and deterministic anisotropy for the retrieval of the molecular geometry |Ψ(r)|2 in gas-phase diffraction experiments](https://arxiv.org/abs/2207.09600).


Attributions
------------
Please cite [Bayesian inferencing and deterministic anisotropy for the retrieval of the molecular geometry |Ψ(r)|2 in gas-phase diffraction experiments](https://arxiv.org/abs/2207.09600) if you used this package.


License
-------
This package is free software offered under the MIT license: details in `License.txt`


