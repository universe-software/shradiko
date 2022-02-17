# Shradiko
## Make Portable AppImages from Distro Packages

Shradiko is a tool for making portable AppImages from distro packages. Currently, it only supports DEB packages.

## How

Shradiko works by installing the package and its dependencies onto a minimal package manager installation inside a chroot (currently Ubuntu Base 20.04), which is stored in the AppImage. When the AppImage is run, it creates a new mount namespace and mounts overlay filesystems onto several system directories to make the package think it is installed on the system. That way, even non-portable packages can be used portably.

## Notes

* To build the AppImage, Shradiko must be run as root
* Shradiko requires an internet connection to download Ubuntu Base and any package dependencies
* Since Shradiko uses overlay filesystems, the packaged program should not expect changes to files in system directories to be permanent. For example, editing files in `/etc` with a packaged text editor will not actually change those files permanently. Changes to files in user home directories will still be permanent.
* The program will think it is running as root. Some programs refuse to run as root or require special options for that to work. For example Chromium requires the `--no-sandbox` flag to run as root. This does not, however, give actual root permissions.
* See the *Troubleshooting* guide below

## Installation

Shradiko is itself an AppImage. Go to [the releases page](https://github.com/universe-software/shradiko/releases) and download the latest `shradiko-apt`. Then, to make it executable, run the following in a terminal opened to the directory with `shradiko-apt`:

```bash
chmod +x shradiko-apt
```

## Usage

**Shradiko requires root permissions.** To build an AppImage with Shradiko, you need:

* A `.deb` package file
* A [desktop entry file](https://wiki.archlinux.org/title/Desktop_entries) - Note that if you use the desktop entry from inside the package, some packages have entries that use unregistered categories, which AppImageKit forbids. Edit the desktop entry and remove any `Categories` that aren't in the [registered categories list](https://specifications.freedesktop.org/menu-spec/latest/apa.html) first.
* An icon file (`.png` or `.svg`) with a base name that matches the `Icon` value in the desktop entry

**Shradiko requires provided file paths to be absolute.** To easily do so for files in the current directory, provide paths with `"$(pwd)/..."`. To build an AppImage, assuming `shradiko-apt` and the necessary files are in the current directory, run:

```
sudo ./shradiko-apt "$(pwd)/PACKAGE.deb" COMMAND "$(pwd)/DESKTOP_ENTRY.desktop" "$(pwd)/ICON" "$(pwd)/PACKAGE.AppImage"
```

replacing `PACKAGE.deb` with the name of your DEB file, `COMMAND` with the command to use to start the installed program (e.g. if you installed LibreOffice Writer, the command would be `lowriter`), `DESKTOP_ENTRY.desktop` with the name of your desktop entry, `ICON` with the name of you icon file (including `.png` or `.svg extension`), and `PACKAGE.AppImage` with the name you want to give your new AppImage.

Shradiko will then download and extract Ubuntu Base, install the package and its dependencies, clean up, and wrap the environment in an AppImage. To allow your regular user profile to access the AppImage, run

```
sudo chown $USERNAME:$USERNAME PACKAGE.AppImage
```

Replacing `PACKAGE.AppImage` with the name of your AppImage file.

## Troubleshooting

Sometimes, if Shradiko fails or is interrupted, trying to run it again will give errors when trying to delete the old instance files because some directories are still mounted. If this happens, use the following commands to unmount those directories first:

```
sudo fuser -km /root/.shradiko/AppImage/root/dev
sudo umount /root/.shradiko/AppImage/root/dev
sudo fuser -km /root/.shradiko/AppImage/root/tmp
sudo umount /root/.shradiko/AppImage/root/tmp
sudo fuser -km /root/.shradiko/AppImage/root/proc
sudo umount /root/.shradiko/AppImage/root/proc
```