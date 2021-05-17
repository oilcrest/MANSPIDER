# MAN-SPIDER
### Crawl SMB shares for juicy information. File content searching + regex is supported!

![manspider](https://user-images.githubusercontent.com/20261699/74963251-6a08de80-53df-11ea-88f4-60c39665dfa2.gif)

### File types supported:
- `PDF`
- `DOCX`
- `XLSX`
- `PPTX`
- any text-based format
- and many more!!

### MAN-SPIDER will crawl every share on every target system. If provided creds don't work, it will fall back to "guest", then to a null session.
![manspider](https://user-images.githubusercontent.com/20261699/80316979-f9ab7e80-87ce-11ea-9628-3c22a07e8378.png)

### Installation:
(Optional) Install these dependencies to add additional file parsing capability:
~~~
# for images (png, jpeg)
$ sudo apt install tesseract tesseract-data-eng

# for legacy document support (.doc)
$ sudo apt install antiword
~~~
Install manspider (please be patient, this can take a while):
~~~
$ pip install pipx
$ pipx install manspider
~~~

### Example #1: Search the network for filenames that may contain creds
NOTE: matching files are automatically downloaded into `$HOME/.manspider/loot`! (`-n` to disable)
~~~
$ manspider 192.168.0.0/24 -f passw user admin account network login logon cred -d evilcorp -u bob -p Passw0rd
~~~

### Example #2: Search for XLSX files containing "password"
~~~
$ manspider share.evilcorp.local -c password -e xlsx -d evilcorp -u bob -p Passw0rd
~~~

### Example #3: Search for interesting file extensions
~~~
$ manspider share.evilcorp.local -e bat com vbs ps1 psd1 psm1 pem key rsa pub reg txt cfg conf config -d evilcorp -u bob -p Passw0rd
~~~

### Example #4: Search for finance-related files
This example searches financy-sounding directories for filenames containing 5 or more consecutive numbers (e.g. `000202006.EFT`)
~~~
$ manspider share.evilcorp.local --dirnames bank financ payable payment reconcil remit voucher vendor eft swift -f '[0-9]{5,}' -d evilcorp -u bob -p Passw0rd
~~~

### Usage Tip #1:
You can run multiple instances of manspider at once. This is useful when you want to search local files (similar to `grep -R`). You can also specify the keyword `loot` as the target, which searches the downloaded files in `$HOME/.manspider/loot`.

### Usage Tip #2:
Reasonable defaults help prevent unwanted scenarios like getting stuck on a single target. All of these can be overridden:
- **default spider depth: 10** (override with `-m`)
- **default max filesize: 10MB** (override with `-s`)
- **default threads: 5** (override with `-t`)
- **shares excluded: `C$`, `IPC$`, `ADMIN$`, `PRINT$`** (override with `--exclude-sharenames`)

### Usage Tip #3:
Manspider accepts any combination of the following as targets:
- IPs
- hostnames
- subnets (CIDR format)
- files containing any of the above
- local folders containing files

For example, you could specify any or all of these:
- **`192.168.1.250`**
- **`share.evilcorp.local`**
- **`192.168.1.0/24`**
- **`smb_hosts.txt`**
- **`loot`** (to search already-downloaded files)
- **`/mnt/share`** (to recursively search a directory)
    - NOTE: when searching local files, you must specify a directory, not an individual file

## Usage:
~~~
usage: manspider [-h] [-u USERNAME] [-p PASSWORD] [-d DOMAIN] [-m MAXDEPTH] [-H HASH] [-t THREADS] [-f REGEX [REGEX ...]] [-e EXT [EXT ...]] [--exclude-extensions EXT [EXT ...]]
                 [-c REGEX [REGEX ...]] [--sharenames SHARE [SHARE ...]] [--exclude-sharenames [SHARE ...]] [--dirnames DIR [DIR ...]] [--exclude-dirnames DIR [DIR ...]] [-q] [-n]
                 [-mfail INT] [-o] [-s SIZE] [-v]
                 targets [targets ...]

Scan for juicy data on SMB shares. Matching files and logs are stored in $HOME/.manspider. All filters are case-insensitive.

positional arguments:
  targets               IPs, Hostnames, CIDR ranges, or files containing targets to spider (NOTE: local searching also supported, specify directory name or keyword "loot" to search
                        downloaded files)

optional arguments:
  -h, --help            show this help message and exit
  -u USERNAME, --username USERNAME
                        username for authentication
  -p PASSWORD, --password PASSWORD
                        password for authentication
  -d DOMAIN, --domain DOMAIN
                        domain for authentication
  -m MAXDEPTH, --maxdepth MAXDEPTH
                        maximum depth to spider (default: 10)
  -H HASH, --hash HASH  NTLM hash for authentication
  -t THREADS, --threads THREADS
                        concurrent threads (default: 5)
  -f REGEX [REGEX ...], --filenames REGEX [REGEX ...]
                        filter filenames using regex (space-separated)
  -e EXT [EXT ...], --extensions EXT [EXT ...]
                        only show filenames with these extensions (space-separated, e.g. `docx xlsx` for only word & excel docs)
  --exclude-extensions EXT [EXT ...]
                        ignore files with these extensions
  -c REGEX [REGEX ...], --content REGEX [REGEX ...]
                        search for file content using regex (multiple supported)
  --sharenames SHARE [SHARE ...]
                        only search shares with these names (multiple supported)
  --exclude-sharenames [SHARE ...]
                        don't search shares with these names (multiple supported)
  --dirnames DIR [DIR ...]
                        only search directories containing these strings (multiple supported)
  --exclude-dirnames DIR [DIR ...]
                        don't search directories containing these strings (multiple supported)
  -q, --quiet           don't display matching file content
  -n, --no-download     don't download matching files
  -mfail INT, --max-failed-logons INT
                        limit failed logons
  -o, --or-logic        use OR logic instead of AND (files are downloaded if filename OR extension OR content match)
  -s SIZE, --max-filesize SIZE
                        don't retrieve files over this size, e.g. "500K" or ".5M" (default: 10M)
  -v, --verbose         show debugging messages
~~~