# docker-kdesrc-build

This project aims to provide a lightweight and ready to use KDE development
environment by wrapping the `kdesrc-build` tool in Docker.

This way, you can compile and work on the latest version of the KDE project 
and keep your main system clean of unwanted development packages.

You can run a KDE application on your current desktop by sharing your X
server instance inside the Docker container

Moreover, you can run and try the entire Plasma Desktop in another tty !

# Requirements

- `python` `>=` `3.2`
- `docopt`
- `docker`

# Usage

## Quick and simple

Just choose a distro as a base system for the container (*`archlinux` 
is recommended, as it has the latest dependencies*), and run the script :

    ./run.py --base archlinux

The script will do the following operations :

* check for a `Dockerfile-archlinux` file
* create a dir under `$HOME/kdebuild/archlinux` to be mounted under `/work`
* build or update the Docker image `archlinux-kdedev` if necessary
* mount `bashrc` as a volume (`/home/kdedev/.bashrc`)
* mount `kdesrc-buildrc` as a volume (`/home/kdedev/.kdesrc-buildrc`)
* run the container
* run `/bin/bash`

## Run the Plasma Desktop

### 1 - Using a shared X11 socket

To run an entire Plasma Desktop session, we need to create a new X server instance
, running on a new `tty`, for example, `tty8`.

    # startx -display :1 -- :1 vt8

Also you have to **explicitly authorize access** to allow clients inside the
container to use this xserver instance :

    # DISPLAY=':1' xhost +

Then, run the container and use the `--display` option to specify the right
tty to be used by applications :

    ./run.py --base archlinux --display ':1'

Execute `startkde` from your install directory (which is `/work` by default)

Note that you should have build the `workspace` set first (`kdesrc-build workspace`)

Inside the container :

    $ /work/install/bin/startkde

And the KDE desktop should be starting on `tty8` !

### 2 - Using a VNC Server

Another method consist to run a `Virtual FrameBuffer` inside the container,
as well as a `VNC` server to view it's content.

Run the environement :

    ./run.py --base archlinux 

Run the `Xvfb` server :

    sudo Xvfb $DISPLAY +extension GLX +render -screen 0 1024x780x24 &

Run the `VNC server :`

    sudo x11vnc -usepw -display $DISPLAY

Now you should inspect the container on the host
    
    docker inspect <container name | ID>

and check for it's IP address:

    "IPAddress": "172.17.0.10"

You can connect to the VNC server `container-ip-address`:`5900` with a VNC client
on the host !

Now you can run any applications, including `startkde` !

## Command line options reference

    Usage: build.py [options]

    Options:
        -b --base DISTRO        Use DISTRO as base system [Default: all]
        --no-cache              Do not use cache when building the image [Default: False]
        --rm                    Automatically remove the container when it exits [Default: True]
        --display DISPLAY       Change the DISPLAY environment variable passed to the container [Default: :0]
        --vnc                   Enable port forwarding and disable X11 socket sharing
        --qt PATH               Set the PATH to your a specified Qt installation (mounted as /qt) [Default: False]
        -h --help               Display this message


## kdesrc-buildrc configuration

To configure `kdesrc-buildrc`, take a look at [http://kdesrc-build.kde.org/documentation/](http://kdesrc-build.kde.org/documentation/)

## Use a specific version of Qt

If you want to change the Qt version used during the compilation, you can
provide a the path to a Qt installation on the host with :

    ./run.py --base archlinux --qt /path/to/qt

This path will be mounted under `/qt`

Don't forget to change the `qtdir` variable in the `kdesrc-buildrc`

# Dependencies installed

|              | Archlinux | Fedora | OpenSUSE | Ubuntu |
|--------------|-----------|--------|----------|--------|
| Frameworks   |     ✓     |    ✗   |     ✗    |    ✗   |
| Workspace    |     ✓     |    ✗   |     ✗    |    ✗   |
| Applications |     ✗     |    ✗   |     ✗    |    ✗   |
