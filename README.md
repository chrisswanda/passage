# passage - age based password/secrets manager

A simple password manager using [age](https://github.com/FiloSottile/age) written in POSIX `bash`. Based on [pash](https://github.com/dylanaraps/pash) by [dylanaraps](https://github.com/dylanaraps). I forked this project from [pa](https://github.com/biox/pa) by [biox](https://github.com/biox/).  Also, this implementation of passage has nothing to do with [passage](https://github.com/stchris/passage) which was based on Rust and that project appears to be archived.

- Automatically generates an `age` key if one is not detected.
- Only `120~` LOC (*minus blank lines and comments*).
- Configurable password generation using `/dev/urandom`.
- Guards against `set -x`, `ps` and `/proc` leakage.
- Easily extendible through the shell.
- Ability to edit passwords using `$EDITOR`

I am just a ham fisted knucklehead and have never claimed to be a developer.  I have been a long time user of [pass](www.passwordstore.org) and have been following [age](https://github.com/FiloSottile/age) for quite some time.  I have been waiting for Age and Pass to get together at some point, so when I saw `pa` as a pass+age type password manager, figured I would mess around with it for my own purposes.

Changes thus far for my usage are: using `.passage` for storage, I use `~/.config/age` to store my keypairs since they are synced via my personal git repo across multiple machines and `age-keygen` is a password protected file because I do not want to sync my keypairs in plain text, since MacOS does not use `/dev/shm` I added a random `/tmp` directory using OpenSSL, and I am using `pbcopy` since I run MacOS.

I'm also throwing in a few scripts that I have used in the past for age encryption and decryption, as I have integrated `age` into my day to day usage.  The gist can be seen here also -> https://gist.github.com/chrisswanda/bc537f87df7ab958773b3dab2d8f1f44


## Dependencies

- `age`
- `age-keygen`
- `oathtool` 

## Usage

Examples: 
```
  passage show github
  passage copy Travel/Uber
  passage otp  Development/Github
  passage list
  passage add Web/gmail
  passage edit Finance/ETrade
  passage del Social/Facebook
  passage git {pull}{push}{status}
  ```

```
USAGE

- show [name]    - Show password for an entry.
- copy [name]    - Copy password to clipboard. Clears in 30 seconds.
- otp  [name]    - Copy OTP to clipboard. Clears in 30 seconds.
- list           - List all entries.
- add  [name]    - Create a new password, randomly generated.
- edit [name]    - Edit a password entry with vim.
- del  [name]    - Delete a password entry.
- git  [command] - push, pull, status, add, commit
```

I have included something that resembles autocomplete.
```
$ passage {tab}
Travel/ Web/ Social/ Foo/ add  copy  del  edit  git  list  otp  show
```
Add this to your autocomplete directory.

`$ cp passage_autocomplete /usr/local/etc/bash_completion.d/passage_autocomplete`

Then you can source it `$source /usr/local/etc/bash_completion.d/passage_autocomplete`

or add it to your `~.bashrc`

`[[ -r "/usr/local/etc/bash_completion.d/passage_autocomplete" ]] && source "/usr/local/etc/bash_completion.d/passage_autocomplete"`


## FAQ

### Where are passwords stored?

The passwords are stored in `age` encrypted files located at `${XDG_DATA_HOME:=$HOME/}.passage}`.

### How does the copy command work?

The copy command takes the very first line of your passage entry, and copies it to your clipboard.

For example, here is an entry for Foo/bar

```
$ passage show Foo/bar
Enter passphrase for identity file "{your age private key location}":
4cWLle2RB2hPDFMkw
login: my_user_name
URL: www.example.com
Notes: free form notes

Recovery keys:
blah
cruft
things
```
When you run the `copy` command:

```
$ passage copy Foo/bar
Enter passphrase for identity file "{your age private key location}":
Clearing clipboard in 30 seconds.

$ 4cWLle2RB2hPDFMkw
```

### How do I change the password store location?

Set the environment variable `PASSAGE_DIR` to a directory.

```sh
# Default: '~/.passage'.
export PASSAGE_DIR=~/.passage

```
Or you can set it to whatever directory you want:
```sh
export PASSAGE_DIR=~/.local/some_other_dir
```

### Any other environment variables?

```
You can change the password length
# Password length:   export PASSAGE_LENGTH=21

And you can set your password characters
# Password pattern:  export PASSAGE_PATTERN=_A-Z-a-z-0-9
```

### How can I rename my passwords?

You can just drop into your $PASSAGE_DIR, and merely just rename the file.  `$mv test_file.age new_test_file.age` Your passwords are just files stored in a directory, so use any POSIX commands that you would use to manage any files normally.  Do not forget to name your files with an *.age extention.  

### How can I extend `passage`?

A shell function can be used to add new commands and functionality to `passage`. The following example adds `passage git` to execute `git` commands on the password store.

```sh
passage() {
    case $1 in
        g*)
            cd "${PASSAGE_DIR:=${XDG_DATA_HOME:=$HOME/}.passage}"
            shift
            git "$@"
        ;;

        *)
            command passage "$@"
        ;;
    esac
}
```

### What if I want to try out your version of `passage`?

Just note that I made this for my MacOS environment.  If you are using some other linux distro, you will need to make a few tweaks.

- For pw_edit(), I am copying to my `$TMPDIR` since MacOS does not have a `/dev/shm` and I sure as hell don't want to make a ram drive.
- For pw_copy(), I am using `pbcopy`.  For your linux environment, you can use `xclip`.
- I am using a password protected private key for my age credentials.  Granted, my hard drive is encrypted and if someone is on my local machine, I have bigger issues.  But, since I sync `~/.config/age` to my personal git repo, I figured might as well keep this key protected since age does not offer forward security.  To generate your password protected age credentials use `age-keygen | age -p > private_key`.
```
    age-keygen | age -p > ~/.config/age/username.priv.key

    Public key: age16wm8r7a6hzghjcqpze4302jwthvwrux46ud78zj9fsjn4c9eyp3qljm0gn
    Enter passphrase (leave empty to autogenerate a secure one): xxxxxxxx
    Confirm passphrase: xxxxxxxx
```
   I take the output of my public key and put it into a file named username.pub.key and put it in my ~/.config/age directory.

 ```echo "age16wm8r7a6hzghjcqpze4302jwthvwrux46ud78zj9fsjn4c9eyp3qljm0gn" > ~/.config/age/username.pub.key```

