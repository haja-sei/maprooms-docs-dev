# 9. Maintain the Maproom

## 9.1. Git process

The Git workflow is a standard development process. This section provides a reminder of the main steps used to maintain and share Maproom changes.

During development, create a working branch with *git checkout -b `<branch-name>`* and make the changes on that branch. Use *git status* to inspect modified, staged, and untracked files, and *git diff* to review changes to tracked files.

Before integrating remote changes, save the intended local work in a commit. Use *git add `<file1> <file2>`* to select the files, followed by *git commit -m "Describe the change"*.

The command *git commit -am "Describe the change"* automatically stages modifications and deletions of tracked files, but it does not include new, untracked files. Selecting files explicitly is useful when the working directory also contains changes that should remain outside the commit. For ordinary Maproom development, leave the *app/dst_api* submodule revision unchanged unless updating it is part of the intended change.

Retrieve the latest remote changes with *git fetch origin*. If the remote main branch contains updates that need to be included, merge them into the working branch with *git merge -m "Merge main updates" origin/main*. Fetching retrieves the remote history; merging integrates the selected changes into the current branch.

If conflicts occur, compare the affected sections carefully. The section beginning with *<<<<<<< HEAD* contains the current branch's version, while the section between *======= and >>>>>>>* contains the incoming version. Select or combine the required content, then remove the conflict markers. Keeping both versions is appropriate only when the combined result is valid.

After resolving the conflicts, save the files, stage them with *git add `<file>`*, and complete the merge with *git merge --continue*. [Test the local Maproom](4-installation.md#44-test-the-maproom-locally) to confirm that the affected functionality still works.

Before sharing the branch, fetch again to check whether additional changes have appeared remotely during development. If another merge is required, resolve any conflicts and repeat the relevant local checks.

Once the changes have been tested, upload the branch with *git push origin `<branch-name>`*. Then open a pull request on GitHub, selecting the intended target branch and describing the changes and their verification.

When developing through a [fork](3-requirement.md#3252-fork-clone), push the working branch to the fork and open the pull request against the original repository. If the original repository is configured as upstream, retrieve its updates with *git fetch upstream and merge from upstream/main*.

The main commands are summarized below. Replace values inside <...> with the appropriate names or paths.

| Command | Purpose |
|---|---|
| git checkout -b `<branch-name>` | Create a working branch and switch to it |
| git status | Show the current branch and modified, staged, or untracked files |
| git diff | Review unstaged changes to tracked files |
| git add `<file1> <file2>` | Stage the selected files, including new files |
| git diff --cached | Review the changes staged for the next commit |
| git commit -m "Describe the change" | Commit the staged changes |
| git commit -am "Describe the change" | Stage and commit changes to tracked files; new files are excluded |
| git fetch origin | Retrieve updates from the configured origin repository |
| git merge -m "Merge main updates" origin/main | Integrate the fetched main branch into the current branch |
| git merge --continue | Complete a merge after resolving and staging conflicts |
| git push origin `<branch-name>` | Upload the working branch to GitHub |
| git fetch upstream | Retrieve updates from the original repository when using a fork |
| git merge upstream/main | Integrate the original repository's updates into the current branch |

A push uploads the branch. The pull request is then created and reviewed on GitHub before the changes are merged into the target branch.

## 9.2. dev- files and local datasets-config.yaml

The Bash script update_maprooms.sh provides a common process for updating a local Maproom installation. It updates the application and *dst_api*, copies selected [configuration templates](5-maproom-structure.md#515-the-role-of-dev-files), and installs the Python packages required by the application.

The beginning of the script defines the installation paths:

```bash
#!/bin/bash

ROOT_DIR="/home/aranaivoson/ENACTS/MAPROOMS"
PYTHON_CMD="/home/aranaivoson/ENACTS/MAPROOMS/venv/bin/python"

MAPROOM_DIR="${ROOT_DIR}/maprooms"
APP_DIR="${MAPROOM_DIR}/app"

SEP="==============================================================================="

echo "$SEP"
echo -e "\nUpdating the maprooms .....................\n"
...
```

Set **ROOT_DIR** to the directory containing the local maprooms repository and **PYTHON_CMD** to the Python executable in its virtual environment. The script derives **MAPROOM_DIR** and **APP_DIR** from these values.

Before running the update, save pending code changes and preserve local configuration edits that must be retained. Run the script with Bash using bash *update_maprooms.sh*.

The script performs the following operations:

1. It enters the Maproom repository and runs git pull origin main to retrieve and integrate updates. This command operates on the current checkout; the script does not switch to the main branch first.
2. It enters app/dst_api and runs git pull origin main there as well. This updates the submodule independently and can change the submodule revision reported by the parent repository.
3. It searches beneath app/ for directories named yaml. In each directory, it copies matching dev- files to active filenames with the dev- prefix removed.
4. It skips the five configuration templates listed below. It also preserves an existing maproom_items.yaml; the corresponding template is copied only when that active file is absent.
5. It returns to the Maproom repository and runs 

```bash
"$PYTHON_CMD" -m pip install -r requirements.txt
```
to install packages according to the application's requirements.

The following templates are always excluded from copying:

| Excluded template | Active local file preserved |
|---|---|
| dev-nmhs-info.yaml | nmhs-info.yaml |
| dev-datasets-config.yaml | datasets-config.yaml |
| dev-subdivision-config.yaml | subdivision-config.yaml |
| dev-disp-teleconnections.yaml | disp-teleconnections.yaml |
| dev-season-definition.yaml | season-definition.yaml |

These [active files](4-installation.md#413-prepare-local-configuration-files-for-development) must be maintained locally. Compare them with updated templates when new configuration keys are introduced, while preserving the values required by the local implementation.

Other matching active files are overwritten by the template-copying step. Local changes in those files therefore need to be preserved or incorporated into the appropriate shared templates before updating.

Review the command output before considering the update complete, because the script does not automatically stop after every failed command. The script also does not restart the application. After a successful update, restart it using the local operating procedure and verify the affected Maproom pages.

