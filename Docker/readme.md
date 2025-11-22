# Building EDuke32 for FunKey-S

This guide details how to cross-compile EDuke32 for the FunKey-S handheld. It uses a Docker container to ensure a consistent build environment without polluting your host machine.

## Prerequisites

* **OS:** Windows 10/11 (PowerShell)
* **Software:**
    * [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running.
    * [Git](https://git-scm.com/) installed.

## 1. Project Setup

1.  Create a working directory on your PC: `C:\Duke`
2.  Clone this repository into that folder.
3.  Copy the file named `Dockerfile.eduke32` to `C:\Duke`

## 2. Build the Environment

Open **PowerShell**, navigate to your folder, and build the Docker image. This will download the FunKey-S SDK and install necessary build tools.

```powershell
cd C:\Duke
docker build -t eduke32-build -f Dockerfile.eduke32 .
```

## 3. Run the Container

Once the build is complete, launch the container. This mounts your local C:\Duke folder to /workspace inside the container.

```powershell
docker run -it --rm -v "C:\Duke:/workspace" eduke32-build
```

## 4. Compile EDuke32
You are now inside the Linux container (you should see a bash prompt). Run the following commands to compile the game.

## 1. Navigate to the source directory: (Adjust this path if your repository structure is different)
It will build the envirnment. Once thats completed, run:
```
cd fks-eduke32/polymer/eduke32
```

## 2. Define the SDK configuration path:
```
export CROSS_SDL_CONFIG=/opt/FunKey-sdk-2.3.0/arm-funkey-linux-musleabihf/sysroot/usr/bin/sdl-config
```

## 3. Clean previous builds:
```
make clean
```

## 4. Run the Make command:
```
make RENDERTYPE=SDL \
     SDLCONFIG=$CROSS_SDL_CONFIG \
     SDL_FRAMEWORK=0 \
     ARCH="-mcpu=cortex-a7 -mfpu=neon-vfpv4 -mfloat-abi=hard -ftree-vectorize" \
     MISCLINKOPTS="$($CROSS_SDL_CONFIG --libs)"
```

## 5. Package the OPK

Once the compilation finishes successfully, you need to package the executable into an .opk file.

Note: This assumes your repository contains an opk/ folder with the required assets (icon, .desktop file, libraries).

```
# 1. Copy the newly compiled binary into the packaging folder
cp eduke32 opk/

# 2. Create the SquashFS file (The OPK)
mksquashfs opk eduke32.opk -all-root -no-xattrs -noappend -no-exports

# 3. Copy the final OPK back to your Windows workspace
cp eduke32.opk /workspace/
```
