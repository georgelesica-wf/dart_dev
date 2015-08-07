# Dart Dev Tools

> Centralized tooling for Dart projects. Consistent interface across projects. Easily configurable.

- [**Motivation**](#motivation)
- [**Supported Tasks**](#supported-tasks)
- [**Getting Started**](#getting-started)
- [**Project Configuration**](#project-configuration)
- [**CLI Usage**](#cli-usage)
- [**Programmatic Usage**](#programmatic-usage)

## Motivation

All Dart (https://dartlang.org) projects eventually share a common set of development requirements:

- Tests (unit, integration, and functional)
- Consistent code formatting
- Static analysis to detect issues
- Examples for manual testing/exploration

Together, the Dart SDK and a couple of packages from the Dart team supply the necessary tooling to support the above
requirements. But, the usage is inconsistent, configuration is limited to command-line arguments, and you inevitably end
up with a slew of shell scripts in the `tool/` directory. While this works, it lacks a consistent usage pattern across
multiple projects and requires an unnecessary amount of error-prone work to set up.

This package improves on the above process by providing a number of benefits:

#### Centralized Tooling
By housing the APIs and CLIs for these various dev workflows in a single location, you no longer have to worry about
keeping scripts in parity across multiple projects. Simply add the `dart_dev` package as a dependency, and you're ready
to go.

#### Versioned Tooling
Any breaking changes to the APIs or CLIs within this package will be reflected by an appropriate version bump according
to semver. You can safely upgrade your tooling to take advantage of continuous improvements and new features with
minimal maintenance.

#### Separation of Concerns
Every task supported in `dart_dev` is separated into three pieces:

1. API - programmatic execution via Dart code.
2. CLI - script-based execution via the `dart_dev` executable.
3. Configuration - singleton configuration instances for simple per-project configuration.

#### Consistent Interface
By providing a single executable (`dart_dev`) that supports multiple tasks with standardized options, project developers
have a consistent interface for development across all projects that utilize this package. Configuration is handled on a
per-project basis via a single Dart file, meaning that you don't have to know anything about a project to run tests or
static analysis - you just need to know how to use the `dart_dev` tool.


> **Note:** This is __not__ a replacement for the tooling provided by the Dart SDK and packages like `test` or
`dart_style`. Rather, `dart_dev` is a unified interface for interacting with said tooling in a simplified manner.


## Supported Tasks

- **Tests:** runs test suites (unit, integration, and functional) via the [`test` package test runner](https://github.com/dart-lang/test).
- **Code Formatting:** runs the [`dartfmt` tool from the `dart_style` package](https://github.com/dart-lang/dart_style) over source code.
- **Static Analysis:** runs the [`dartanalyzer`](https://www.dartlang.org/tools/analyzer/) over source code.
- **Serving Examples:** uses [`pub serve`](https://www.dartlang.org/tools/pub/cmd/pub-serve.html) to serve the project examples.


## Getting Started

###### Install `dart_dev`
Add the following to your `pubspec.yaml`:
```yaml
dev_dependencies:
  dart_dev: any
```

###### Create an Alias (optional)
Add the following to your bash or zsh profile for convenience:
```
alias ddev='pub run dart_dev'
```

###### Configuration
In order to configure `dart_dev` for a specific project, run `ddev init` or `pub run dart_dev init` to generate the
configuration file. This should create a `tool/dev.dart` file where each task can be configured as needed.

```dart
import 'package:dart_dev/dart_dev.dart';

main(args) async {
  // Define the entry points for static analysis.
  config.analyze.entryPoints = ['lib/', 'test/', 'tool/'];
  
  // Configure the port on which examples should be served.
  config.examples.port = 9000;
  
  // Define the directories to include when running the
  // Dart formatter.
  config.format.directories = ['lib/', 'test/', 'tool/'];
  
  // Define the location of your test suites.
  config.test
    ..unitTests = ['test/unit/']
    ..integrationTests = ['test/integration/'];

  // Execute the dart_dev tooling.
  await dev(args);
}
```

[Full list of configuration options](#project-configuration).


###### Try It Out
The tooling in `dart_dev` works out of the box with happy defaults for each task. Run `ddev` or `pub run dart_dev` to
see the help usage. Try it out by running any of the following tasks:

```
# with the alias
ddev analyze
ddev examples
ddev format
ddev test

# without the alias
pub run dart_dev analyze
pub run dart_dev examples
pub run dart_dev format
pub run dart_dev test
```

Add the `-h` flag to any of the above commands to receive additional help information specific to that task.


## Project Configuration
Project configuration occurs in the `tool/dev.dart` file where the `config` instance is imported from the `dart_dev`
package. The bare minimum for this file is:

```dart
import 'package:dart_dev/dart_dev.dart';

main(args) async {
  // Available config objects:
  //   config.analyze
  //   config.examples
  //   config.format
  //   config.init
  //   config.test
  
  await dev(args);
}
```

### `analyze` Config
All configuration options for the `analyze` task are found on the `config.analyze` object.

Name            | Type           | Default    | Description
--------------- | -------------- | ---------- | -----------
`entryPoints`   | `List<String>` | `['lib/']` | Entry points to analyze. Items in this list can be directories and/or files. Directories will be expanded (depth=1) to find Dart files.
`fatalWarnings` | `bool`         | `true`     | Treat non-type warnings as fatal.
`hints`         | `bool`         | `true`     | Show hint results.

### `examples` Config
All configuration options for the `examples` task are found on the `config.examples` object.

Name       | Type     | Default       | Description
---------- | -------- | ------------- | -----------
`hostname` | `String` | `'localhost'` | The host name to listen on.
`port`     | `int`    | `8080`        | The base port to listen on.

### `format` Config
All configuration options for the `format` task are found on the `config.format` object.

Name          | Type           | Default     | Description
------------- | -------------- | ----------- | -----------
`check`       | `bool`         | `false`     | Dry-run; checks if formatter needs to be run and sets exit code accordingly.
`directories` | `List<String>` | `['lib/']`  | Directories to run the formatter on. All files (any depth) in the given directories will be formatted.

### `test` Config
All configuration options for the `test` task are found on the `config.test` object.

Name               | Type           | Default     | Description
------------------ | -------------- | ----------- | -----------
`integrationTests` | `List<String>` | `[]`        | Integration test locations. Items in this list can be directories and/or files.
`platforms`        | `List<String>` | `[]`        | Platforms on which to run the tests (handled by the Dart test runner). See https://github.com/dart-lang/test#platform-selector-syntax for a full list of supported platforms.
`unitTests`        | `List<String>` | `['test/']` | Unit test locations. Items in this list can be directories and/or files.


## CLI Usage
This package comes with a single executable: `dart_dev`. To run this executable: `ddev` or `pub run dart_dev`. This
usage will simply display the usage help text along with a list of supported tasks:

```
$ ddev
Standardized tooling for Dart projects.

Usage: pub run dart_dev [task] [options]

    --[no-]color    Colorize the output.
                    (defaults to on)

-h, --help          Shows this usage.
-q, --quiet         Minimizes the logging output.
    --version       Shows the dart_dev package version.

Supported tasks:

    analyze
    examples
    format
    init
    test
```

- Static analysis: `ddev analyze`
- Serving examples: `ddev examples`
- Dart formatter: `ddev format`
- Initialization: `ddev init`
- Tests: `ddev test`

Add the `-h` flag to any of the above commands to see task-specific flags and options.

> Any project configuration defined in the `tool/dev.dart` file should be reflected in the execution of the above
commands. CLI flags and options will override said configuration.


## Programmatic Usage
The tooling facilitated by this package can also be executed via a programmatic Dart API:

```dart
import 'package:dart_dev/api.dart' as api;

main() async {
  await api.analyze();
  await api.serveExamples();
  await api.format();
  await api.init();
  await api.test();
}
```

Check out the source of these API methods for additional documentation.

> In order to provide a clean API, these methods do not leverage the configuration instances that the command-line
interfaces do. Because of this, the default usage may be different. You can access said configurations from the main
`package:dart_dev/dart_dev.dart` import.