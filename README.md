# hera-singularity
[![https://www.singularity-hub.org/static/img/hosted-singularity--hub-%23e32929.svg](https://www.singularity-hub.org/static/img/hosted-singularity--hub-%23e32929.svg)](https://singularity-hub.org/collections/4892)

## Notice

__April 27, 2021__:
[Singularity Hub remote built service is no longer available.](https://singularityhub.github.io/singularityhub-docs/2021/going-read-only/) We are considering other alternative. The old Singularity Hub builds can still be accessed via the badge link above, which now redirects to DataLad. The `singularity pull` can also do the pull from datalad URL. We will manually build and upload to Ilifu for the time being.

__April 28, 2021__:
The containers have been built and are available at `/ilifu/astro/projects/hera/containers`, with a few additional new containers that will be documented soon. We will keep rebuilding and replacing these files weekly, to keep the software stack up to date with the development, until we have an automated solution.

---

This repository contains recipe files for building singularity containers for the HERA software suits. The containers are remotely built on Singularity Hub when the recipes are pushed to the `main` branch. Container images can be directly download from the the badge link above or by using the `singularity pull` command line (see [below](##-Singularity-Commands)). Ilifu users, make sure to read [Specific Usages for Ilifu](###-Specific-Usages-for-Ilifu) section and check the relevant page on the HERA wiki.


## About Container and Singularity
Containers are encapsulated software environments and abstract the software and applications from the underlying operating system. This allows users to run workflows in customized environments, switch between environments, and to share these environments with colleagues and research teams.

Singularity is a free, cross-platform and open-source computer program that performs operating-system-level virtualization also known as containerization (another widely used one being Docker).

A singularity container is required for computing on the Ilifu cloud-computing cluster, which HERA has access (see the HERA wiki page on this).

Suggestion for other container recipes and implementations are welcome!


## Container Content

### Python Packages
All containers are built with `Ubuntu 20.04` and `miniconda` with `python=3.8` unless otherwise specify [below](###-Different-Between-Containers:), and come standard with the following packages:

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

<a name="myfootnote2">1</a>: with CASA measurement sets, HEALPix beam, and CST beam functionalities\
<a name="myfootnote1">2</a>: with profiling and full simulator

### Different Between Containers:

- `hera1`:
  - Intended for general-purpose computing with most of the commonly used data analysis, astronomical, and HERA software packages
- `rtp`:
  - For running the RTP pipeline and analysis with `makeflow`.
  - Equivalent to `hera1` with an addition of `hera_opm`, `hera_mc`, and  `hera_notebook_templates`.
  - `hera_pipelines` is cloned to `/usr/local`
- `casa_imaging`:
  - Equivalent to `hera1` with a full installation of `casa-6`
- `casa6_modular`:
  - Equivalent to `hera1` with a pip-wheel installation of `casa-6`, making `casatasks`, `casatools`, and `casampi` packages available (see https://casa-pip.nrao.edu/).
  - Based on `Python 3.6` and `Ubuntu 18.04` for casa-pip compatibility.


### Environment Variables
The following environment variables are also exported in all containers:

```
CONDA_INSTALL_PATH="/usr/local/miniconda3"
CONDA_INIT_SCRIPT="$CONDA_INSTALL_PATH/etc/profile.d/conda.sh"
```

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

By default, `shell` invokes `/bin/sh --norc`, which means that `.bashrc` will not be executed (more on this [here](https://github.com/hpcng/singularity/issues/643)) and thus `conda` will not be initialized. To have `conda` working, you can do one of the following:

a) Run `exec $SHELL` inside the singularity shell. If `$SHELL` is `\bin\bash` (as in our Ubuntu build), `.bashrc` will be read.
```
$ singularity shell rtp.sif
Singularity> exec $SHELL
```

b) Manually execute the conda initialization script inside singularity shell. A `CONDA_INIT_SCRIPT` environment variable pointing to the absolute path of the script (`/usr/local/miniconda3/etc/profile.d/conda.sh`), is made available for this purpose. Note that `.` must be used as `source` won't work under `sh`.
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

By default, the user home directory on the host system will be mapped to the user home directory in the container, preserving all file permission. On Ilifu, the shared data paths on the host are also mapped.

### Specific Usages for Ilifu

#### Container File Locations
Recent builds are available at `/ilifu/astro/projects/hera/containers`

#### Using a HERA container as a Jupyter kernel

See [this page](https://docs.ilifu.ac.za/#/tech_docs/software_environments?id=using-a-custom-container-as-a-jupyter-kernel) on Ilifu documentation. We may try to semi-automate this process for users with a shell script in the future.
