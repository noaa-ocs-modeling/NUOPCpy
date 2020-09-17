# NEMSpy
### Python wrapper for the NOAA Environmental Modeling System

[![tests](https://github.com/noaa-ocs-modeling/NEMSpy/workflows/tests/badge.svg)](https://github.com/noaa-ocs-modeling/NEMSpy/actions?query=workflow%3Atests)
[![build](https://github.com/noaa-ocs-modeling/NEMSpy/workflows/build/badge.svg)](https://github.com/noaa-ocs-modeling/NEMSpy/actions?query=workflow%3Abuild)

This repository implements the [National Unified Operational Prediction Capability (NUOPC)](https://www.earthsystemcog.org/projects/nuopc/).

#### Usage:
```python
from datetime import timedelta

from nemspy import ModelingSystem
from nemspy.model import ADCIRC, AtmosphericMesh, NationalWaterModel, WaveMesh

interval = timedelta(hours=1)
ocean_model = ADCIRC(300)
wave_mesh = WaveMesh()
atmospheric_mesh = AtmosphericMesh()
hydrological_model = NationalWaterModel(769)

nems = ModelingSystem(interval, ocean=ocean_model, wave=wave_mesh, atmospheric=atmospheric_mesh, hydrological=hydrological_model)
nems.connect('atmospheric', 'ocean')
nems.connect('wave', 'ocean')
nems.connect('atmospheric', 'hydrological')
nems.connect('wave', 'hydrological')
nems.connect('ocean', 'hydrological')

nems.write('nems.configure')
```

the resulting `nems.configure` file looks like this:
```fortran
#############################################
####  NEMS Run-Time Configuration File  #####
#############################################

# EARTH #
EARTH_component_list: ATM WAV OCN HYD
EARTH_attributes::
  Verbosity = min
::

# ATM #
ATM_model:                      atmesh
ATM_petlist_bounds:             0 0
ATM_attributes::
  Verbosity = min
::

# WAV #
WAV_model:                      ww3data
WAV_petlist_bounds:             1 1
WAV_attributes::
  Verbosity = min
::

# OCN #
OCN_model:                      adcirc
OCN_petlist_bounds:             2 12
OCN_attributes::
  Verbosity = min
::

# HYD #
HYD_model:                      nwm
HYD_petlist_bounds:             13 781
HYD_attributes::
  Verbosity = min
::

# Run Sequence #
runSeq::
  @3600
    ATM -> OCN   :remapMethod=redist
    WAV -> OCN   :remapMethod=redist
    ATM -> HYD   :remapMethod=redist
    WAV -> HYD   :remapMethod=redist
    OCN -> HYD   :remapMethod=redist
    ATM
    WAV
    OCN
    HYD
  @
::
```

#### Related Repositories:
- https://github.com/NOAA-EMC/NEMS
- https://github.com/esmf-org/esmf
