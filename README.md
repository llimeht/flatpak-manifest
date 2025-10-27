# Flatpak Manifest

This is a draft of the Flatpak manifest file which allows SasView to be built as a Flatpak.

To build, you need to have [Flatpak](https://www.flatpak.org/setup/), and `flatpak-builder`. If you don't have `flatpak-builder` installed, you can run the following command to install it:

``` shell
flatpak install -y flathub org.flatpak.Builder
```

Also install the SDK and base for the project:

``` shell
flatpak install -u org.kde.Sdk//6.9 org.kde.Platform//6.9 io.qt.qtwebengine.BaseApp//6.9
```

Then, to build the manifest, run the following commands.

``` shell
./prep-wheelhouse
flatpak-builder --force-clean build/ org.sasview.sasview.yml
```

To install it to your local machine, add the `--install` flag to the above command. Then, you can run the Flatpak with:

``` shell
flatpak run org.sasview.sasview
```

# Design Notes

- Flathub's [best practices](https://docs.flathub.org/docs/for-app-authors/requirements#best-practices) state that 'Applications should build all components of the manifest from source when possible.' This is currently not the case as the vast number of Python dependencies cannot be simply rebuilt — unfortunately, deploying tools like `flatpak_pip_generator` only serves to find missing build-requires that cause the wheels to fail to build.
- SasView requires there to be a C compiler in its environment, otherwise it will crash when trying to select a model. Since a C compiler is only available in the SDK, and not the runtime, this is achieved by `tccbox` as a module for the time being; however, it isn't doesn't manage to compile.
- Flatpak currently has full access to the home directory but there other other mechanisms that might be better, such as  Flatpak's [portal system](https://docs.flatpak.org/en/latest/portal-api-reference.html) or configured via permissions tools like [Flatseal](https://flathub.org/en/apps/com.github.tchx84.Flatseal), or desktop-specific tools such as [Flatpak KCM (kde-config-flatpak)](https://invent.kde.org/plasma/flatpak-kcm).

# Debugging the runtime environment
If building fails at some point, you can enter the build environment with
```
flatpak-builder --run build-dir org.sasview.sasview.yml bash
```

If you can get the flatpak to build then you can enter the runtime environment via sasview itself:

```
flatpak run org.sasview.sasview -c "import subprocess; subprocess.run('bash', env={'PS1':'$ '})"
```


# Known Issues

- There are no OpenCL drivers in the Flatpak so any functionality related to OpenCL will not work.
- In the about page, SasView states its installation path as `/app/bin` because, since its in a containerised environment, it only knows where it is installed in that environment itself, and not on the host system.

# TODO
- C compiler
- OpenCL / llvm access
