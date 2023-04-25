# Building on Pop!_OS 22.04 LTS
```bash
# Prerequisites
echo "deb http://security.ubuntu.com/ubuntu bionic-security main" | sudo tee /etc/apt/sources.list.d/ssl-1-0-dev.list
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 3B4FE6ACC0B21F32
sudo apt update && apt-cache policy libssl1.0-dev
sudo apt-get install libssl1.0-dev

# Building the project
git clone https://github.com/psabadac/vanitygen.git
cd vanitygen
make
```

I'd like to present a standalone command line vanity address generator 
called vanitygen.

There are plenty of quality tools to do this right now already.  So why 
use vanitygen?  The main reason is that it is fast, more than an order 
of magnitude faster than the official bitcoin client with the vanity 
address patch applied.  This is despite the fact that it runs on the 
CPU and does not use OpenCL or CUDA.  Vanitygen is also a bit more 
user-friendly in that it provides feedback on its rate of progress and 
how many keys it has checked.

```bash
./vanitygen 
Vanitygen 0.22 (OpenSSL 1.0.2n  7 Dec 2017)
Usage: ./vanitygen [-vqnrik1NT] [-t <threads>] [-f <filename>|-] [<pattern>...]
Generates a bitcoin receiving address matching <pattern>, and outputs the
address and associated private key.  The private key may be stored in a safe
location or imported into a bitcoin client to spend any balance received on
the address.
By default, <pattern> is interpreted as an exact prefix.

Options:
-v            Verbose output
-q            Quiet output
-n            Simulate
-r            Use regular expression match instead of prefix
              (Feasibility of expression is not checked)
-i            Case-insensitive prefix search
-k            Keep pattern and continue search after finding a match
-1            Stop after first match
-N            Generate namecoin address
-T            Generate bitcoin testnet address
-X <version>  Generate address with the given version
-F <format>   Generate address with the given format (pubkey or script)
-P <pubkey>   Specify base public key for piecewise key generation
-e            Encrypt private keys, prompt for password
-E <password> Encrypt private keys with <password> (UNSAFE)
-t <threads>  Set number of worker threads (Default: number of CPUs)
-f <file>     File containing list of patterns, one per line
              (Use "-" as the file name for stdin)
-o <file>     Write pattern matches to <file>
-s <file>     Seed random number generator from <file>

```

Vanitygen is written in C, and is provided in source code form and 
pre-built Win32 binaries.  At present, vanitygen can be built on Linux, 
and requires the openssl and pcre libraries.

Vanitygen can generate regular bitcoin addresses, namecoin addresses, 
and testnet addresses.

Vanitygen can search for exact prefixes or regular expression matches.  
When searching for exact prefixes, vanitygen will ensure that the 
prefix is possible, will provide a difficulty estimate, and will run 
about 30% faster.  Exact prefixes are case-sensitive by default, but 
may be searched case-insensitively using the "-i" option.  Regular 
expression patterns follow the Perl-compatible regular expression 
language.

Vanitygen can accept a list of patterns to search for, either on the 
command line, or from a file or stdin using the "-f" option.  File 
sources should have one pattern per line.  When searching for N exact 
prefixes, performance of O(logN) can be expected, and extremely long 
lists of prefixes will have little effect on search rate.  Searching 
for N regular expressions will have varied performance depending on the 
complexity of the expressions, but O(N) performance can be expected.

By default, vanitygen will spawn one worker thread for each CPU in your 
system.  If you wish to limit the number of worker threads created by 
vanitygen, use the "-t" option.

The example below completed quicker than average, and took about 45 sec 
to finish, using both cores of my aging Core 2 Duo E6600:
```bash
$ ./vanitygen 1Love
Difficulty: 4476342
[48165 K/s][total 2080000][Prob 37.2%][50% in 21.2s]                           
Pattern: 1Love
Address: 1LoveRg5t2NCDLUZh6Q8ixv74M5YGVxXaN
Privkey: 5JLUmjZiirgziDmWmNprPsNx8DYwfecUNk1FQXmDPaoKB36fX1o
```

Currently, it is difficult to import the private key into bitcoin.  
Sipa's showwallet branch has a new command called "importprivkey" that 
accepts the base-58 encoded private key.  Vanitygen has been tested to 
work with that version of bitcoin.
