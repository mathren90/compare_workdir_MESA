
While experimenting with [MESA](http://mesa.sourceforge.net/) and developing setups, I find myself
rather often comparing inlists (for single and/or binary stars) with
each other. A simple diff is often not particularly informative
because of the comments, non-matching order of the entries, or missing
entries in one file that are present in the other file but have no
real effect because they are set to the defaults. For this reason,
I've written the script `compare_inlists.py` which can print a
line-by-line diff of two inlists, ignoring comments and empty lines,
and checking the defaults from the documentation when entries are
missing. It *assumes you use the same MESA version* for this last task.

This can be used inside of scripts or notebooks, or from command line, thanks to [`click`](https://github.com/pallets/click).


# How to use `compare_inlists.py`


```
python compare_inlists.py --help
Usage: compare_inlists.py [OPTIONS] INLIST1 INLIST2

Options:
  --pgstar TEXT    Show also diff of pgstar namelists.
  --mesa_dir TEXT  use customized location of $MESA_DIR. Will use environment
                   variable if empty.
  --vb TEXT        Show also matching lines using green.
  --help           Show this message and exit.
```

It uses [`termcolor`](https://pypi.org/project/termcolor/) to print in red entries that differ between
two inlists, and, if invoked adding `--vb=True` (for verbose) from the command
line it will also print in green entries that are equal. If one entry
is in one inlist but not the other, but the inlist that contains it 
has the default value, it is considered as a "green" entry.

This has been tested with MESA version 12778, and works for inlists
for single stars and binaries (but will refuse to compare input for a
single star and for a binary with each other). It can take a
customized `$MESA_DIR` with the optional argument `--mesa_dir`. If not
provided it will use the `$MESA_DIR` environment variable currently
set.

By default the comparison between pgstar namelist is disabled because
I need it less, but it can be enabled using `--pgstar=True`.

# Example

A screenshot of an example with `--vb=True` and `$MESA_DIR` not set as
environment variable, but passed as command line option:

![example](/examples/verbose.png?raw=true "verbose output")


# How to use `compare_all_work_dir_inlists.py`

MESA allows to nest namelists (i.e., star_job, controls, and/or
pgstar, and their binary counterparts) using `read_extra_star_job_inlist*` and
`extra_star_job_inlist*_name` (and similar). The script `compare_all_work_dir_inlists.py` uses
the functions defined in `compare_inlists.py`to compare the entire MESA
setup of two work directories. 

It will not work to compare a work directory for a single star with a
work directory for a binary. However, the functions in the
`compare_inlists.py` and `compare_all_work_dir_inlists.py` could be
used to create a comparison (e.g., of a single star with the donor of
a binary), it's just not ready yet.

As `compare_inlists.py`, it can be used inside of scripts or notebooks, or from command line,
thanks to [`click`](https://github.com/pallets/click).

```
python compare_all_work_dir_inlists.py --help
Usage: compare_all_work_dir_inlists.py [OPTIONS] WORK_DIR1 WORK_DIR2

Options:
  --pgstar TEXT    Show also diff of pgstar namelists.
  --mesa_dir TEXT  use customized location of $MESA_DIR. Will use environment
                   variable if empty and return an error if empty.
  --vb TEXT        Show also matching lines using green.
  --help           Show this message and exit.
```

If called on a pair of folders both for single stars (or both for
binaries), it first builds a "master" dictionary with of all the
options MESA reads for each namelist, starting from `inlist` and
checking if it contains nested namelists. It then performs the
comparison of the "master" dictionaries of the two folders. If the
same `read_extra_star_job_inlist*` (i.e. same number instead of the
`*`) appears in multiple nested inlists the last read will over-write
the previous, which is the same behavior as in MESA. Same for controls
and pgstar and the corresponding binary namelists. In the case of
binary folders, it will also compare all the namelists for each of the
stars in the binary. The comparison between the pgstar namelists is
also disabled by default and can be done using `--pgstar=True` from
command line.

The output is similar to the example above for individual inlists.
