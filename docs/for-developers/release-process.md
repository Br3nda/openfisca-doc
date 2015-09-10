# Release process

> This section is for maintainers who want to build and release a Python package of OpenFisca
> on the [PyPI](https://pypi.python.org/pypi) repository.

Here are the steps to follow to build and release a Python package.
Execute them on each Git repository you want to publish, in that order:
* [OpenFisca-Core](https://github.com/openfisca/openfisca-core)
* [OpenFisca-France](https://github.com/openfisca/openfisca-france)
* [OpenFisca-Parsers](https://github.com/openfisca/openfisca-parsers)
* [OpenFisca-Web-API](https://github.com/openfisca/openfisca-web-api)

As explained in the [branching model](development-process.md#branching-model) section,
each repository uses `master` and `next` git branches.
Releasing a new version implies merging the `next` branch int `master`,
but we want only *one merge commit* to appear in the `master` branch.
All the pre-release related stuff is commited in the `next` branch.

For each step we specify the branch on which you should be, like that:

    (branch_name) command line

See also:
* [PEP 440: public version identifiers](http://legacy.python.org/dev/peps/pep-0440/#public-version-identifiers)
* [distributing packages guide](https://python-packaging-user-guide.readthedocs.org/en/latest/distributing.html)
* [setuptools](https://pythonhosted.org/setuptools/setuptools.html)
* [semver](http://semver.org/)

## Steps to execute

Execute tests and check that there is no error:

```bash
(next) make test
or
(next) nosetests
```

> This first step has to be executed on all the released repositories before continuing.

If the project is internationalized with [GNU gettext](https://www.gnu.org/software/gettext/) via [Babel](http://babel.pocoo.org/):

* Extract strings to translate from source code:

  ```bash
  (next) python setup.py extract_messages
  ```

* Update catalog (aka `.po` files) from `.pot` file:

  ```bash
  (next) python setup.py update_catalog
  ```

* Translate them if needed (using [poedit](https://poedit.net/) for example):

  ```bash
  (next) poedit xxx/i18n/fr/LC_MESSAGES/yyy.po
  ```

* Ensure that `Project-Id-Version` in `.pot` and `.po` files are correct.

* If there are modified files, commit them.

* Compile catalog:

  ```bash
  (next) python setup.py compile_catalog
  ```

* Should display `(100%) translated`.

Close the "next release" section in `CHANGELOG.md` and fill the changes list:

  ```
  ## NEW_RELEASE_NUMBER.dev0 - next release
becomes
  ## NEW_RELEASE_NUMBER

delete line:
  * TODO Fill this changes list while developing
```

> Use these commands to dig the Git history:

```bash
(next) git log OLD_RELEASE_NUMBER..
(next) git shortlog OLD_RELEASE_NUMBER..
```

> OLD_RELEASE_NUMBER has to be replaced by a real value (ie `0.5.0` without ".dev0" suffix), assuming the corresponding git tag was set.

Edit `setup.py`:

> Set the release version number which can be determined:
> * by removing the ".dev0" suffix (ie from "X.Y.Z.dev0" to "X.Y.Z")
> * and, depending on the changelog contents, by increasing the major or minor version number
>
> Check that everything is OK, in particular requirements (install_requires and extras_require).

```python
setup(
    [...]
    version = 'NEW_RELEASE_NUMBER',
    [...]
    )
```

Comment the dependencies installed by Git in `requirements.txt`:

```
#-e git+https://github.com/openfisca/openfisca-core.git@master#egg=OpenFisca-Core
and others perhaps
```

Commit changes (message: "Release X.Y.Z") and push.

Merge the `next` branch into `master`:

```bash
(next) git checkout master
(master) git merge next
```

Register the package on the [PyPI test instance](https://wiki.python.org/moin/TestPyPI), only the first time, but can be done many times:

> Note: this operation is protected by an authentication, as well as the other commands dealing with PyPI.

```bash
(master) python setup.py register -r https://testpypi.python.org/pypi
```

Build and [upload](https://python-packaging-user-guide.readthedocs.org/en/latest/distributing.html#uploading-your-project-to-pypi) the package to the PyPI test instance:

```bash
(master) python setup.py sdist bdist_wheel upload -r https://testpypi.python.org/pypi
```

Check if package install correctly from the PyPI test instance:

```bash
# TODO: this does not work!
(master) pip install -i https://testpypi.python.org/pypi <package name>
```

Tag the new release and upload it to git server:

```bash
(master) git tag NEW_RELEASE_NUMBER
(master) git push origin NEW_RELEASE_NUMBER
(master) git push
```

Register the package on PyPI, only the first time, but can be done many times:

```bash
(master) python setup.py register
```

Build and upload the package to PyPI:

```bash
(master) python setup.py sdist bdist_wheel upload
```

In a new shell check if package install correctly from PyPI, using a [virtualenv](https://virtualenv.pypa.io/en/latest/):

```bash
cd ~/tmp
virtualenv openfisca
cd openfisca
source bin/activate
pip install <package name> (ie OpenFisca-Core)
python
import <module name> (ie openfisca_core)
deactivate
```

Switch back to the previous shell and checkout the `next` branch:

```bash
(master) git checkout next
```

Edit `setup.py` to change version number (ie increase patch number and add ".dev0" suffix):

```python
setup(
    [...]
    version = 'NEW_FUTURE_RELEASE_NUMBER.dev0',
    [...]
    )
```

Create the next release section in `CHANGELOG.md`, ie:

```
## NEW_FUTURE_RELEASE_NUMBER.dev0 - next release

* TODO Fill this changes list while developing
```

> Keep the "TODO" list item as is.

Uncomment the dependencies installed by Git in `requirements.txt`:

```
-e git+https://github.com/openfisca/openfisca-core.git@master#egg=OpenFisca-Core
and others perhaps
```

Commit changes and push:

```bash
(next) git add setup.py
(next) git commit -m "Update to next dev version"
(next) git push
```

Announce the new release on the website news, Twitter, the mailing list and [TF1](http://www.tf1.fr/).
