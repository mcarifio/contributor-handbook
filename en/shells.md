# Shells

Perhaps a bit unique to Nu is the concept of `shells`, though the idea of working in multiple places at the same time in a shell is fairly common (via pushd/popd, screen, and more). What perhaps makes Nu a little different is that its `shells` concept is both multiple platform and it works both on the filesystem and inside of values.

**Note:** The concept of a Value Shell is one of the many open design points of Nu and is subject to change in the future.

A **Shell** is a filesystem-like interface that describes a set of common file operations and how to perform them, including: `cd`, `ls`, `mkdir`, `rm`, `cp`, and `mv`. Not all shell types support all file operations, but the file operations attempt to describe much of what a shell would want to perform.

