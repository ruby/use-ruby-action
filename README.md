# setup-ruby

This action downloads a prebuilt ruby and adds it to the `PATH`.

It is very efficient and takes about 5 seconds to download, extract and add the given Ruby to the `PATH`.
No extra packages need to be installed.

Compared to [actions/setup-ruby](https://github.com/actions/setup-ruby),
this actions supports many more versions and features.

### Supported Versions

This action currently supports these versions of MRI, JRuby and TruffleRuby:

| Interpreter | Versions |
| ----------- | -------- |
| `ruby` | 1.9.3, 2.0.0, 2.1.9, 2.2, all versions from 2.3.0 until 3.1.0-preview1, head, debug, mingw, mswin |
| `jruby` | 9.1.17.0 - 9.3.1.0, head |
| `truffleruby` | 19.3.0 - 21.3.0, head |
| `truffleruby+graalvm` | 21.2.0 - 21.3.0, head |

`ruby-debug` is the same as `ruby-head` but with assertions enabled (`-DRUBY_DEBUG=1`).  
On Windows, `mingw` and `mswin` are `ruby-head` builds using the MSYS2/MinGW and the MSVC toolchains respectively.

Preview and RC versions of Ruby might be available too on Ubuntu and macOS (not on Windows).
However, it is recommended to test against `ruby-head` rather than previews,
as it provides more useful feedback for the Ruby core team and for upcoming changes.

Only versions published by [RubyInstaller](https://rubyinstaller.org/downloads/archives/) are available on Windows.
Due to that, Ruby 2.2 resolves to 2.2.6 on Windows and 2.2.10 on other platforms.
And Ruby 2.3 on Windows only has builds for 2.3.0, 2.3.1 and 2.3.3.

Note that Ruby ≤ 2.4 and the OpenSSL version it needs (1.0.2) are both end-of-life,
which means Ruby ≤ 2.4 is unmaintained and considered insecure.

### Supported Platforms

The action works for all [GitHub-hosted runners](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/virtual-environments-for-github-hosted-runners).

| Operating System | Recommended | Other Supported Versions |
| ----------- | -------- | -------- |
| Ubuntu  | `ubuntu-latest`  (= `ubuntu-20.04`) | `ubuntu-18.04`, `ubuntu-16.04` |
| macOS   | `macos-latest`   (= `macos-10.15`)  | `macos-11.0` |
| Windows | `windows-latest` (= `windows-2019`) | `windows-2016` |

The prebuilt releases are generated by [ruby-builder](https://github.com/ruby/ruby-builder)
and on Windows by [RubyInstaller2](https://github.com/oneclick/rubyinstaller2).
`mingw` and `mswin` builds are generated by [ruby-loco](https://github.com/MSP-Greg/ruby-loco).
`ruby-head` is generated by [ruby-dev-builder](https://github.com/ruby/ruby-dev-builder),
`jruby-head` is generated by [jruby-dev-builder](https://github.com/ruby/jruby-dev-builder),
`truffleruby-head` is generated by [truffleruby-dev-builder](https://github.com/ruby/truffleruby-dev-builder)
and `truffleruby+graalvm` is generated by [graalvm-ce-dev-builds](https://github.com/graalvm/graalvm-ce-dev-builds).
The full list of available Ruby versions can be seen in [ruby-builder-versions.js](ruby-builder-versions.js)
for Ubuntu and macOS and in [windows-versions.js](windows-versions.js) for Windows.

## Usage

### Single Job

```yaml
name: My workflow
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: ruby/setup-ruby@v1
      with:
        ruby-version: 2.6 # Not needed with a .ruby-version file
        bundler-cache: true # runs 'bundle install' and caches installed gems automatically
    - run: bundle exec rake
```

### Matrix of Ruby Versions

This matrix tests all stable releases and `head` versions of MRI, JRuby and TruffleRuby on Ubuntu and macOS.

```yaml
name: My workflow
on: [push, pull_request]
jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest]
        # Due to https://github.com/actions/runner/issues/849, we have to use quotes for '3.0'
        ruby: [2.5, 2.6, 2.7, '3.0', head, jruby, jruby-head, truffleruby, truffleruby-head]
    runs-on: ${{ matrix.os }}
    steps:
    - uses: actions/checkout@v2
    - uses: ruby/setup-ruby@v1
      with:
        ruby-version: ${{ matrix.ruby }}
        bundler-cache: true # runs 'bundle install' and caches installed gems automatically
    - run: bundle exec rake
```

### Matrix of Gemfiles

```yaml
name: My workflow
on: [push, pull_request]
jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        gemfile: [ rails5, rails6 ]
    runs-on: ubuntu-latest
    env: # $BUNDLE_GEMFILE must be set at the job level, so it is set for all steps
      BUNDLE_GEMFILE: ${{ github.workspace }}/gemfiles/${{ matrix.gemfile }}.gemfile
    steps:
      - uses: actions/checkout@v2
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: 2.6
          bundler-cache: true # runs 'bundle install' and caches installed gems automatically
      - run: bundle exec rake
```

See the GitHub Actions documentation for more details about the
[workflow syntax](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions)
and the [condition and expression syntax](https://help.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions).

### Supported Version Syntax

* engine-version like `ruby-2.6.5` and `truffleruby-19.3.0`
* short version like `2.6`, automatically using the latest release matching that version (`2.6.5`)
* version only like `2.6.5`, assumes MRI for the engine
* engine only like `truffleruby`, uses the latest stable release of that implementation
* `.ruby-version` reads from the project's `.ruby-version` file
* `.tool-versions` reads from the project's `.tool-versions` file
* If the `ruby-version` input is not specified, `.ruby-version` is tried first, followed by `.tool-versions`

### Working Directory

The `working-directory` input can be set to resolve `.ruby-version`, `.tool-versions` and `Gemfile.lock`
if they are not at the root of the repository, see [action.yml](action.yml) for details.

### Bundler

By default, if there is a `Gemfile.lock` file (or `$BUNDLE_GEMFILE.lock` or `gems.locked`) with a `BUNDLED WITH` section,
that version of Bundler will be installed and used.
Otherwise, the latest compatible Bundler version is installed (Bundler 2 on Ruby >= 2.4, Bundler 1 on Ruby < 2.4).

This behavior can be customized, see [action.yml](action.yml) for more details about the `bundler` input.

### Caching `bundle install` automatically

This action provides a way to automatically run `bundle install` and cache the result:
```yaml
    - uses: ruby/setup-ruby@v1
      with:
        bundler-cache: true
```

Note that any step doing `bundle install` (for the root `Gemfile`) or `gem install bundler` can be removed with `bundler-cache: true`.

This caching speeds up installing gems significantly and avoids too many requests to RubyGems.org.  
It needs a `Gemfile` (or `$BUNDLE_GEMFILE` or `gems.rb`) under the [`working-directory`](#working-directory).  
If there is a `Gemfile.lock` (or `$BUNDLE_GEMFILE.lock` or `gems.locked`), `bundle config --local deployment true` is used.

To use a `Gemfile` which is not at the root or has a different name, set `BUNDLE_GEMFILE` in the `env` at the job level
as shown in the [example](#matrix-of-gemfiles).

#### bundle config

When using `bundler-cache: true` you might notice there is no good place to run `bundle config ...` commands.
These can be replaced by `BUNDLE_*` environment variables, which are also faster.
They should be set in the `env` at the job level as shown in the [example](#matrix-of-gemfiles).
To find the correct the environment variable name, see the [Bundler docs](https://bundler.io/man/bundle-config.1.html) or look at `.bundle/config` after running `bundle config --local KEY VALUE` locally.
You might need to `"`-quote the environment variable name in YAML if it has unusual characters like `/`.

To perform caching, this action will use `bundle config --local path $PWD/vendor/bundle`.  
Therefore, the Bundler `path` should not be changed in your workflow for the cache to work (no `bundle config path`).

#### How it Works

When there is no lockfile, one is generated with `bundle lock`, which is the same as `bundle install` would do first before actually fetching any gem.
In other words, it works exactly like `bundle install`.
The hash of the generated lockfile is then used for caching, which is the only correct approach.

#### Dealing with a corrupted cache

In some rare scenarios (like using gems with C extensions whose functionality depends on libraries found on the system
at the time of the gem's build) it may be necessary to ignore contents of the cache and get and build all the gems anew.
In order to achieve this, set the `cache-version` option to any value other than `0` (or change it to a new unique value
if you have already used it before.)

```yaml
    - uses: ruby/setup-ruby@v1
      with:
        bundler-cache: true
        cache-version: 1
```

#### Caching `bundle install` manually

It is also possible to cache gems manually, but this is not recommended because it is verbose and *very difficult* to do correctly.
There are many concerns which means using `actions/cache` is never enough for caching gems (e.g., incomplete cache key, cleaning old gems when restoring from another key, correctly hashing the lockfile if not checked in, OS versions, ABI compatibility for `ruby-head`, etc).
So, please use `bundler-cache: true` instead and report any issue.

## Windows

Note that running CI on Windows can be quite challenging if you are not very familiar with Windows.
It is recommended to first get your build working on Ubuntu and macOS before trying Windows.

* Use Bundler 2.2.18+ on Windows (older versions have [bugs](https://github.com/ruby/setup-ruby/issues/209#issuecomment-889064123)) by not setting the `bundler:` input and ensuring there is no `BUNDLED WITH 1.x.y` in a checked-in `Gemfile.lock`.
* The default shell on Windows is not Bash but [PowerShell](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions#using-a-specific-shell).
  This can lead issues such as multi-line scripts [not working as expected](https://github.com/ruby/setup-ruby/issues/13).
* The `PATH` contains [multiple compiler toolchains](https://github.com/ruby/setup-ruby/issues/19). Use `where.exe` to debug which tool is used.
* For Ruby ≥ 2.4, MSYS2 is prepended to the `Path`, similar to what RubyInstaller2 does.
* For Ruby < 2.4, the DevKit MSYS tools are installed and prepended to the `Path`.
* JRuby on Windows has a known bug that `bundle exec rake` [fails](https://github.com/ruby/setup-ruby/issues/18).

## Versioning

It is highly recommended to use `ruby/setup-ruby@v1` for the version of this action.
This will provide the best experience by automatically getting bug fixes, new Ruby versions and new features.

If you instead choose a specific version (v1.2.3) or a commit sha, there will be no automatic bug fixes and
it will be your responsibility to update every time the action no longer works.
Make sure to always use the latest release before reporting an issue on GitHub.

This action follows semantic versioning with a moving `v1` branch.
This follows the [recommendations](https://github.com/actions/toolkit/blob/master/docs/action-versioning.md) of GitHub Actions.

## Using self-hosted runners

This action might work with [self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)
if the [virtual environment](https://github.com/actions/virtual-environments) is very similar to the ones used by GitHub runners. Notably:

* Make sure to use the same operating system and version.
* Set the environment variable `ImageOS` on the runner to the corresponding value on GitHub-hosted runners (e.g. `ubuntu18`/`macos1015`/`win19`). This is necessary to detect the operating system and version.
* Make sure to use the same version of libssl.
* Make sure that the operating system has `libyaml-0` installed
* The default tool cache directory (`/opt/hostedtoolcache` on Linux, `/Users/runner/hostedtoolcache` on macOS,
  `C:/hostedtoolcache/windows` on Windows) must be writable by the `runner` user.
  This is necessary since the Ruby builds embed the install path when built and cannot be moved around.
* `/home/runner` must be writable by the `runner` user.

In other cases, please use a system Ruby or [install Ruby manually](https://github.com/postmodern/chruby/wiki#installing-rubies) instead.

On a self-hosted runner you need to define the `ImageOs` as an evironment variable on the host, you can do this in the `~/actions-runner/.env` file (See [#230](https://github.com/ruby/setup-ruby/issues/230)).

## History

This action used to be at `eregon/use-ruby-action` and was moved to the `ruby` organization.
Please [update](https://github.com/ruby/setup-ruby/releases/tag/v1.13.0) if you are using `eregon/use-ruby-action`.

## Credits

The current maintainer of this action is @eregon.
Most of the Windows logic is based on work by MSP-Greg.
Many thanks to MSP-Greg and Lars Kanis for the help with Ruby Installer.
