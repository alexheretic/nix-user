# nix packages sans root
Scripts and info for using nix packages without root access. Install and use nixpkgs instead of using apt or compiling manually.

## Install
Steps to setup on Linux (e.g. Ubuntu 22.04) without root.

* Install **rustup**
    ```sh
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    ```
* Install **nix-user-chroot**
    ```sh
    cargo install nix-user-chroot
    ```
* Setup ~/.nix
    ```sh
    mkdir -m 0755 ~/.nix
    nix-user-chroot ~/.nix bash -c 'curl -L https://nixos.org/nix/install | sh'
    ```
    See https://nixos.wiki/wiki/Nix_Installation_Guide
* Auto startup using nix: Add to ~/.bashrc
    ```sh
    if [ -z "$NIX_CONF_DIR" ]; then
        exec nix-user-chroot ~/.nix bash
    else
        # nix-user mode
        . "$HOME/.nix-profile/etc/profile.d/nix.sh"
        
        if [[ $(ps --no-header --pid=$PPID --format=comm) != "fish" && -z $BASH_EXECUTION_STRING ]] && type -P fish >/dev/null
        then
            exec fish
        fi
    fi
    ```

### Install scripts
This repo contains some helper scripts for using nix packages.

The main motivation is the poor performance of upstream operations (like search & update) managing _nix-env_ adhoc packages.

```sh
git clone https://github.com/alexheretic/nix-user ~/nix
```

#### Usage
* `nix/search [--refresh] PACKAGE_REGEX`
* `nix/update [--check]`

Use _nix-env_ for installing/uninstalling packages e.g. `nix-env -iA nixpkgs.foo`.

## Issues
* Using _nix-env_ to search, check for updates & update all packages is **very** slow and memory intensive. The scripts in this repo provide fast alternatives.
* It can be difficult to use programs inside the nix-root from outside. It's best to try and do everything inside, e.g. install and use nix vscode instead of the ubuntu system version.
* Graphical apps & daemons can break if the terminal they were started with is exitted.
* Dynamic link dependency packages (like openssl) don't work by default for compiling. On nix you are intended to use _nix-shell_ and explicitly declare linking dependencies. An alternative is installing such link dependencies on the system or compiling them into $HOME and setting the necessary env vars.
