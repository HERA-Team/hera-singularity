# hera-rtp-singularity
This repository contains recipes of singularity containers for HERA software. The recipes are remotely built on Singularity Hub. Currently, there are two recipes in the repo:

- rtp: a container for the HERA RTP software pipeline
- debug: a container for experimental build that contain RTP among other things

We are currently experimenting with running the HERA RTP pipeline on the ilifu cluster with these two containers.

Suggestion for other containers for the broader collaboration and contribution are welcome!

## About Container and Singularity
Containers are encapsulated software environments and abstract the software and applications from the underlying operating system. This allows users to run workflows in customized environments, switch between environments, and to share these environments with colleagues and research teams.

Singularity is a free, cross-platform and open-source computer program that performs operating-system-level virtualization also known as containerization (another widely used one being Docker).

## Singularity Commands

### Pull
Use `singularity pull` to download the container from Singularity Hub
```
$ singularity pull [name_to_save_the_imagee_(optional)] shub://HERA-Team/hera-rtp-singularity:<recipe>
```
For example,
```
$ singularity pull rtp.sif shub://HERA-Team/hera-rtp-singularity:rtp
INFO:    Downloading shub image
 1.98 GiB / 1.98 GiB [=======================================================] 100.00% 13.12 MiB/s 2m34s
 ```

### Shell
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

### Executing Commands
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

## File Persmission and Bind Path
Singularity containers run as the user and share host services and features. When Singularity ‘swaps’ the host operating system for the one inside your container, the host file systems becomes inaccessible. (And root on the host system is not the same as root in the container!)

By default, only the user home directory on the system will be mapped to the user home directory in the container, including all file permission. The `shell`
