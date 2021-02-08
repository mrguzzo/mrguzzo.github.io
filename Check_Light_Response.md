# Check the light response

After running the simulation, you can generate a plot of the number of photons per channel. To do so, follow the steps below.

## Save light information

The light information is not saved as default. To change that, you should run `pmtresponse`. Pick the `prodsingle.fcl` of your preference and add the following lines.

```
#include "opticaldetectormodules_sbnd.fcl"
```

In the `physics:{ }` block

```
physics:
{
  producers:
  {
    (...)
  }
  
  analyzers:
  {
    pmtresponse: @local::sbnd_simphotoncounter
  }
  
  simulate: [ (...) ]
  analyzeIt: [pmtresponse]
  end_paths: [analyzeIt]
}
```

Make sure to add `pmtresponse` in `analyzeIt:[ ]`, and to add `analyzeIt` in `end_paths:[ ]`.

## Run the simulation

Run the simulation chain with the `prodsingle.fcl` you just modified.

```
lar -c prodsingle.fcl -n <nevents> -o gen.root
lar -c standard_g4_ref_sbnd.fcl -s gen.root -o g4.root
lar -c standard_detsim_ref_sbnd.fcl -s g4.root -o detsim.root
```

## Make the plots

Run `/sbnd/app/users/mguzzo/my_codes/geometry/plot_pds_response_geometry_<version>.cc` for the geometry version you are running.

```
root -l 'plot_pds_response_geometry_<version>.cc("detsim.root")'
```
