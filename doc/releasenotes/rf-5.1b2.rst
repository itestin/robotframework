==========================
Robot Framework 5.1 beta 2
==========================

.. default-role:: code

`Robot Framework`_ 5.1 is a new feature release that starts Robot Framework's
localization efforts and also brings in other nice enhancements.
Robot Framework 5.1 preview releases are targeted especially
for people interested in translations.

All issues targeted for Robot Framework 5.1 can be found
from the `issue tracker milestone`_.
Questions and comments related to the release can be sent to the
`robotframework-users`_ mailing list or to `Robot Framework Slack`_,
and possible bugs submitted to the `issue tracker`_.

If you have pip_ installed, just run

::

   pip install --pre --upgrade robotframework

to install the latest available release or use

::

   pip install robotframework==5.1b2

to install exactly this version. Alternatively you can download the source
distribution from PyPI_ and install it manually. For more details and other
installation approaches, see the `installation instructions`_.

Robot Framework 5.1 beta 2 was released on Wednesday September 21, 2022.

.. _Robot Framework: http://robotframework.org
.. _Robot Framework Foundation: http://robotframework.org/foundation
.. _pip: http://pip-installer.org
.. _PyPI: https://pypi.python.org/pypi/robotframework
.. _issue tracker milestone: https://github.com/robotframework/robotframework/issues?q=milestone%3Av5.1
.. _issue tracker: https://github.com/robotframework/robotframework/issues
.. _robotframework-users: http://groups.google.com/group/robotframework-users
.. _Robot Framework Slack: https://robotframework-slack-invite.herokuapp.com
.. _installation instructions: ../../INSTALL.rst

.. contents::
   :depth: 2
   :local:

Most important enhancements
===========================

Localization
------------

Robot Framework 5.1 starts our localization efforts by making it possible to translate
various markers used in the data. It is possible to translate headers (e.g. `Test Cases`)
and settings (e.g. `Documentation`) (`#4096`_), `Given/When/Then` prefixes used in BDD
(`#519`_), as well as true and false strings used in Boolean argument conversion (`#4400`_).
Future versions may allow translating syntax like `IF` and `FOR`, contents of logs and
reports, error messages, and so on.

Languages to use are specified when starting execution using the `--language` command
line option. With languages supported by Robot Framework out-of-the-box, it is possible
to use just a language code or name like `--language fi` or `--language Finnish`.
It is also possible to create a custom language file and use it like `--language MyLang.py`.
If there is a need to support multiple languages, the `--language` option can be
used multiple times like `--language de --language uk`.

In addition to specifying the language from the command line, it is possible to
specify it in the data file itself using `language: <lang>` syntax, where `<lang>` is
a language code or name, before the first section::

    language: fi

    *** Asetukset ***
    Dokumentaatio        Example using Finnish.

Due to technical reasons this per-file language configuration affects also parsing
subsequent files, but that behavior is likely to change and *should not* be dependent
on. Either use `language: <lang>` in each parsed file or specify the language to
use from the command line.

Robot Framework 5.1 beta 2 contains built-in support for these languages in addition
to English that is automatically supported:

- Bosnian (BS)
- Czech (CS)
- Dutch (NL)
- Finnish (FI)
- French (FR)
- German (DE)
- Polish (PL)
- Portuguese (PT) and Brazilian Portuguese (PT-BR)
- Russian (RU)
- Simplified Chinese (ZH-CN)
- Spanish (ES)
- Thai (TH)
- Turkish (TR)
- Ukrainian (UK)

All these translations have been provided by our awesome community and we hope to get
more community contributed translations still before Robot Framework 5.1 final
release. If you are interested to help, head to Crowdin__ that we use
for collaboration. For more instructions see issue `#4390`_ and for general
discussion and questions join the `#localization` channel on our Slack.

__ https://robotframework.crowdin.com/robot-framework

Enhancements for setting keyword and test tags
----------------------------------------------

It is now possible to set tags for all keywords in a certain file by using
the new `Keyword Tags` setting (`#4373`_). It works in resource files and also
in test case and suite initialization files. When used in initialization files,
it only affects keywords in that file and does not propagate to lower level suites.

The `Force Tags` setting has been renamed to `Test Tags` (`#4368`_). The motivation
is to make settings related to tests more consistent (`Test Setup`, `Test Timeout`,
`Test Tags`, ...) and to better separate settings for specifying test and keyword tags.
Consistent naming also easies translations. The old `Force Tags` setting still works but it
will be `deprecated in the future`__. When creating tasks, it is possible to use
`Task Tags` alias instead of `Test Tags`.

To simplify setting tags, the `Default Tags` setting will `also be deprecated`__.
The functionality it provides, setting tags that some but no all tests get,
will be enabled in the future by using `-tag` syntax with the `[Tags]` setting
to indicate that a test should not get tag `tag`. This syntax will then work
also in combination with the new `Keyword Tags`. For more details see `#4374`__.

__ `Force Tags and Default Tags settings`_
__ `Force Tags and Default Tags settings`_
__ https://github.com/robotframework/robotframework/issues/4374

Enhancements to using keywords with embedded arguments
------------------------------------------------------

When using keywords with embedded arguments, it is pretty common that a keyword
that is used matches multiple keyword implementations. For example,
`Execute "ls" with "-lh"` in this example matches both of the keywords:

.. sourcecode:: robotframework

   *** Test Cases ***
   Automatic conflict resolution
       Execute "ls"
       Execute "ls" with "-lh"

   *** Keywords ***
   Execute "${cmd}"
       Log    Running command '${cmd}'.

   Execute "${cmd}" with "${opts}"
       Log    Running command '${cmd}' with options '${opts}'.

Earlier when such conflicts have occurred, execution has failed due to there
being multiple matching keywords. Nowadays Robot Framework tries to find the
best match and use that. In the above example, `Execute "${cmd}" with "${opts}"`
is considered a better match than the more generic `Execute "${cmd}"` and
the example thus succeeds. (`#4454`_)

There can, however, be cases where there is no single match that would be better
than others. In such cases conflicts cannot be automatically resolved and
execution fails as earlier.

Another nice enhancement related to keywords using embedded arguments is that
if they are used with `Run Keyword` or its variants, arguments aren't anymore
always converted to strings. This allows passing arguments containing other
values as variables. (`#1595`_)

Enhancements to keyword namespaces
----------------------------------

It is possible to mark keywords in resource files as private by adding
`robot:private` tag to them (`#430`_). If such a keyword is used by keywords
outside that resource file, there will be a warning. These keywords are also
excluded from HTML library documentation generated by Libdoc.

If a keyword exists in the same resource file as a keyword using it, it will
be used even if there would be keyword with the same name in another resource
file (`#4366`_). Earlier this situation caused a conflict.

If a keyword exists in the same resource file as a keyword using it and there
is a keyword with the same name in the test case file, the keyword in the test
case file will be used as it has been used earlier. This behavior is nowadays
deprecated__, though, and in the future local keywords will have precedence also
in these cases.

__ `Keywords in test case files having precedence over local keywords in resource files`_

Possibility to disable continue-on-failure mode
-----------------------------------------------

Robot Framework generally stops executing a keyword or a test case if there
is a failure. Exceptions to this rule include teardowns, templates and
cases where the continue-on-failure mode has been explicitly enabled with
`robot:continue-on-failure` or `robot:recursive-continue-on-failure`
tags. Robot Framework 5.1 makes it possible to disable the implicit or explicit
continue-on-failure mode when needed by using `robot:stop-on-failure` and
`robot:recursive-stop-on-failure` tags (`#4303`_).

`start/end_keyword` listener methods get more information about control structures
----------------------------------------------------------------------------------

When using the listener API v2, `start_keyword` and `end_keyword` methods are not
only used with keywords but also with all control structures. Earlier these methods
always got exactly the same information, but nowadays there is additional context
specific details with control structures (`#4335`_).

Performance enhancements for executing user keywords
----------------------------------------------------

The overhead in executing user keywords has been reduced. The difference
can be seen especially if user keywords fail often, for example, when using
`Wait Until Keyword Succeeds` or a loop with `TRY/EXCEPT`. (`#4388`_)

Python 3.11 support
--------------------

Robot Framework 5.1 officially supports the forthcoming Python 3.11
release (`#4401`_). Incompatibilities were not too big, so also the earlier
versions work fairly well.

At the other end of the spectrum, Python 3.6 is deprecated and will not
anymore be supported by Robot Framework 6.0 (`#4295`_).


Backwards incompatible changes
==============================

- Space is required after `Given/When/Then` prefixes used with BDD scenarios. (`#4379`_)
- Dictionary related keywords in `Collections` require dictionaries to inherit `Mapping`. (`#4413`_)
- `Dictionary Should Contain Item` from the Collections library does not anymore convert
  values to strings before comparison. (`#4408`_)
- Generation time in XML and JSON spec files generated by Libdoc has been changed to
  `2022-05-27T19:07:15+00:00`. With XML specs the format used to be `2022-05-27T19:07:15Z`
  that is equivalent with the new format. JSON spec files did not include the timezone
  information at all and the format was `2022-05-27 19:07:15`. (`#4262`_)
- `BuiltIn.run_keyword()` nowadays resolves variables in the name of the keyword to
  execute when earlier they were resolved by Robot Framework before calling the keyword.
  This affects programmatic usage if the used name contains variables or backslashes.
  The change was done when enhancing how keywords with embedded arguments work with
  `BuiltIn.run_keyword()`. (`#1595`_)


Deprecated features
===================

`Force Tags` and `Default Tags` settings
----------------------------------------

As `discussed above`__, new `Test Tags` setting has been added to replace `Force Tags`
and there is a plan to remove `Default Tags` altogether. Both of these settings still
work but they are considered deprecated. There is no visible deprecation warning yet,
but such a warning will be emitted starting from Robot Framework 6.0 and eventually these
settings will be removed. (`#4368`_)

The plan is to add new `-tag` syntax that can be used with the `[Tags]` setting
to enable similar functionality that the `Default Tags` setting provides. Because
of that, using tags starting with a hyphen with the `[Tags]` setting is now deprecated.
If such literal values are needed, it is possible to use escaped format like `\-tag`.
(`#4380`_)

__ `Enhancements for setting keyword and test tags`_

Keywords in test case files having precedence over local keywords in resource files
-----------------------------------------------------------------------------------

Keywords in test cases files currently always have the highest precedence. They
are used even when a keyword in a resource file uses a keyword that would exist also
in the same resource file. This will change so that local keywords always have
highest precedence and the current behavior is deprecated. (`#4366`_)

`WITH NAME` in favor of `AS` when giving alias to imported library
------------------------------------------------------------------

`WITH NAME` marker that is used when giving an alias to an imported library
will be renamed to `AS` (`#4371`_). The motivation is to be consistent with
Python that uses `as` for similar purpose. We also already use `AS` with
`TRY/EXCEPT` and reusing the same marker and internally used token simplifies
the syntax. Having less markers will also ease translations (but these markers
cannot yet be translated).

In Robot Framework 5.1 both `AS` and `WITH NAME` work when setting an alias
for a library. `WITH NAME` is considered deprecated, but there will not be
visible deprecation warnings until Robot Framework 6.0.

Singular section headers like `Test Case`
-----------------------------------------

Robot Framework has earlier accepted both plural (e.g. `Test Cases`) and singular
(e.g. `Test Case`) section headers. The singular variants are now deprecated
and their support will eventually be removed (`#4431`_). The is no visible
deprecation warning yet, but they will most likely be emitted starting from
Robot Framework 6.0.

Using variables with embedded arguments so that value does not match custom pattern
-----------------------------------------------------------------------------------

When keywords accepting embedded arguments are used so that arguments are
passed as variables, variable values are not checked against possible custom
regular expressions. Keywords being called with arguments they explicitly do not
accept is problematic and this behavior will be changed. Due to the backwards
compatibility it is now only deprecated, but validation will be more strict
in the future. (`#4462`_)

Custom patterns have often been used to avoid conflicts when using embedded arguments.
That need is nowadays smaller because Robot Framework 5.1 can typically resolve
conflicts automatically. (`#4454`_)

Python 3.6 support
------------------

Python 3.6 `reached end-of-life`__ in December 2021. It will be still supported
by Robot Framework 5.1 and all future RF 5.x releases, but not anymore by
Robot Framework 6.0 (`#4295`_). Users are recommended to upgrade to newer
versions already now.

__  https://endoflife.date/python


Acknowledgements
================

Robot Framework development is sponsored by the `Robot Framework Foundation`_
and its ~50 member organizations. Robot Framework 5.1 team funded by the foundation
consisted of `Pekka Klärck <https://github.com/pekkaklarck>`_ and
`Janne Härkönen <https://github.com/yanne>`_ (part time).
In addition to that, the wider open source community has provided several
great contributions:

- `Elout van Leeuwen <https://github.com/leeuwe>`_ has lead the localization efforts
  (`#4390`_). Individual translations have been provided by the following people:

  - Bosnian by `Namik <https://github.com/Delilovic>`_
  - Czech by `Václav Fuksa <https://github.com/MoreFamed>`_
  - Dutch by `Pim Jansen <https://github.com/pimjansen>`_ and
    `Elout van Leeuwen <https://github.com/leeuwe>`_
  - French by `@lesnake <https://github.com/lesnake>`_
  - German by `René <https://github.com/Snooz82>`_ and `Markus <https://github.com/Noordsestern>`_
  - Polish by `Bartłomiej Hirsz <https://github.com/bhirsz>`_
  - Portuguese and Brazilian Portuguese by `Hélio Guilherme <https://github.com/HelioGuilherme66>`_
  - Russian by `Anatoly Kolpakov <https://github.com/axxyhtrx>`_
  - Simplified Chinese by `charis <https://github.com/mawentao119>`_ and `@nixuewei <https://github.com/nixuewei>`_
  - Spanish by Miguel Angel Apolayo Mendoza
  - Thai by `Somkiat Puisungnoen <https://github.com/up1>`_
  - Turkish by `Yusuf Can Bayrak <https://github.com/yusufcanb>`_
  - Ukrainian by `@Sunshine0000000 <https://github.com/Sunshine0000000>`_

- `Oliver Boehmer <https://github.com/oboehmer>`_ provide several contributions:

  - Support to disable the continue-on-failure mode using `robot:stop-on-failure` and
    `robot:recursive-stop-on-failure` tags. (`#4303`_)
  - Document that failing test setup stops execution even if the continue-on-failure
    mode is active. (`#4404`_)
  - Default value to `Get From Dictionary` keyword. (`#4398`_)
  - Allow passing explicit flags to regexp related keywords. (`#4429`_)

- `Ossi R. <https://github.com/osrjv>`_ added more information to `start/end_keyword`
  listener methods when they are used with control structures (`#4335`_).

- `René <https://github.com/Snooz82>`_ fixed Libdoc's HTML outputs if type hints
  matched Javascript variables in browser namespace (`#4464`_) or keyword names (`#4471`_).

- `J. Foederer <https://github.com/JFoederer>`_ enhanced performance of
  `Keyword Should Exist` when a keyword is not found (`#4470`_).

- `Fabio Zadrozny <https://github.com/fabioz>`_ provided a pull request speeding up
  user keyword execution (`#4353`_).

- `@Apteryks <https://github.com/Apteryks>`_ added support to generate deterministic
  library documentation by using `SOURCE_DATE_EPOCH`__ environment variable (`#4262`_).

__ https://reproducible-builds.org/specs/source-date-epoch/

Thanks also to all community members who have submitted bug reports, helped debugging
problems, or otherwise helped with the release.

| `Pekka Klärck <https://github.com/pekkaklarck>`__
| Robot Framework Creator

Full list of fixes and enhancements
===================================

.. list-table::
    :header-rows: 1

    * - ID
      - Type
      - Priority
      - Summary
      - Added
    * - `#4096`_
      - enhancement
      - critical
      - Multilanguage support for markers used in data
      - alpha 1
    * - `#519`_
      - enhancement
      - critical
      - Given/When/Then should support other languages than English
      - alpha 1
    * - `#1595`_
      - bug
      - high
      - Embedded arguments are not passed as objects when executed with `Run Keyword` or its variants
      - beta 2
    * - `#4295`_
      - enhancement
      - high
      - Deprecate Python 3.6
      - alpha 1
    * - `#430`_
      - enhancement
      - high
      - Keyword visibility modifiers for resource files
      - alpha 1
    * - `#4303`_
      - enhancement
      - high
      - Support disabling continue-on-failure mode using `robot:stop-on-failure` and `robot:recursive-stop-on-failure` tags
      - alpha 1
    * - `#4335`_
      - enhancement
      - high
      - Pass more information about control structures to `start/end_keyword` listener methods
      - beta 1
    * - `#4366`_
      - enhancement
      - high
      - Give local keywords precedence over imported keywords in resource files
      - alpha 1
    * - `#4368`_
      - enhancement
      - high
      - New `Test Tags` setting as an alias for `Force Tags`
      - alpha 1
    * - `#4373`_
      - enhancement
      - high
      - Support adding tags for all keywords using `Keyword Tags` setting
      - alpha 1
    * - `#4380`_
      - enhancement
      - high
      - Deprecate setting tags starting with a hyphen like `-tag` using the `[Tags]` setting
      - alpha 1
    * - `#4388`_
      - enhancement
      - high
      - Enhance performance of executing user keywords especially when they fail
      - alpha 1
    * - `#4400`_
      - enhancement
      - high
      - Allow translating True and False words used in Boolean argument conversion
      - beta 1
    * - `#4401`_
      - enhancement
      - high
      - Python 3.11 compatibility
      - alpha 1
    * - `#4454`_
      - enhancement
      - high
      - Automatically select "best" match if there is conflict with keywords using embedded arguments
      - beta 2
    * - `#4351`_
      - bug
      - medium
      - Libdoc can give bad error message if library argument has extension matching resource files
      - alpha 1
    * - `#4355`_
      - bug
      - medium
      - Continuable failures terminate WHILE loops
      - alpha 1
    * - `#4357`_
      - bug
      - medium
      - Parsing model: Creating `TRY` and `WHILE` statements using `from_params` is not possible
      - alpha 1
    * - `#4359`_
      - bug
      - medium
      - Parsing model: `Variable.from_params` doesn't handle list values properly
      - alpha 1
    * - `#4364`_
      - bug
      - medium
      - `@{list}` used as embedded argument not anymore expanded if keyword accepts varargs
      - beta 1
    * - `#4381`_
      - bug
      - medium
      - Parsing errors are recognized as EmptyLines
      - alpha 1
    * - `#4384`_
      - bug
      - medium
      - RPA aliases for settings do not work in suite initialization files
      - alpha 1
    * - `#4387`_
      - bug
      - medium
      - Libdoc: Fix storing information about deprecated keywords to spec files
      - alpha 1
    * - `#4408`_
      - bug
      - medium
      - Collection: `Dictionary Should Contain Item` incorrectly casts values to strings before comparison
      - alpha 1
    * - `#4418`_
      - bug
      - medium
      - Dictionaries insider lists in YAML variable files not converted to DotDict objects
      - beta 1
    * - `#4438`_
      - bug
      - medium
      - `Get Time` returns current time if it is given input time that matches epoch
      - beta 2
    * - `#4441`_
      - bug
      - medium
      - Regression: Empty `--include/--exclude/--test/--suite` are not ignored
      - beta 2
    * - `#4447`_
      - bug
      - medium
      - Evaluating expressions that modify evaluation namespace (locals) fail
      - beta 1
    * - `#4455`_
      - bug
      - medium
      - Standard libraries don't support `pathlib.Path` objects
      - beta 2
    * - `#4464`_
      - bug
      - medium
      - Libdoc: Type hints aren't shown for types with same name as Javascript variables available in browser namespace
      - beta 2
    * - `#4262`_
      - enhancement
      - medium
      - Honor `SOURCE_DATE_EPOCH` environment variable when generating library documentation
      - alpha 1
    * - `#4312`_
      - enhancement
      - medium
      - Add project URLs to PyPI
      - alpha 1
    * - `#4353`_
      - enhancement
      - medium
      - Performance enhancements to parsing
      - alpha 1
    * - `#4354`_
      - enhancement
      - medium
      - When merging suites with Rebot, copy documentation and metadata from merged suites
      - beta 1
    * - `#4371`_
      - enhancement
      - medium
      - Add `AS` alias for `WITH NAME` in library imports
      - alpha 1
    * - `#4379`_
      - enhancement
      - medium
      - Require space after Given/When/Then prefixes
      - alpha 1
    * - `#4398`_
      - enhancement
      - medium
      - Collections: `Get From Dictionary` should accept a default value
      - alpha 1
    * - `#4404`_
      - enhancement
      - medium
      - Document that failing test setup stops execution even if continue-on-failure mode is active
      - alpha 1
    * - `#4413`_
      - enhancement
      - medium
      - Dictionary related keywords in `Collections` are more script about accepted values
      - alpha 1
    * - `#4429`_
      - enhancement
      - medium
      - Allow passing flags to regexp related keywords using explicit `flags` argument
      - beta 1
    * - `#4431`_
      - enhancement
      - medium
      - Deprecate using singular section headers
      - beta 1
    * - `#4440`_
      - enhancement
      - medium
      - Allow using `None` as custom argument converter to enable strict type validation
      - beta 1
    * - `#4461`_
      - enhancement
      - medium
      - Automatic argument conversion for `pathlib.Path`
      - beta 2
    * - `#4462`_
      - enhancement
      - medium
      - Deprecate using embedded arguments using variables that do not match custom regexp
      - beta 2
    * - `#4470`_
      - enhancement
      - medium
      - Enhance `Keyword Should Exist` performance by not looking for possible recommendations
      - beta 2
    * - `#4349`_
      - bug
      - low
      - User Guide: Example related to YAML variable files is buggy
      - alpha 1
    * - `#4358`_
      - bug
      - low
      - User Guide: Errors in examples related to TRY/EXCEPT
      - alpha 1
    * - `#4453`_
      - bug
      - low
      - `Run Keywords`: Execution is not continued in teardown if keyword name contains non-existing variable
      - beta 2
    * - `#4471`_
      - bug
      - low
      - Libdoc: If keyword and type have same case-insensitive name, opening type info opens keyword documentation
      - beta 2
    * - `#4346`_
      - enhancement
      - low
      - Enhance documentation of the `--timestampoutputs` option
      - alpha 1
    * - `#4372`_
      - enhancement
      - low
      - Document how to import resource files bundled into Python packages
      - alpha 1
    * - `#4394`_
      - bug
      - ---
      - Error when `--doc` or `--metadata` value matches an existing directory
      - alpha 1

Altogether 52 issues. View on the `issue tracker <https://github.com/robotframework/robotframework/issues?q=milestone%3Av5.1>`__.

.. _#4096: https://github.com/robotframework/robotframework/issues/4096
.. _#519: https://github.com/robotframework/robotframework/issues/519
.. _#1595: https://github.com/robotframework/robotframework/issues/1595
.. _#4295: https://github.com/robotframework/robotframework/issues/4295
.. _#430: https://github.com/robotframework/robotframework/issues/430
.. _#4303: https://github.com/robotframework/robotframework/issues/4303
.. _#4335: https://github.com/robotframework/robotframework/issues/4335
.. _#4366: https://github.com/robotframework/robotframework/issues/4366
.. _#4368: https://github.com/robotframework/robotframework/issues/4368
.. _#4373: https://github.com/robotframework/robotframework/issues/4373
.. _#4380: https://github.com/robotframework/robotframework/issues/4380
.. _#4388: https://github.com/robotframework/robotframework/issues/4388
.. _#4400: https://github.com/robotframework/robotframework/issues/4400
.. _#4401: https://github.com/robotframework/robotframework/issues/4401
.. _#4454: https://github.com/robotframework/robotframework/issues/4454
.. _#4351: https://github.com/robotframework/robotframework/issues/4351
.. _#4355: https://github.com/robotframework/robotframework/issues/4355
.. _#4357: https://github.com/robotframework/robotframework/issues/4357
.. _#4359: https://github.com/robotframework/robotframework/issues/4359
.. _#4364: https://github.com/robotframework/robotframework/issues/4364
.. _#4381: https://github.com/robotframework/robotframework/issues/4381
.. _#4384: https://github.com/robotframework/robotframework/issues/4384
.. _#4387: https://github.com/robotframework/robotframework/issues/4387
.. _#4408: https://github.com/robotframework/robotframework/issues/4408
.. _#4418: https://github.com/robotframework/robotframework/issues/4418
.. _#4438: https://github.com/robotframework/robotframework/issues/4438
.. _#4441: https://github.com/robotframework/robotframework/issues/4441
.. _#4447: https://github.com/robotframework/robotframework/issues/4447
.. _#4455: https://github.com/robotframework/robotframework/issues/4455
.. _#4464: https://github.com/robotframework/robotframework/issues/4464
.. _#4262: https://github.com/robotframework/robotframework/issues/4262
.. _#4312: https://github.com/robotframework/robotframework/issues/4312
.. _#4353: https://github.com/robotframework/robotframework/issues/4353
.. _#4354: https://github.com/robotframework/robotframework/issues/4354
.. _#4371: https://github.com/robotframework/robotframework/issues/4371
.. _#4379: https://github.com/robotframework/robotframework/issues/4379
.. _#4398: https://github.com/robotframework/robotframework/issues/4398
.. _#4404: https://github.com/robotframework/robotframework/issues/4404
.. _#4413: https://github.com/robotframework/robotframework/issues/4413
.. _#4429: https://github.com/robotframework/robotframework/issues/4429
.. _#4431: https://github.com/robotframework/robotframework/issues/4431
.. _#4440: https://github.com/robotframework/robotframework/issues/4440
.. _#4461: https://github.com/robotframework/robotframework/issues/4461
.. _#4462: https://github.com/robotframework/robotframework/issues/4462
.. _#4470: https://github.com/robotframework/robotframework/issues/4470
.. _#4349: https://github.com/robotframework/robotframework/issues/4349
.. _#4358: https://github.com/robotframework/robotframework/issues/4358
.. _#4453: https://github.com/robotframework/robotframework/issues/4453
.. _#4471: https://github.com/robotframework/robotframework/issues/4471
.. _#4346: https://github.com/robotframework/robotframework/issues/4346
.. _#4372: https://github.com/robotframework/robotframework/issues/4372
.. _#4394: https://github.com/robotframework/robotframework/issues/4394
.. _#4390: https://github.com/robotframework/robotframework/issues/4390
