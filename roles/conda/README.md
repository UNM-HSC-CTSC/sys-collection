# ctsc.sys.conda

Installs a system-wide [Miniconda](https://docs.anaconda.com/miniconda/) for
a multi-user environment on Ubuntu: downloads and runs the installer into
`conda_install_dir`, locks down permissions afterwards so only root can
modify the base install (`go-w`, recursive), and adds
`{{ conda_install_dir }}/bin` to `PATH` for interactive human logins only
(UID >= 1000) via `/etc/profile.d/conda.sh`. System/service accounts are
left alone, since conda's base env shadowing the OS-packaged `python3` can
silently break scripts those accounts run.

Users still need to run `conda init` themselves to set up shell integration
(the `.bashrc` hooks conda normally adds on a per-user install).

## Variables

| Variable               | Default                                                                 | Description                                                                             |
| ---------------------- | ----------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `conda_install_dir`    | `/opt/conda`                                                            | Directory Miniconda is installed into.                                                  |
| `conda_miniconda_url`  | `https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh` | Miniconda installer download URL. Override for other architectures or to pin a version. |
| `conda_installer_dest` | `/tmp/miniconda.sh`                                                     | Local path the installer is downloaded to before running (removed afterwards).          |

## Example

```yaml
- hosts: all
  become: true
  roles:
    - role: ctsc.sys.conda
```
