# LitMod3D ver 3.1

**LitMod3D** is a software for 3D integrated geophysical-petrological interactive modelling of the lithosphere and underlying upper mantle using a variety of input datasets: potential fields (gravity and magnetic), surface heat flow, elevation (isostasy), seismics, magnetotellurics and geochemical.

Ver 3.1 incorporates a highly optimised Python thermal solver (bi-conjugate gradient squared method), crustal petrology features (thermodynamic equilibrium and metastable) and a parallel gravity forward solver. The new version is intended to work with program get-inp (customized interface to Perple_X, http://www.perplex.ethz.ch/) to generate the inputcrustal and mantle compositional files.



## Installation
Download the distibution archive (https://github.com/javfurchu/litmod/releases/download/v3.0/LitMod3D_V3.1.tgz) to your computer and unpack it into a folder (for example `~/LitMod_proj`): tar -xvzf LitMod3D_V3.1.tgz

### Linux
You have to have [Python 3.0](https://www.python.org/), [PGPLOT](http://www.astro.caltech.edu/~tjp/pgplot/), [gfortran](https://gcc.gnu.org/fortran/), [zsh](http://www.zsh.org/) and [GMT5](http://gmt.soest.hawaii.edu) installed on your computer. In Linux, set **zsh** as a main shell. Change directory to the folder with unpacked files. Then run the script:
```
./PREPARE.job
```
This script will create all necessary folders and compile the programs.

### macOS
In **macOS**, first install [iTerm2](https://iterm2.com). Within **iTerm2** terminal install [brew](https://brew.sh) with command:
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Than install **zsh** with command :
```
brew install zsh
```

Make **zsh** a default shell in **iTerm2** by clicking iTerm2 -> Profiles -> Command: /bin/zsh.

Restart **iTerm2** and install [Oh My Zsh](https://ohmyz.sh) with command:
```
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

Now, install [XQuartz](https://www.xquartz.org):
```
brew cask install xquartz
```

After, install [pgplot](http://www.astro.caltech.edu/~tjp/pgplot/) with command:
```
brew install pgplot
```

Finally, to compile and prepare **LitMod**, use:
```
./PREPARE.job
```
Note that if you already had **pgplot** and/or **XQuartz** installed on your Mac, you may have to change the path to libraries. Instructions above were tested on macOS Mojave.


## Usage

### Preparing GMT4
LitMod executables and main calling script `LITMOD_3D.job` use [GMT5](http://gmt.soest.hawaii.edu), to pre and postprocess files and therefore **GMT5** must be set up to be called directly from the the local LitMod folder, such as:
```
gmt gmtinfo -C ./layers/layer1.xyz
```

To install **GMT5** on **macOS**, run:
```
brew install gmt5
```
and add the path to **GMT5** into `.zshrc` config file in your homefolder:
```
echo 'export PATH="/usr/local/opt/gmt@5/bin:$PATH"' >> .zshrc
```
In **Linux**, follow instructions on http://gmt.soest.hawaii.edu to install **GMT4**.

### Input files
Examples of input files required to run **LitMod** are included into the archive (https://github.com/javfurchu/litmod/releases/download/v3.0/LiMod3D_V3.1.tgz)

LitMod requires a number of input files that are managed from the calling script `LITMOD_3D.job`:

- `LITMOD3D.info` model parameters such as mesh size and the thermal boundary conditions.
- `mnt.info` lists the compositional files to be used in the thermodynamic calculations.

In addition other input files are required:

- `layers.info` header file that describes the physical and petrological properties of each layer in the model.
- Geometry of model layers in the folder 'layers' (geographical coordinates),and the folder 'layers_xy' (Cartesian coordinates).
- Compositional files in the folder 'mant_data'.

### LITMOD_3D.job
`LITMOD_3D.job`  contains all the input values required to run LiMod. The script interfaces with [GMT](https://github.com/GenericMappingTools) and generates `LITMOD3D.info` and `mnt.info` files rquired by LitMod. `LITMOD_3D.job`  also processes the files contained in the folder 'layers' (geographical layers) and puts the output (Cartesian layers) in the folder 'layers_xy' as required by LitMod. 

At the beginning of `LITMOD_3D.job` the user can set up the geopgraphical boundaries of the modelling region (lon_min, lon_max, lat_min, lat_max), the number of nodes in the model (N_x, N_y, N_z) and other parameters. 

`LITMOD_3D.job` preprocesses input grids for geophysical data (e.g., gravity anomaly, surface topography, etc.) and reprojects them onto the modelling region (within lon_min, lon_max, lat_min, lat_max) with defined grid spacing (as per N_x, N_y variables). To make a first run, set parameter pre_pro to 1 and run `LITMOD_3D.job`:
```
./LITMOD_3D.job
```
The distibution provides sample observed data grids in the folder `example_obs`. The user can provide customized data grids with the format lon; lat; value that will be preprocessed by `LITMOD_3D.job`. For the gravity gradients a small program (LNOF2MRF) is provided to rotate the tensor from the Local North Oriented Reference Frame (https://earth.esa.int/web/guest/data-access/view-data-product/-/article/goce-gravity-gradients-in-lnof-5775) to the model reference frame in the Cartesian coordinate system used by LitMod. 


After running `LITMOD_3D.job` in preprocessing mode, switch pre_pro variable to 0, the mode variable to 1  and run `LITMOD_3D.job` again. This will run **LitMod** forward modelling code. The ouput geophysical data sets and the 3D lithospheric model can be visualized using **LitMod** graphical interface, which also can be used to interactively modify the 3D model:
```
./LITMOD3D_INTF
```


A detailed manual is included in the distribution as a PDF file: `LitMod_userguide.pdf`.
