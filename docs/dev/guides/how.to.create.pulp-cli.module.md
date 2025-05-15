# How to Create a Pulp CLI Module Using Cookiecutter

This guide shows you how to quickly create a Pulp CLI module using the provided cookiecutter templates. Instead of manually creating all the required files, the cookiecutter bootstrap process will automatically generate the correct structure and files for you.

---

## 1. Overview

Pulp CLI modules extend the functionality of the Pulp CLI by adding custom commands and features. The pulp-cli project includes cookiecutter templates that make it easy to bootstrap a new module with the correct structure and configuration.

---

## 2. Prerequisites

Before you begin, ensure you have the following installed:

```bash
pip install cookiecutter click pyyaml tomlkit
```

---

## 3. Bootstrap a New Pulp CLI Module

### 3.1 Using the Bootstrap Template

Navigate to the directory where you want to create your new module and run:

```bash
# Clone the pulp-cli repository if you don't have it already
git clone https://github.com/pulp/pulp-cli.git

# Navigate to the cookiecutter directory
cd pulp-cli/cookiecutter

# Run the bootstrap script
python apply_templates.py --bootstrap
```

### 3.2 Answer the Prompts

The cookiecutter will prompt you for several values:

```
  [1/7] app_label (noname):
  [2/7] glue [y/n] (y): 
  [3/7] docs [y/n] (n): 
  [4/7] translations [y/n] (n): 
  [5/7] version (0.0.1.dev0): 
  [6/7] repository (https://github.com/pulp/<app_label>): 
  [7/7] test_matrix (default): 
```

- **app_label**: The name of your module (used for imports and commands)
- **glue**: Include pulp-glue integration (recommended)
- **docs**: Include documentation setup
- **translations**: Include translation support
- **version**: Starting version number
- **repository**: Git repository URL for your project
- **test_matrix**: Test matrix configuration for CI

### 3.3 Apply Additional Templates

After bootstrapping, the script automatically applies CI templates and (if selected) documentation templates:

```
Bootstrap new CLI plugin.
New plugin repository created in pulp-cli-my-module.
Apply ci template
Apply docs template
```

---

## 4. Resulting Directory Structure

After running the bootstrap, you'll have a fully structured project. Here's the actual structure generated for an example module named `my-module`:

```plaintext
pulp-cli-my-module/
├── .ci/                          # CI configuration files
├── .github/                      # GitHub workflows
│   └── workflows/
│       └── test.yml
├── CHANGES/                      # Directory for changelog entries
├── CHANGES.md                    # Changelog summary
├── Makefile                      # Build automation
├── lint_requirements.txt         # Dependencies for linting
├── lower_bounds_constraints.lock # Minimum dependency versions
├── pulp-glue-my-module/           # Glue package (if selected)
│   ├── pulp_glue/
│   │   └── my-module/
│   │       ├── __init__.py
│   │       ├── context.py        # API context class
│   │       └── py.typed          # Type checking marker
│   └── pyproject.toml            # Glue package configuration
├── pulpcore/                     # CLI implementation
│   └── cli/
│       └── my-module/
│           ├── __init__.py       # Main entry point
│           └── py.typed          # Type checking marker
├── pyproject.toml                # Project configuration
└── test_requirements.txt         # Testing dependencies
```

### 4.1 Key Files and Directories

The bootstrap process creates several important directories and files:

- `pulpcore/cli/my-module/__init__.py`: Main entry point where you define command groups
- `pulp-glue-my-module/pulp_glue/my-module/context.py`: API context class for interacting with Pulp
- `pyproject.toml`: Project metadata, dependencies, and build configuration
- `CHANGES/`: Directory for changelog fragments (.feature, .bugfix, etc. files)
- `.github/workflows/`: CI configuration for automated testing

---

## 5. Customizing Your Module

### 5.1 Modify Command Group

Edit `pulpcore/cli/my_module/__init__.py` to define your command groups and add functionality:

```python
import typing as t

import click
from pulp_cli.generic import pulp_group
from pulp_glue.common.i18n import get_translation

# Import your command modules
from pulpcore.cli.my_module.my_command import my_command_group

translation = get_translation(__package__)
_ = translation.gettext

__version__ = "0.1.0"  # Matches version in pyproject.toml


@pulp_group(name="my_module")
def my_module_group() -> None:
    """My module commands."""
    pass


def mount(main: click.Group, **kwargs: t.Any) -> None:
    # Add your command groups here
    my_module_group.add_command(my_command_group)
    main.add_command(my_module_group)
```

### 5.2 Create Command Modules

Create a new file for your commands, e.g., `pulpcore/cli/my_module/my_command.py`:

```python
import click
from pulp_cli.common.context import pass_pulp_context
from pulp_glue.common.context import PulpContext


@click.group()
def my_command_group():
    """My custom commands."""
    pass


@my_command_group.command()
@click.option("--limit", type=int, help="Limit the number of items")
@pass_pulp_context
def list(pulp_ctx: PulpContext, limit: int):
    """List items."""
    # Implement your command logic here
    click.echo(f"Listing items with limit: {limit}")
```

---

## 6. Development Workflow

### 6.1 Install Your Module in Development Mode

```bash
cd pulp-cli-my-module
pip install -e .
```

### 6.2 Test Your Commands

After installation, you can test your commands:

```bash
pulp my-module my-command list --limit 10
```

---

## 7. Project Maintenance

### 7.1 Adding Changelog Entries

The project uses Towncrier to manage the changelog. To add a changelog entry, create a new file in the `CHANGES/` directory with an appropriate extension:

```bash
# Create a changelog entry for a new feature
echo "Added new command for syncing repositories" > CHANGES/123.feature

# Example for a bugfix
echo "Fixed authentication issue with token-based login" > CHANGES/124.bugfix
```

The filename should be unique (typically use a ticket/issue number) with one of these extensions:
- `.feature` - For new features
- `.bugfix` - For bug fixes
- `.doc` - For documentation improvements
- `.removal` - For deprecations or removals
- `.misc` - For other changes

When the project is released, these fragments will be automatically compiled into a formatted changelog in `CHANGES.md` using the Towncrier template.

### 7.2 Update Templates

If you need to update the project structure with new template changes:

```bash
python ~/pulp-cli/cookiecutter/apply_templates.py
```

---

## 8. Best Practices

1. **Follow the Project Structure**: Maintain the structure created by the cookiecutter.
2. **Use Click Decorators**: Leverage Click's features for robust CLI commands.
3. **Keep Documentation Updated**: If you enabled docs, maintain them.
4. **Use the Glue Layer**: Keep API interaction code in the glue layer for better separation.

---

By using the cookiecutter bootstrap process, you can quickly set up a properly structured Pulp CLI module and focus on implementing your custom commands rather than worrying about project structure.