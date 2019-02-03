# LitMod

Some things you can do with LitMod:

- Integrated geophysical, petrological, and thermochemical imaging of lithosphere and underlying upper mantle using inversion and forward approaches and different data sets at global and regional scales.
- 3D Lithospheric modelling integrating different geophysical data: potential fields, surface heat flow, elevation, seismic velocities, electrical conductivity, and mantle composition.


## Usage

LitMod is written mostly in Fortran. Compile using a fortran compiler, preferably gfortran:

```
gfortran modules.for LITMOD3D.for SUB* -o litmod
```

or use make. Make sure that the output directory is specified correctly in the makefile.

Input files required to run LitMod may be [downloaded](https://doi.org/10.5281/zenodo.1468861) separately and extracted into the working directory.

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.1468861.svg)](https://doi.org/10.5281/zenodo.1468861)

LitMod requires the following input files:

- `LITMOD3D.info` model parameters such as mesh size and boundary conditions.
- `layers.info` header file that describes the petrological properties of each layer.
- `mnt.info` lists the thermodynamic files to be used.
- Stratigraphic layers in the folder 'layers' (geographical coordinates).
- Stratigraphic layers in the folder 'layers_xy' (Cartesian coordinates).
- Thermodynamic files in the folder 'mant_data'.

`LITMOD3D.job` interfaces with [GMT](https://github.com/GenericMappingTools) and generates a `LITMOD3D.info` file read by LitMod. `LITMOD3D.job` processes the files contained in a folder 'layers' (geographical layers) and puts the output (Cartesian layers) in the folder 'layers_xy'. Note that there is no need to re-run `LITMOD3D.job` unless you change the coordinated of the projects or the number of nodes. LitMod interfaces with [Perple_X](http://www.perplex.ethz.ch/) to produce the thermodynamic files read from the 'mant_data' folder.


## Code structure

Steps 1-6 outline the setup phase of a LitMod simulation; steps 7-11 list the internal routines used by LitMod to compute properties geophysical, petrological, and thermochemical properties of the lithosphere.

1. Define material constants (water, density of sublithospheric mantle, gravity, etc.)
2. Extract geological bodies `layers.info`.
3. Extract sublithospheric mantle bodies `sublith_an.xyz`.
4. Load input parameters from `litmod3d.info` - includes domain size, flags for certain functions.
5. Calibrate elevation.
6. Cut off anything below the transition zone.
7. Call **MATER** - extract thermal properties.
8. Call **TEMPERATURE_3D** - temperature calculation.
9. Call **THERM_COND** - calculate effective thermal conductivity.
10. Call **columns_mat(0)** - calculate elevation without sublithospheric contribution.
11. Call **columns_mat(1)** - calculate density, pressure, gravity, geoid anomaly with sublithospheric contribution.


### List of subroutines

- **MATER** `sub_mater.for`
  - takes the properties from *Perple_X* (werami) using *getimp*.
- **TEMPERATURE_3D** `SUB_TEMPERATURE_3D.for`
  - Solves the steady-state heat equation in 3D.
  - Uses a least-squares solver.
- **THERM_COND** `SUB_THERMAL_COND.for`
  - calculates effective thermal conductivity.
  - T-P dependence based on Hofmeister (1999) with radiative contribution.
- **COLUMNS** `SUB_COLUMNS.for`
  - calculate density, pressure, gravity, geoid anomaly using *Perple_X*.
  - calls to LK_interpol, ELECT_COND, initia, meemum, GRAV_GRAD3D, GEO_GRAD3D, U_SECOND_DER, LSQ_PLANE.
- **LK_INTERPOL** `SUB_LK_interpol.for`
  - this is a simple interpolator
- **ELEC_COND** `SUB_ELEC_COND.for`
  - compute the electrical conductivity
- **initia** `SUB_per_initia.f`
  - initialise Perple_X in case of change in composition
- **meemum** `SUB_per_meemum4_sub2.f`
  - computes the composition for a given temperature/pressure without calculating the whole range like in vertex.
  - calls lpopt0 which does the Gibbs free energy minimisation.
- **GRAV_GRAD3D** `SUB_Grav_Grad3D.for`
  - calculates the gravity anomaly from the density structure.
- **GEOGRAV_GRAD3D** `SUB_GeoGrav_Grad3D.for`
  - same as above but accounting for geoid.
- **U_SECOND_DER** `SUB_U_SECOND_DER.for`
  - calculates the second derivatives of the gravity anomaly
- **LSQ_PLANE** `SUB_LSQ_PLANE.for`
  - I think this has to do with computing the geoid.
- **STATT** `SUB_STAT.for`
  - gets the average, standard deviations.


## Notes (Mattia).
- **layers.info**
  - Last number is 0 if the properties are fixed, =1 is the layer density is read
    from an external file, >1 if the layer is a crustal or mantle body with properties
    computed by LitMod.

## Notes (Eldar)
- **gravcalc_parallel** `gravcalc_parallel.for` and **gravity_calculator** `gravity_calculator.py`
  - external parallel gravity calculator

# LitMod Graphical Interface

Graphical interface for LitMod

## Compilation

Program is written in Fortran. Compile using a fortran compiler, preferably gfortran:
```
gfortran LITMOD3D_INTF.f SUB* -o litmod3d_intf -lpgplot -L/usr/lib -lX11 -fd-lines-as-comments -lpng
```

In **macOS**, first install [iTerm2](https://iterm2.com). Within **iTerm2** terminal install [brew](https://brew.sh) with command:
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Than install [zsh]() with command :
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

Finally, to compile, use:
```
gfortran LITMOD3D_INTF.f -o litmod3d_intf SUB* -lpgplot -L/usr/lib -L/usr/X11/lib -fd-lines-as-comments -lpng
```
Note that is you already had **pgplot** and/or **XQuartz** installed on your Mac, you may have to change the path to libraries. Instructions above were tested on macOS Mojave.

Program and main script `LITMOD_3D.job` uses [GMT4](http://gmt.soest.hawaii.edu), and **GMT4** must be set up in such way that subroutines are called directly, such as:
```
minmax -C ./layers/layer1.xyz
```

To install **GMT4** on Mac, run:
```
brew install gmt4
```
and add the path to **GMT4** into `.zshrc` config file in your homefolder:
```
echo 'export PATH="/usr/local/opt/gmt@4/bin:$PATH"' >> .zshrc
```
