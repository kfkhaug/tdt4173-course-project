# How to Work with the Code

This document includes instructions for development setup as well as a quick introduction to tools used to aid development.

## Development Setup

1. Begin by installing uv following [this](https://docs.astral.sh/uv/getting-started/installation/#standalone-installer) guide. 

    - On Windows 11, you need to run this command: 
        ```
        powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
        ```
    - On MacOS, you need to run this command:
        ```
        curl -LsSf https://astral.sh/uv/install.sh | less"
        ```
    - On Linux, you need to run this command:   
2. After installing uv, open the project folder in VS Code and press ```Ctrl + J``` to start a terminal thats already navigated to the working folder. This command is very nice to use by default every time you need a terminal.

3. Enter ```uv venv``` in the terminal. This will create a .venv folder that only exists for you on your PC. It contains the virtual environment for this specific python project on your PC.

4. Enter ```uv sync --all-extras``` in the terminal. This will automatically download the specific python version and packages..

5. Set the python interpreter used by VS Code to be the same as the one used by the virtual environment that should have been created. The path should include something like ```tdt4173-course-project``` and have ```.venv``` in the name. This ensures that you use the python setup used only by this project.

6. Place the raw data files in `data/1_raw/`. I order to avoid having a lot of folders to navigate through, all files from the `kernel`- and and `extended`-folders are put in the same folder as `prediction-mapping` and `sample_submission`. They are also prefixed with respectively `kernel_` and `extended_` so as to remember where the files came from. The `data/1_raw/` folder should then have the following structure:

    ```
    tdt4173-course-project/
    ├── data/
    │   ├── 1_raw/
    │   │   ├── extended_materials.csv
    │   │   ├── extended_transportation.csv
    │   │   ├── kernel_purchase_orders.csv
    │   │   ├── kernel_receivals.csv
    │   │   ├── prediction_mapping.csv
    │   │   └── sample_submission.csv
    │   ├── 2_interim/
    │   ├── 3_procecced/
    │   └── 4_predicted/
    ```

7. Read the next section about project structure to begin understanding where to code!

## Folder Structure

To get started, the main folders you'll need to focus on are `docs`, `data`, and `scripts`. The `docs` folder contains all official project materials and other important documents, so it's a good first stop for understanding the project requirements. All coding and analysis should be done within the `scripts` folder. The `data` folder is where our datasets are stored. Since the data files are too large for GitHub, you must remember to place them in this directory manually.

Looking at the other folders, the `models` directory is where our trained model files will be saved. These are the final outputs used to generate predictions. Lastly, the `delivery` folder holds the placeholder files for our final submission. These are currently mostly empty and will be filled out once we've completed the work in scripts.

## Development Tools You Should Know

### Uv
[Uv](https://docs.astral.sh/uv/) is a tool for managing the python version and the packages you use. It completely relpaces pip. **Never use pip**, that will break the project.

The reason it is genious is that normally, each person working on a project may have different versions of python and therefore experience different bugs or functionality. Having the same python version removes this problem.

In the same vein, having differring package verisons also introduces bugs for large projects. Uv handles all of this.


#### Useful Uv Commands

- **```uv sync --all-extras```** reads the uv config files in the repo and automatically installs what you need to be up to date. Use this if you know that other project members have added or removed packages that the code now relies on. The `--all-extras` flag ensures packages in the `dev` group are installed as well. Dev contains packages relevant only for development, but not for running the code.

- **```uv add [package]```** installs a package. For example ```uv add scikit-learn pandas``` has already been been used to install the two packages `pandas` and `scikit-learn`. This command replaces the old ```pip install [package]``` way of doing things. From now on, never use `pip install`. Instead rely on `uv add`. If you want to add a package to the `dev` group, use ```uv add --dev [package]```.


- **```uv remove [package]```** is used to uninstall packages. If you want to remove a package from the `dev` group, use `uv remove --dev [package]` instead.

- **`uv self update`** updates uv itself. It is not neccessary in any way, but now you know how to update uv.

### Ruff
[Ruff](https://docs.astral.sh/ruff/#testimonials) re-formats code to use the same style and automatically fixes common errors. This ensures both that easily overseen errors are corrected immediately and that everyone can read your the entire codebase. Ruff should be run each time you are done editing a file.

I have set it to automatically run for all files whenever you use git commit. The result should be shown in the terminal. It should also be installed automatically when you run ```uv sync```, but i advise you to also install the VS Code extension for it that can be found [here](https://marketplace.visualstudio.com/items?itemName=charliermarsh.ruff).

If you want to run it manually, use any of these:
```
# Lint files in `path/to/code`. #
ruff check [path/to/code/]
# Eg. ruff check main.py

# Lint files in the current directory. #
ruff check

# Lint files in the current directory and fix any fixable errors. #
ruff check --fix

# Lint files in the current directory and re-lint on change. #
ruff check --watch
```

I don't have any experience using this, so it may bring problems in the beginning. Still, try to research how to fix the problem before giving up on the tool, it is very powerful.

### Ty

[Ty](https://docs.astral.sh/ty/) is a bleeding edge type checker. Meaning that is so new that it is still in alpha, and that it supposed to catch runtime errors before running the code. Ty should be used each time you are done editing a file.

Remember using Java how it would complain if it saw that inputs to the code weren't compatible further along the code? It basicly implements this in python.

Begin by installing [this](https://marketplace.visualstudio.com/items?itemName=astral-sh.ty) extension in VS Code. It is also installed automatically when you run ```uv sync```, but my impression is that the extension works best.

Since Ty is so new, Chat can't help explain how to use it. Instead, resort to [this](https://docs.astral.sh/ty/reference/cli/#ty-help) list of commands. Most commonly, you will want to run ```ty ckeck [path/to/code/]``` in order check the specific file you are working on.