# Lefthook installation reference

Lefthook distributes as a standalone, no-deps binary. There are multiple ways to install lefthook but the most common is via package manager for your programming language.

You can also download just the [binary](https://github.com/evilmartians/lefthook/releases/latest) for your OS and architecture and put it somewhere in your `$PATH` and update it with

```sh
lefthook self-update
```

The sections below cover every supported installation method.

## Homebrew

Homebrew for MacOS and Linux:

```bash
brew install lefthook
```

## Snap

Snap for Linux:

```sh
snap install --classic lefthook
```

## Scoop

Scoop for Windows:

```sh
scoop install lefthook
```

## Winget

Winget for Windows:

```sh
winget install evilmartians.lefthook
```

## Debian-based (APT)

APT packages for Debian/Ubuntu Linux:

```sh
curl -1sLf 'https://dl.cloudsmith.io/public/evilmartians/lefthook/setup.deb.sh' | sudo -E bash
sudo apt install lefthook
```

See all instructions: `https://cloudsmith.io/~evilmartians/repos/lefthook/setup/#formats-deb`

> Debian package repository hosting is graciously provided by [Cloudsmith](https://cloudsmith.com).

## RPM-based

RPM packages for CentOS/Fedora Linux:

```sh
curl -1sLf 'https://dl.cloudsmith.io/public/evilmartians/lefthook/setup.rpm.sh' | sudo -E bash
sudo yum install lefthook
```

See all instructions: `https://cloudsmith.io/~evilmartians/repos/lefthook/setup/#repository-setup-yum`

> RPM package repository hosting is graciously provided by [Cloudsmith](https://cloudsmith.com).

## Alpine (APK)

APK packages for Alpine:

```sh
sudo apk add --no-cache bash curl
curl -1sLf 'https://dl.cloudsmith.io/public/evilmartians/lefthook/setup.alpine.sh' | sudo -E bash
sudo apk add lefthook
```

See all instructions: `https://cloudsmith.io/~evilmartians/repos/lefthook/setup/#formats-alpine`

> APK package repository hosting is graciously provided by [Cloudsmith](https://cloudsmith.com).

## Arch Linux (AUR)

AUR for Arch:

- Official [AUR package](https://aur.archlinux.org/packages/lefthook) (compiles from sources)
- Community [AUR package](https://aur.archlinux.org/packages/lefthook-bin) (delivers pre-compiled binaries)

```sh
# To compile from sources
yay -S lefthook

# To install only executable
yay -S lefthook-bin
```

## Devbox

Add lefthook in the devbox environment.
lefthook already exists in the [Nix package](https://search.nixos.org/packages?channel=25.05&show=lefthook&from=0&size=50&sort=relevance&type=packages&query=lefthook)

```bash
devbox add lefthook@latest
```

> **Note:** The devbox plugin for lefthook is maintained by the community. While we appreciate their contribution, the lefthook team cannot provide direct support for devbox-specific installation issues.

## Mise

> See [https://github.com/jdx/mise](https://github.com/jdx/mise)

```bash
mise use lefthook@latest
```

> **Note:** The mise plugin for lefthook is maintained by the community. While we appreciate their contribution, the lefthook team cannot provide direct support for mise-specific installation issues.

## npm (Node)

NPM package:

```bash
npm install --save-dev lefthook
```

```bash
yarn add --dev lefthook
```

```bash
pnpm add -D lefthook
```

> **Note:** If you use `pnpm` package manager make sure to update `pnpm-workspace.yaml`s
> `onlyBuiltDependencies` with `lefthook` and add `lefthook` to `pnpm.onlyBuiltDependencies`
> in your root `package.json`, otherwise the `postinstall` script of the `lefthook` package
> won't be executed and hooks won't be installed.

### Choose right package

Lefthook supports three NPM packages with different ways to deliver the executables

1. [lefthook](https://www.npmjs.com/package/lefthook) installs one executable for your system

   ```bash
   npm install --save-dev lefthook
   ```

2. **legacy**[^1] [@evilmartians/lefthook](https://www.npmjs.com/package/@evilmartians/lefthook) installs executables for all OS

   ```bash
   npm install --save-dev @evilmartians/lefthook
   ```

3. **legacy**[^1] [@evilmartians/lefthook-installer](https://www.npmjs.com/package/@evilmartians/lefthook-installer) fetches the right executable on installation

   ```bash
   npm install --save-dev @evilmartians/lefthook-installer
   ```

[^1]: Legacy distributions are still maintained but they will be shut down in the future.

## Python

```sh
python -m pip install --user lefthook
```

```sh
uv add --dev lefthook
```

```sh
pipx install lefthook
```

## Ruby

```ruby
# Gemfile

group :development do
  gem "lefthook", require: false
end
```

Or globally

```bash
gem install lefthook
```

### Troubleshooting

If you see the error `lefthook: command not found` you need to check your $PATH. Also try to restart your terminal.

## Go

The minimum Go version required is 1.26 and you can install

- as global package

```bash
go install github.com/evilmartians/lefthook/v2@v2.1.9
```

- or as a go tool in your project

```bash
go get -tool github.com/evilmartians/lefthook/v2
```

## Swift

You can find the [Swift wrapper plugin](https://github.com/csjones/lefthook-plugin) on GitHub.

Utilize lefthook in your Swift project using Swift Package Manager:

```swift
.package(url: "https://github.com/csjones/lefthook-plugin.git", exact: "2.1.9"),
```

Or, with [mint](https://github.com/yonaskolb/Mint):

```bash
mint run csjones/lefthook-plugin
```

## Manual (prebuilt executable)

Manual installation with prebuilt executable.

Download binaries from [latest release](https://github.com/evilmartians/lefthook/releases/latest) and install manually.
