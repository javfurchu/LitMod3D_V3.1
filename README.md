# LitMod

Some things you can do with **LitMod**:

- Integrated geophysical, petrological, and thermochemical imaging of lithosphere and underlying upper mantle using inversion and forward approaches and different data sets at global and regional scales.
- 3D Lithospheric modelling integrating different geophysical data: potential fields, surface heat flow, elevation, seismic velocities, electrical conductivity, and mantle composition.

## Installation
Download prepared archive (https://github.com/javfurchu/litmod/releases/download/v3.0/litmod_3.0.zip) to your computer and unpack it into the folder (for example `~/litmod`). 

### Linux
You have to have [Python 2.7](https://www.python.org/), [PGPLOT](http://www.astro.caltech.edu/~tjp/pgplot/), [gfortran](https://gcc.gnu.org/fortran/), [zsh](http://www.zsh.org/) and [GMT4](http://gmt.soest.hawaii.edu) installed on your computer. In Linux, set **zsh** as a main shell. Change directory to the folder with unpacked files. Than run the script:
```
./PREPARE.job
```
This script will create all necessary folders and compile the program.

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
Note that is you already had **pgplot** and/or **XQuartz** installed on your Mac, you may have to change the path to libraries. Instructions above were tested on macOS Mojave.


## Usage

### Preparing GMT4
Program and main script `LITMOD_3D.job` uses [GMT4](http://gmt.soest.hawaii.edu), and **GMT4** must be set up in such way that subroutines are called directly, such as:
```
minmax -C ./layers/layer1.xyz
```

To install **GMT4** on **macOS**, run:
```
brew install gmt4
```
and add the path to **GMT4** into `.zshrc` config file in your homefolder:
```
echo 'export PATH="/usr/local/opt/gmt@4/bin:$PATH"' >> .zshrc
```
In **Linux**, follow instructions on http://gmt.soest.hawaii.edu to install **GMT4**.

### Input files
Examples of input files required to run **LitMod** are included into the archive (https://github.com/javfurchu/litmod/releases/download/v3.0/litmod_3.0.zip)

LitMod requires the following input files:

- `LITMOD3D.info` model parameters such as mesh size and boundary conditions.
- `layers.info` header file that describes the petrological properties of each layer.
- `mnt.info` lists the thermodynamic files to be used.
- Stratigraphic layers in the folder 'layers' (geographical coordinates).
- Stratigraphic layers in the folder 'layers_xy' (Cartesian coordinates).
- Thermodynamic files in the folder 'mant_data'.

### LITMOD_3D.job
`LITMOD_3D.job` interfaces with [GMT](https://github.com/GenericMappingTools) and generates a `LITMOD3D.info` file read by LitMod. `LITMOD_3D.job` processes the files contained in a folder 'layers' (geographical layers) and puts the output (Cartesian layers) in the folder 'layers_xy'. 

In the beginning of `LITMOD_3D.job` user can set up boundaries of the region (lon_min, lon_max, lat_min, lat_max), number of nodes in the model (N_x, N_y, N_z) and other parameters. 

`LITMOD_3D.job` preprocesses input observed grids and reprojects them onto preferred region (within lon_min, lon_max, lat_min, lat_max) with defined (in N_x, N_y, N_z) spacing. To make a first run, set parameter pre_pro to 1 and run `LITMOD_3D.job`:
```
./LITMOD_3D.job
```
Sample observed data grids are placed in the folder `example_obs`, and all of them have format lon; lat; value except GOCE gravity gradients grid `GO_CONS_GGG_255_G03S_20100201T000000_20131111T235959_0002`. ONe can put their own grids instead of sample grids provided with the release archive.

After running `LITMOD_3D.job` in preprocessing mode, switch pre_pro to 0 and run `LITMOD_3D.job` again. This will run **LitMod** and the result of calculations can be observed using **LitMod** graphical interface, which is called by command:
```
./litmod_intf
```

Detailed manual is included into archive as a PDF file `LitMod_userguide.pdf`.
