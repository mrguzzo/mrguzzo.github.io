## Useful links/paths

- Checkout files: `/pnfs/uboone/persistent/users/hanyuwei/WC-LEE/files/data_mc_cv/`
- Example of macro: `/uboone/data/users/rfitzpat/numi_checkout/numi_plots.C`
- [Numu-Argon Cross-Section Internal Note](https://microboone-docdb.fnal.gov/cgi-bin/private/RetrieveFile?docid=33766&filename=MicroBooNE_Wire_Cell_Xs_Analysis_Internal_Note_Jan11.pdf&version=1)

## Information saved in the Checkout files

- Modules used to save information from reco2 files to checkout files (from uboonecode v52): `/ubana/ubana/MicroBooNEWireCell`

The information in the reco2 files follow this structure
- `sim::MCParticles` contains both truth and reconstructed information. It contains more particle propagation information. More information [here](https://nusoft.fnal.gov/larsoft/doxsvn/html/classsimb_1_1MCParticle.html)
- `sim::MCTruth` contains truth information. More information [here](https://nusoft.fnal.gov/larsoft/doxsvn/html/classsimb_1_1MCTruth.html)

You can change `ubana/ubana/MicroBooNEWireCell/WireCellAnaTree_module.cc` if you want to save different information into the checkout file. Currently, the variables saved are relevant to LEE analysis mostly.

## Topology classification

The topologies are classified as follows
```
    // check if cosmic
    if (eval_info.truth_energyInside != 0)
      if ((eval_info.match_completeness_energy/eval_info.truth_energyInside) < 0.1) isCosmic = true;

    // check if outside fiducial volume
    if (eval_info.match_completeness_energy/eval_info.truth_energyInside>=0.1 && eval_info.truth_vtxInside==0) isOutFV = true;

    // check for pi0s
    if (pfeval_info.truth_NprimPio > 0) hasPi0 = true;

    // get interaction type
    if (eval_info.truth_isCC) {
      if (eval_info.truth_nuPdg == 12) isNueCC = true;
      else if (eval_info.truth_nuPdg == -12) isNueBarCC = true;
      else if (abs(eval_info.truth_nuPdg) == 14) isNumuCC = true;
    }
    else { isNC = true; }

    // topology
    int top = -1;

    if (isCosmic) { top = cosmic; nCosmic++; }
    else if (isOutFV) { top = outFV; nOutFV++; }
    else if (isNueCC) { top = -1; nNueCC++; } // use intrinsic nue 
    else if (isNueBarCC) { top = -1; nNueBarCC++; }  // use intrinsic nue
    else if (isNumuCC) {
      if (hasPi0) { top = ccnumupi0; nNumuCC_pi0++; }
      else { top = ccnumu; nNumuCC++; }
    }
    else if (isNC) {
      if (hasPi0) { top = ncpi0; nNC_pi0++; }
      else { top = nc;  nNC++; }
    }
```

More information: [https://github.com/BNLIF/wcp-uboone-bdt/blob/main/inc/WCPLEEANA/cuts.h](https://github.com/BNLIF/wcp-uboone-bdt/blob/main/inc/WCPLEEANA/cuts.h)

## Selection cut

- Remove `tagger_info.numu_cc_flag < 0`
- Remove `!inFidVolRecoWCP(pfeval_info)`
- Electron-neutrino: `tagger_info.numu_cc_flag >=0 && tagger_info.nue_score > 7.0`
- Fully contained events: `eval_info.match_isFC==1` (==0 for partially contained)
