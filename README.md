# boot-tools-deps

A Boot task that uses `tools.deps(.alpha)` to read in `deps.edn` files in the same way that the `clj` script uses them. Updates Boot's resources, sources, and classpath based on the paths, extra paths, and classpath computed by `tools.deps`. Can also update Boot's dependencies (for use by tasks like `uber`).

[![Clojars Project](https://img.shields.io/clojars/v/seancorfield/boot-tools-deps.svg)](https://clojars.org/seancorfield/boot-tools-deps)

## Usage

You can either add this to your `build.boot` (or `profile.boot`) file's `:dependencies`:

    [seancorfield/boot-tools-deps "0.2.1"]

and then expose the task with:

    (require '[boot-tools-deps.core :refer [deps]])

or you can just add it as command line dependency:

    boot -d seancorfield/boot-tools-deps:0.2.1 ...

The available arguments are:

* `-c` `--config-files` -- specify the `deps.edn` files to be used
* `-D` `--config-data` -- provide an EDN string that is treated as an additional, final `deps.edn` file
* `-C` `--classpath-aliases` -- specify the aliases for classpath additions
* `-R` `--resolve-aliases` -- specify the aliases for resolving dependencies
* `-A` `--aliases` -- shorthand for specifying `-R` and `-C` with the same alias
* `-B` `--overwrite-boot-deps` -- in addition to setting up the classpath (and `:resource-paths` and `source-paths`), overwrite Boot's `:dependencies` with those returned from `tools.deps` -- note: this is only required for Boot tasks such as `uber` to function correctly!
* `-r` `--repeatable` -- use only the local `deps.edn` file (or the `-c` specified files) -- note: the `-D` option is still read and used!
* `-v` `--verbose` -- explain what the task is doing (`-vv` also makes `tools.deps` verbose)

### Specifying aliases for tasks in `build.boot`

You can specify different classpaths for different tasks. For example, to use the default classpath for the `build` task:

    (deftask build
      "Build and install the project locally."
      []
      (comp (deps) (pom :project 'foo/bar :version "0.1.0" ) (jar :main 'clojure.main) (install)))

And to add the `:test` alias when testing:

    (require '[adzerk.boot-test :as boot-test])

    (deftask test
      "Runs tests"
      []
      (comp (deps :aliases [:test])
            (boot-test/test)))

## Differences from how `clj` works

* The "system" `deps.edn` file is not read but `boot-tools-deps` merges in a copy taken from the `clojure/brew-install` repository so the effect should be the same. _This means the default `deps.edn` information may lag behind the latest, distributed/installed version, or may be ahead of the version you actually have installed (if you have not updated `clojure` recently). The version of Clojure used is whatever version is running by the time this task is run -- you can only change that via `BOOT_CLOJURE_VERSION` or `~/.boot/boot.properties`, not via `deps.edn`._
* `clj` computes the full classpath and caches it in a local file. _`boot-tools-deps` computes the classpath on every invocation._
* Whatever value of `:paths` comes back from `tools.deps` is used as the `:resource-paths` value for Boot.
* Whatever value of `:extra-paths` comes back from `tools.deps` is used as the base `:source-paths` value for Boot. You can use Boot's `sift` task to treat them as resources instead.
* Any additional folders found on the computed classpath produced by `tools.deps` are added to the `:source-paths` and any JAR files on the computed classpath are added directly to Boot's in-memory classpath. _Boot's `:dependencies` are not updated by default._
* In order to run tasks that depend on Boot's `:dependencies`, such as `uber`, you need to specify the `-B` option to overwrite Boot's `:dependencies` with the computed dependencies produced by `tools.deps`. _Note: transitive dependencies do not inherit the `:scope` of the dependency that caused them to be included!_

## Changes

* 0.2.1 -- 01/29/2018 -- Make Clojure a `:provided` dependency for consistency.`
* 0.2.0 -- 01/28/2018 -- Update to use the most recent `tools.deps.alpha` release; directly update the Boot classpath (which means Git and Local dependencies are now supported!); run `tools.deps` inside a Boot pod; no longer update Boot's `:dependencies` by default.
* 0.1.4 -- 12/06/2017 -- Fix #3 by updating `deps.edn` template from `brew-install` (changes Clojars repo URL); fix #4 by correcting how `-r` and `-c` options affect the list of `deps.edn` files used; switches from `HOME` environment variable to `user.home` system property; adds `-A` option for when you need the same alias on both `-R` and `-C`; now relies on `tools.deps.alpha.makecp` loading all the specific providers (instead of loading them manually).
* 0.1.3 -- 11/15/2017 -- Fix #2 by using `deps.edn` template from `brew-install` repo as defaults.
* 0.1.2 -- 11/13/2017 -- Expose `deps` task machinery as a function, `load-deps`, for more flexibility.
* 0.1.1 -- 11/12/2017 -- First working version.

## License

Copyright © 2017-2018 Sean Corfield, all rights reserved.

Distributed under the Eclipse Public License version 1.0.
