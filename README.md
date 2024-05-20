# HTNDX

HTNDX is a dedicated desktop process manager for [Hoosat Network node](https://github.com/Hoosat-Oy/HTND).


HTNDX offers a miniature console using which user can re-build the Hoosat Network stack, upgrading Hoosat Network to the latest version directly from GitHub. The build process is automated via a series of scripts that, if
needed, fetch required tools (git, go, gcc) and build Hoosat Network on the host computer (the build includes various Hoosat Network utilities including, `htnwallet`, `htnctl` and others and can be executed against any specific Git branch).  HTNDX console can also be used to migrate Hoosat Network database if building a version with an updated database schema.

HTNDX process configuration (available via a simple JSON editor) allows user to specify command-line arguments for executables, as such it is possible to configure HTNDX to run multiple instances of Hoosat Network or potentially run multiple networks simultaneously (provided Hoosat Network nodes do not pro-actively auto-discover each-other)

Like many desktop applications, HTNDX can run in the tray bar, out of the way.

HTNDX is built using [NWJS](https://nwjs.io) and is compatible Windows, Linux and Mac OS X.


## Building HTNDX

### Pre-requisites

- [Node.js 14.0.0+](https://nodejs.org/)
- Emanator - `npm install emanator@latest`
- Rust (latest, used for building kaspa miner at https://github.com/aspectron/kaspa-miner)
- Cuda linraries for kaspa miner (depends on the platform)

**NOTE:** HTNDX build process builds and includes latest Hoosat Network binaries from Git master branches. 
To build from specific branches, you can use `--branch...` flags (see below).

#### Generating HTNDX installers
```
npm install emanator@latest
git clone git@github.com:Hoosat-Oy/htndx
cd htndx
# run emanate with one or multiple flags below
#  --portable   create a portable zipped application
#  --innosetup  generate Windows setup executable
#  --dmg        generate a DMG image for Mac OS X
#  --all        generate all OS compatible packages
# following flags can be used to reset the environment
#  --clean		clean build folders: purges cloned `GOPATH` folder
#  --reset		`--clean` + deletes downloaded/cached NWJS and NODE binaries
emanate [--portable | --innosetup | --dmg | --all]
```


DMG - Building DMG images on Mac OS requires `sudo` access in order to use system tools such as `diskutil` to generate images: 
```
sudo emanate --dmg
```

To build the windows portable deployment, run the following command:
```
emanate --portable
```

To build the Windows installer, you need to install [Innosetup](https://jrsoftware.org/isdl.php) and run:
```
emanate --innosetup
```


Emanator stores build files in the `~/emanator` folder

#### Running HTNDX from development environment


In addition to Node.js, please download and install [Latest NWJS SDK https://nwjs.io](https://nwjs.io/) - make sure that `nw` executable is available in the system PATH and that you can run `nw` from command line.

On Linux / Darwin, as good way to install node and nwjs is as follows:

```
cd ~
mkdir bin
cd bin

#node - (must be 14.0+)
wget https://nodejs.org/dist/v14.4.0/node-v14.4.0-linux-x64.tar.xz
tar xvf node-v14.4.0-linux-x64.tar.xz
ln -s node-v14.4.0-linux-x64 node

#nwjs
wget https://dl.nwjs.io/v0.46.2/nwjs-sdk-v0.46.2-linux-x64.tar.gz
tar xvf nwjs-sdk-v0.46.2-linux-x64.tar.gz
ln -s nwjs-sdk-v0.46.2-linux-x64 nwjs

```
Once done add the following to `~/.bashrc`
```
export PATH = /home/<user>/bin/node/bin:/home/<user>/bin/nwjs:$PATH
```
The above method allows you to deploy latest binaries and manage versions by re-targeting symlinks pointing to target folders.

Once you have node and nwjs working, you can continue with HTNDX.

HTNDX installation:
```
npm install emanator@latest
git clone git@github.com:Hoosat-Oy/htndx
cd htndx
npm install
emanate --local-binaries
nw .
```

#### Building installers from specific Hoosat Network Git branches

`--branch` argument specifies common branch name for kaspa and kasparov, for example:
```
emanate --branch=v0.4.0-dev 
```
The branch for each repository can be overriden using `--branch-<repo-name>=<branch-name>` arguments as follows:
```
emanate --branch=v0.4.0-dev --branch-htnd=v0.3.0-dev
emanate --branch-miningsimulator=v0.1.2-dev
```

**NOTE:** HTNDX `build` command in HTNDX console operates in the same manner and accepts `--branch...` arguments.


## HTNDX Process Manager

### Configuration

HTNDX runtime configuration is declared using a JSON object.  

Each instance of the process is declared using it's **type** (for example: `htnd`) and a unique **identifier** (`kd0`).  Most process configuration objects support `args` property that allows
passing arguments or configuration options directly to the process executable.  Depending on the process type, the configuration is passed via command line arguments (kasparov*) or configuration file (htnd).

Supported process types:
- `htnd` - Hoosat Network full node
- `htnminer` - Hoosat Network sha256 miner

**NOTE:** For Hoosat Network, to specify multiple connection endpoints, you must use an array of addresses as follows: ` "args" : { "connect" : [ "peer-addr-port-a", "peer-addr-port-b", ...] }`

#### Default Configuration File
```js
{
	"htnd:kd0": {
		"args": {
			"rpclisten": "0.0.0.0:42420",
			"listen": "0.0.0.0:16211",
			"profile": 7000,
			"rpcuser": "user",
			"rpcpass": "pass"
		}
	},
	"htnd:kd1": {
		"args": {
			"rpclisten": "0.0.0.0:16310",
			"listen": "0.0.0.0:16311",
			"profile": 7001,
			"connect": "0.0.0.0:16211",
			"rpcuser": "user",
			"rpcpass": "pass"
		}
	},
	"simulator:sim0": {
        "blockdelay" : 2000,
		"peers": [ "127.0.0.1:16310" ]
	},
	"pgsql:db0": {
		"port": 18787
	},
	"mqtt:mq0": {
		"port": 18792
	},
	"kasparovsyncd:kvsd0": {
		"args": {
			"rpcserver": "localhost:16310",
			"dbaddress": "localhost:18787"
			"mqttaddress": "localhost:18792",
			"mqttuser" : "user",
			"mqttpass" : "pass"
		}
	},
	"kasparovd:kvd0": {
		"args": {
			"listen": "localhost:11224",
			"rpcserver": "localhost:16310",
			"dbaddress": "localhost:18787"
		}
	}
}
```

### Data Storage

HTNDX stores it's configuration file as `~/.htndx/config.json`.  Each configured process data is stored in `<datadir>/<process-type>-<process-identifier>` where `datadir` is a user-configurable location.  The default `datadir` location is `~/.htndx/data/`.  For example, `htnd` process with identifier `kd0` will be stored in `~/.htndx/data/htnd-kd0/` and it's logs in `~/.htndx/data/htnd-kd0/logs/htnd.log`

### Hoosat Network Binaries

HTNDX can run Hoosat Network from 2 locations - an integrated `bin` folder that is included with HTNDX redistributables and `~/.htndx/bin` folder that is created during the Hoosat Network build process. 

## HTNDX Console

HTNDX Console provides following functionality:
- Upgrading kasparov using `migrate` command
- `start` and `stop` controls stack runtime
- Hoosat Networkd RPC command execution
- Use of test wallet app (HTNDX auto-configures kasparov address)
- Rebuilding Hoosat Network software stack from within the console

### Using Hoosat Networkd RPC

Hoosat Networkd RPC can be accessed via HTNDX Console using the process identifier. For example:
```
$ kd0 help
$ kd0 getinfo
```
Note that RPC methods are case insensitive.

To supply RPC call arguments, you must supply and array of JSON-compliant values (numbers, double-quote-enclosed strings and 'true'/'false').  For example:
```
$ kd0 getblock "000000b22ce2fcea335cbaf5bc5e4911b0d4d43c1421415846509fc77ec643a7"
{
  "hash": "000000b22ce2fcea335cbaf5bc5e4911b0d4d43c1421415846509fc77ec643a7",
  "confirmations": 83,
  "size": 673,
  "blueScore": 46241,
  ...
}
```

