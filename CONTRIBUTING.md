# Contributing to lsst/templates

This page describes how to maintain and contribute to this templates repository.
You'll learn about the structure of templates and how to document them.

Broadly, use the regular [Data Management Development Workflow](https://developer.lsst.io/processes/workflow.html) when contributing to this repository.
Because of the broad impact of these templates, a [Request for Comments (RFC)](https://developer.lsst.io/processes/decision_process.html) may be appropriate.

**Contents:**

1. [Set up for development](#set-up-for-development)
2. [Making a file template](#making-a-file-template)
3. [Making a project template](#making-a-project-template)
4. [Continuous integration](#continuous-integration)
5. [templatekit](#templatekit)
6. [FAQ](#faq)

## Set up for development

To effectively contribute templates, you'll need to install the [templatekit Python package](#templatekit) that includes helpers for rendering templates, as well as sets dependencies on third-party tools like Cookiecutter and Scons:

```bash
git clone https://github.com/lsst/templates
cd templates
python setup.py install
```

*It's a good idea to do this in an isolated Python virtual environment.*

To regenerate the examples, type:

```bash
scons
```

The [Making a file template](#making-a-file-template) and [Making a project template](#making-a-project-template) sections describe how to add and modify templates.

## Making a file template

Each **file template** has its own sub-directory in the `file_templates` root directory.
The standard components of a file template directory are:

- [README.md](#file-template-readme-file) file.
- [template.ext.jinja](#file-template-jinja2-template-file).
  This file is a [Jinja2 template](http://jinja.pocoo.org/docs/2.9/templates/).
  If the template is for a named file, use that name with a final `jinja` extension. For example, `COPYRIGHT.jinja`.
- [cookiecutter.json](#file-template-cookiecutterjson) file.
- [Sconscript](#file-template-sconscript) file.
- [example.ext](#template-example-file) file.

When you add a new file template, add a link to it from the root `README.md` file.

The next sections describe these files in more detail.

### File template README file

The title of the README is the same as the file template's directory name.

Provide a summary sentence or paragraph below the title.

Create a section called `Template variables`.
Name each subsection in `Template variables` after a Jinja variable.
Describe the use and format of each variable in these subsections.

### File template Jinja2 template file

The template file itself has a `jinja` extension.
For more information about the Jinja2 templating system, see the [Jinja2 template designer documentation](http://jinja.pocoo.org/docs/2.9/templates/).

The template file is designed to work with [Cookiecutter](https://cookiecutter.readthedocs.io/en/latest/).
This means that all template variables must be attributes of a `cookiecutter` variable.
For example:

```jinja
Copyright {{ cookiecutter.copyright_year }} {{ cookiecutter.copyright_holder }}
```

### File template SConscript

Include a `SConscript` file with the file template.
This example is from the `stack_license_py` template:

```python
from templatekit.builder import file_template_builder


env = Environment(BUILDERS={'FileTemplate': file_template_builder})
env.FileTemplate('example.py', 'template.py.jinja')
```

The variables that need to be updated for you template are:

- `example.py`: set this to be the name of the example file generated from `scons` and Cookiecutter.
- `template.py.jinja`: set this to be the name of the template source file.

Remember to include this `SConscript` file's path to the `SConstruct` file at the root of the repository.

### Template example file

The example file is generated via `scons` and should be committed in the Git repository whenever the template changes.
The file name of the example comes from the `SConscript`, and the template values are the defaults set in `cookiecutter.json`.

## Making a project template

Each project template is its own sub-directory in the `project_templates` root directory.
The standard components of a project template directory are:

- [README.md](#project-template-readme-file) file.
- [SConscript](#project-template-sconscript-file) file.
- [cookiecutter.json](#project-template-cookiecutterjson-file) file.
- [cookiecutter.project_name](#project-template-cookiecutter-directory) directory.
- [example](#project-template-example-directory) directory.

When you add a new project template, add a link to it from the root `README.md` file.

The next sections describe these files and directories in more detail.

### Project template README file

The title for a project's `README.md` should match the project's directory name.

Below the title, include a summary paragraph that describes what kinds of projects this project template can create.

Include a section called `Template variables`.
Subsections should match keys in the `cookiecutter.json` file and document each variable.

Include a section called `Files`.
Each subsection should be the name of a file (in a project cookiecutter directory).
Here you can document each component of that file, and describe how developers should extend the file for their own project.

### Project template SConscript file

Include a `SConscript` with the project template:

```python
from templatekit.builder import cookiecutter_project_builder


env = Environment(BUILDERS={'Cookiecutter': cookiecutter_project_builder})
env.Cookiecutter('cookiecutter.json')
```

This `SConscript` is ready-to-use for most project templates.
The only reason to extend this SConscript file is if the template requires additional tooling to render.

Remember to include this `SConscript` file's path in the `SConstruct` file at the root of the repository.

### Project template cookiecutter.json file

The `cookiecutter.json` file is used by [Cookiecutter](https://cookiecutter.readthedocs.io/en/latest/) to get input from a user and create a project.

The JSON file contains a single JSON object.
Keys are names of Jinja templates and values are default values.
For example:

```json
{
  "project_name": "example",
  "copyright_year": "{% now 'utc', '%Y' %}",
  "copyright_holder": "Association of Universities for Research in Astronomy",
  "_extensions": ["jinja2_time.TimeExtension"]
}
```

Besides scalar default values, you can instead specify [choice variables](https://cookiecutter.readthedocs.io/en/latest/advanced/choice_variables.html) and [dictionary variables](https://cookiecutter.readthedocs.io/en/latest/advanced/dict_variables.html).

See the [Cookiecutter documentation](https://cookiecutter.readthedocs.io/en/latest/), and other project templates, for more information and examples.

### Project template cookiecutter directory

In a cookiecutter project template, both the paths of files and directories, and the files's content, are Jinja2 template variables.

The root directory of a cookiecutter template is also a Jinja2 template.
For example, if the  `cookiecutter.json` file has a field `project_name`, and the directory of the project should be that name, the root directory of the cookiecutter template is:

```jinja
{{cookiecutter.project_name}}
```

### Project template example directory

The template example is generated by `scons`.
The name of the example directory should ideally be `example`.
You can set this from the `project_name` field in `cookiecutter.json`, assuming that the template cookiecutter directory is named `{{cookiecutter.project_name}}`.
Be sure to commit the example files to the Git repository whenever the template changes.

## Continuous integration

[Travis CI](https://travis-ci.org/lsst/templates) tests the repository when you push to GitHub.
The tests have two roles:

1. Ensure that the Cookiecutter templates are renderable.
2. Ensure that example files and projects are consistent with the templates.

This is done by running `scons` from the root of the templates repository.
The root `SConscript` file invokes the individual `SConscript` files of each template.
In turn, these `SConscript` files render the template using the defaults in the template's `cookiecutter.json` file.

If Travis CI detects an unclean state in the template Git repository after running `scons`, it marks the commit as a failure.
This is because an unclean Git state implies that the examples aren't reproducible from the template.

If this happens, run `scons` locally and commit the rendered example.
If files have been deleted from the template project, be sure to delete them from the rendered example as well.

See the [.travis.yml](.travis.yml) file for more details on how the tests are run.

## templatekit

`templatekit` is a Python package, included in the templates repository, that provides helpers for rendering file and project templates (including custom Scons builders).
`templatekit` also defines the dependencies needed to render the templates.

To install `templatekit` for development, run `python setup.py develop` from the root of the repository.

You can invoke the unit tests by running `python setup.py test`.
We use [pytest](https://docs.pytest.org/en/latest/) as the testing framework.

## FAQ

### Why Markdown and why README.md?

The templates repository is meant to be used at the code level, either as a local clone or on GitHub.
Since Markdown is GitHub's best-supported markup syntax, it makes sense to use it here rather than reStructuredText.
See the [GitHub Flavored Markdown documentation](https://help.github.com/articles/basic-writing-and-formatting-syntax/) for guidance on what you can do.

GitHub features `README.md` files in its interface.
By using `README.md` we're able to show documentation alongside directories in GitHub repositories.

### Why are file templates distinguished from projects templates?

**Project templates** help you bootstrap a new Git repository.
They only contain the essential boilerplate though.
You shouldn't need to sort out and delete extraneous template files from a newly-created project.

To make a Git repository useful, though, you'll need to add new files like Python modules, tests, or documentation.
**File templates** provide the form of these files (or parts of files).

In other words, you *start* a new Git repository with a Project template and continue developing it by drawing from file templates.

By separating files from projects we better document those file templates, and also use file templates across many different types projects.

### How do I keep a project consistent with a template?

It's inevitable that projects (and templates) will duplicate content from other templates.
For example, Python modules in a project will contain license boilerplate that originates in a template.
This duplication can cause some maintenance concerns.
Here are some strategies for dealing with duplicate content:

1. Symlink the template Jinja file into the project template.
   This works if the template is for a whole file, and that template is used by multiple projects.

2. Document the dependency so that projects are updated when an underlying template changes.
   This documentation should appear in the describes of individual files in the project's `README.md`.
   Link to the template by name so that the dependency is searchable.
   Maintainers and reviewers should check for template consistency.