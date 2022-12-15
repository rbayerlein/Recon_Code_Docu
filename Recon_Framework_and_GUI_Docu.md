# UC Davis In-House PET Image Reconstruction Framework

## Documentation of framework architecture and GUI

### Recon GUI
- auxFcnCompilation: collection of helper functions that can be accessed via an object of class auxFcnCompilation('name_tag')
- start GUI using command 
    ```
    cd install_dir/GUI
    LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6 matlab -nodesktop -r "recon_GUI"
    ```
    `install_dir` is the directory where the recon package is located and compiled
    Pre-loading the library `libstdc++.so.6` is necessary since calling a C++ executable might not be possible. Matlab might struggle to find the correct library in its own install directory. Therefore, force Matlab to use the system's library.
- Gui must be started from **within** the `GUI/` folder otherwise the install directory for the recon will be wrong!
- The user must have reading **and** writing permissions for all folders. That is the directory where the code package is executed, the list mode directory (or study directory) and obviously the output directory for the image data!

#### Handles overview
| parameter | explanation |
| ------ | ------ |
| `install_dir` | Top level where the source code is located. |
| `install_dir_server` | Top level where the source code was located on server in old code version. Now it's all in one place, but the parameter might be used in some other program parts. |
| `frame_choose_base` | File name of the *.raw list mode file |
| `path_choose` | Full path to raw list mode data without filename |
| `path_choose_server` | Same as `path_chose`. Needed because former version used it and it is potnetially used in many other files. |
| `scan_length` | total length of the scan, taken from the dicom header. Given in seconds. |
| `study_dir` | Study parent folder. Contains PET, Image, CT, UCD |
| `study_dir_ucd` | `study_dir/UCD`, here, scatter data are stored and the multi-bed config file |
| `dcm_dir_init` | `study_dir/Image` |
| `dcm_dir_init_ucd` | `study_dir/UCD/Image` |
| `study_dir_server` | `study_dir` |
| `dcm_dir_init_server` | `dcm_dir_init` |
| `dcm_dir_init_ucd_server` | `dcm_dir_init_ucd` |
| `AC_path` | Folder where the CT data are stored. |
| `AC_path_server` | `AC_path` |
| `sensitivity_path` | Path to the sensitivity image with extension *sen_img |
| `recon_data_dir` | Directory where the output folder will be created. This used to be set to `/mnt/ssd/userName/explorer/recon_data_temp/` automatically. This handles entry did exist in the old GUI version. |
| `scatter_onoff` | toggle scatter correction on or off |
| `psf_onoff` | toggle PSF modeling on or off |
| `randoms_onoff` | toggle randoms correction on or off |
| `spatial_subsampling` | Spatial subsampling for multi-bed imaging or short scanner geometry |
| `cycles_per_bed` | multi-bed recon parameter, number of cycles per bed position |
| `num_beds` | multi-bed recon parameter, number of bed positions |
| `time_per_bed` | multi-bed recon parameter, time per bed in seconds |
| `start_ring` | multi-bed recon parameter, first ring |
| `rings_per_bed` | multi-bed recon parameter, number of rings of one bed position |
| `overlap` | multi-bed recon parameter, be overlap given in number of rings |
| `subsample` | temporal subsampling, this functionality has not been implemented yet. |
| `reconFrames` | array with recon frame numbers, e.g. 3 frames would result in an array of [0 1 2]. |
| `sensitivity_image_name` | Sensitivity image name, without folder, just the *.sen_img file |
| `fname_mb_config` | Filename of config file for multi-bed imaging. It will be created regardless whether subsampling is on or off. If no subsampling is used, defauls values are taken that represent total-body scan. |
| ` tker_path` and `aker_path` | LUTs for PSF modeling |
| ` max_par_recon ` | Required by recon_scheduler to calculate number of frames that are done. Value is 12. |
| `pet_img_size` | vector defined as `[x y z]` |
| `pet_voxel_size` | vector defined as `[x y z]` |
| `AddAttn2add_fac` | path to AddAttn2add_fac executable which adds attenuation factors to additive correction factors. |
| `lm_fp_exp` | path to lm_fp_exp executable that performs the forward projection to calculate the attenuation along every LOR in the data set. |
| `simset_dir` | base directory of simset installation. e.g. `/home/username/.../2.9.2_new_mat_table` |
| `user_name` | user name extracted from current work directory. |
| `scat_cor_dir` | directory where SC framework is located. `scat_corr_recon.m` must be there. |
| `use_existing_SC_data` | Determines whether the program should use scatter data from previous simulations or recreate the data set with new simulations. |
| `tof_onoff` | toggle TOF on or off. |
| `large_memory_avail` | For servers with sufficient memory (at least 128 GB) one can switch this to the on position. For workstations, this should be switched off, which causes the program to execute certain parts consecutively instead of parallel. Otherwise, Matlab will just crash due to lack or memory. |
| `` |  |
| `` |  |
| `` |  |
**Further recon parameters needed by read_lm executable**
| parameter | explanation |
| ------ | ------ |
| `run_recon` | explanation needed |
| `write_lm` | explanation needed |
| `write_sino` | explanation needed |
| `write_sino_block` | explanation needed |
| `write_histo_img` | explanation needed |

### For new users
**Install GUI on new work station**
- copy code to workstation under the users home directory e.g. `/home/new_user/code/`
- recompile executables using `./install.sh`.
- (if not existing) create new user name and home directory for new user using `sudo useradd -m user_name`. The `-m` is necessary for creating a home directory.
- for easier accessibilty, use different shell for new user: `sudo chsh -s /bin/bash user_name`
- add new user to the group recon_data using `sudo usermod -aG recon_data user_name`, which will allow them to write to /mnt/data/other_user_name, where image or raw data might be stored and that need to be shared between users.

### To Do List
- Replace hard coded values and parameters and let programs collect them from handler
    - voxel size
    - image size
- remove folderSizeMonitor or finalize the monitoring tool for SimSET
- Implement parallel programming for combine listmode executable (speed up executable)
- SC parameters should be part of GUI (*num events -> done*, num iterations, scaling method,...)
- Calculate scatter fraction
- clean up read_lm executable especially the outputs!
- In the end, copy mu-map into output directory. It's useful for image analysis.
- Add TOF to file name
- Deactivate TOF switch after Start Image Reconstruction button was pressed
- Fix lm2blocksino_P_and_D so that 5D sino scaling can be used in phase 5 (see phase 8 code for details!)

### List of programs that need to be compiled when code is transfered
- AddAttn2add_fac
- read_lm/src: process_listmode executable
- AddScatter2add_fac
- hist2lm
- lm2blocksino5D
- lm2blocksino_P_and_D
- 

MADE CHANGES!

