# Dotfiles management

This is [my](https://github.com/dcreager/) take on the "share your
dotfiles" craze.  Maybe it will be useful for you, too.

My first principles are:

1. I have several files that I need available under `$HOME`, on a variety
   of machines.

2. I'd like to use git to keep track of those files.

3. I need separate "collections" of dotfiles, though, so that I can
   publish the ones that don't contain any private information or keys,
   and can keep the private information in repos that only I can access.

Principle #3, in particular, means that I can't (easily) just have my
$HOME directory be a git repository.

Instead, the approach that I take is:

* Any directory named `$HOME/.dotfiles.*` is assumed to be a git
  repository containing dotfiles that you want to track.

* Within those repositories, any file or directory named `*.symlink`
  will be symlinked into `$HOME`, with the trailing `.symlink` removed.
  Any directory structure within the repositories will be maintained.

This repository contains a `dotfiles` script that takes care of
installing and removing these symlinks for you.  The intention is that
it will be one (of several) dotfile repositories that `dotfiles` will
manage.  (That way the `dotfiles` script in this repository will be
symlinked into `$HOME/bin` for you.)

The `dotfiles` script is written in straight-up Bourne shell, so it
should work on any system with a working `/bin/sh`.  (I.e., no
dependencies on Ruby, Python, Make, or anything like that.)

## Installation

To install the `dotfiles` script for the first time, do the following:

    $ cd ~
    $ git clone git://github.com/dcreager/dotfiles-base.git .dotfiles.base
    $ .dotfiles.base/bin/dotfiles.symlink install

That should create a `$HOME/bin/dotfiles` symlink for you.  From that
point on, you can just run `dotfiles` from anywhere, assuming that
`$HOME/bin` is in your `PATH`.

## Usage

* `dotfiles install`

    Recursively searches for any `*.symlink` file in any
    `$HOME/.dotfiles.*` directory, and symlinks them into `$HOME`.  It
    will output the names of any new symlinks that are created.

    If any of the symlinks can't be created because there's already a
    (non-symlink) file with the same name, you'll get a "collision"
    warning, and that symlink won't be created.  If you want to replace
    the existing regular file, delete it and then re-run `dotfiles
    install`.

* `dotfiles uninstall`

    Unlinks the symlinks that `dotfiles install` creates.  Just like
    with `dotfiles install`, we print out a collision warning if we
    expect to find a `dotfiles`-managed symlink but find a regular file
    instead.

    As a special case, we won't delete the symlink for the `dotfiles`
    script itself, since that would prevent you from being able to
    (easily) run the `dotfiles` script later.  If you really want to
    completely remove all symlinks, set the `FULL` environment variable:

        $ FULL=1 dotfiles uninstall

* `dotfiles list`

    Lists each of the symlink files in all of your dotfiles
    repositories, along with the location where the symlink for that
    file would be created.

* `dotfiles clean`

    A more thorough version of `dotfiles uninstall`.  The `uninstall`
    command only looks for symlinks that correspond to a `*.symlink`
    file in one of your dotfiles repositories.  If you remove
    `*.symlink` file, then `dotfiles uninstall` won't know to delete the
    (now stale and dangling) symlink from `$HOME`.  `dotfiles clean`
    will recursively look through the entire tree rooted under `$HOME`,
    looking for symlinks that point into a dotfiles repository, and
    deleting any symlink that points at a nonexistent file.  This can
    take quite awhile if you've got a large home directory.

    An alternative is to use the following pattern:

        [ assuming you're in one of your dotfiles repositories ]
        $ dotfiles uninstall
        $ git rm <file to delete>
        $ dotfiles install

    This allows `dotfiles uninstall` to (quickly) remove the
    about-to-be-stale symlink while it's still around.  Then you delete
    it, and reinstall everything that remains.  (This is why the default
    for `dotfiles uninstall` is to not uninstall the `dotfiles` script
    itself.)

## Additional repositories

As noted above, this repository only contains the `dotfiles` script
itself.  It might eventually might contain other scripts for handling
these kinds of dotfile symlinks, but it doesn't contain any of my actual
dotfiles.  If you're using zsh and git, you might want to check out my
[actual collection of
dotfiles](https://github.com/dcreager/dotfiles-public/), too.  And if
you're not, or if you don't like the choices I've made in configuring my
system, you can create your own `~/.dotfiles.public` repository and
still use the `dotfiles` script to manage it.
