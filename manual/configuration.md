# Configuration

The behavior of RuboCop can be controlled via the
[.rubocop.yml](https://github.com/rubocop-hq/rubocop/blob/master/.rubocop.yml)
configuration file. It makes it possible to enable/disable certain cops
(checks) and to alter their behavior if they accept any parameters. The file
can be placed in your home directory, XDG config directory, or in some project
directory.

The file has the following format:

```yaml
inherit_from: ../.rubocop.yml

Style/Encoding:
  Enabled: false

Layout/LineLength:
  Max: 99
```

!!! Note

    Qualifying cop name with its type, e.g., `Style`, is recommended,
    but not necessary as long as the cop name is unique across all types.

## Config file locations

RuboCop will start looking for the configuration file in the directory
where the inspected file is and continue its way up to the root directory.

If it cannot be found until reaching the project's root directory, then it will
be searched for in the user's global config locations, which consists of a
dotfile or a config file inside the [XDG Base Directory
specification][xdg-basedir-spec].

* `~/.rubocop.yml`
* `$XDG_CONFIG_HOME/rubocop/config.yml` (expands to `~/.config/rubocop/config.yml`
  if `$XDG_CONFIG_HOME` is not set)

If both files exist, the dotfile will be selected.

As an example, if RuboCop is invoked from inside `/path/to/project/lib/utils`,
then RuboCop will use the config as specified inside the first of the following
files:

* `/path/to/project/lib/utils/.rubocop.yml`
* `/path/to/project/lib/.rubocop.yml`
* `/path/to/project/.rubocop.yml`
* `/.rubocop.yml`
* `~/.rubocop.yml`
* `~/.config/rubocop/config.yml`
* [RuboCop's default configuration][1]

## Inheritance

All configuration inherits from [RuboCop's default configuration][1] (See
"Defaults").

RuboCop also supports inheritance in user's configuration files. The most common
example would be the `.rubocop_todo.yml` file (See "Automatically Generated
Configuration" below).

Settings in the child file (that which inherits) override those in the parent
(that which is inherited), with the following caveats.

### Inheritance of hashes vs. other types

Configuration parameters that are hashes, for example `PreferredMethods` in
`Style/CollectionMethods`, are merged with the same parameter in the parent
configuration. This means that any key-value pairs given in child configuration
override the same keys in parent configuration. Giving `~`, YAML's
representation of `nil`, as a value cancels the setting of the corresponding
key in the parent configuration. For example:

```yaml
Style/CollectionMethods:
  Enabled: true
  PreferredMethods:
    # No preference for collect, keep all others from default config.
    collect: ~
```

Other types, such as `AllCops` / `Include` (an array), are overridden by the
child setting.

Arrays override because if they were merged, there would be no way to
remove elements in child files.

However, advanced users can still merge arrays using the `inherit_mode` setting.
See "Merging arrays using inherit_mode" below.

### Inheriting from another configuration file in the project

The optional `inherit_from` directive is used to include configuration
from one or more files. This makes it possible to have the common
project settings in the `.rubocop.yml` file at the project root, and
then only the deviations from those rules in the subdirectories. The
files can be given with absolute paths or paths relative to the file
where they are referenced. The settings after an `inherit_from`
directive override any settings in the file(s) inherited from. When
multiple files are included, the first file in the list has the lowest
precedence and the last one has the highest. The format for multiple
inheritance is:

```yaml
inherit_from:
  - ../.rubocop.yml
  - ../conf/.rubocop.yml
```

## Inheriting configuration from a remote URL

The optional `inherit_from` directive can contain a full url to a remote
file. This makes it possible to have common project settings stored on a http
server and shared between many projects.

The remote config file is cached locally and is only updated if:

- The file does not exist.
- The file has not been updated in the last 24 hours.
- The remote copy has a newer modification time than the local copy.

You can inherit from both remote and local files in the same config and the
same inheritance rules apply to remote URLs and inheriting from local
files where the first file in the list has the lowest precedence and the
last one has the highest. The format for multiple inheritance using URLs is:

```yaml
inherit_from:
  - http://www.example.com/rubocop.yml
  - ../.rubocop.yml
```

### Inheriting configuration from a dependency gem

The optional `inherit_gem` directive is used to include configuration from
one or more gems external to the current project. This makes it possible to
inherit a shared dependency's RuboCop configuration that can be used from
multiple disparate projects.

Configurations inherited in this way will be essentially *prepended* to the
`inherit_from` directive, such that the `inherit_gem` configurations will be
loaded first, then the `inherit_from` relative file paths will be loaded
(overriding the configurations from the gems), and finally the remaining
directives in the configuration file will supersede any of the inherited
configurations. This means the configurations inherited from one or more gems
have the lowest precedence of inheritance.

The directive should be formatted as a YAML Hash using the gem name as the
key and the relative path within the gem as the value:

```yaml
inherit_gem:
  my-shared-gem: .rubocop.yml
  cucumber: conf/rubocop.yml
```

An array can also be used as the value to include multiple configuration files
from a single gem:

```yaml
inherit_gem:
  my-shared-gem:
    - default.yml
    - strict.yml
```

**Note**: If the shared dependency is declared using a [Bundler](https://bundler.io/)
Gemfile and the gem was installed using `bundle install`, it would be
necessary to also invoke RuboCop using Bundler in order to find the
dependency's installation path at runtime:

```sh
$ bundle exec rubocop <options...>
```

### Merging arrays using inherit_mode

The optional directive `inherit_mode` specifies which configuration keys that
have array values should be merged together instead of overriding the inherited
value.

This applies to explicit inheritance using `inherit_from` as well as implicit
inheritance from [the default configuration][1].

Given the following config:
```yaml
# .rubocop.yml
inherit_from:
  - shared.yml

inherit_mode:
  merge:
    - Exclude

AllCops:
  Exclude:
    - 'generated/**/*.rb'

Style/For:
  Exclude:
    - bar.rb
```

```yaml
# .shared.yml
Style/For:
  Exclude:
    - foo.rb
```

The list of `Exclude`s for the `Style/For` cop in this example will be
`['foo.rb', 'bar.rb']`. Similarly, the `AllCops:Exclude` list will contain all
the default patterns plus the `generated/**/*.rb` entry that was added locally.

The directive can also be used on individual cop configurations to override
the global setting.


```yaml
inherit_from:
  - shared.yml

inherit_mode:
  merge:
    - Exclude

Style/For:
  inherit_mode:
    override:
      - Exclude
  Exclude:
    - bar.rb
```

In this example the `Exclude` would only include `bar.rb`.


## Defaults

The file [config/default.yml][1] under the RuboCop home directory contains the
default settings that all configurations inherit from. Project and personal
`.rubocop.yml` files need only make settings that are different from the
default ones. If there is no `.rubocop.yml` file in the project, home or XDG
directories, `config/default.yml` will be used.

## Including/Excluding files

RuboCop does a recursive file search starting from the directory it is
run in, or directories given as command line arguments. Files that
match any pattern listed under `AllCops`/`Include` and extensionless
files with a hash-bang (`#!`) declaration containing one of the known
ruby interpreters listed under `AllCops`/`RubyInterpreters` are
inspected, unless the file also matches a pattern in
`AllCops`/`Exclude`. Hidden directories (i.e., directories whose names
start with a dot) are not searched by default.

Here is an example that might be used for a Rails project:

```yaml
AllCops:
  Exclude:
    - 'db/**/*'
    - 'config/**/*'
    - 'script/**/*'
    - 'bin/{rails,rake}'
    - !ruby/regexp /old_and_unused\.rb$/

# other configuration
# ...
```

**Note**: When inspecting a certain directory(or file)
given as RuboCop's command line arguments,
patterns listed under `AllCops` / `Exclude` are also inspected.
If you want to apply `AllCops` / `Exclude` rules in this circumstance,
add `--force-exclusion` to the command line argument.

Here is an example:

```yaml
# .rubocop.yml
AllCops:
  Exclude:
    - foo.rb
```

If `foo.rb` specified as a RuboCop's command line argument, the result is:

```sh
# RuboCop inspects foo.rb.
$ bundle exec rubocop foo.rb

# RuboCop does not inspect foo.rb.
$ bundle exec rubocop --force-exclusion foo.rb
```

### Path relativity

In `.rubocop.yml` and any other configuration file beginning with `.rubocop`,
files, and directories are specified relative to the directory where the
configuration file is. In configuration files that don't begin with `.rubocop`,
e.g. `our_company_defaults.yml`, paths are relative to the directory where
`rubocop` is run.

### Unusual files, that would not be included by default

RuboCop comes with a comprehensive list of common ruby file names and
extensions. But, if you'd like RuboCop to check files that are not included by
default, you'll need to pass them in on the command line, or to add entries for
them under `AllCops`/`Include`. Remember that your configuration files override
[RuboCops's defaults][1]. In the following example, we want to include
`foo.unusual_extension`, but we also must copy any other patterns we need from
the overridden `default.yml`.

```yaml
AllCops:
  Include:
    - foo.unusual_extension
    - '**/*.rb'
    - '**/*.gemfile'
    - '**/*.gemspec'
    - '**/*.rake'
    - '**/*.ru'
    - '**/Gemfile'
    - '**/Rakefile'
```

This behavior of `Include` (overriding `default.yml`) was introduced in
[0.56.0](https://github.com/rubocop-hq/rubocop/blob/master/CHANGELOG.md#0560-2018-05-14)
via [#5882](https://github.com/rubocop-hq/rubocop/pull/5882). This change allows
people to include/exclude precisely what they need to, without the defaults
getting in the way.

#### Another example, using `inherit_mode`

```yaml
inherit_mode:
  merge:
    - Include

AllCops:
  Include:
    - foo.unusual_extension
```

See "Merging arrays using inherit_mode" above.

### Deprecated patterns

Patterns that are just a file name, e.g. `Rakefile`, will match
that file name in any directory, but this pattern style is deprecated. The
correct way to match the file in any directory, including the current, is
`**/Rakefile`.

The pattern `config/**` will match any file recursively under
`config`, but this pattern style is deprecated and should be replaced by
`config/**/*`.

#### `Include` and `Exclude` are relative to their directory

The `Include` and `Exclude` parameters are special. They are
valid for the directory tree starting where they are defined. They are not
shadowed by the setting of `Include` and `Exclude` in other `.rubocop.yml`
files in subdirectories. This is different from all other parameters, who
follow RuboCop's general principle that configuration for an inspected file
is taken from the nearest `.rubocop.yml`, searching upwards. _This behavior
will be overridden if you specify the `--ignore-parent-exclusion` command line
argument_.

### Cop-specific `Include` and `Exclude`

Cops can be run only on specific sets of files when that's needed (for
instance you might want to run some Rails model checks only on files whose
paths match `app/models/*.rb`). All cops support the
`Include` param.

```yaml
Rails/HasAndBelongsToMany:
  Include:
    - app/models/*.rb
```

Cops can also exclude only specific sets of files when that's needed (for
instance you might want to run some cop only on a specific file). All cops support the
`Exclude` param.

```yaml
Rails/HasAndBelongsToMany:
  Exclude:
    - app/models/problematic.rb
```

## Generic configuration parameters

In addition to `Include` and `Exclude`, the following parameters are available
for every cop.

### Enabled

Specific cops can be disabled by setting `Enabled` to `false` for that specific cop.

```yaml
Layout/LineLength:
  Enabled: false
```

Most cops are enabled by default. Cops, introduced or significantly updated
between major versions, are in a special pending status (read more in
["Versioning"](versioning.md)). Some cops, configured the above `Enabled: false`
in [config/default.yml](https://github.com/rubocop-hq/rubocop/blob/master/config/default.yml),
are disabled by default. The cop enabling process can be altered by
setting `DisabledByDefault` or `EnabledByDefault` (but not both) to `true`.

```yaml
AllCops:
  DisabledByDefault: true
```

All cops are then disabled by default. Only cops appearing in user
configuration files are enabled. `Enabled: true` does not have to be
set for cops in user configuration. They will be enabled anyway. It is also
possible to enable entire departments by adding for example

```yaml
Style:
  Enabled: true
```

All cops are then enabled by default. Only cops explicitly disabled
using `Enabled: false` in user configuration files are disabled.

### Severity

Each cop has a default severity level based on which department it belongs
to. The level is normally `warning` for `Lint` and `convention` for all the
others, but this can be changed in user configuration. Cops can customize their
severity level. Allowed values are `refactor`, `convention`, `warning`, `error`
and `fatal`.

There is one exception from the general rule above and that is `Lint/Syntax`, a
special cop that checks for syntax errors before the other cops are invoked. It
cannot be disabled and its severity (`fatal`) cannot be changed in
configuration.

```yaml
Lint:
  Severity: error

Metrics/CyclomaticComplexity:
  Severity: warning
```

### Details

Individual cops can be embellished with extra details in offense messages:

```yaml
Layout/LineLength:
  Details: >-
    If lines are too short, text becomes hard to read because you must
    constantly jump from one line to the next while reading. If lines are too
    long, the line jumping becomes too hard because you "lose the line" while
    going back to the start of the next line. 80 characters is a good
    compromise.
```

These details will only be seen when RuboCop is run with the `--extra-details` flag or if `ExtraDetails` is set to true in your global RuboCop configuration.

### AutoCorrect

Cops that support the `--auto-correct` option can have that support
disabled. For example:

```yaml
Style/PerlBackrefs:
  AutoCorrect: false
```

## Setting the target Ruby version

Some checks are dependent on the version of the Ruby interpreter which the
inspected code must run on. For example, enforcing using Ruby 2.3+ safe
navigation operator rather than `try` can help make your code shorter and
more consistent... _unless_ it must run on Ruby 2.2.

Users may let RuboCop know the oldest version of Ruby which your project
supports with:

```yaml
AllCops:
  TargetRubyVersion: 2.2
```

Otherwise, RuboCop will then check your project for `.ruby-version` and
use the version specified by it.

## Automatically Generated Configuration

If you have a code base with an overwhelming amount of offenses, it can
be a good idea to use `rubocop --auto-gen-config`, which creates
`.rubocop_todo.yml` and adds `inherit_from: .rubocop_todo.yml` in your
`.rubocop.yml`. The generated file `.rubocop_todo.yml` contains
configuration to disable cops that currently detect an offense in the
code by changing the configuration for the cop, excluding the offending
files, or disabling the cop altogether once a file count limit has been
reached.

By adding the option `--exclude-limit COUNT`, e.g., `rubocop
--auto-gen-config --exclude-limit 5`, you can change how many files are
excluded before the cop is entirely disabled. The default COUNT is 15.

The next step is to cut and paste configuration from `.rubocop_todo.yml`
into `.rubocop.yml` for everything that you think is in line with your
(organization's) code style and not a good fit for a todo list. Pay
attention to the comments above each entry. They can reveal configuration
parameters such as `EnforcedStyle`, which can be used to modify the
behavior of a cop instead of disabling it completely.

Then you can start removing the entries in the generated
`.rubocop_todo.yml` file one by one as you work through all the offenses
in the code.

Another way of silencing offense reports, aside from configuration, is
through source code comments. These can be added manually or
automatically. See "Disabling Cops within Source Code" below.

The cops in the `Metrics` department will by default get `Max` parameters
generated in `.rubocop_todo.yml`. The value of these will be just high enough
so that no offenses are reported the next time you run `rubocop`. If you
prefer to exclude files, like for other cops, add `--auto-gen-only-exclude`
when running with `--auto-gen-config`. It will still change the maximum if the
number of excluded files is higher than the exclude limit.

## Updating the configuration file

When you update RuboCop version, sometimes you need to change `.rubocop.yml`.
If you use [mry](https://github.com/pocke/mry), you can update `.rubocop.yml`
to latest version automatically.


```sh
$ gem install mry
# Update to latest version
$ mry .rubocop.yml
# Update to specified version
$ mry --target=0.48.0 .rubocop.yml
```

See [https://github.com/pocke/mry](https://github.com/pocke/mry) for more information.

## Disabling Cops within Source Code

One or more individual cops can be disabled locally in a section of a
file by adding a comment such as

```ruby
# rubocop:disable Layout/LineLength, Style/StringLiterals
[...]
# rubocop:enable Layout/LineLength, Style/StringLiterals
```

You can also disable *all* cops with

```ruby
# rubocop:disable all
[...]
# rubocop:enable all
```

In cases where you want to differentiate intentional disabled vs disables that
you'd like to revisit later, you can use disable:todo as an alias of rubocop:disable.

```ruby
# rubocop:todo Layout/LineLength, Style/StringLiterals
[...]
# rubocop:enable Layout/LineLength, Style/StringLiterals
```

One or more cops can be disabled on a single line with an end-of-line
comment.

```ruby
for x in (0..19) # rubocop:disable Style/For
```

If you want to disable a cop that inspects comments, you can do so by
adding an "inner comment" on the comment line.

```ruby
# coding: utf-8 # rubocop:disable Style/Encoding
```

Running `rubocop --[safe-]auto-correct --disable-uncorrectable` will
create comments to disable all offenses that can't be automatically
corrected.

## Setting the style guide URL

You can specify the base URL of the style guide using `StyleGuideBaseURL`.
If specified under `AllCops`, all cops are targeted.

```yaml
AllCops:
  StyleGuideBaseURL: https://rubystyle.guide
```

`StyleGuideBaseURL` is combined with `StyleGuide` specified to the cop.

```yaml
Lint/UselessAssignment:
  StyleGuide: '#underscore-unused-vars'
```

The style guide URL is https://rubystyle.guide#underscore-unused-vars.

If specified under a specific department, it takes precedence over `AllCops`.
The following is an example of specifying `Rails` department.

```yaml
Rails:
  StyleGuideBaseURL: https://rails.rubystyle.guide
```

```yaml
Rails/TimeZone:
  StyleGuide: '#time'
```

The style guide URL is https://rails.rubystyle.guide#time.

[1]: https://github.com/rubocop-hq/rubocop/blob/master/config/default.yml
[xdg-basedir-spec]: https://specifications.freedesktop.org/basedir-spec/latest/index.html
