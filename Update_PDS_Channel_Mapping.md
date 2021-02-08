# Update the number of PDS channels
---

If you have added or removed photodetectors in your geometry, you should let LArSoft know about that. You do that by updating `$MRB_SOURCE/sbndcode/sbndcode/OpDetSim/sbnd_pds_mapping.json` which is the channel mapping of our simulation. Meaning, this is the file where the relation between the channel number and the kind of detector is established. There are a few steps you should do in order to update the number/label of your photodetectors.

## Getting current channel mapping

First, run `analyseIt:[pmtresponse]` in your FHICL file to get the position of each channel. Create a copy of `$MRB_SOURCE/sbndcode/sbndcode/JobConfigurations/prodsingle_sbnd.fcl` and call it `prodsingle_sbnd_givemechannels.fcl`, for example. Add the following lines:

```
#include "opticaldetectormodules_sbnd.fcl"
```

In the `physics:{ }` block:

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

Please make sure to place `analyzeIt:[ ]` after `simulate:[ ]`, and to add analyzeIt in `end_paths:[analyzeIt]`. And, finally, at the end of your FHICL file, change the default SimPhotonsLite to false, because the SimPhotonCounter_module uses SimPhotons instead to fill the different trees.

```
# channel mapping
services.LArG4Parameters.UseLitePhotons: false
```

Now run your FHICL file

```
lar -c prodsingle_sbnd_givemechannels.fcl -n 1
```

And notice that a channel map will be printed on your terminal. The map looks like

```
0 212.415 135.3 472.62
1 -212.415 135.3 472.62
2 212.415 124.7 472.62
3 -212.415 124.7 472.62
```

and it obbeys: `channel number - X position - Y position - Z position` of each photodetector in the simulation.

## Identify the channels

Now, identify the channels manually and include the proper label for each of them in `$MRB_SOURCE/sbndcode/sbndcode/OpDetSim/sbnd_pds_mapping.json`. To do so do:

1. Create a spreadsheet with the columns you want (we are currently using `channel | pd_type | pds_box | sensible_to_vuv | sensible_to_vis | tpc`, just like you can see [here](https://docs.google.com/spreadsheets/d/1sC1uMdWCc9Qyir6qS60y43RNDyfQxHaXRdVc-ACg9Q8/edit#gid=0)).
2. Download the spreadsheet as .cvs
3. Run this [script](https://github.com/mrguzzo/SBND_geometry/blob/ongoingWorkBranch/create_channel_mapping.ipynb) over your .cvs spreadsheet. It will create a unreadable .json file.
4. Open the .json file in the Firefox browser > Raw Data > Pretty Print > Save. Save it as `sbnd_pds_mapping.json`. It will be like the following:

```
[
  {
    "channel": 0,
    "pd_type": "xarapuca_vis",
    "pds_box": 0,
    "sensible_to_vuv": false,
    "sensible_to_vis": true,
    "tpc": 0
  },
  {
    "channel": 1,
    "pd_type": "xarapuca_vis",
    "pds_box": 1,
    "sensible_to_vuv": false,
    "sensible_to_vis": true,
    "tpc": 1
  },
  (...)
```
5. Move this file to `$MRB_SOURCE/sbndcode/sbndcode/OpDetSim/sbnd_pds_mapping.json`.
6. Compile your environment

````
cd $MRB_BUILDDIR
mrb z
mrbsetenv
mrb i -j4
````

You should be able to run your simulation now using the new PDS channel mapping.
