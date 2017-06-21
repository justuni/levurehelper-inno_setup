# inno_setup

Inno Setup is a free installer system for Windows available at http://www.jrsoftware.org/isinfo.php. This helper fills in a template `.iss` file with the files and folders in your application each time you package your application.

## Contents

* [Activate the inno_setup framework helper](#activate-the-inno_setup-framework-helper)
* [Generating an .iss file when packaging your application](#generating-an-iss-file-when-packaging-your-application)
* [Setting up the `.iss` file](#setting-up-the-iss-file)

## Activate the inno_setup framework helper

To add the Inno Setup helper to your application add it under the `helpers` section of the `app.yml` file:

```
# app.yml

helpers:
  - folder: ./helpers
  - filename: "./helpers/inno_setup"
```

## Generating an .iss file when packaging your application

The Inno Setup helper works by inserting references to all of the files and folders in the root directory of your application folder into an existing `.iss` file. The helper will replace the variable `[[FilesAndFoldersToInstall]]` with the file and folder references.

You will add configuration information to two locations within the `app.yml` file.

### `inno setup` section

The first location will be a new configuration setting named `inno setup`. It looks like this:

```
# app.yml

inno setup:
  file flags:
  folder flags:
  executable flags:
  compiler:
```

The `file flags`, `folder flags`, and `executable flags` properties specify the flags that will be added to the file and folder entries in the `.iss` file. The `compiler` property is set to the path to Inno Setup `Compil32.exe` compiler. When running on Windows the helper will try to compile the `.iss` script.

`file flags`: Flags that will be added to all files added to the `.iss` file.
`folder flags`: Flags that will be added to all folders added to the `.iss` file.
`executable flags`: Flags that will be added to the executable file that is added to the `.iss` file.

Here is an example:

```
# app.yml

inno setup:
  file flags: ignoreversion
  folder flags: ignoreversion recursesubdirs createallsubdirs
  executable flags: ignorevesion sign
  compiler: C:\Program Files (x86)\Inno Setup 5\Compil32.exe
```

```
# .iss file
...

Source: ".\windows\Externals\*"; DestDir: "{app}\Externals"; Flags: ignoreversion recursesubdirs createallsubdirs
Source: ".\windows\helpers\*"; DestDir: "{app}\helpers"; Flags: ignoreversion recursesubdirs createallsubdirs
Source: ".\windows\locales\*"; DestDir: "{app}\locales"; Flags: ignoreversion recursesubdirs createallsubdirs
Source: ".\windows\ui\*"; DestDir: "{app}\ui"; Flags: ignoreversion recursesubdirs createallsubdirs
Source: ".\windows\app.livecodescript"; DestDir: "{app}"; Flags: ignoreversion
Source: ".\windows\behaviors.livecode"; DestDir: "{app}"; Flags: ignoreversion
Source: ".\windows\frontscripts.livecode"; DestDir: "{app}"; Flags: ignoreversion
Source: ".\windows\levure.livecodescript"; DestDir: "{app}"; Flags: ignoreversion
Source: ".\windows\libraries.livecode"; DestDir: "{app}"; Flags: ignoreversion
Source: ".\windows\revsecurity.dll"; DestDir: "{app}"; Flags: ignoreversion
Source: ".\windows\ScreenSteps.exe"; DestDir: "{app}"; Flags: ignoreversion sign

...
```

### `copy files` section

The second location you will be in one or more `copy files` sections under `build profiles`. You will add an `inno setup` section with references to the `.iss` file(s) that you want to copy over into the package folder when you package an application. Here is an example that copies the `./build files/MyApp.iss` file for all build profiles. Note the optional `compile` parameter. Set to `false` if the helper should not try to compile the script.

```
# app.yml

build profiles:
  all profiles:
    post build script:
    copy files:
      inno setup:
        - filename: ../build files/MyApp.iss
          compile: true
```

## Setting up the `.iss` file

When setting up the `.iss` file that the Inno Setup helper will use you will configure every setting except for your application files in the `[Files]` section. In the `[Files]` section you will include the string `[[FilesAndFoldersToInstall]]` on a line by itself. There are some other variables you can use in the file as well.

`[[NAME]]`: The `name` property from `app.yml`.
`[[APP_VERSION]]`: The first two numbers from `version` in `app.yml` (e.g. 2.0).
`[[MAJOR_VERSION]]`: The first number from the `version` property in `app.yml` (e.g. 2).
`[[BUILD]]`: The `build` property from `app.yml`.
`[[BUILD_PROFILE]]`: The build profile used to package the application.


### Example:

```
...
[Setup]
SignTool=signtool
AppName=[[NAME]]
AppVerName=[[NAME]] [[APP_VERSION]]
...

[Files]

[[FilesAndFoldersToInstall]]

[Tasks]
Name: "desktopicon"; Description: "{cm:CreateDesktopIcon}"; GroupDescription: "{cm:AdditionalIcons}"; Flags:
...
```
