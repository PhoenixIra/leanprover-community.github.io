# Lean Tips and Tricks

This document contains some tips for intermediate and advanced users of Lean.
It is currently focussed around Lean 3.

### Commenting out large proofs using `sorry`

Sometimes commenting out large chunks of a proof is helpful, while the default comment characters are `--` or `/- multi-line comment-/`, however often a more convenient way is to temporarily skip a long subproof (of a focussed goal surrounded by braces) by prepending `sorry;`
```
sorry; { rw blah,
  ...
  HUGE PROOF,
  ... }
```
With this method only one line needs to be changed, syntax highlighting is usually preserved, and jumping to definition still works. It also has the advantage that it is extremely easy to remove all instances of `sorry;` from a file when finished using find and replace.

### Memory and deterministic timeout
Lean has two configurable flags for configuring its resource usage, memory and execution time.
While the defaults for these are usually fairly sensible, when writing proofs it can be helpful to increase these options, if deterministic timeouts or out of memory issues slow down development.
This is especially helpful when using tactics like `library_search` that even if they take a long time will be replaced by a faster tactic if they succeed.

To change these settings we can add the flags `-M  <some number of megabytes>` (default 4096) for the memory limit and `-T <number of allocations>` (default 100000).
In VSCode this can be configured by opening the settings and searching for `Lean Memory Limit` and `Lean Time Limit`.


### Old mode
If you change a file early on in the import hierarchy and then attempt to work on a file that depends on it much later, then Lean will try to check all files in between to ensure they didn't break.
This behaviour can cause the Lean server to become unresponsive, or simply slow down development.
You can call lean with the "--old" flag to help in these situations, though it does come with some caveats.
In VSCode you can enable it by opening the VSCode settings and searching for "Lean: extra options" then add the flag `--old`.
This causes lean not to recompile oleans for unedited files and just assume everything worked, so in this situation it is helpful to avoid recompilation.
However you do then run the risk that your change broke or changed the meaning of some file in-between. Thus it most applicable when you are adding a new lemma/definition early in the hierarchy that you are sure won't break things in between, rather than changing existing definitions.
It is possible to "prove false" running lean with the old flag on, by changing a definition in an early file later results about that definition may become incorrect if Lean is asked not to rebuild those files.
The old flag should therefore not be used when consistency of the whole project is important (e.g. in CI), though it can speed up development.

### The `#exit` command and `.`

The command `#exit` will cause lean to stop reading the file at the location it is inserted.
This can be helpful when working on proofs in the middle of a file with many results or slow results below the current location.
This is related to switching to `check visible lines` mode, but gives the user more precise control over which parts of the file are slow.

Likewise when working at the start of a file Lean will recheck the import lines, this can be prevented by placing a `.` character (which delimits lean declarations) on a newline after the import lines.

### Profiling proofs

Sometimes long proofs are slow to execute and isolating the slow tactic or subgoals in an attempt to refactor or speed them up is necessary.
The basic option to enable this is `set_option profiler true`, which will print out timing information for tactic execution.
Commenting out subproofs (with the sorry trick above) you can isolate the slow part of the proof and explore options for rewriting it.
