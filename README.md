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
    * `lm_info_f#`, where `#` stands for the frame number starting at 0. Contains start and end time, frame start and length and the `byte_location`.
    * 
##### Variables
* `byte_location` for event identification. Gets incremented in multiples of 8 byte = 64 bits (which is the length of one event in the raw data)

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
This section has the purpose of explaining certain parts of the code or even single lines more thoroughly. No content is too simple to be added in this section as long as it helps understanding what's going on... 

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