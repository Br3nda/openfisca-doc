# OpenFisca extensions

*Note: The following is, today, only implemented in openfisca-france*

Extensions allow you to add formulas to OpenFisca that are not included in the openfisca-france repository (e.g. local prestations).

Extensions folders located in `openfisca_france/model/extensions/` will be **automatically loaded** at the initialization of the tax benefit system.
They can be brought there manually or by any custom mechanism up to your convenience.

If you need to import an extension located out of the `extensions` folder, you can use the following function:

```py
openfisca_france.model.extensions.import_extension('/path/to/external/extension/folder')
```

##Extension architecture

The architecture of an extension folder is the following:

```sh
extensions/{extension_name}/ # The folder name is by convention the name of the extension.
    extensions/{extension_name}/__init__.py # Empty file.
    extensions/{extension_name}/parameters.xml # Optional parameters file.
    extensions/{extension_name}/{some_formula}.py # File containing formulas
    extensions/{extension_name}/{other_formula}.py
    extensions/{extension_name}/{some_formula}.yaml # Optional test files
    extensions/{extension_name}/{other_formula}.yaml
```
All python files located directly in `extensions/{extension_name}/` are imported in the tax benefit system.

Subdirectories are ignored, as well as any other xml file than `parameters.xml`.

The syntax of the formulas within extension python files is the same than in the general openfisca-france formulas, except that imports should not be relative (e.g. `from openfisca_france.model.base import *`).

Variables inside an extension should not have the same name than any existing formula, nor than any formula in another extension being used.
