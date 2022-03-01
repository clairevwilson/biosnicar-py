# Input Configuration Guide for BioSNICAR

BioSNICAR is configured by updating the values in the config file `inputs.yaml`. This file is divided into subsections for variables that corrspond to specific parts of the model. These are:

1) `CTRL`: variables that control how the program executes ("metaconfig")
2) `RTM`: variables that configure the radiative transfer model
3) `ICE`: variables that adjust the ice physical properties
4) `PATHS`: paths to specific files
5) `IMPURITIES`: properties of impurities included in the simulation
6) `PLOT`: configuration for plotting output data

The user sets the desired values in `inputs.yaml` and then there is no need to interact with any other script in the repository apart from simpyl runnign the driver `python ./src/snicar_driver.py`. However, the value chpices are quite nuanced and require some explanation. In this document, the meaning of each field in `inputs.yaml` is defined along with any constraints and defaults.

## PATHS
The paths to speficic files are defined here. The user must update the `DIR_BASE` parameter to their own base directory - i.e. the path to the BioSNICAR top-level repository on the users own machine. This depends on where the user has downloaded the source code. All other paths are relative to the base directory and do not need to be updated.

## CTRL
Parameters related to the model execution.

### `SMOOTH`:
#### Definition:
Boolean that toggles smoothing of the predicted albedo. If `SMOOTH==True` a Savistky-Golay smoothign function is applied to the spectral albedo, with a window size and polynomial order defined by `WINDOW_SIZE` and `POLY_ORDER` respectively.

#### Values:
`True` or `False`

#### Default:
`False`

### `WINDOW_SIZE`:
#### Definition:
Interpolation window width used in Savisky-Golay filter if `SMOOTH==True`.

#### Values:
Positive integer. Technically valid between 2-480 but practical between ~5-20.

#### Default:
9


### `POLY_ORDER`:
#### Definition:
Order of polynomial interpolation function used in Savisky-Golay filter if `SMOOTH==True`.

#### Values:
Positive integer. Technically any valid but practical between ~5-20.
#### Default:
`3`

## RTM
### `DIRECT`:
#### Definition:
Toggles between incoming irradiance being diffuse or direct. It's effect is to change a string stub that generates the filepath used to load in iorradiance from file. If direct irradiance is toggled the model reads in a file whose name contains "_clr_" and a solar zenith angle. 

#### Values:
`0` for diffuse irradiance. `1` for direct irradiance.
#### Default:
`1`

### `NBR_WVL`:
#### Definition
Number of individual wavelengths considered by the model. Under normal circumstances the wavelength range is from 0.205 um to 4.995 um in steps of 10 nm, giving 480 individual wavelengths.

#### Values:
positive integer

#### Default:
`480`

### `APRX_TYP`:
#### Definition:
Chooses two-stream approximation type. There are three choices: Eddington, Quadrature and Hemispheric mean. Each is a way to integrate over angle to give a single upwards and downwards flux. The hemispheric mean is derived by assuming the phase function to equal `1 + g` (where `g`= asymmetry parameter) in the forward scattering hemisphere and `1-g` in the backward scattering hemisphere. The hemispheric mean is useful for resolving instabilities in the infrared wavelengths. The equations for each of these approximations is provided in Toon et al. (1989) Table 1. 
    
These choices are only available for the Toon solver - the adding-doubling solver uses hemispheric mean automatically.

#### Values:
`0: Eddington`
`1: Quadrature`
`2: Hemispheric mean`

#### Default:
`2`

### `DELTA`:
#### Definition:
Toggles whether or not the delta function approximation is applied. The delta function truncates the forward scattering peak in the phase function and compensates by adjusting the voluem scattering coefficient, as described [here](https://ui.adsabs.harvard.edu/abs/1970JAtS...27..943P/abstract). 

This choice is only available for the Toon solver - the adding-doubling applies the delta function automatically.

#### Values:
`True: Delta function`
`False: No delta function`
    
#### Default:
`True`


### `SOLZEN`:
#### Definition:
Solar zenith angle. This refers to the angular position of the solar disc relative to vertical (nadir = 0, horizon = 90) expressed in degrees.

#### Values:
Positive integer degrees.
For the Ton solver there are known instabilities for solzen outside the range 50-89. For the AD solver any positive integer between 1-89 is valid.
    
#### Default:
`50`

### `INCOMING`:
#### Definition:
Chooses spectal profile for incoming irradiance from one of seven presets. These are:
```    
0 = mid-latitude winter

1 = mid-latitude summer

2 = sub-Arctic winter

3 = sub-Arctic summer

4 = Summit,Greenland (sub-Arctic summer, surface pressure of 796hPa)

5 = High Mountain (summer, surface pressure of 556 hPa)

6 = Top-of-atmosphere

```
Note that clear-sky spectral fluxes are loaded when direct_beam=1,
and cloudy-sky spectral fluxes are loaded when direct_beam=0

#### Values:
Positive integer between 0 and 6
    
#### Default:
`4`


### `ILLUMINATION_FILE_STUBS`:
#### Definition:
These file stubs are used to select the correct file to open for the incoming irradiance depending on the value of `incoming` and `solzen`.

#### Values:
The file stubs are strings and they are constant - do not change.

#### Default:
```["swnb_480bnd_mlw", "swnb_480bnd_mls", "swnb_480bnd_saw", "swnb_480bnd_sas",  "swnb_480bnd_smm", "swnb_480bnd_hmn", "swnb_480bnd_trp"]```


### `VIS_MAX_IDX`:
#### Definition:
The index in `wavelengths` of the upper wavelength in the visible range.

#### Values:
Technically any positive integer between 1 and 480 is valid, but practically values close to 39 are appropriate as this corresponds to `wavelength` == 0.7um.

#### Default:
`39 `

### `NIR_MAX_IDX`:
#### Definition:
The index in `wavelengths` of the upper wavelength in the near-infrared range.

#### Values:
Technically any positive integer between 1 and 480 is valid, but practically values close to 480 are appropriate as this corresponds to `wavelength` == 4.995 um.

#### Default:
`480 `


## ICE
### `DZ`:
#### Definition:
Thickness of each layer in the model in units of meters.

#### Values:
array with `length == nbr_lyr`. Each element is the thickness of a layer in meters. `dz[0]` is the top layer.

#### Default:
`[0.1, 0.1] `

### `LAYER_TYPE`
#### Definition:
There are two layer tupes available in the model: granular or solid ice. Granular ice corresponds to a bulk medium of air with discrete ice grains. Solid ice refers to ice being the bulk medium with air inclusions. Solid ice layers have Fresnel reflection at their upper surfaces.

The toon solver can only accept granular layers. The adding-doubling solver accepts either type including a mix of both in any order.

#### Values:
`0` for granular ice
`1` for solid ice

#### Default:
`[0, 0] `


### `RHO`
#### Definition:
Ice density in each layer measured in kg/m3. 

#### Values:
Positive integers up to 917 (density of pure ice).

#### Default:
`[400, 400]`

### `RF`
#### Definition:
Choice of refractive index for ice. The refractive index of pure ice has been measured several times over the past 5 decades. Here we toggle between several of those datasets. The options are:

```
0: Warren et al. (1984)

1: Warren et al. (2008)

2: Picard et al. (2016)
```

#### Values:
Positive integer between 0 and 2.

#### Default:
`2` 

### `CDOM`
#### Definition:
Chromophoric dissolved organic matter (CDOM) is coloured organic matter that can be found in glacier meltwater. This variable toggles its presence on or off in each layer. This is an experimental feature that only allows a single concentration and it can only be added to solid ice layers (layer_type=1). The CDOM absorption data come from an upcoming paper by Halbach et al. Toggling CDOM on means adjustign the imaginary refractive index of the ice for the presence of CDOM, assuming it to well mixed through the ice layer.

#### Values:
0 for no CDOM, 1 for CDOM

#### Default:
`[0, 0] `


### `SHP`
#### Definition:
Shape of ice grains. Adjustments for grain shape can onyl be made when layer_type ==0. The available shapes are:
```
0 = sphere

1 = spheroid

2 = hexagonal plate

3 = koch snowflake

4 = hexagonal prisms
```
<br>
The adjustments to the loaded optical properties for spheroids, hexagonal plates and Koch snowflakes are according to Fu et al. 2007 and He et al. 2016. Hexagonal columns are calculated offline using geometric optics calculations and loaded in as separate files.

#### Values:
Positive integer between 0 and 4

#### Default:
`[0, 0]`

### `RDS`
#### Definition:
Effective radius of ice grains if `layer_type==0` and effective radius of air bubbles if `layer_type==1`. The valid range varies according to choice of RF - we have a larger database for Picard et al. 2016 refractive indices than the other because it is our default. Please check the `/Data/OP_data/480band/` directory for available radii.

#### Values:
Positive integer between 30 and 20000 (if `rf==2`).

#### Default:
`[1000, 1000]`

### `WATER`
#### Definition:
Thickness of liquid water coating around ice grains. Only valid when `shp==0` and `layer_type==0`. The value given should be the total radius of the grain and its water coating. For example for a 100 um grain with a 10 um water coating, the value of `WATER` should be 110. Providing a value that is less than the value provided for `rds` effectively toggles this off. The water coatings are quite expensive calculations as they run a Mir scattering code on the fly. Therefore they are off by default.

#### Values:
Positive integer.

#### Default:
`[0, 0]`

### `HEX_SIDE`
#### Definition:
Length of one of the six sides of the hexagonal face of the iuce grain when `shp==4`. The available size combinations can be seen by browsing `/Data/OP_data/480band/ice_hexagonal_columns`.

#### Values:
Positive integer.

#### Default:
`[10000, 10000]` 

### `HEX_LENGTH`
#### Definition:
Length of the long axis of the ice grain when `shp==4`. The available size combinations can be seen by browsing `/Data/OP_data/480band/ice_hexagonal_columns`.

#### Values:
Positive integer.

#### Default:
`[10000, 10000]`

### `SHP_FCTR`
#### Definition:
Ratio of nonspherical grain effective radii to that of an equal volume sphere. This is only active when `shp>1` i.e. the grains are non-spherical. 

#### Values:
Float between 0-1.

#### Default:
`[0, 0]`


### `AR`
#### Definition:
Aspect ratio of grain, i.e. ratio of width to length. 

#### Values:
Positive float or integer.

#### Default:
`[0, 0]` 

## IMPURITIES
Impurities are provided as instances of the Impurity class. The clas has fixed attributes that must be populated in order for the impurity to be included in the mdoel. These attributes are: `NAME`, `FILE`, `CFACTOR`, `COATED`, `UNIT`, and `CONC`. These are user defined in `inputs.yaml`. There is no upper or lower limit to the number of different impurities that can be added - the user can simple continue to stack additional impurities in the `IMPURITIES` section of `inputs.yaml`. The basic structure of an impurity as defined in `inputs.yaml` is as follows:

```
IMPURITIES:
  BC:
    NAME: "bc"
    FILE: "bc_ChCB_rn40_dns1270.nc"
    CFACTOR: 1
    COATED: False
    UNIT: 0
    CONC: [0 ,0]
```

### `NAME`
The impurity name is simply a user defined tag to recognise the instance of the impurity. It can be anything the user chooses but we suggest a simple descriptive word or acronym. 

### `FILE`
The file is the precise filename in `/Data/OP_data/480band/lap/` that contains the optical properties of the impurity to be included. The available options are quite comprehensive, including:

```
"bc_ChCB_rn40_dns1270.nc": uncoated black carbon (Bohren and Huffman, 1983)
"miecot_slfsot_ChC90_dns_1317.nc": sulfate-coated BC (Bohren and Huffman, 1983)
"brC_Kirch_BCsd.nc": uncoated brown carbon (Kirchstetter et al. (2004).)
"brC_Kirch_BCsd_slfcot.nc": sulfate-coated brown carbon (Kirchstetter et al. (2004)
"dust_balkanski_central_size1.nc": dust size 1 (r=0.05-0.5um) (Balkanski et al 2007)
"dust_balkanski_central_size2.nc": dust size 2 (r=0.5-1.25um) (Balkanski et al 2007)
"dust_balkanski_central_size3.nc": dust size 3 (r=1.25-2.5um) (Balkanski et al 2007)
"dust_balkanski_central_size4.nc": dust size 4 (r=2.5-5.0um)  (Balkanski et al 2007)
"dust_balkanski_central_size5.nc": dust size 5 (r=5.0-50um)  (Balkanski et al 2007)
"volc_ash_eyja_central_size1.nc": volcanic ash size 1 (r=0.05-0.5um) (Flanner et al 2014)
"volc_ash_eyja_central_size2.nc": volcanic ash size 2 (r=0.5-1.25um) (Flanner et al 2014)
"volc_ash_eyja_central_size3.nc": volcanic ash size 3 (r=1.25-2.5um) (Flanner et al 2014)
"volc_ash_eyja_central_size4.nc": volcanic ash size 4 (r=2.5-5.0um) (Flanner et al 2014)
"volc_ash_eyja_central_size5.nc": volcanic ash size 5 (r=5.0-50um) (Flanner et al 2014)
"volc_ash_mtsthelens_20081011.nc": ashes from Mount Saint Helen's
"dust_skiles_size1.nc": Colorado dust 1 (Skiles et al 2017)
"dust_skiles_size2.nc": Colorado dust 2 (Skiles et al 2017)
"dust_skiles_size3.nc": Colorado dust 3 (Skiles et al 2017)
"dust_skiles_size4.nc": Colorado dust 4 (Skiles et al 2017)
"dust_skiles_size5.nc": Colorado dust 5 (Skiles et al 2017)
"dust_greenland_central_size1.nc": Greenland dust 1 (Polashenski et al 2015)
"dust_greenland_central_size2.nc": Greenland dust 2 (Polashenski et al 2015)
"dust_greenland_central_size3.nc": Greenland dust 3 (Polashenski et al 2015)
"dust_greenland_central_size4.nc": Greenland dust 4 (Polashenski et al 2015)
"dust_greenland_central_size5.nc": Greenlanddust 5 (Polashenski et al 2015)
"dust_greenland_Cook_LOW_20190911.nc": Dark Zone dust 1 (Cook et al. 2019 "LOW")
"dust_greenland_Cook_CENTRAL_20190911.nc": Dark Zone dust 2 (Cook et al. 2019 "mean") 
"dust_greenland_Cook_HIGH_20190911.nc": Dark Zone dust 3 (Cook et al. 2019 "HIGH") 
"snw_alg_r025um_chla020_chlb025_cara150_carb140.nc": Snow Algae (Cook et al. 2017a, spherical, C nivalis)
"SA_Chevrollier2022_r8.99.nc": Snow algae (Chevrollier et al 2022) NOT YET FUNCTIONAL
"Cook2020_glacier_algae_4_40.nc": Glacier Algae (Cook et al. 2020)
"GA_Chevrollier2022_r4.9_L18.8.nc": Glacier algae (Chevrollier et al 2022) NOT YET FUNCTIONAL
```

### `UNIT`
All these impurities have their mass absorption coefficients in units of ppb, and therefore their concentration (`CONC`) should be expressed normalised to mass in the model - i.e. in ng/g or ppb. This is achieved by setting `UNIT ==0`. There are two exceptions to this: `"SA_Chevrollier2022_r8.99.nc"` and `"GA_Chevrollier2022_r4.9_L18.8.nc"`. These are empirically measured optical properties for snbow and glacier algae whose absorption coefficient is expressed in m2/cell. Their concentration (`CONC`) should therefore be expressed in cells/mL. This is achieved by setting `UNIT==1`. There are validity checks in place to ensure these values are set correctly.



### `CONC`

Array containing the concentration in each layer in the model in units defined by `UNIT`. 

### `CFACTOR`
The `CFACTOR` attribute is a multiplier that accounts for field measurments having a coarse vertical resolution but the model havign fine vertical resolution. This is mostly applicable toi glacier algae that are known to accumulate on the upper surface of the ice, but field samples generally are onyl able to remove the top approx. 2cm for cell counting. this leads to an underestimate of the concentration int he upper mm by including ~2cm of dilute clean ice below. `CFACTOR` accounts for this problem. It is a simple multiplier so it must be set to `1` by default.

### `COATED`
Boolean that toggles sulfate coating on or off. This is only applicable to black and brown carbon where there are files for both coated an uncoated versions. The parameter is requird because a different field in the optical property file is loaded for coated compared to uncoated impurities, so that the mass normalization takes into account the mass of both the impurity and its coating.


## PLOT
These parameters control how the output albedo plot is formatted. Recommend leaving them all at their default values. The user may, however, want to toggle figure saving on and off, in which case update the value of `SAVE`.
