# NoStress Kommando Kustom Slackware Tool.

Garry Iglesias (c) 2013-2016.

## Introduction

> **NoStress Kustom Slackware** tool (**kuSlack**) is a command line utility which
> provide an assisted way to generate a customized _Slackware_ 'ISO' image. This image
> can be an _advanced installer_ (_Slackware_ Installation DVD, tweaked for your own
> usage) or a pure (or less pure ;) ) Live Slackware system.

> **_WARNING:_** This tool is still work in progress and in it's early public
> release. It is not complete, and might be broken for some usage. You should use
> it with a minimal understanding of what it does, shell scripting and what an OS
> is.

### Some history

> **kuSlack** development started between 2012 and 2013 to provide automatic, non
> interactive installation of Slackware on workstations and servers. It was allowing
> automatic hard-drive partitioning, SCSI setup and selected package installation.
> As an in-house tool it was quite 'rushed', but the perspectives were already there.
>
> Time have passed and I used the tool occasionally, lacking time to invest to improve
> it. While working on preparing open-source publishing of my "Kommando" collection,
> it appeared that a new version of _Slackware_ (14.2) were coming, with a bunch of
> new interesting features, like the 4.x kernel allowing file system overlays, greatly
> easing setup of live distributions. On the Linux Questions forums, a couple of people
> was talking about a 'live' system, and tweaking the _Slackware_ installer, which brought
> me to the conclusion that publishing an upgraded and cleaned  **kuSlack** could be of
> some interests for the Slackware community.
>
> And last but far from being the least, _Alien Bob_'s Slackware contributor released his
> "Project X" Live Slackware script, which was a good occasion to attempt to add the
> 'live' generation to **kuSlack**. Alien Bob's work saved me quite some time not to
> have to dig into sparse documentation, as it was a working pipeline which could serve
> as the basis for the live ISO setup. So indirectly and involuntary, Alien Bob contributed
> to the fast Live ISO integration, and I thank him for this and all his work for the
> Slackware community.
>
> __Early work in progress note:__
>
> So here it is, **kuSlack**, at the time of writing will become public soon, I focused
> first on the Live ISO generation as it seems to be the most requested feature, and
> I currently lack of resources to be able to do the whole upgrade of the installer
> tweaking feature, it will come in time... "when it's ready".

## Getting started

> **kuSlack** rely on "Flavor" descriptions. You can define your own flavor of Slackware
> DVD. A Flavor can depend on some other flavor, giving more flexibility to manage
> multiple flavors easily.
>
> **kuSlack** provide some default flavors, you can use them as a basis to build your
> own flavors.
>
> **kuSlack** reads the flavor description and generate a *recipe* script which
> can be invoked to generate the actual ISO image.

### Installation

> Simply clone or unpack **kuSlack** somewhere you choose and add it's directory to the
> environment **PATH** variable.

> **Hint:** you can run **kuSlack** with the _**make-env**_ command to automatically
> generate a code snippet for the **PATH** variable and configuration.

>     $ path/to/kuSlack make-env

> You can append to your session startup script with a one-liner:

>     $ cat "$(path/to/kuSlack make-env 2>/dev/null)" >>~/.bash_profile

> **Note:** you can use *kuSlack* as a normal user. But you must execute the *recipe* script
> as root. So you can install *kuSlack* local to one user, and, at the same time, you must
> have administrator permissions on the workstation (su or sudo).

### Usage

> Simple usage:
>
>     $ kuSlack [options] <command> [command options] [<command arguments>]
>
> Online help is available through:
>
>     $ kuSlack help

#### Configuration:

> Before using *kuSlack*, you should configure it.
> You can have several configuration files if you want, but the default one is
> located at: **$HOME**/etc/kuSlack.conf.

> There are many way to change the configuration path:

> * Set the environment variable **KUSLACK\_CONFIG\_PATH**
> * Use option **_--config_** _</path/to/file.conf>_
> * Use flag **_--here_** or **_-H_** to look in the current directory a file named
> 'kuSlack.conf'
> * Use option **_--this_** _<configname>_ to look in the current directory a file
> named '_configname_'.

> If no configuration exists at the look-up path, a new default configuration will
> be generated, and if your **EDITOR** environment variable is set, it will be use
> to propose you a direct edition.

#### Generating a recipe and cooking it:

> To build a specific flavor of _Kustom Slackware_ you need to invoke **kuSlack**
> in order to generate a 'recipe' script. This recipe must be executed as _root_
> and it will 'cook' the final _Kustom Slackware_.

> **kuSlack** already provide 'stock' flavors, some on them providing some features
> to build your own flavor, but also 'terminal' flavors which are the "official _Kustom
> Slackware_ flavors". So we can just create one of them as an example, or if you
> want as "yet another" Slackware Live ISO.

>     $ kuSlack cook --stock livekom-slack

> Shortly after typing this command, **kuSlack** should return successfully, pointing
> to the recipe script. You can review it if you want to check what it does, although
> it's not easy to read if you're not used to _BASH_ scripting.

> You can also invoke **kuSlack** so it cooks the recipe as soon as it is generated.
> If you're not logged as _root_, **kuSlack** will attempt to use _**sudo**_ to call
> the _recipe_ script.

>     $ kuSlack cook -x --stock livekom-slack

## Creating a flavor.

> Creating a **kuSlack** flavor is straightforward.
> Once configured simply:

>     $ kuSlack new-flavor <flavor-name>

> **Note:** if configuration is not at the default location, you must add corresponding
> arguments or set the configuration variable (**KUSLACK\_CONFIG\_PATH**).

> **kuSlack** is a _BASH_ written tool, and all configuration files or customisation
> script (flavor main script) are source as BASH scripts. You can of course call any
> command available from those scripts.

### Flavor structure.

> A flavor is defined with this directory structure:

> * [**./rc.flavor**] Main flavor script.
> * [**./flavor.dep**] Flavor dependencies.
> * [**./rc.modules/**] Optional modules directory.
> * [**./live.tree/**] Optional live tree template directory.
> * [**./data.tree/**] Optional data tree template directory.

> The main flavor script (**rc.flavor**) is sourced from the generated recipe script.
> The recipe call all dependent (**flavor.dep**) main flavor scripts in order.

> The flavor can register some functions to recipe's hooks. This is the top-level entry
> to customizing the ISO generation process.

> The flavor can "include" modules (BASH sub-scripts) from the main script. The **source**
> (**.**) command is wrapped to take care of the module path implicitly (recipe is not
> store, neither called from the flavor directory...).

> The recipe, after sourcing all flavors used, execute the ISO generation process, calling
> the corresponding hooks at some specific steps.

## Recipe ISO generation process.

> The recipe defines the pipeline to generate an **Kustom Slackware** ISO. The process is
> linear, with optional steps.

> There are two kind of generated ISO:

> * **Installation ISO:** this process by-bass the extended "live" generation. It is based
> on the stock Slackware installer. The default installation initrd is used as basis and
> tweaked for your needs. Then the ISO's 'data' (visible) is built, and often provide
> a (trimmed) 'mirror' of the **Slackware** repository, or a selection of stock packages
> mixed with some other repo selection (SBO, home-brew...).
> * **Live ISO:** this does a full process, generate a dedicated initrd with "extended"
> live capabilities. So a "live" installation is first done using _squashfs_ to generated
> live file system image. Then the ISO's 'data' tree is built, it contains the _squashfs_
> image files, but you can also add any data you want to be accessed through the mounted
> ISO image. Finally, a init ramdisk image is packed and the ISO image generated.

## More to come

> **_Work in progress_**
