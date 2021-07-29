# hera-singularity

## Notice

__July 15, 2021__:
We are currently manually building and uploading the containers to the HERA project directory on Ilifu on an irregular basis. Please check the built dates of the container files and contact @piyanatk if you need the containers to be rebuilt. Scheduled daily re-building is being planned.

---

This repository contains recipe files of the Singularity containers for the HERA software stack.

Ilifu users, please make sure to read the relevant page on the HERA wiki. A singularity container is required for computing on the Ilifu. If you need specific Python modules to be installed in the containers, please contact @piyanatk.

## About Container and Singularity
Containers are encapsulated software environments and abstract the software and applications from the underlying operating system. This allows users to run workflows in customized environments, switch between environments, and to share these environments with colleagues and research teams.

Singularity is a free, cross-platform and open-source computer program that performs operating-system-level virtualization also known as containerization (another widely used one being Docker).


## Container Content

### Python Packages
All containers are built with `Ubuntu 20.04` and `miniconda` with `python=3.8` unless otherwise specify [below](###-Different-Between-Containers:). All variances come standard with the following packages:

| Data Analysis  | Astronomical       | HERA         |
| -------------- | ------------------ | ------------ |
| `dask`         | `aipy`             | `linsolve`   |
| `jupyterlab`   | `astropy`          | `uvtools`    |
| `matplotlib`   | `astropy-healpix`  | `hera_qm`    |
| `numpy`        | `astroquery`       | `hera_cal`   |
| `pandas`       | `cartopy`          | `hera_sim`   |
| `scipy`        | `healpy`           | `hera_psepc` |
| `scikit-learn` | `pyuvdata`<sup>[1](#myfootnote1)</sup>         |
| `xarray`       | `pyuvsim`<sup>[2](#myfootnote2)</sup>          |
|                | `SSINS`<sup>[3](#myfootnote3)</sup>          |

<a name="myfootnote1">1</a>: With CASA measurement sets, HEALPix beam, and CST beam functionalities, see https://github.com/RadioAstronomySoftwareGroup/pyuvdata\
<a name="myfootnote2">2</a>: With profiling and full simulator, see https://github.com/RadioAstronomySoftwareGroup/pyuvsim
<a name="myfootnote3">3</a>: See https://github.com/mwilensky768/SSINS

### Variances:

We are currently building the following variances.

- `hera1`:
  - Include all packages in the table above. Intended for general-purpose computing.
- `casa6_full`:
  - Equivalent to `hera1` with a full installation of `casa-6`, and `APLpy` for visualisation.
- `casa6_modular`:
  - Equivalent to `hera1` with a pip-wheel installation of `casa-6`, making `casatasks`, `casatools`, and `casampi` packages (see https://casa-pip.nrao.edu/), and `APLpy`
  - Based on `Python 3.6` and `Ubuntu 18.04` for casa-pip compatibility.
- `rtp`:
  - For testing the `makeflow` pipeline.
  - Equivalent to `hera1` with an addition of `hera_opm`, `hera_mc`, and  `hera_notebook_templates`.
  - `hera_pipelines` is cloned to `/usr/local`
- `h4c`:
  - Almost equivalent to `rtp` except some specific branches on `hera_cal` and `pspec` for H4C analysis.
- `tau`:
  - This container is `hera1` with extra tools for simulation, machine learning, and etc. Specifically, it contains the following additions:
    - emupy (https://github.com/nkern/emupy)
    - zreion (https://github.com/plaplant/zreion)
    - 21cmFAST=3.1.1
    - powerbox
    - tensorflow
    - pytorch
    - keras
    - sympy
    - numexpr

### Python Environment

All containers use Miniconda3, which are installed at `/usr/local/miniconda3/` inside the containers.

The name of Conda environment in each container is the same as the container name, e.g. `hera1`, `casa6_full`, and etc, The default conda environment `base` is not used.


### Environment Variables
The following environment variables are also exported in all containers:

```
CONDA_INSTALL_PATH="/usr/local/miniconda3"
CONDA_INIT_SCRIPT="$CONDA_INSTALL_PATH/etc/profile.d/conda.sh"
```

The latter is especially useful to make the `conda` command available inside the container (see the section on [`singularly shell` usage](####-`shell`) below).

The `rtp` container has an additional environment variable that point to `hera_pipelines`.

```
HERA_PIPELINES_PATH="/usr/local/hera_pipelines"
```

## Usage

### Singularity Commands

#### `pull`
Use `singularity pull` to download the container from Singularity Hub
```
$ singularity pull [name_to_save_the_image_(optional)] shub://HERA-Team/hera-singularity:<recipe>
```
For example,
```
$ singularity pull rtp.sif shub://HERA-Team/hera-singularity:rtp
INFO:    Downloading shub image
 1.98 GiB / 1.98 GiB [=======================================================] 100.00% 13.12 MiB/s 2m34s
 ```

#### `shell`
The `singularity shell` command allows you to spawn a new shell within your container and interact with it as though it were a small virtual machine.

By default, `shell` invokes `/bin/sh --norc`, which means that `.bashrc` will not be executed (more on this [here](https://github.com/hpcng/singularity/issues/643)) and thus Conda will not be initialized. To make the `conda` command available, you can do one of the following:

a) Run `exec $SHELL` inside the singularity shell. If `$SHELL` is `\bin\bash` (as in our Ubuntu build), `.bashrc` will be read.
```
$ singularity shell rtp.sif
Singularity> exec $SHELL
```

b) Manually execute the conda initialization script inside singularity shell. The `CONDA_INIT_SCRIPT` environment variable pointing to the absolute path of the script (`/usr/local/miniconda3/etc/profile.d/conda.sh`), is made available for this purpose. Note that `.` must be used as `source` won't work under `sh`.
```
$ singularity shell rtp.sif
Singularity> . $CONDA_INIT_SCRIPT
```

b) Specify `\bin\bash` as a shell to use when executing the `shell` command, either by using the `SINGULARITY_SHELL` environment variable,
```
$ SINGULARITY_SHELL=/bin/bash singularity shell hera-rtp.sif
```
or `-s` option,
```
$ singularity shell -s /bin/bash hera-rtp.sif
```

#### `exec`
The `singularity exec` command allows you to execute a custom command within a container by specifying the image file.
```
$ singularity exec rtp.sif echo "Hello World!"
Hello World!
```
```
$ cat myscript.sh
Hello World!
$ singularity exec rtp.sif bash myscript.sh
Hello World!
```

### File Permission and Bind Path
Singularity containers run as the user and share host services. When Singularity ‘switch’ from the host operating system to the containerized operating system, the OS-level system files on the host becomes inaccessible. (the root user on the host system is also different from the root in the container!)

### Specific Usages for Ilifu

Plese see the relevant page on the HERA wiki.
