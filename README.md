# Image Reconstruction Code Documentation
Code: _Eric Berg_, Documentation: _Reimund Bayerlein_

## General Workflow 
* The reconstruction is started by executing 
`recon_s1` located in folder `uex/`
it's a bash script and loads some libraries and starts the GUI via invoking the following matlab script:
* `uex/code/explorer-master/run_server_server_recon/respiratory_gui.m` implements the GUI interface for the user to select input file name, recon parameters and sensitivity image (either create a new one or load one from file)
* The GUI invokes further functions and programs:
    * `/read_lm/process_lm_sino_explorer_singlefile_release`: reads in the list mode data and saves them in orderly manner to be used in the recon.
* The GUI will print `done` at the end of the recon, if no errors have occurred. 

## Functions and Programs

#### `respiratory_gui.m`
##### General Info
* Matlab
* Creates a config file with 8 lines to be opened by `process_lm_sino_explorer_singlefile_release`. 
* Starts the creation of the sensitivity map
##### Making Sensivity Map
* Processes that are performed in `respiratory_gui.m`:
    * line 274: `function make_sens_img_Callback(hObject, eventdata, handles)`
    * Checks and sets paths and checks for existing sens maps to prevent overwriting 
    * disables GUI buttons for the mean time
    * Collects from server: AC dicom, .nc file
    * creates plane and crystal efficiencies and adds the gaps and saves them to server
    * makes the mu map
    * creates files names `run_sens.sh` and `run_sens.#.sh` with `#` from 1 to 5 corresponding to unit differences 0 to 4.
    * line 403: `succ = make_runsens_server_sh(handles)` creates the actual files `run_sens.sh` and `run_sens.#.sh` and fills them with the commands to be executed on the server
    * Cleans up the data temp folder on server 
    * moves the sensitivity image data to the server, i.e. the run scripts, the efficiency files, the mu map. They end up in `/mnt/ssd/userName/explorer/sens_data_temp/`
    * Runs the scripts on the server by invoking `run_sens.sh` which itself invokes the other `run_sens.#.sh` files.
    * The individual sensitivity images from the 5 scripts are copied back to the workstation and summed up, then pushed back to the server into `/mnt/data/userName/explorer/projectFolder/.../UCD/Image/`
    * GUI buttens are enabled again
* Actual sensitivity map creation process (outside the GUI)
    * `run-sens.sh` in invoked from the `respiratory_gui.m`, which in turn invokes `run_sens.#.sh`, which again invoke the corresponging `make_senimg#.m` files; in this case `#` runs from 0 to 4 indicating the unit difference treated in this specific script.
    * `make_senimg#.m`:
        * `blockringdiff_no` indicates the unit difference and is encoded in the file name in `#`
        * Creates file name and sets parameters like voxel size, number of voxels, etc...
        * Check, if mu map exist and is correct. then open it
        * define mu map image and voxel size
        * Open plane and crys eff maps
        * add paths to  `./PETsystem`, which contains the script invoking functions for forward and backprojection etc. This is where things are happening
        * a scanner object is being built using the matlab function `buildPET.m`. It creates an object of class `PETsystem` defined in `/PETsystem/@PETsystem/PETsystem.m` and which then holds the correct scanner parameters and geometry 
        * then sensitivity image is created in line 110: `bp = scanner.cal_senimg_uih(plaeff_wgap, cryseff_wgap, att_image, att_image_size, att_voxel_size, image_size, voxel_size, blockringdiff_no, num_blockrings, num_bins_axial_reduced);`
        * Save sensitivity image
    * `scanner.cal_senimg_uih(...)`  in line 498 of `PETsystem.m` -- this is where the heavy lifting happens
        * Parameters:
            * plaeff_wgap
            * cryseff_wgap
            * att_image
            * att_image_size
            * att_voxel_size
            * image_size (of PET image)
            * voxel_size (of PET image)
            * blockringdiff_no (unit difference)
            * num_blockrings (=8)
            * num_bins_axial_reduced (=84, probably the number of axial crystal per unit)
        * Functionality:
            * f
#### `process_lm_sino_explorer_singlefile_release`
##### General Info
* C++
* Folder `uex/code/explorer-master/read_lm/`
* Make file is in subfolder `src/`
* Dependency `#include "../../include/coinc.h"`: library that allows to read in the binary raw data and extract the individual segments encoding time stamp, etc

##### Description (house keeping in the first part)
* Read config file created in the respiratory GUI. (`scan_details_fullpath = argv[1];`). Contains 8 lines:
    * output folder `outfolder`
    * input raw file path (lm data) `infile_fullpath`
    * raw data number (1 to 8)
    * Static or dynamic (1,2)
    * dynamic frames (1, 100, 2, 200, etc)
    * write processed listmode data (0 or 1)
    * write 3D sinograms (0 or 1)
* Define scanner parameters and global variables
    *  scanner parameters
    * `uint64_t *pRawBuffer = new uint64_t[BUFFER_SIZE];` Buffer to read in events, BUFFER_SIZE is 65536.
    * for reading lm files
    * time tags
    * block count rate variables
    * coincidence counters
    * sinogram variables
    * HistoTOF image variables
    * randoms, dead time, dynamic, counters, etc
* Read listmode
    * check if input file `pInputFile` exists. and check file size (divisible by 8)
    * define output files (e.g. `outfile_fullpath_p`)
* Load LUTs
    * reads in normalization coefficients `vector<float> nc_crys(num_crystals_all);` which is not used again.
    * reads in plane efficiency `vector<float> plaeff(672 * 672);` which is not used again.
    * Same happens with timing LUTs

##### Main Run Program (second part)
* find first time stamp
* get intital count rates
* `if (write_lm == true || run == false || feof(pInputFile))` write to file:
    * `lm_info_f#`
    * 
##### Variables
* `byte_location` for event identification. Gets incremented in multiples of 8 byte = 64 bits (which is the length of one event in the raw data)
* `year00`, `month00`, etc. time stamp of the FIRST event
* `unitA = COINC::GetUiA(pRawBuffer[i]);`	// 0:7 NOT 1:8
* `crys1 = COINC::GetCrysA(pRawBuffer[i]);` // crystal ID within a bank, goes from 0 to 5879
* `bankk = COINC::GetBankPair(pRawBuffer[i]) - 1;` // returns bank pair index 0:53
* `transA = crys1 % 70;` // transaxial crystal ID within a bank
* `modA = bank_lut[bankk][0];`	// returns bank A
* `transA = transA + (modA * 70);`	// absolute transaxial crystal ID; there are 70 transaxial crystals per bank (bank = 2*module)
* `axA = floor(crys1 / 70) + (unitA * 84);`	// absolute axial crystal ID A; there are 84 axial crystals per module/unit

##### Output files
* `lm_info_f#`, where `#` stands for the frame number starting at 0. Contains start and end time, frame start and length and the `byte_location`.
* `lm_reorder_f0_prompts.1.add_fac` . addition factors. random mean of an LOR, also scatters
* `lm_reorder_f0_prompts.1.mul_fac` multiplicative factors. contains normalization and dead time corrections
* `lm_reorder_f0_prompts.1.lm` just list mode data in 10 byte format
* 

## Supplementary Files
### Reconstruction_Parameters_X
* X stands for the unit number 1:8
* Gets created by `respiratory_gui.m` and is handed to `process_lm_sino_explorer_singlefile_release`.
* contains the following 8 lines of information
    * output folder `outfolder`
    * input raw file path (lm data) `infile_fullpath`
    * raw data number (1 to 8)
    * Static or dynamic (1,2)
    * dynamic frames (1, 100, 2, 200, etc)
    * write processed listmode data (0 or 1)
    * write 3D sinograms (0 or 1)


## Code snippets explained
This section has the purpose of explaining certain parts of the code or even single lines more thoroughly. 
**No content is too simple to be added in this section as long as it helps understanding what's going on!**

```
size_t fwrite ( const void * ptr, size_t size, size_t count, FILE * stream );
```
Writes `count` elements each of size `size` from the block of memory pointet to by `ptr` into the current position of the stream `stream`.
```
size_t fread ( void * ptr, size_t size, size_t count, FILE * stream );
```
Writes `count` elements each of size `size` from the stream `stream` and stores them in a block or memory specified by `ptr`.

**bold text**

*italics*

_also italics_