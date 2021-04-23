# synapse-nspawn

Run [Synapse](https://github.com/matrix-org/synapse) on localhost in a Debian container.

## Prerequisites (minimum)

- systemd 233 and [systemd-nspawn](https://wiki.archlinux.org/index.php/Systemd-nspawn)
- [mkosi](https://github.com/systemd/mkosi) version 10 (requires Python 3.7)

#### To install prerequisites on Debian-based distributions, run:
```
sudo apt install systemd-container debootstrap debian-archive-keyring
python3 -m pip install --user git+https://github.com/systemd/mkosi.git
```
#### To install prerequisites on other systemd-based platforms:

- If your distribution lacks `systemd-nspawn` and `machinectl`, install the package `systemd-container`
- Install the packages `debootstrap` and `debian-archive-keyring`, then [install mkosi from source](https://github.com/systemd/mkosi#installation)

## Quickstart

#### To spawn a minimal Synapse homeserver on `localhost:8008`, run:
```
git clone https://github.com/matrix-org/synapse.git
git clone [THIS REPO]
cd synapse-nspawn
sudo -E python3 -m mkosi --build-sources ../synapse --minimize
sudo systemd-nspawn --machine synapse --settings trusted
```
## Usage

mkosi uses a [two-phase process](https://github.com/systemd/mkosi/blob/main/mkosi.md#build-phases) to allow fast incremental builds of code in the container. Run mkosi with the `--incremental` flag and with `--build-sources` set to the root of your existing Synapse code tree. On subsequent builds, use the `--force` flag to install your modified code:
```
sudo -E python3 -m mkosi --build-sources ../synapse --incremental --force
```
The image `synapse.raw` is created in `/var/lib/machines/`, and can be easily booted with systemd-nspawn:
```
sudo systemd-nspawn --machine synapse --settings trusted
```

## Notes

- Ignore the btrfs error at the end of the mkosi build process
- mkosi does not cache the Python packages installed in `mkosi.prepare`, so these packages are downloaded twice during the initial build process
- The included Synapse service, sysuser, and tmpfile configs are based on those of the [Arch package](https://github.com/archlinux/svntogit-community/tree/9fdd2fa67a9d4cf89873ecb01a9ce78927f24952/trunk)
- The included systemd log config is from the official [Synapse repo](https://github.com/matrix-org/synapse/tree/47db2c3673290ca1e0dff3bd4fb9f461c97c67c3/contrib/systemd)