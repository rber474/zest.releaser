Entrypoints: extending/changing zest.releaser
=============================================

A zest.releaser entrypoint gets passed a data dictionary and that's about it.
You can do tasks like generating documentation.  Or downloading external files
you don't want to store in your repository but that you do want to have
included in your egg.

Every release step (prerelease, release and postrelease) has three points
where you can hook in an entry point:

before
    Only the ``workingdir`` and ``name`` are available in the data
    dictionary, nothing has happened yet.

middle
    All data dictionary items are available and some questions (like new
    version number) have been asked.  No filesystem changes have been made
    yet.

after
    The action has happened, everything has been written to disk or uploaded
    to pypi or whatever.


For the release step it made sense to create extra entry points:

after_checkout
    The middle entry point has been handled, the tag has been made, a
    checkout of that tag has been made and we are now in that checkout
    directory.  Of course, when the user chooses not to do a checkout,
    this entry point never triggers.

before_upload
    The source distribution and maybe the wheel have been made.  We
    are about to upload to PyPI with ``python setup.py`` or ``twine
    upload dist/*``.  You may want to use this hook to to sign a
    distribution before twine uploads it.

Note that an entry point can be specific for one package (usually the
package that you are now releasing) or generic for all packages.  An
example of a generic one is `gocept.zestreleaser.customupload`_, which
offers to upload the generated distribution to a chosen destination
(like a server for internal company use).  If your entry point is
specific for the current package only, you should add an extra check
to make sure it is not run while releasing other packages; something
like this should do the trick::

    def my_entry_point(data):
        if data['name'] != 'my.package':
            return
        ...

.. _`gocept.zestreleaser.customupload`: https://pypi.org/project/gocept.zestreleaser.customupload


Entry point specification
-------------------------

An entry point is configured like this in your setup.py::

      entry_points={
          #'console_scripts': [
          #    'myscript = my.package.scripts:main'],
          'zest.releaser.prereleaser.middle': [
              'dosomething = my.package.some:some_entrypoint',
              ]},

or like this in your pyproject.toml::

    [project.entry-points."zest.releaser.prereleaser.middle"]
    dosomething = "my.package.some:some_entrypoint"

Entry-points can also be specified in the setup.cfg file like this
(The options will be split by white-space and processed in the given
order.)::

    [zest.releaser]
    prereleaser.middle =
       my.package.some.some_entrypoint
       our.package.other_module.other_function


In ``zest.releaser.prereleaser.middle`` resp. ``prereleaser.middle``
replace ``prereleaser``
with the respective command name (`prerelease`, `release`, `postrelease`,
etc.)
and ``middle`` with the respective hook name (`before`, `middle`, `after`,
`after_checkout`, etc.)
as needed.

Similarly, they can also be placed in the ``tool.zest-releaser`` config section of
pyproject.toml like this::

    [tool.zest-releaser]
    prereleaser.middle = [
        "my.package.some.some_entrypoint",
        "our.package.other_module.other_function"
    ]

See the ``setup.py`` of ``zest.releaser`` itself for some real world examples.

You'll have to make sure that the zest.releaser scripts know about your entry
points, for instance by placing your egg (with entry point) in the same
zc.recipe.egg section in your buildout as where you placed zest.releaser.  Or,
if you installed zest.releaser globally, your egg-with-entrypoint has to be
globally installed, too.

Notes:

* Entry-points given in ``setup.cfg`` will be processed before
  entry-point defined via installed packages.

* The order in which entry-point defined via installed packages are
  processed is undefined.

* *If* you use an entry point defined in the package you're releasing *and*
  your package has the code inside a ``src/`` dir, you might have to add
  ``hook_package_dir = src`` to the ``[tool.zest-releaser]`` section.


Comments about data dict items
------------------------------

Your entry point gets a data dictionary: the items you get in that dictionary
are documented below.  Some comments about them:

- Not all items are available.  If no history/changelog file is found, there
  won't be any ``data['history_lines']`` either.

- Items that are templates are normal python string templates.  They use
  dictionary replacement: they're actually passed the same data dict.  For
  instance, prerelease's ``data['commit_message']`` is by default ``Preparing
  release %(new_version)s``.  A "middle" entry point could modify this
  template to get a different commit message.

- Entry-points can change the the ``data`` dictionary, thus one hook
  can prepare data another one is using. But be aware that changing
  entries in the dict may lead to malfunction.


.. ### AUTOGENERATED FROM HERE ###

Common data dict items
----------------------

These items are shared among all commands.

commit_msg
    Message template used when committing

has_released_header
    Latest header is for a released version with a date

headings
    Extracted headings from the history file

history_encoding
    The detected encoding of the history file

history_file
    Filename of history/changelog file (when found)

history_header
    Header template used for 1st history header

history_insert_line_here
    Line number where an extra changelog entry can be inserted.

history_last_release
    Full text of all history entries of the current release

history_lines
    List with all history file lines (when found)

name
    Name of the project being released

new_version
    New version to write, possibly with development marker

nothing_changed_yet
    First line in new changelog section, warn when this is still in there before releasing

original_version
    Original package version before any changes

reporoot
    Root of the version control repository

required_changelog_text
    Text that must be present in the changelog. Can be a string or a list, for example ["New:", "Fixes:"]. For a list, only one of them needs to be present.

update_history
    Should zest.releaser update the history file?

workingdir
    Original working directory

``prerelease`` data dict items
------------------------------

today
    Date string used in history header

``release`` data dict items
---------------------------

tag
    Tag we're releasing

tag-message
    Commit message for the tag

tag-signing
    Sign tag using gpg or pgp

tag_already_exists
    Internal detail, don't touch this :-)

tagdir
    Directory where the tag checkout is placed (*if* a tag
    checkout has been made)

tagworkingdir
    Working directory inside the tag checkout. This is
    the same, except when you make a release from within a sub directory.
    We then make sure you end up in the same relative directory after a
    checkout is done.

version
    Version we're releasing

``postrelease`` data dict items
-------------------------------

breaking
    True if we handle a breaking (major) change

dev_version
    New version with development marker (so 1.1.dev0)

dev_version_template
    Template for development version number

development_marker
    String to be appended to version after postrelease

feature
    True if we handle a feature (minor) change

final
    True if we handle a final release

new_version
    New version, without development marker (so 1.1)

``addchangelogentry`` data dict items
-------------------------------------

commit_msg
    Message template used when committing. Default: same as the message passed on the command line.

message
    The message we want to add

``bumpversion`` data dict items
-------------------------------

breaking
    True if we handle a breaking (major) change

clean_new_version
    Clean new version (say 1.1)

feature
    True if we handle a feature (minor) change

final
    True if we handle a final release

release
    Type of release: breaking, feature, normal, final
