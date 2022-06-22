# UC Davis In-House PET Image Reconstruction Framework

## Documentation of framework architecture and GUI

### Recon GUI
- auxFcnCompilation: collection of helper functions that can be accessed via an object of class auxFcnCompilation('name_tag')
- start GUI using command 
    ```sh
    matlab -nodesktop -r "recon_GUI"
    ```
- d

#### handles overview
| parameter | explanation |
| ------ | ------ |
| `install_dir` | Top level where the source code is located. |
| `frame_choose_base` | File name of the *.raw list mode file |
| `path_choose` | Full path to raw list mode data without filename |
| `path_choose_server` | Same as `path_chose`. Needed because former version used it and it is potnetially used in many other files. |


### ??