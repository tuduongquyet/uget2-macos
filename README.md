Geany for Mac OS
================
Geany for Mac OS is a project that contains all the necessary configuration
files, themes, scripts and instructions to create the Geany app bundle and 
a dmg installer image for Mac OS.

Binaries
--------
The Mac OS binaries can be downloaded from the Geany Releases page:

<https://www.geany.org/Download/Releases>

Files and Directories
---------------------
A brief description of the contents of the project directory:

### Directories
*	*Faience*: Faience GTK 2 icon theme combined with Faenza-Cupertino 
	icon theme (for better folder icons) and with lots of unneeded icons
	removed to save space.
*	*Papirus*: Papirus GTK 3 icon theme with lots of unneeded icons
	removed to save space.
*	*Greybird*: Greybird GTK 2 theme which has been modified to look more
	like Mac OS.
*	*Sierra-light-solid*: Sierra GTK 3 Theme
*	*iconbuilder.iconset*: contains source icons for the Geany.icns
	file. Not needed for normal build, present just in case the icns file
	needs to be recreated for some reason.
*	*patches*: patches fixing VTE under Mac OS and enabling VTE bundling. 

### Configuration files
*	*geany.modules*: JHBuild modules file with Geany dependencies.
*	*geany-gtk2.bundle, geany-gtk3.bundle*: configuration files describing
	the contents of the app bundle.
*	*Info.plist*: Mac OS application configuration file containing some basic
	information such as application name, version, etc. but also additional
	configuration including file types the application can open.
*	*Geany.icns*: Mac OS Geany icon file.
*	*settings.ini*: theme configuration file for GTK 3. 

### Scripts
*	*launcher-gtk2.sh, launcher-gtk3.sh*: launcher script from the
	gtk-mac-bundler project setting all the necessary environment variables.
*	*plist_filetypes.py*: script generating the file type portion of the
	Info.plist file from Geany's filetype_extensions.conf configuration
	file.
*	*create_dmg.sh*: script calling create-dmg to create the dmg installer
	image. 

General Instructions
--------------------
For more general instructions about building and bundling Mac OS applications
please visit

<https://gitlab.gnome.org/GNOME/gtk-osx/>

The HOWTO below contains just the portions necessary/relevant for
building and bundling Geany.

Prerequisities
--------------
*	OS X
*	Xcode and command-line tools

JHBuild Installation
--------------------
To create the bundle, you need to first install JHBuild and GTK as described below.

1.	Create a new account for jhbuild (not necessary but this makes sure
	jhbuild will not interfere with some other command-line tools installed
	on your system).

2.	Get `gtk-osx-setup.sh` by

	```
	curl -L -o gtk-osx-setup.sh https://gitlab.gnome.org/GNOME/gtk-osx/raw/master/gtk-osx-setup.sh
	```
	
	and run it:
	
	```
	sh gtk-osx-setup.sh
	```

3.	Run

	```
	export PATH=$PATH:"$HOME/.new_local/bin"
	```

	to set path to jhbuild installed in the previous step.

4.	Update the `setup_sdk()` call in `~/.jhbuildrc-custom` to something like

	```
	setup_sdk(target="10.9", sdk_version="native", architectures=["x86_64"])
	```

	so the build creates a 64-bit binary that works on OS X 10.9 and later.
	OS X 10.9 is the first version which uses libc++ by default which is
	now required by Scintilla and VTE libraries because of C++11 support.

5.	By default, jhbuild compiles without optimization flags. To enable
	optimization, add `setup_release()` at the end of `~/.jhbuildrc-custom`.

6.	Install GTK and all of its dependencies by running one of the following
	commands:
	* **GTK 2**
		```
		jhbuild bootstrap-gtk-osx && jhbuild build meta-gtk-osx-freetype meta-gtk-osx-bootstrap meta-gtk-osx-core
		```
	* **GTK 3**
		```
		jhbuild bootstrap-gtk-osx && jhbuild build meta-gtk-osx-freetype meta-gtk-osx-bootstrap meta-gtk-osx-gtk3
		```
	This is the moment when you have to make a decision whether to build
	Geany with GTK 2 or GTK 3 - they cannot be installed side by side.

Geany Installation
------------------
1.	Docutils will fail if you do not set the following environment variables:

	```
	export LC_ALL=en_US.UTF-8; export LANG=en_US.UTF-8; export PYTHON=python3
	```

2.	To build Geany, plugins and all of their dependencies, run one of 
	the following commands inside the `geany-osx` directory  depending on
	the GTK version used and whether to use Geany sources from the latest
	release tarball or current git master:
	* **GTK 2**
		* **Geany from release tarball**
			```
			jhbuild -m `pwd`/geany.modules build geany-bundle-release-gtk2
			```
		* **Geany from git master**
			```
			jhbuild -m `pwd`/geany.modules build geany-bundle-git-gtk2
			```
	* **GTK 3**
		* **Geany from release tarball**
			```
			jhbuild -m `pwd`/geany.modules build geany-bundle-release-gtk3
			```
		* **Geany from git master**
			```
			jhbuild -m `pwd`/geany.modules build geany-bundle-git-gtk3
			```

Bundling
--------
1.  To build the GTK3 binary launcher, run
	```
	xcodebuild -project LauncherGtk3/geany/geany.xcodeproj
	```
2.	Run

	```
	jhbuild shell
	```
	to start jhbuild shell. 

	*The rest of this section assumes you are running from within the jhbuild shell.*

3.	To bundle all available Geany themes, get them from

	<https://github.com/geany/geany-themes>

	and copy the `colorschemes` directory under `$PREFIX/share/geany`.

4.	Go to the `geany-osx` directory and copy the icon theme to the GTK
	icons directory:
	* **GTK 2**
		```
		cp -R Faience $PREFIX/share/icons
		```
	* **GTK 3**
		```
		cp -R Papirus $PREFIX/share/icons
		```

5.	Inside the `geany-osx` directory run the following command to create
	the app bundle.
	* **GTK 2**
		```
		~/.local/bin/gtk-mac-bundler geany-gtk2.bundle
		```
	* **GTK 3**
		```
		~/.local/bin/gtk-mac-bundler geany-gtk3.bundle
		```

6.	Optionally if you have a development account at Apple and want to sign the
	resulting bundle so it can be started without warning dialogs, use

	```
	export SIGN_CERTIFICATE="your certificate name"
	```

	The certificate should be installed in your login keychain. You can get the
	certificate name by running `security find-identity -p codesigning` and
	checking  for "Developer ID Application" - the whole name in apostrophes is
	the certificate name.

	Then, run

	```
	./sign.sh
	```


Distribution
------------
1.	Get the `create-dmg` script from

	<https://github.com/andreyvit/create-dmg.git>

	and put it to your `$PATH`.

2.	Create the dmg installation image by calling
	
	```
	./create_dmg.sh
	```

	from within the `geany-osx` directory. If the `SIGN_CERTIFICATE` variable is
	defined, the image gets signed by the specified certificate.

3.	Optionally, to get the image notarized by
	[Apple notary service](https://developer.apple.com/documentation/security/notarizing_your_app_before_distribution),
	run
	```
	./notarize.sh <dmg_file> <apple_id>
	```
	where `<dmg_file>` is the dmg file generated above and `<apple_id>`
	is the Apple ID used for your developer account. The script then
	prompts for an [app-specific password](https://support.apple.com/en-us/HT204397)
	generated for your Apple ID.

Maintenance
-----------
This section describes some maintenance-related activities which do not
have to be performed during normal bundle/installer creation:

*	To get the `Info.plist` file associations in sync with 
	`filetype_extensions.conf`, copy the filetype extension portion from
	`filetype_extensions.conf` to the marked place in `plist_filetypes.py`
	and run the script. Copy the output of the script to the marked
	place in `Info.plist`.

*	The `Geany.icns` icon file can be regenerated from the `iconbuilder.iconset`
	directory using

	```
	iconutil -c icns ./iconbuilder.iconset
	```

*	Before the release update the Geany version and copyright years inside
	`Info.plist` and `create_dmg.sh`. Also update the `-release` targets in
	`geany.modules` file to point to the new release. Dependencies inside
	`geany.modules` can also be updated to newer versions.
	
*	To make sure nothing is left from the previous build when making a
	new release, run

	```
	rm -rf .new_local Source gtk .cache/jhbuild
	```

---

Jiri Techet, 2019
