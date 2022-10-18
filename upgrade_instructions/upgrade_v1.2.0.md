# Upgrade Instruction

## Preping your node

### Update your node

Don't just COPY & PASTE, read and ensure the commands meet your needs/paths

Now might be a good time to update your node

```bash
apt-get update && apt-get upgrade
```

Install `build-essential` to compile source code

```bash
apt-get install -y build-essential
```

## GitHub

### Fresh clone of the repo
If you haven't already, clone the github repo.

The following instructions should be done as your `vidulum` user.

```bash
git clone https://github.com/vidulum/mainnet vidulum-mainnet
cd vidulum-mainnet
git fetch -a
git checkout v1.2.0
```

### Existing repo

Skip this step if you just got a Fresh Clone of the repo.

If you already have a cloned folder, update it and pull the version to upgrade to. Change `CLONED_REPO_DIR` to the appropriate directory.

```bash
cd CLONED_REPO_DIR
git fetch -a
git checkout v1.2.0
```

### Build the binary

Build the new binary but do not install/move it.

```bash
make build
```

This will compile the binary and place it in the location

`./build/vidulumd`

Now we want to ensure it's the correct version.

```
./build/vidulumd version --long
```

There will be a long list of go packages. At the bottom will be the `cosmos_sdk_version`.  Your output should match minus the go version.  It is recommended to update.

```
name: vidulum-1
server_name: <appd>
version: 1.2.0
commit: '"latest"'
build_tags: ""
go: go version go1.19 linux/amd64
build_deps:

cosmos_sdk_version: v0.45.9
```

## Implementing the upgrade

### For those who where there for the halt

Do not START your daemon yet until word has been given to do so in discord.

Change your halt height back to 0 in your `app.toml`

For those that do fall in this category, you can move your copy of the new binary to the `go/bin` folder.

```bash
# Inside the repo as your vidulum user
make install
rm ~/.local/bin/vidulumd
ln -s ~/go/bin/vidulumd ~/.local/bin/vidulumd
```

If you have followed the Vidulum Documentation for installation, the above is recommended.  `make install` will place the binary in `~/go/bin` automatically, so this just creates a link to the defined location from the installation instructions.

When the time comes to restart the chain, as your admin user run

```bash
sudo systemctl start vidulum && journalctl -u vidulum -f
```

### Upgrading after the halt/chain restart - Beyond Oct 18th

We will stop the node, move the files over and then restart the node and follow the journal output to ensure the upgrade worked.  This assumes you've ran `make build` in the repo and the new binary is ready for copying.

You may need to change the directories locations in these commands.  Do not just copy & paste.

```bash
systemctl stop vidulumd && mv /home/vidulum/vidulum-mainnet/build/vidulumd /home/vidulum/go/bin/vidulumd && systemctl start vidulum && journalctl -u vidulum -f
```

If you have previously stored your binary in `~/.local/bin` for the vidulum user, you can do the following so that you don't need to update your service file

As the vidulum user
```bash
rm ~/.local/bin/vidulumd
ln -s ~/go/bin/vidulumd ~/.local/bin/vidulumd
```

This creates a symlink so that when you run `make install` and new binary's are installed to `~/go/bin`, you don't need to copy/move to `~/.local/bin`.

