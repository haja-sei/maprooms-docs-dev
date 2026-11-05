
# 3. Requirements

## 3.1. Operating system


Developers can maintain and update Maprooms on Linux, macOS, or Windows through the Windows Subsystem for Linux (WSL), provided the environment supports the required Python packages and development tools.

Use an up-to-date operating system or Linux distribution. A terminal supporting Git and SSH is required for retrieving code, accessing development resources, and working with remote servers.

The terminal environment must be able to:

- Retrieve code from GitHub.
- Access the separately provided installation and configuration files.
- Use SSH/SCP to access the designated data server when datasets are distributed that way.
- Read all configured dataset locations.
- Install and run the environment described by the supplied installer.

The supplied installation script uses Ubuntu-oriented commands, including `apt`, `ufw`, and GNU-style `sed`. Other operating systems require an appropriately adapted installation procedure.

**For WSL on Windows:**

Use an up-to-date WSL installation and a Linux distribution. Run the following commands in **Windows PowerShell**, following [Microsoft's WSL installation instructions](https://learn.microsoft.com/en-us/windows/wsl/install) and [command-reference instructions](https://learn.microsoft.com/en-us/windows/wsl/basic-commands).

```powershell
# New installation: open PowerShell as administrator.
wsl --install

# Existing installation.
wsl --update
wsl --list --verbose
```

After installation, run Git, Python, pip, SSH, the supplied Bash scripts, and `python run.py` inside the Linux distribution.

Keep the project and virtual environment in that Linux environment. Update configuration paths to Linux paths; a Windows path such as `C:\data` cannot be used directly as a Linux path.

For better filesystem performance, keep the project and, where practical, its datasets under the WSL Linux filesystem, for example:

```text
/home/<username>/ENACTS/
```

Datasets can also be stored elsewhere, provided their paths are accessible from WSL and correctly configured.

After installing the Linux distribution, additional development tools may be needed to build Python packages. For Ubuntu, the basic compiler toolchain can be installed with:

```bash
sudo apt install build-essential
```

This command supplies general build tools. Some dependencies may require additional system libraries according to the maintained installation procedure.


##   3.2. Development environment
###    3.2.1. Python environment and compatible Python version

Maproom development requires a stable Python release compatible with the application's dependencies.

The development setup described here uses Python 3.14, which is also selected by the supplied installation script. The corresponding interpreter must already be available before the script creates its virtual environment.

Check it with:

```bash
python3.14 --version
```

The package metadata in `setup.py` declares Python `>=3.11`. This declaration does not by itself confirm that every dependency combination works with every newer Python release.

The Python dependency versions in `requirements.txt` are not pinned. Using recent stable releases is the intended approach, but compatibility must be checked for the selected Python version.

Use the project virtual environment to keep Maproom dependencies separate from other Python projects. The installation script creates this environment as `venv/`.


###   3.2.2. Required libraries

The supplied installation script installs Python dependencies from `requirements.txt`.

These dependencies cover:

- Flask and its extensions.
- Numerical and statistical processing with NumPy, SciPy, and related packages.
- Multidimensional data processing with xarray, Dask, Zarr, and netCDF4.
- Geospatial processing.
- Image and chart generation.
- Data exchange and supporting utilities.
- Application-serving dependencies.

Frontend packages and their versions are defined in:

```text
package.json
package-lock.json
```

Node.js and npm are required when preparing these browser dependencies.

The vendor build uses:

```text
scripts/copy-vendor-assets.js
```

It copies the required browser assets into:

```text
app/static/libs/
```

These assets include libraries used for maps, charts, controls, and page layout.

The supplied installation Bash script installs the Python dependencies but does not run the npm preparation commands. Section 4.1.4 describes that step.


###   3.2.3. Git installation and GitHub account

Git is required to retrieve the application, inspect changes, and manage development history. A GitHub account is required when accessing restricted repositories or contributing changes.

Developers can work directly as collaborators on `sei-africa/maprooms` or through their own fork, where repository permissions allow it.

| Method | Use it when | Repository to clone |
|---|---|---|
| Collaborator | The maintainers have granted your GitHub account the required access | sei-africa/maprooms |
| Fork | You develop in your own GitHub copy and propose changes upstream | Your fork of sei-africa/maprooms |

Both methods still require the separately supplied installation/configuration files and datasets.

Access to the parent repository does not automatically establish access to the dst_api submodule. Verify both.


###   3.2.4. SSH authentication and repository access

Create the SSH key pair on your own development computer, or use an existing suitable key. Add the public key to your GitHub account and keep the private key on your computer.

Follow [GitHub's SSH-key instructions](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) for your operating system.

A typical Linux or WSL sequence is:

```bash
# Use an existing suitable key, or create a new one.
# Do not overwrite an existing key that you still need.
ssh-keygen -t ed25519 -C "YOUR_GITHUB_EMAIL"

eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Copy this PUBLIC key into GitHub:
# Settings → SSH and GPG keys.
cat ~/.ssh/id_ed25519.pub

# Test authentication.
ssh -T git@github.com
```

If you choose another key filename, use that filename in ssh-add and when displaying the public key.

macOS agent and keychain configuration can differ; follow [GitHub's platform-specific instructions](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=mac).

The successful GitHub SSH test says that authentication succeeded and that GitHub does not provide shell access. That command can return exit code 1 even after successful authentication. See [GitHub's SSH connection test](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/testing-your-ssh-connection).

The dst_api submodule is declared in .gitmodules:

```text
Path: app/dst_api
URL:  git@github.com:sei-africa/dst_api.git
```

Ensure your account can access this repository as well as the parent Maprooms repository.


###   3.2.5. Clone the repository

This section explains the available clone methods. The supplied installation script already handles cloning, so these commands are provided as reference for understanding that process or following a manual installation procedure.

Use one installation path. Avoid running a fresh-clone installer over an existing checkout without first following the maintainer's procedure.

#### 3.2.5.1. Collaborator clone

For developers with access to the main repository:

```bash
git clone git@github.com:sei-africa/maprooms.git
cd maprooms
```

#### 3.2.5.2. Fork clone

To work through a fork:

1. Open `sei-africa/maprooms` on GitHub.
2. Create a fork under your account.
3. Clone the fork to your laptop.
4. Add the original repository as `upstream` if it is not already configured.

```bash
git clone git@github.com:YOUR_GITHUB_USERNAME/maprooms.git
cd maprooms
```

HTTPS is another option for the parent repository:

```bash
git clone https://github.com/YOUR_GITHUB_USERNAME/maprooms.git
```

See [GitHub's fork workflow](https://docs.github.com/en/pull-requests/how-tos/work-with-forks/fork-a-repo). 

**Choose the appropriate SSH or HTTPS method rather than running both clone commands.**

#### 3.2.5.3. Initialize the dst_api submodule

For a manual installation, initialize the submodule from the top-level maprooms/ directory:

```bash
git submodule update --init --recursive
```

The supplied installer already performs submodule initialization.

Cloning the parent repository through HTTPS does not change the SSH URL declared for dst_api in .gitmodules. SSH authentication may therefore still be required for the submodule.


##   3.3. Storage

The application source requires less storage than the datasets. However, the development environment also needs space for Python dependencies, frontend packages, generated browser assets, cache, and user files.

The required datasets must be copied to the developer's computer or made available through accessible configured locations.

The following example retains the dataset inventory used for this development setup:

       DATA/                    76.50 GB reported total
       ├── ENACTS/              41.88 GB
       ├── CMIP6/               30.42 GB
       ├── SHP/                134.89 MB
       ├── SOILGRIDS/          100.31 MB
       └── FORECASTS/           12.09 MB

These figures describe an example dataset collection, not a fixed requirement for every country.

The reported total differs from the sum of the listed folder sizes. Confirm the inventory before using it as a storage-capacity estimate.

The ENACTS directory includes NetCDF source files and prepared Zarr data for the required timescales, including daily, dekadal, monthly, and climatological products.

Allow additional space for conversion outputs and future updates. Keeping both source NetCDF files and prepared Zarr stores requires storage for both representations.

The application also maintains runtime files under **MAPROOMS_DATA_DIR**:

       MAPROOMS_DATA_DIR/
       ├── app_cache/
       ├── users_data/
       ├── maprooms-sqlite.db
       └── enso-sqlite.db

- app_cache/ stores cached results used by the application.
- users_data/ stores supported user-related files.
- maprooms-sqlite.db stores application user information.
- enso-sqlite.db stores ENSO-related information.

The SQLite databases are generally much smaller than the climate datasets, while cache and user-file storage can grow with application use.

The account running Maprooms needs write access to the runtime directory. Data preparation also requires write access to the configured Zarr output locations.
