# Producing files on the log-in node (locally)

To run LArSoft commands on Theta, you should do use a bash (aka singularity) code like the one below. 

Please, make sure that all paths on Theta should have `/lus/theta-fs0/` + the `pwd` output.

It is crucial to open the directory where you want the FCL file to be run inside the singularity, otherwise, Theta tries to run it in the home directory, and it does not have permission to write. You should run it using:

```
./singularity_example.sh
```

Everything between `EOF ... EOF` is being run inside singularity. The `EOF` thing is called a `heredoc`, and its being used to essencially pass these commands to singularity like an argument to this `singularity run` call.

<details>
    <summary>singularity_example.sh</summary>

    
    #!/bin/bash

    # ----- launch singularity container

    singularity run --no-home -B /lus:/lus -B /soft:/soft /lus/theta-fs0/projects/ReconMBNE/containers/singularity_slf7-balsam.sif << EOF

        # ----- Setup the pulled version of uboonecode (this is like sourcing cvmfs setup_uboone.sh)

        source /lus/theta-fs0/projects/ReconMBNE/uboonecode_v08_00_00_01b/setup

        # ----- Now we can setup uboonecode

        setup uboonecode v08_00_00_01b -q e17:prof

        # ----- Opening folder where you want the LArSoft outputs to be saved
        # ----- this step if very important otherwise your singularity will
        # ----- try to save the output in the home directory and will fail

        cd /lus/theta-fs0/projects/ReconMBNE/testing_cosmic_production/

        # ----- LArSoft commands

        lar -c ...

    EOF

    # After the EOF, we have now exited the singularity container.
    echo "Exited Container" 
    
</details>

---

# Producing files on the compute nodes (submitted through Balsam)

---
## Activate the database

Run every time you log in on a Theta gpvm.

```
cd /projects/ReconMBNE/
source env/bin/activate
module load postgresql
balsam init /lus/theta-fs0/projects/ReconMBNE/uboone_balsam      # only the first time
source balsamactivate uboone_balsam
```

---
## Create a Singularity

The singularity files are saved in `/lus/theta-fs0/projects/ReconMBNE/applications`.

<details>
    <summary>singularity_example.sh</summary>

    ```
    #!/bin/bash

    # launch singularity container
    singularity run --no-home -B /lus:/lus -B /soft:/soft /lus/theta-fs0/projects/ReconMBNE/containers/singularity_slf7-balsam.sif << EOF

        # ----- Setup the pulled version of uboonecode (this is like sourcing cvmfs setup_uboone.sh)
        source /lus/theta-fs0/projects/ReconMBNE/uboonecode\_v08\_00\_00\_01b/setup

        # ----- Now we can setup uboonecode
        setup uboonecode v08\_00\_00_01b -q e17:prof

        # ----- Opening folder
        cd /lus/theta-fs0/projects/ReconMBNE/testing\_cosmic\_production/

        # ----- Running Corsika
        lar -c /lus/theta-fs0/projects/ReconMBNE/testing\_cosmic\_production/prodcorsika\_on\_theta.fcl -n 1 -o corsika.root

        # ----- Running Geant4
        lar -c wirecell\_g4\_uboone.fcl -s corsika.root -o g4.root

        # ----- Running Detsim
        lar -c wirecell\_detsim\_uboone.fcl -s g4.root -o detsim.root

        # ----- Running Reco1&2
        lar -c reco\_uboone\_mcc9\_8\_driver\_stage1.fcl -s detsim.root -o reco1\_reco2.root

    EOF

    # After the EOF, we have now exited the singularity container.
    echo "Exited Container"
    ```

</details>

---
## Applications

Add application. It uses the path of the executable, so you can change it as much as you want after creating the application. Remember to use the full path to the executable
```
balsam app --name <application name> --executable /lus/theta-fs0/projects/ReconMBNE/applications/<file>.sh
```

Check list of applications:
```
balsam ls apps
```

Remove an application:
```
balsam rm apps --name <application name>
```

---
- **Create workflow and populate a database**

    The scripts are saved in `/lus/theta-fs0/projects/RecomMBNE/scrips`.

    ```
    python add_workflow_<file>.py
    ```

    You should be able to see the list of files that will be processed by typing

    ```
    balsam ls
    ```

---
- **Submit jobs**

    ```
    balsam submit-launch -n 8 -t 60 -A <project_name> -q debug-flat-quad --job-mode serial --wf-filter <wf_name>
    ```

    Where:

    ```
    -n <n_nodes>          = 8                    // number of nodes
    -t <length>           = 60                   // length
    -A <project_name>     = ReconMBNE            // name of the project
    ```

    After submitting jobs, you can check the status of it by running one of the options below, depending on your goal.

    ```
    balsam ls                                 // returns a table with the status of each file
    balsam ls --by-state                      // returns a summary of the status in general
    balsam ls --by-state --wf <wf_name>       // returns a summary of the status in general for a specific workflow
    ```

    The possible outputs of the command above are:

    - CREATED:
    - PREPROCESSED:
    - JOB_FINISHED:
    - RESTART_READY:
    - FAILED:

    You can also check the status of your jobs [here](https://status.alcf.anl.gov/theta/activity)

---
# Re-run jobs using the same files
 
Once your jobs are done, you can check the general success rate with `balsam ls --by-state`, which will give you the number of jobs that successfully finished (JOB_FINISHED), the number that failed (FAILED) and the number that were not successfull but

## Re-run failed jobs

1. Change the status of the FAILED ones to RESTART_READY.
    ```
    python scripts/workflow_database_failed_to_restart_ready.py
    ```
2. Submit your jobs again. It will re-run all the RESTART_READY ones.
    ```
    balsam submit-launch -n 8 -t 60 -A <project_name> -q debug-flat-quad --job-mode serial --wf-filter <wf_name>
    ```

## Re-run all files again


If you want to re-run your Balsam jobs on the same files, you should do the following:

1. Clean the balsam database
    ```
    python scripts/workflow_database_clean.py
    balsam ls                                        // should be empty
    ```
2. Populate the database again:
    ```
    python scripts/add_workflow_<file>.py
    balsam ls                                        // should return the list of files
    ```
3. Delete the `/uboone_balsam/data/<workflow>/` folder.
    ```
    rm -r /lus/theta-fs0/projects/ReconMBNE/uboone_balsam/data/<workflow>
    ```
4. Submit your jobs again.
    ```
    balsam submit-launch -n 8 -t 60 -A <project_name> -q debug-flat-quad --job-mode serial --wf-filter <wf_name>
    ```