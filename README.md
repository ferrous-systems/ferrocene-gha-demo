# Demo: Ferrocene and its toolchain within GitHub Action(s)

This is a simplified demo.

## Prerequisites

- Access to a GitHub account where you can create repo(s).
- Access to [Ferrocene Customer Portal].
- TL;DR: You can see the [`build.yml`] file for a fully working sample for this demo project.

## Steps to use Ferrocene in your project

### Create a new project directory

We will create a brand new project that works with Ferrocene.
Create a directory where you will have your project files and `cd` into it.

```sh
mkdir ferrocene-demo
cd ferrocene-demo
```

### Add the Ferrocene project manifest to `ferrocene-demo` directory

Ferrocene comes with its own toolchain and toolchain manager. The main tool is called
[CriticalUp], which is Ferrocene's toolchain manager.

Create a file in project root: `criticalup.toml` and paste the following content in it:

```toml
manifest-version = 1

[products.ferrocene]
release = "stable-24.08.0"
packages = [
    "rustc-${rustc-host}",
    "cargo-${rustc-host}",
    "rust-std-${rustc-host}"
]
```

Some highlights about CriticalUp and the project manifest:

- This is the CriticalUp project manifest file. It is usually named `criticalup.toml`.
  CriticalUp tries to find it within your project folder or its parent directory.
  - You can override this and provide an explicit `--project` flag with a path to your `criticalup.toml` file for your project.
  - `criticalup install --project /path/to/my/manifest/criticalup.toml`.
- Used to install Ferrocene and its releases for your project, and packages and dependencies for the release (like [rustup]).
- Also used to build/run your project (like [cargo]).
- You can use `-${rustc-host}` suffix to automatically have CriticalUp fill the current architecture triple.

### Get the CriticalUp Token to authenticate

- To install any Ferrocene product/toolchain, you will need to get a token from the [Ferrocene Customer Portal].
  This token can be created in your account.
- The tokens are at the [Ferrocene CriticalUp Tokens] section of the portal.
  - Once you are on the page, click "New Token", provide a memorable title for it, and once generated copy the token.
  - The token is only shown once for security.

### Install CriticalUp

The [Installing CriticalUp](https://criticalup.ferrocene.dev/install.html) section of the [Criticalup documentation] says to run:

```shell
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/ferrocene/criticalup/releases/latest/download/criticalup-installer.sh | sh
```

> For GitHub Actions: See the step 'Make sure CriticalUp is installed and available' in [`build.yml`].

### Test CriticalUp is installed

The following command does not require a token or authentication and can tell you available subcommands for CriticalUp.

```shell
criticalup --help 
```

> For GitHub Actions: See the step 'Test if CriticalUp is installed' in [`build.yml`].

### Authenticate CriticalUp

**This section assumes you have done the following from above:**

- Get the CriticalUp Token to authenticate
- Add the CriticalUp Token to GitHub Action secrets

In the following command to authenticate, paste the token in place of `<CRITICALUP_TOKEN>`:

```sh
criticalup auth set <CRITICALUP_TOKEN>
```

**Note:** In your GitHub Action you can use the secret as:

```shell
criticalup auth set ${{ secrets.CRITICALUP_TOKEN }}
```

> For GitHub Actions: See the step 'Authenticate CriticalUp' in [`build.yml`].

### Install Ferrocene toolchain

**This step assumes you have already done the following from above:**

- Add a project manifest to your repo

Running the following command will install the toolchain listed in your project manifest (`criticalup.toml`).

```shell
criticalup install
```

> For GitHub Actions: See the step 'Install Ferrocene toolchain from the project manifest (criticalup.toml)' in [`build.yml`].

### Create new binary project

**Note:** This section is for regular x86_64 binary project. If you are doing an embedded project,
then please ignore this section and continue with the next section: "Alternatively create new embedded project".

Once the toolchain is installed, you will have `rustc`, `cargo`, and `rust-std` available in the toolchain.

We will run a command to create a new binary project:

```sh
criticalup run cargo init
```

or explicitly passing `--project` manifest:

```sh
criticalup run --project criticalup.toml cargo init
```

This will initialize a new project in the directory `ferrocene-demo`.
Note how we pass `cargo init` to the `criticalup run` command.

### Alternatively create new embedded project

**Note:** This section is an alternative to the prior section. If you are not doing an embedded, please ignore this section.

We will assume this embedded project is for ARM Cortex-M microcontrollers
and will use the [template from rust-embedded project](https://github.com/rust-embedded/cortex-m-quickstart).

#### Project manifest changes for embedded

The `criticalup.toml` needs update to be able to cross-compile for your embedded architecture.
Here's what you can use for our demo:

```toml
manifest-version = 1

[products.ferrocene]
release = "stable-24.08.0"
packages = [
    "rustc-${rustc-host}",
    "cargo-${rustc-host}",
    "rust-std-${rustc-host}",
    "rust-std-thumbv7em-none-eabi",
    "rust-std-thumbv7em-none-eabihf"
]
```

#### Generate new embedded project from template

```sh
criticalup run cargo generate --git https://github.com/rust-embedded/cortex-m-quickstart
```

A few things to note:

- The template will ask you to provide a project name.
- This will generate a project directory with the provided name, inside `ferrocene-demo` since we are using a template.
- We will use the project name: `app`.

#### Update Cargo config: `app/.cargo/config.toml`

Open file `app/.cargo/config.toml`. The template will generate a lot of comments in the file but we want
to make sure the uncommented content matches the following:

```toml
[target.'cfg(all(target_arch = "arm", target_os = "none"))']
rustflags = []

[build]
target = "thumbv7em-none-eabihf"     # Cortex-M4F and Cortex-M7F (with FPU)
```

#### Update the memory layout: `app/memory.x`

Open file `app/memory.x` file and paste the following content:

```
MEMORY
{
  /* NOTE 1 K = 1 KiBi = 1024 bytes */
  FLASH : ORIGIN = 0x08000000, LENGTH = 256K
  RAM : ORIGIN = 0x20000000, LENGTH = 40K
}
```

### Build your app using CriticalUp

The following command uses installed Ferrocene:

```shell
criticalup run cargo build --release
```

As you can see, you can pass `cargo` as a subcommand. Also, note that the system recognizes
the architecture you want to build for (in case of embedded) and cross-compiles for you. You
can check this by simple using the `file` command on Linux:

```sh
$ file target/thumbv7em-none-eabihf/release/app

target/thumbv7em-none-eabihf/release/app: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, with debug_info, not stripped
```

### Run your app using CriticalUp

The following command uses installed Ferrocene:

```shell
criticalup run cargo run --release
```

> For GitHub Actions: See the step 'Run my app via Ferrocene and its toolchain' in [`build.yml`].

## GitHub settings for Actions

### Push your project to GitHub

First, log into github.com and create a new project repo and copy the remote URL.
The URL will be in the format of `git remote add origin https://github.com/OWNER/REPOSITORY.git`.

Git related commands are beyond the scope of this document but the steps are:

- Initialize your project as a Git repo: `git init`.
- Add and commit all the files to Git: `git -am 'my awesome project'`.
- Add the remote (from above): `git remote add origin https://github.com/OWNER/REPOSITORY.git`.
- Push the code: `git push`.

### Add the CriticalUp Token to GitHub Action secrets

The CriticalUp Token you got from the [Ferrocene Customer Portal] must be set in your GitHub repo's Settings.

- Go to the 'Settings' tab of your GitHub repo.
- Click 'Secrets and variables'.
- Click 'Actions'.
- Click 'New repository secret'.
- Add Name as `CRITICALUP_TOKEN` and past the token from [Ferrocene Customer Portal] in the 'Secret' text area.

In your GitHub Action, now you can use the secret as:

```shell
criticalup auth set ${{ secrets.CRITICALUP_TOKEN }}
```

### Create a simple GitHub Action

An example of a fully working Github CI workflow file can be found in the workflow file [`build.yml`] of this demo project.

- When no workflow file in your project exists, copy the `build.yml` into the folder `.github/workflows`, otherwise
  copy the CI job `install-criticalup-build-run-my-app` into your existing workflow file.
- Adapt the workflow if necessary, for example, to compile the project instead of run it.
- We will use a single job so we don't need to cache anything. The job consists of multiple steps.
- We will showcase only Ubuntu 20.04 in this exercise.

## References

- [Ferrocene documentation]
- [Ferrocene Customer Portal]
- [CriticalUp]
- [CriticalUp documentation]
- [GitHub Actions documentation](https://docs.github.com/en/actions)
- [rustup]
- [cargo]

[CriticalUp]: https://github.com/ferrocene/criticalup
[CriticalUp documentation]: https://criticalup.ferrocene.dev/index.html
[Ferrocene Customer Portal]: https://customers.ferrocene.dev
[Ferrocene CriticalUp Tokens]: https://customers.ferrocene.dev/users/tokens
[Ferrocene documentation]: https://public-docs.ferrocene.dev/main/index.html
[rustup]: https://rustup.rs/
[cargo]: https://doc.rust-lang.org/cargo/
[`build.yml`]: ./.github/workflows/build.yml
