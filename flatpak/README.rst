https://gerrit.libreoffice.org/gitweb?p=dev-tools.git;a=blob;f=flatpak/build.sh;h=8b381c2fed1a9b47b923e8a205f348473d03f419;hb=HEAD
https://git.gnome.org/browse/pitivi/tree/build/flatpak


Building
--------------

Build the software in flatpak using a command like this:

    flatpak-builder  --force-clean -v  --repo=/tmp/fb.repo --gpg-sign=tobiasmue@gnome.org  /tmp/fpbuilder org.gnome.Keysign.json

That will have populated the *repository* in /tmp/fb.repo with the build.


Then, you have to make the build known in the repository:

    flatpak build-update-repo --title='GNOME Keysign' --generate-static-deltas --prune --prune-depth=5  --gpg-sign="tobiasmue@gnome.org"  /tmp/fb.repo


To create a bundle file, use something like the following:


    gpg2 --output="$HOME/tobiasmue@gnome.org.gpg.asc" --armor --export \
            "tobiasmue@gnome.org"


    flatpak build-bundle \
        --repo-url=https://muelli.cryptobitch.de/flatpak \
        --gpg-keys="$HOME/tobiasmue@gnome.org.gpg.asc" /tmp/fb.repo   \
        GNOME-Keysign.flatpak org.gnome.Keysign           \
        master


Installation
--------------

Before you can install an application, you need to install a runtime.
For now, we use the GNOME 3.20 runtime.
If you haven't installed it yet, you can get it by running the following commands:

    wget https://sdk.gnome.org/keys/gnome-sdk.gpg
    flatpak --user remote-add --gpg-import=gnome-sdk.gpg gnome https://sdk.gnome.org/repo/
    flatpak --user remote-add --gpg-import=gnome-sdk.gpg gnome-apps https://sdk.gnome.org/repo-apps/
    flatpak --user install gnome org.gnome.Platform 3.20


Then, if you have a bundle file of GNOME Keysign, you can do

    flatpak install --user --bundle GNOME-Keysign.flatpak


Otherwise, you have to manually add the repository and install from there:

    flatpak remote-add --user --gpg-import=$HOME/tobiasmue@gnome.org.gpg.asc   gks https://muelli.cryptobitch.de/flatpak/

    flatpak install --verbose --user gks org.gnome.Keysign




Repository
--------------

If you haven't done so already, you need to make the latest build
known to the (local) repository with the *build-update-repo* mentioned
above.

Then, you can simply copy the local repository directory to a remote location:

    rsync --delete --numeric-ids  -r  --partial --progress -v --links /tmp/fb.repo/ server:~/public_html/flatpak/

Note that if "summary" and "summary.sig" do not get copied last,
a client won't be able to pull the lastest changes while they
are still in transit.