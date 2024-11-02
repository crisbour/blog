---
title: "Unreal Engine on Nix (Linux)"
author: "Cristian Bourceanu"
date: 2024-11-02T12:39:00Z
categories: [nix, container, setup]
---

# Unreal Engine

The Unreal Engine will be useful for us to generate synthetic data for our
research. They provide a prebuild binary for Ubuntu and Centos, however, since I
am running NixOS (non FHS system), the prebuilt binary can't link to the
requested libraries.

**TL;DR**
I have tried a few approaches to get it working on Nix or LXD under Nix,
however, in the end I resorted to using their `docker` image.

## Running on Nix

Prebuilt binaries expecting to run on Linux normally have hard-coded paths in
the executable, such as `/lib/libstdc++.6.so` or even worse the linker itself
cannot be found. 

### FHSUserEnv
In some cases workaround to build and FHS environment fixes the
issue. In this scenario, the program is really big (larger than 30GB), so
populating a derivation with that amount of extra storage is not acceptable for
my laptop, but I have tried to do so:

```nix
{ pkgs ? import<nixpkgs>{}}:
#stdenv, buildFHSUserEnv, fetchFromGitHub, git, python3, mono, clang, cmake, libGL, xorg }:

let
  inherit (pkgs) stdenv buildFHSUserEnv fetchFromGitHub git python3 mono clang cmake libGL xorg;
  unreal-engine = stdenv.mkDerivation {
    pname = "unreal-engine";
    version = "5.4.4"; # Adjust version as needed

	src = ./.;

    nativeBuildInputs = [ git python3 pkgs.bash stdenv.cc.cc stdenv.cc.cc.lib];


	buildInputs = [ 
		libGL 
		xorg.libXi 
		xorg.libXcursor 
		xorg.libXrandr 
		xorg.libXinerama 
	];

    buildPhase = ''
      patchShebangs .
      # Run Setup.sh
      ./Setup.sh
      
      # Generate project files
      ./GenerateProjectFiles.sh
      
      # Build the engine
	  make
    '';

    installPhase = ''
      mkdir -p $out
      cp -r . $out
    '';

    meta = with pkgs.lib; {
      description = "Unreal Engine source code";
      homepage = "https://www.unrealengine.com/";
      license = licenses.unfree;
      platforms = platforms.linux;
    };
  };

in
buildFHSUserEnv {
  name = "unreal-engine-env";
  targetPkgs = pkgs: with pkgs; [
    unreal-engine
    # Add any additional runtime dependencies here
  ];
  runScript = "bash";
}
```
`patchShebangs` overwrites the `#!/usb/bin/<executable>` paths with the nixpkgs
paths in order to fix the build issue. However, this still seems to error with a binary trying to call an utility from
a hard-coded path.

### `nix-ld`

Also tried to setup an environment using [nix-ld](https://github.com/nix-community/nix-ld) to run with libraries paths being patched automagically.
==IDK how nix-ld actually works as I have checked out their source code==

```nix
with import <nixpkgs> {};
mkShell {
  NIX_LD_LIBRARY_PATH = lib.makeLibraryPath [
    stdenv.cc.cc
    stdenv.cc.cc.lib
    SDL2
    openssl
    xorg.libX11
    xorg.libXext
    xorg.libXcursor
    xorg.libXrandr
    xorg.libXinerama
    libGL
    libuv
    # ...
  ];
  NIX_LD = lib.fileContents "${stdenv.cc}/nix-support/dynamic-linker";
  shellHook = ''
    #LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$NIX_LD_LIBRARY_PATH:${pkgs.stdenv.cc.cc.lib}/lib
    LD_LIBRARY_PATH=${pkgs.stdenv.cc.cc.lib}/lib
  '';
}
```

### LXD

Next obvious answer to me was to work in a Distro that UnrealEngine supports by
default. Getting LXD to work in NixOS had it's own complication with forwarding
network.

In order to get LXD working properly on NixOS, the following is added to the
`configuration.nix`:

```nix
  # Adding lxd and overriding the package
  virtualisation = {
    lxd = {
      enable=true;

      # using the package of our overlay
      # package = pkgs.lxd-vmx.lxd.override {useQemu = true;};
      recommendedSysctlSettings=true;
    };

    lxc = {
      enable = true;
      lxcfs.enable = true;

      # This enables lxcfs, which is a FUSE fs that sets up some things so that
      # things like /proc and cgroups work better in lxd containers.
      # See https://linuxcontainers.org/lxcfs/introduction/ for more info.
      #
      # Also note that the lxcfs NixOS option says that in order to make use of
      # lxcfs in the container, you need to include the following NixOS setting
      # in the NixOS container guest configuration:
      #
      defaultConfig = "lxc.include = ''${pkgs.lxcfs}/share/lxc/config/common.conf.d/00-lxcfs.conf";

      # TODO Add systemConfig to store lxcpath and btrfs.root paths
      #systemConfig =
      #  ''
      #    lxc.lxcpath = /var/lib/lxd/storage-pools # The location in which all containers are stored.
      #  '';
    };

    libvirtd = {
      enable = true;
      qemu.runAsRoot = true;
    };
  };

  # kernel module for forwarding to work
  boot.kernelModules = [ "nf_nat_ftp" ];

  # Set up networking bridge for LXD
  networking = {
    bridges = {
      lxdbr0 = {
        interfaces = [];
      };
    };
    nat = {
      enable = true;
      internalInterfaces = ["lxdbr0"];
      externalInterface = "eth0"; # Replace with your actual external interface
    };
  };
  networking.firewall.trustedInterfaces = [ "lxdbr0" ];
```

While the github repository compiled, it didn't finish linking due to an error I
couldn't debug.
==I haven't tried the prebuilt binary for this one==

### Docker

Luckly it seems that EpicGames provide a docker image for their releases of
Unreal as well.

In order to access them you need to generate a personal autorisation token:
`Profile > Settings > Developer settings > Personal access token > Tokens
(classic)`. Enable `read: registry` for this token. Then login into the `ghcr`
with docker:

```sh
export GH_PAT=<copy_token_here>
echo $GH_PAT | docker login ghcr.io -u <username> --password-stdin
```

Then pull the image of interest, i.e.
```sh
docker pull ghcr.io/epicgames/unreal-engine:dev-5.4.4
```

Now in order to run your container with Xserver support and GPU passthrough,
run:

```sh
docker run --rm -ti --device nvidia.com/gpu=all -v/tmp/.X11-unix:/tmp/.X11-unix:rw -e DISPLAY --network host ghcr.io/epicgames/unreal-engine:dev-5.4.3 bash
```

#### GPU, Xserver and network to Docker

There are a few details to iron out in order to make the above command work. For
NixOS the followings:

1. Ensure docker is enable and has support for nvidia:
```nix
  virtualisation.docker = {
    enable = true;
  };
  hardware.nvidia-container-toolkit.enable = true;

  users.users.cristi.extraGroups = ["docker"];

  environment.systemPackages = with pkgs; [
    docker
  ];

```
2. Ensure xserver is enabled and can allow docker
```nix
services.xserver = {
  enable = true;
  displayManager.lightdm.enable = true;
};
environment.systemPackages = with pkgs; [ 
  xorg.xhost # Necessary for allowing docker X11 access 
];
```
3. xhost add `docker` to local drivers with `xhost +local:docker` or configure
from NixOS (warn: I haven't tried this yet):
```nix
systemd.services.xhost-docker = {
  description = "Allow Docker containers to access X11";
  after = [ "display-manager.service" ];
  script = ''
    ${pkgs.xorg.xhost}/bin/xhost +local:docker
  '';
  serviceConfig = {
    Type = "oneshot";
    User = "root";
  };
  wantedBy = [ "multi-user.target" ];
};
```




