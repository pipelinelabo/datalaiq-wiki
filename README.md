### DatalaiQ Wiki
Current version of DatalaiQ documentation can always be found at http://docs.datalaiq.io
This repo is served up at that url and can be cloned for offline use.

There is a shell.nix provided in the repo. To install Nix, visit https://nixos.org/download.html
Once nix is installed, just run `nix-shell` in the wiki root directory. 

Inside of the nix-shell:
- build the documentation by running `make html`
- view changes as you edit by running `make livehtml` and visit http://127.0.0.1:8000
