# Demo: Ferrocene and its toolchain within GitHub Action(s)

This is a very simple demo.

## Prerequisites

- Access to a GitHub account where you can create repo(s).
- Access to [Ferrocene Customer Portal].
- TL;DR: You can see the [`build.yml`] file for a fully working sample for this demo project.

## Steps to use Ferrocene in your project

Note: This section will assume that you have access to GitHub and already have a Rust project you want to use Ferrocene for.

### Add a project manifest to your repo

Ferrocene comes with its own toolchain and toolchain manager. Main tool is called [CriticalUp], which is Ferrocene's toolchain manager. 

Create a file in project root: `criticalup.toml` and paste the following content in it:

```toml
manifest-version = 1

[products.ferrocene]
release = "stable-24.05.0"
packages = [
    "rustc-${rustc-host}",
    "cargo-${rustc-host}",
    "rust-std-${rustc-host}"
]
```

Some highlights about CriticalUp and the project manifest:

- This is CritalUp project manifest file. It is usually named `criticalup.toml`. CriticalUp tries to find it within your project folder or its parent directory.
  - You can override this and provide an explicit `--project` flag with path to your own `criticalup.toml` file for your project.
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

### Add the CriticalUp Token to GitHub Action secrets

The CriticalUp Token you got from the [Ferrocene Customer Portal] must be set in your GitHub repo's Settings.

- Go to 'Settings' tab of your GitHub repo.
- Click 'Secrets and variables'.
- Click 'Actions'.
- Click 'New repository secret'.
- Add Name as `CRITICALUP_TOKEN` and past the token from [Ferrocene Customer Portal] in the 'Secret' text area.

### Create a simple GitHub Action

An example of a fully working Github CI workflow file can be found in the workflow file [`build.yml`] of this demo project.

- When no workflow file in your project exists, copy the `build.yml` into the folder `.github/workflows`, otherwise
  copy the CI job `install-criticalup-build-run-my-app` into your existing workflow file.
- Adapt the workflow if necessary, for example to compile the project instead of run it.
- We will use a single job so we don't need to cache anything. The job consists of multiple steps.
- We will showcase only Ubuntu 20.04 in this exercise.

#### Install CriticalUp

The [Installing CriticalUp](https://criticalup.ferrocene.dev/install.html) section of the [Criticalup documentation] says to run:

```shell
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/ferrocene/criticalup/releases/latest/download/criticalup-installer.sh | sh
```

> See the step 'Make sure CriticalUp is installed and available' in [`build.yml`].

#### Test CriticalUp is installed

The following command does not require a token or authentication and can tell you available subcommands for CriticalUp.

```shell
criticalup --help 
```

> See the step 'Test if CriticalUp is installed' in [`build.yml`].

#### Authenticate CriticalUp

_This section assumes you have done the following from above:

- Get the CriticalUp Token to authenticate
- Add the CriticalUp Token to GitHub Action secrets_

In your GitHub Action you can use the secret now as:

```shell
criticalup auth set ${{ secrets.CRITICALUP_TOKEN }}
```

> See the step 'Authenticate CriticalUp' in [`build.yml`].

#### Install Ferrocene toolchain

_This step assumes you have already done the following from above:

- Add a project manifest to your repo_

Just running the following command will install the toolchain listed in your project manifest (`criticalup.toml`).

```shell
criticalup install
```

> See the step 'Install Ferrocene toolchain from the project manifest (criticalup.toml)' in [`build.yml`].

#### Run your app using CriticalUp

The following command uses installed Ferrocene:

```shell
criticalup run cargo run --release
```

As you can see, you can simply pass `cargo` as a subcommand.

> See the step 'Run my app via Ferrocene and its toolchain' in [`build.yml`].

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