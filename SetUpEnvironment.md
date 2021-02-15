# Setting up uboonecode

Log in into the GPVM

```
kinit username@FNAL.GOV
ssh -KXY username@uboonegpvm04.fnal.gov
```

Set up uboonecode

```
source /cvmfs/uboone.opensciencegrid.org/products/setup_uboone.sh
ups list -aK+ uboonecode
setup uboonecode <version> -q <qual>
cd /uboone/app/users/$USER
mkdir uboonecode_<version>
cd uboonecode_<version>
mrb newDev -v <version> -q <qual>
source localProducts_uboonecode_<version>_<qual>/setup
cd $MRB_SOURCE
mrb g -t <tag> uboonecode     # mrb gitCheckout uboonecode (for the most up to date version)
cd $MRB_BUILDDIR
mrbsetenv
mrb install -j4
```

Include other packages

 ```
 cd $MRB_SOURCE
 mrb g -t UBOONE_SUITE_<version> package    # mrb gitCheckout package (for the most up to date version)
 cd $MRB_BUILDDIR
 mrbsetenv
 mrb i -j4
 ```

If you are not installing the most recent uboonecode, make sure to download it (and the packages) using the version tag to avoid version conflicts. 
