# Compiling, running and making changes on Apple Silicon Macs

The following instructions allow you to build and run the client and other services on an Apple Silicon
Mac without resorting to having a separate x86_64 `homebrew` install.

This guide assumes you have `homebrew` installed on your Mac already.

## Install `python` 2.7 and 3.

Use a version manager, such as `asdf` or `pyenv` to install both python 2.7 and 3.

eg
```bash
brew install asdf
asdf plugin-add python
asdf install python 2.7.18
asdf install python 3.11.1
```

## Install `IntelliJ` 

Install the free community edition of IntelliJ IDEA.

## Clone `serenade`

Clone `serenade` to a directory in which you would like all your Serenade related things to go, eg 

```bash
mkdir -p ~/voicecoding/
cd voicecoding/
git clone https://github.com/serenadeai/serenade.git
```

## Set environment variables

Serenade uses two environment variables to describe where source code and dependencies will be on the filesystem:

- `SERENADE_SOURCE_ROOT`: The location of the `serenade` repository. Defaults to `~/serenade`.
- `SERENADE_LIBRARY_ROOT`: The location of Serenade dependencies. Defaults to `~/libserenade`.

For example, adding the following to `~/.bashrc` & `~/.bash_profile` or `~/.zshrc`, replacing
the `/path/to` for your parent directory in which you cloned `serenade`.

```
export SERENADE_SOURCE_ROOT="/path/to/serenade"
export SERENADE_LIBRARY_ROOT="/path/to/libserenade"
```

Note that next setup step script will create your `SERENADE_LIBRARY_ROOT` directory if it doesn't exist.

## Run `scripts/setup/setup-mac.sh`

To install necessary system-wide dependencies onto your system, run:

```bash
scripts/setup/setup-mac.sh
```

After this we must also set the `JAVA_HOME` env var to point to the JDK dep & you should add the gradle dep to your `PATH`.
These will be set to paths that will be created by the `build-dependencies-apple-silicon.sh` you will run shortly.

Pay attention to the output from this script as it will give you what to add to your `~/.bashrc` or `~/.zshrc`

```
export JAVA_HOME="/path/to/libserenade/jdk-14.0.1.jdk/Contents/Home"
export PATH=/path/to/libserenade/jdk-14.0.1.jdk/Contents/Home/bin:/path/to/libserenade/gradle-7.4.2/bin:$PATH
```

## Create a new IntelliJ project

In IntelliJ IDEA, open your `serenade` clone using 'File->New->Project from existing sources...'

Once open, go to 'File->Project Structure' and go to the 'Project' tab. For the SDK option, click the dropdown, 
and choose 'Add SDK->JDK...' and select the JDK 14 in `libserenade/`, eg at `~/voicecoding/libserenade/jdk-14.0.1.jdk/`

IntelliJ automatically preforms build and project setup based on the gradle file, but you can also configure more
Configurations which trigger whatever gradle action you want.

## Build dependencies

Then, to download and build Serenade libraries and models, run:

```bash
scripts/setup/build-dependencies-apple-silicon.sh
```

As with the Docker image, this will build all of the dependencies needed to run Serenade as well as train models. To build only the dependencies needed to run Serenade, you can instead run:

```bash
scripts/setup/build-dependencies-apple-silicon.sh --minimal
```

## Compile

Check everything can now compile by running

```bash
gradle installDist
```

Hopefully everything compiles. There maybe warnings from the linker about architectures of dylibs but they seem to not
be used so can be ignored.

## Run the tests

```bash
scripts/serenade/bin/run.py --tests 'gradle test -i'
```

And check that everything passes.

## Setup the client app

First you must install the compiled services we just built into the client app's local directory using:

```bash
gradle client:installServer
```

## Start the client

```bash
cd client
bin/dev.py
```

Once the app is open, skip over the setup process and then from the UI switch the server from Cloud to `Local` mode.

This will start the local server that has been installed via the previous step.

If the client never seems to connect, then the server has probably failed to boot. You can see logs in `~/.serenade` to 
see what has gone wrong.

## Install plugins

Eg for JetBrains IDEs, install the `Serenade` Plugin from the Plugin Marketplace and restart your IDE. 
The plugin will then connect to the running instance of serenade.