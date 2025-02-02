# Upgrade Instructions

## Prepping your node

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

### Update Go To The Latest Version

As your Admin/Root user:

```bash
GOVER=$(curl https://go.dev/VERSION?m=text)
wget https://golang.org/dl/${GOVER}.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf ${GOVER}.linux-amd64.tar.gz
```

## GitHub

### Catch up to Block 5212050

*Do this step ONLY if you halted at height 5212000*

We first tried to halt at 5212000 but that didn't work out.  If you halted there and didn't turn back on, you first need to catch up to the latest block on the old version.

On `vidulumd` repo version `v1.0.0`:
Watch the journal first.
```bash
journalct -u vidulum -f
```

Next change your halt height in `app.toml` to be `5212049`
```
halt-height = 5212049
```

Now start the vidulum service.  It should exit and halt the service once it reaches this block.  If not, and your service keeps restarting, you will need to manually stop the service.
This is why we have the journal up to show us logs as it happens!

Once you've caught up to block 5212049, you can move on to upgrading your binary!

## ALTERNATIVE TO PREVIOUS STEP
### Snapshot Method

There were issues due to when and how the chain halted.  Some nodes prevoted on the block that we were trying to gain consensus on.  5212052.
The simple fix is to back up your `priv_validator_state.json` in the `$HOME/.vidulum/data` directory.

Backup state file:
```bash
cp ~/.vidulum/data/priv_validator_state.json ~/.vidulum/
```
Verify that the copy exists before moving on.

Next, download the snapshot here:
https://rpc.erialos.me:8080/vidulum/

To download this to your server:
```bash
cd ~/.vidulum
wget https://rpc.erialos.me:8080/vidulum/vidulum_snapshot_Oct-18-2022.tar.gz
```

Before doing so :stop_sign: ENSURE YOU BACKED UP YOUR STATE FILE :stop_sign:
delete your existing data directory
```bash
rm -rf ~/.vidulum/data
```

Now extract the tar.gz file
```bash
tar zxf vidulum_snapshot_Oct-18-2022.tar.gz
```

This should extract a folder called `data` in the current working directory which should be `~/.vidulum`

Now we need to replace your state file:
```bash
cp priv_validator_state.json data/
```

Your data dir is now all set.  You should see this when you `ls -lha`.  The size of your `priv_validator_state.json` could differ.
```
total 372K
drwx------ 8 vidulum vidulum 4.0K Oct 19 16:33 .
drwxrwxr-x 4 vidulum vidulum 4.0K Oct 18 15:28 ..
drwxr-xr-x 2 vidulum vidulum  16K Oct 19 06:45 application.db
drwxr-xr-x 2 vidulum vidulum 128K Oct 19 06:45 blockstore.db
drwx------ 2 vidulum vidulum 4.0K Oct 19 14:29 cs.wal
drwxr-xr-x 2 vidulum vidulum 4.0K Oct 19 06:45 evidence.db
-rw------- 1 vidulum vidulum  272 Oct 19 16:33 priv_validator_state.json
drwxr-xr-x 3 vidulum vidulum 4.0K Aug  6 19:54 snapshots
drwxr-xr-x 2 vidulum vidulum 196K Oct 19 06:45 state.db
```

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

This will compile the binary and place it in the location: `./build/vidulumd`

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
go: go version go1.19.2 linux/amd64
build_deps:

cosmos_sdk_version: v0.45.9
```

## Implementing the upgrade

### For those who where there for the halt

Change your halt height back to 0 in your `app.toml`

For those that do fall in this category, you can move your copy of the new binary to the `go/bin` folder.

```bash
# Change directories into  the github repo as your vidulum user
make install
rm ~/.local/bin/vidulumd
ln -s ~/go/bin/vidulumd ~/.local/bin/vidulumd
```

If you have followed the Vidulum Documentation for installation, the above is recommended.  `make install` will place the binary in `~/go/bin` automatically, so this just creates a link to the defined location from the installation instructions.

:stop_sign:	When the time comes to restart the chain, as your admin user run :stop_sign:

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

