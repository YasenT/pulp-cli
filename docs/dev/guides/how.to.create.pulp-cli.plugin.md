# How to Create a Pulp CLI Plugin Using Cookiecutter

This guide shows you how to create a Pulp CLI plugin using the provided cookiecutter templates. Instead of manually creating all the required files, the cookiecutter bootstrap process will automatically generate the correct structure and files for you.

---

## Overview

Pulp CLI plugin extend the functionality of the Pulp CLI by adding custom commands and features. The pulp-cli project includes cookiecutter templates that make it easy to bootstrap a new plugin with the correct structure and configuration.

---

## Prerequisites

Before you begin, ensure you have the following installed:

```bash
pip install cookiecutter click pyyaml tomlkit
```

---

## Bootstrap a New Pulp CLI Plugin

### Using the Bootstrap Template

Navigate to the directory where you want to create your new plugin and run:

```bash
# Clone the pulp-cli repository if you don't have it already
git clone https://github.com/pulp/pulp-cli.git

# Run the bootstrap script
pulp-cli/cookiecutter/apply_templates.py --bootstrap

```

### Answer the Prompts

You will be prompted for several values during the bootstrap process. The most important is **app_label**, which should match the Pulp component your plugin targets (for example, use `file` for `pulp-file`). This value determines the import paths and command names for your plugin. You will also see additional prompts for options such as glue integration, documentation, translations, version, repository URL, and CI configuration—answer these as appropriate for your project.

### Key Files and Directories

The bootstrap process creates several important directories and files:

- `pulpcore/cli/my-plugin/__init__.py`: Main entry point where you define command groups
- `pulp-glue-my-plugin/pulp_glue/my-plugin/context.py`: API context class for interacting with Pulp
- `pyproject.toml`: Project metadata, dependencies, and build configuration
- `CHANGES/`: Directory for changelog fragments (.feature, .bugfix, etc. files)
- `.github/workflows/`: CI configuration for automated testing

---

## Customizing Your Plugin

### Modify Command Group

Edit `pulpcore/cli/my_plugin/__init__.py` to define your command groups and add functionality:

```python
import typing as t

import click
from pulp_cli.generic import pulp_group
from pulp_glue.common.i18n import get_translation

# Import your command plugins
from pulpcore.cli.my_plugin.my_command import my_command_group

translation = get_translation(__package__)
_ = translation.gettext

__version__ = "0.1.0"  # Matches version in pyproject.toml


@pulp_group(name="my_plugin")
def my_plugin_group() -> None:
    """My plugin commands."""
    pass


def mount(main: click.Group, **kwargs: t.Any) -> None:
    # Add your command groups here
    my_plugin_group.add_command(my_command_group)
    main.add_command(my_plugin_group)
```

### Create Command plugins

Create a new file for your commands, e.g., `pulpcore/cli/my_plugin/my_command.py`:

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

## Development Workflow

### Install Your plugin in Development Mode

For installation instructions, see here (https://github.com/pulp/pulp-cli/blob/main/docs/user/guides/installation.md#from-a-source-checkout).

### Test Your Commands

After installation, you can test your commands:

```bash
pulp my-plugin my-command list --limit 10
```

---

## Update Templates

If you need to update the project structure with new template changes:

```bash
cd pulp-cli-my-plugin
../pulp-cli/cookiecutter/apply_templates.py
```

---

By using the cookiecutter bootstrap process, you can set up a properly structured Pulp CLI plugin and focus on implementing your custom commands rather than worrying about project structure.