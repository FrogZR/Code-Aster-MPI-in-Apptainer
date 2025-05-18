# Code-Aster-MPI-in-Apptainer

18.05.2025 Build CA dev 17.2.20 and 16.7.11
________________________________________________________________________________________________________

Based on 
- https://github.com/emefff/Code-Aster-MPI-in-Singularity-of-SM2022/blob/main/README.md
- https://github.com/bursica/Code_Aster-MPI-in-Singularity-of-SM2022/blob/main/README.md
Thanks for theire basic work.
________________________________________________________________________________________________________

In the following tutorial we will show how to build the MPI Version of Code Aster 16.7 and 17.2 inside the Singularity Container of Salome-Meca 2022. 

For info on which container to use for which CA version see https://gitlab.com/codeaster-opensource-documentation/opensource-installation-development/-/blob/main/devel/changelog.md). 

A big thanks of course goes to EDF's R&D-Team, the developers of Salome-Meca and Code_Aster 👍. 

This recipe and the resulting container were tested in Fedora 42.(see Code_Aster forum https://code-aster.org/V2/spip.php?rubrique2 for these known minor issues).
________________________________________________________________________________________________________

If you do not have singularity installed, the easiest way is to install the package or install apptainer (succsessor of singularity). Apptainer ([https://apptainer.org/](https://apptainer.org/)) will also be able to run the command singularity.

```
singularity --version
```

create a working folder 
```
mkdir -p ${HOME}/dev/codeaster
cd ${HOME}/dev/codeaster
```
________________________________________________________________________________________________________
Download the necessary Singularity Container of Salome-Meca 2022 with 

```
wget -c https://www.code-aster.org/FICHIERS/singularity/salome_meca-lgpl-2022.1.0-1-20221225-scibian-9.sif
```

The file size should be approx. 6.2GB. 

Alternatively, the container can be downloaded directly from the page https://code-aster.org/V2/spip.php?article303, however, we recommend using wget -c.

Now we put the container file into a sandbox directory
```
sudo singularity build --sandbox ca_sbox salome_meca-lgpl-2022.1.0-1-20221225-scibian-9.sif
```

________________________________________________________________________________________________________
Now we will download the Code_Aster repository from Gitlab. Some of the below instructions can be found here: https://gitlab.com/codeaster-opensource-documentation/opensource-installation-development/-/blob/main/devel/compile.md#preparing-the-access-to-the-container but not all of them are necessary.

We clone the whole repository with

```
git clone https://gitlab.com/codeaster/src.git
git clone https://gitlab.com/codeaster/devtools.git
```

If you want to use and install multiple new code_aster version clone the src into different folders and repeat the following commands for ecah folder, e.g.

```
git clone https://gitlab.com/codeaster/src.git src17220
```

________________________________________________________________________________________________________
Now we select the rigth version of code_aster

```
cd src
git checkout tags/17.2.20
```
And add one file using nano editor 
```
cd code_aster
nano pkginfo.py
```
and add the line 
```
pkginfo = [(17, 2, 20), 'n/a', 'n/a', '16/05/2025', 'n/a', 1, ['no source repository']]

cd ..
cd ..
```
________________________________________________________________________________________________________
All is perpared for  running the conatainer in write mode 

``` 
sudo singularity run --bind ${HOME}/dev/codeaster:/mnt -w ca_sbox shell
```

Now we are at the conatainer 

We change to

```
Singularity> cd /mnt/src
```

Be careful, as we are ***root*** inside the container, the ${HOME} is not your /home/$USER directory anymore.

```
Singularity> 

export TOOLS="/opt/salome_meca/V2022.1.0_scibian_univ/tools"
export ASTER_MPI="${TOOLS}/Code_aster_MPI-17220"
./waf_mpi configure --prefix=${ASTER_MPI} --install-tests --enable-mumps --jobs=8
./waf_mpi build --jobs=8 
./waf_mpi install
cat ${TOOLS}/Code_aster_frontend-202200/etc/codeaster/aster
echo "vers : v17220_mpi:/opt/salome_meca/V2022.1.0_scibian_univ/tools/Code_aster_MPI-17220/share/aster" >> ${TOOLS}/Code_aster_frontend-202200/etc/codeaster/aster
cat ${TOOLS}/Code_aster_frontend-202200/etc/codeaster/aster

Singularity> exit
```

We are now in our normal Linux shell, outside the container. 

The container has to be rebulid
```
sudo singularity build my_new_ca_17220.sif ca_sbox/
```
If you temp folder is too small, you can specifxy a differnet one 

```
export SINGULARITY_TMPDIR=/path/to/your/large/tmpdir
sudo singularity build my_new_ca_17220.sif ca_sbox/
```
or using apptainer 
```
export APPTAINER_TMPDIR=/path/to/your/large/tmpdir
apptainer build my_new_ca_17220.sif ca_sbox/
```

And the sandbox directory ca_sbox can be deleted as well as the orginal sif file.
_______________________________________________________________________________________________________



Exceute the code_aster in batch mode 
```
apptainer exec ~/dec/codeaster/my_new_ca.sif salome shell -- as_run <PATH_TO_EXPORT_FILE>
```

kermit_zr@online.ms






