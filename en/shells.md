# Shells

Perhaps a bit unique to Nu is the concept of `shells`, though the idea of working in multiple places at the same time in a shell is fairly common (via pushd/popd, screen, and more). What perhaps makes Nu a little different is that its `shells` concept is both multiple platform and it works both on the filesystem and inside of values.

**Note:** The concept of a Value Shell is one of the many open design points of Nu and is subject to change in the future.

A **Shell** is a filesystem-like interface that describes a set of common file operations and how to perform them, including: `cd`, `ls`, `mkdir`, `rm`, `cp`, and `mv`. Not all shell types support all file operations, but the file operations attempt to describe much of what a shell would want to perform.

The two types of Shells currently supported are FilesystemShell and ValueShell, though other shells have been discussed.

## Filesystem Shell

The filesystem shell is the shell that works directly with a filesystem and a corresponding paths. By default, Nu opens with a single filesystem shell in the current working directory.

```
> shells
━━━┯━━━━━━━━━━━━┯━━━━━━━━━━━━━━━━
   │ name       │ path 
───┼────────────┼────────────────
 X │ filesystem │ /home/jonathan 
━━━┷━━━━━━━━━━━━┷━━━━━━━━━━━━━━━━
```

We can add a new filesystem shell to this list by using the `enter` command. This will add a new shell+path combination to our ring buffer of shells.

```
> enter Source
/home/jonathan/Source> shells
━━━┯━━━┯━━━━━━━━━━━━┯━━━━━━━━━━━━━━━━━━━━━━━
 # │   │ name       │ path 
───┼───┼────────────┼───────────────────────
 0 │   │ filesystem │ /home/jonathan 
 1 │ X │ filesystem │ /home/jonathan/Source 
━━━┷━━━┷━━━━━━━━━━━━┷━━━━━━━━━━━━━━━━━━━━━━━
```

In this way, you can support jumping between working directories.  Note that currently Nu doesn't support jumping between running applications, only working directories.
