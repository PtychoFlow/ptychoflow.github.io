# Usage

The interaction between the user and the PtychoFlow program is through the *main* object based on the ptychography 
class.

### main attributes:
- params for everything and ptychogram (pairs of scan_pos and diff_pat)
    ```
  - main.params_path      # string
  - main.params           # object
  - main.ptychogram_path  # string
  - main.ptychogram       # object
    ```
- sampling (pixel num, pixel size) of the coordinate of camera, probe (local), and sample (global)
    ```
  - main.camera_sampling
  - main.probe_sampling 
  - main.sample_sampling
    ```
- probe and sample
    ```
  - main.probe   # 4D array with shape        (wavlen, mode, y and x)
  - main.sample  # 5D array with shape (layer, wavlen, mode, y and x)
    ```
- TensorFlow model of cascaded layers
    ```
    - main.model
    ```
- figure handle of dashboard for monitoring the optimization
    ```
    - main.fhandle
    ```
- history of everything
    ```
    - main.history  # loss, wavelength, propagation distance, runtime
    ```
- field-of-view (fov)
    ```
  - main.fov
    ```

### main methods:
- load and check/modify the params and the ptychogram
    ```
  # params
  - main.check_params()
  - main.print_params()
  # ptychogram
  - main.center_scan_pos()
  - main.resize_camera()
  - main.remove_background()
  - main.print_ptychogram()
    ```
- calculate the sampling based on the params (prop method) and the ptychogram
    ```
  - main.calcu_sampling()
  - main.print_sampling()
    ```
- initialize the probe and the sample based on the params and the sampling
    ```
  - main.init_probe()
  - main.init_sample()
    ```
- visualize the probe and the sample, as well as the ptychogram (the scan_pos and the diff_pat)
    ```
  - main.viewer() 
    ```
- initialize and optimize the TensorFlow model
    ```
  - main.build_model()
  - main.optimize(plot=True, plot_interval=1)
    ```
- clear fhandle and history in case of re-initializing and re-optimizing the model
    ```
  - main.clear_fhandle()
  - main.clear_history()
    ```
- interaction between the main object and the model
    ```
    - main.update_model()  # replace all model variables by main attributes
    - main.update_main()   # replace all main attributes by model variables
    ```  
- manipulate individual model variable by name   
    ```   
    - main.set_model_variable(variable_name, variable)
    - variable = main.get_model_variable(variable_name)
    ```
- visualize mode variables and optimization progress
    ```
    - main.dashboard()
    ```
- visualize the probe and (in a particular layer) the sample per wavlen per mode
    ```
    - main.plot_probe(clime=None)
    - main.plot_sample(layer_index = 0, clim=None)
    ```
- calculate and visualize the field-of-view (fov)
    ```
    - main.plot_fov(log10=True)
    ```
- save results (probe and sample) and history and dashboard (eps and png)
    ```
    - main.save_results(file_name) # example file_name = 'dataset'+'date'+'time'
    ```

## Initialization
In order to perform the optimization, PtychoFlow needs to initialize the following attributes:
- params
- ptychogram
- sampling
- initial probe
- initial sample

The initialization can be done either by calling the initialization method or by calling a series 
of methods step-by-step. These two ways of initialization are essentially identical.
- the initialization method
```
from Core.Ptychography import Ptychography

main = Ptychography()
main.params_path = "Data/Experiments/params.txt"
main.ptychogram_path = "Data/Experiments/ptychogram.hdf5"

main.init_main()
```
- a series of methods step-by-step
```
from Core.Ptychography import Ptychography

main = Ptychography()
main.params_path = "Data/Experiments/params.txt"
main.ptychogram_path = "Data/Experiments/ptychogram.hdf5"

# params
main.load_params()
main.check_params()
main.print_params()

# ptychogram
main.load_ptychogram()
main.center_scan_pos()
main.resize_camera()
main.remove_background()
main.print_ptychogram_info()

# sampling
main.calcu_sampling()
main.print_sampling()

# initial probe
main.init_probe()
# initial sample
main.init_sample()
```

## Parameters
In PtychoFlow, the params controls everything: the initialization, the model, and the optimization. The idea is that 
the user only needs the params and the ptychogram to reproduce the reconstruction.

- load

PtychoFlow loads the parameters from the .txt file at main.params_path and part of the parameters from the .hdf5 
file at main.ptychogram_path using method
```
main.load_params()
```
Please see the "translate_hdf5_key" function in the **Files** module for the keys of the parameters in the .hdf5 
file. Notice that PtychoFlow prioritize the parameters in the .txt file than in the .hdf5 file.

- check

PtychoFlow checks the parameters using method
```
main.check_params()
```
It raises error when a required parameter or its value is missing. It completes the list of the optional parameters, 
and fill the missing values by the default values. The program also checks whether the type and the value of each 
parameter are ok. The user can vary the parameters at any time. However, we recommend to always make sure that all 
varied parameters are ok by calling this method. 

- print

The user can print the list of parameters in the console using method 
```
main.print_params()
```
This method will print the key and the value of each parameter.

## Ptychogram

- load 

Ptychoflow loads ptychogram (pairs of scan_pos and diff_pat, as well as the background and the mask) from 
either a .hdf5 file or a .npz file at main.ptychogram_path. The background and the mask will be "None" if 
not provided by the ptychogram file. The background is used to remove the background of the diffraction pattern 
and the mask is used to eliminate certain pixels in the diffraction pattern from optimization.
```
main.load_ptychogram()

# related params
main.params.params['ptychogram_path']

# related attributes
main.ptychogram.scan_pos
main.ptychogram.diff_pat
main.ptychogram.background
main.ptychogram.mask
```

- center scan_pos

Ptychoflow relocates the center of the set of scanning positions at the origin.
This method first finds the "center" scanning position, which is the scanning position in the nearest
vicinity of the center-of-mass of the entire set, and then shift the entire set such that 
the "center" scanning position is relocated at the origin. As a result, The pixelated property of 
the set of scanning positions can be preserved.
```
main.center_scan_pos()

# related params
main.params.params['scan_pos_centering'] = True

# related attributes
main.ptychogram.scan_pos
```

- resize camera related data

Ptychoflow resizes the set of diffraction patterns, as well as the background and the mask.
The resizing factor can be either a scalar (needs to be converted to a tuple using main.check_params()) or a tuple. 
The resizing effectively increase or decrease the field-of-view in the sample plane by splitting or combining the pixels 
in the camera plane. Notice that the user needs to re-calculate the sampling whenever calling this method. 
```
main.resize_camera()

# related params
main.params.params['camera_resizing_factor']

# related attributes
main.ptychogram.diff_pat
main.ptychogram.background
main.ptychogram.mask
```

- remove background 

Ptychoflow removes the background from each diffraction pattern and restricts the range of the pixel value. 
The background will not be removed if the background is "None". Be careful that when using dynamic range extended or 
tilt plane corrected diffraction pattern, the background has already been removed. 
This method also applies thresholds to each diffraction pattern to restrict the range of the pixel value. 
```
main.remove_background()

# related params
main.params.params['background_removal'] = True
main.params.params['saturation']   # diff_pat[diff_pat>=saturation]  = saturation
main.params.params['noise_level']  # diff_pat[diff_pat<=noise_level] = noise_level

# related attributes
main.ptychogram.diff_pat
main.ptychogram.background
```

- print

The user can print the information such as the shape and the datatype of the ptychogram in the console.
```
main.print_ptychogram_info()
```

## sampling

- calculate sampling

PtychoFlow calculates the sampling of the camera, the probe, and the sample.
Each sampling is a tuple consists of the pixel number (ny, nx) and the pixel size (dy, dx). Please note that the 
sampling depends on the ptychogram (the pairs of scan_pos and diff_pat) and the parameters. 
For "fh" and "fr" propagation, the smallest wavelength is used for calculating the sampling. The camera pixel number 
is needed only for simulation purpose.  
```
main.calcu_sampling()

# related params
main.params.params['camera_pixel_num']
main.params.params['camera_pixel_size']
main.params.params['camera_resizing_factor']
main.params.params['prop_method']
main.params.params['wavlen']
main.params.params['prop_dist']

# related attributes
main.ptychogram.scan_pos
main.ptychogram.diff_pat
```

- print

The user can print the sampling pixel number and the sampling pixel size in the console.
```
main.print_ptychogram_info()
```


## Probe
PtychoFlow initializes various kinds and sizes of probes.
The probe type can be either aperture or gaussian. 
The probe size can be either a scalar (needs to be converted to 
a tuple using main.check_params()) or a tuple. 
The default probe type and probe size are "aperture" and "None", respectively.
A "None" probe size will generate a probe whose size equals to half of the probe field-of-view. 

The initialized probe is a 4 dimensional array with 1st dimension the wavelength, the 2nd dimension the spatial mode, 
and the 3rd and the 4th dimensions being the y and the x axes, respectively. 
"weights_wavlen" is a list of ones, while "weights_mode_probe" is a list of random numbers, of which the sum is one.
```
main.init_probe()

# related params
main.params.params['probe_type']
main.params.params['probe_size']
main.params.params['wavlen']
main.params.params['weights_wavlen']
main.params.params['num_mode_probe']
main.params.params['weights_mode_probe']

# related attributes
main.probe_sampling
```

[//]: # (<p align="center">)

[//]: # (  <img src="README/psf.png">)

[//]: # (</p>)

## Sample
PtychoFlow initializes various samples.
The user can determine the type (either 'uniform' or 'random') and the range of the amplitude and the phase of 
the sample individually. The default sample has a uniform one amplitude and uniform zero phase.  

The initialized sample is a 5 dimensional array with the 1st dimension the layer, the 2nd dimension the wavelength, 
the 3rd dimension the spatial mode, 
and the 4th and the 5th dimensions being the y and the x axes, respectively. 
"weights_wavlen" is a list of ones, while "weights_mode_probe" is a list of random numbers, of which the sum is one. 
When the sample is non-dispersive, the 2nd wavelength dimension is always 1.
```
main.init_sample()

# related params
main.params.params['sample_amplitude_type']
main.params.params['sample_phase_type']
main.params.params['sample_amplitude_range']
main.params.params['sample_phase_range']
main.params.params['wavlen']
main.params.params['weights_wavlen']
main.params.params['num_mode_sample']
main.params.params['weights_mode_sample']
main.params.params['dispersive_sample']

# related attributes
main.sample_sampling
```

## Viewer
The user can view the initialization of the *main* object to interactively inspect the ptychogram. 
It also displays the first spatial mode for the first wavelength of the probe and the first sample layer.
```
main.viewer()
 ```

## Model

- build model

Once the *main* object is initialized, the user can build the TensorFlow model. 
```
main.build_model()
```
In case the user would like to rebuild model, we recommend calling the following two methods to clear the history of 
the variables and the handle of the figures: 
```
main.clear_history()
main.clear_fhandle()
```
In case the user would like to get or to set certain variable of the model, we recommend calling the following two 
methods:
```
variable = main.get_model_variable(variable_name)
main.set_model_variable(variable_name, variable)
```
In case the user would like to update the main attributes by the model variables or update the model variables by the 
main attributes, we recommend calling the following two methods:
```
main.update_main()
main.update_model()
```
At this stage, the user can check the variables of the *main.model* by calling
```
main.dashboard()
```

## Optimization
Before optimizing the TensorFlow model *main.model*, we need to determine the optimization strategy, which consists 
of a series of optimizations. Each optimization is determined by the following parameters:

The number of iterations (the optimization goes through the entire dataset in every iteration).
```
- epoch_size: [list]
```
The number of data in the dataset that the optimization goes through in parallel.
``` 
- batch_size: [list]
```
The learning rate for each variable (0 learning rate means the corresponding layer is not trainable).
```
 - learn_rate: [dictionary]
```
The decaying rate of the learning rate. (in every iteration, learn_rate *= decay_rate).
```
- decay_rate:
```
Whether the order of the dataset (pairs of scan_pos and diff_pat) will be randomized.
```
- randomize_order
```

## Regularization
PtychoFlow offers a number of regularization such as
```
- sample l1 norm
- sample total variation (tv)
- probe l1 norm
- probe support
```
The regularizations are controlled by the weights and parameters (probe support). The 623
```
main.regularizer()
main.regularizer.plot_probe_support()
```

PtychoFlow always aligns the length of the lists of optimization parameters with the longest list by extending the 
short list with the last element.

# Usage
Useful test examples of the functions in the sub-modules of the Core module are located in the **Test** folder.

Some validated experimental data and the associated "reconstruction.py" and the "params.txt" files are located in the **
Data** folder.

Raw experimental data can be found
in [our Surfdrive folder](https://surfdrive.surf.nl/files/index.php/s/KpkhxknPq8ZxUHz).

**Recommended usage:**

- Always use the most recent version of the **master** branch.
- Place your experimental data and the associated "reconstruction.py" and the "params.txt" in the **Data/Experiments**
  folder (an ignored folder that will not be synchronized).
- Check the "params_template.txt" in the **Core** folder for the required and the optional parameters. For the basic
  ptychography algorithm, only required parameters are needed.

**Explanation of the required parameters:**

- wavlen: [_list_]
- prop_dist: [_scalar_]
- camera_pixel_size: [_scalar (squared) or tuple (non-squared)_]

<!-- ROADMAP -->

## Roadmap

See the [open issues](https://gitlab.tudelft.nl/linx/joint-adp-project/-/boards) for a board of proposed features.

To-do-list:

- [ ] debugboard:
    - **issue**: Now the debugboard only displays the gradients of the sample and the probe, no diffraction pattern, and
      does not work when changing the optimization variables.
    - **solution**: Add switches in the debugboard so that it works for various number of optimization variables. Add
      display of diffraction pattern for comparison. Update debugboard per epoch instead of per bacth.
- [ ] optimization:
    - **issue**: Need a way to control all optimization parameters.
    - **solution**: optimization should take a list of dictionaries. In the dictionary we specify a lot of parameters:
        - epoch_size
        - batch_size
        - trainables
        - optimizer_type
        - learning_rate
        - other_params
        - loss_function
        - probe_rescale
        - probe_reshift
- [ ] models:
    - **issue**: Propagation between tilted planes needed.
    - **solution**: add a switch that the proprgator can tilt one plane for an arbitary angle along one dimension.
- [ ] gen_scan_pos:
    - **issue**: Now scan_pos is uniformly sampled for regular pattern.
    - **solution**: Add a switch to adjust the sampling intervals. Add a function to perturb the scan_pos.
- [ ] publish results for each dataset on surfdrive.

<!-- CONTRIBUTING -->

## Contributing

Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/NewFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/NewFeature`)
5. Open a Pull Request

<!-- Dependency -->

## Dependency

- numpy
- h5py (.hdf5 files)
- scipy (.mat files)
- pandas (.xlxs files)

<!-- STRUCTURE -->

## Structure

- [ ] Core
    - **Files**: This module contains the functions related to input/output of all kinds of files.
        - ***load_params(file_path): params***\
          This function loads all the parameters from a *.txt* file and return a *dictionary* of parameters.
        - ***save_params(file_path, params):***\
          This function saves a *dictionary* of parameters to a *.txt* file
        - ***load_ptychogram(file_path, precision): scan_pos, diff_pat, background, mask***\
          This function loads ptychogram (pairs of scan_pos and diff_pat) as well as background and mask from either a
          *.hdf5* or a *.npz* file. The value of the background and the mask will be *None*, when not provided. in the file.
        - ***save_xlxs(data, file_path):***\
          This function saves data (˙a list of locations) to a *.xlxs* file.
        - save_mat(data, file_path):
          This functions saves data (numpy arrays) to a *.mat* file.
        - translate_hdf5_key(key): translated_key
        - load_hdf5_params(params): params
    - **Params**
    - **Patterns**
- [ ] CoreTF
