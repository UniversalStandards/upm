# gen_pypi_map

The gen_pypi_map module which generates the `pypi_map.gen.go` file, which
contains the
    1. module -> package mapping for guessing
    2. package -> modules mapping for detecting if a module is already installed
    3. package -> download count mapping for stats

It does this in separate steps, and gives a CLI interface for the admin
to walk through them. To run this program, it is recommended that you
have your working directory (CWD) set to `internal/backends/python`.

## Step 1: download / update pypi download stats

The package download counts are needed for heuristics in the guess algorithm during the generate step, also for upm to sort search results with. The file
`download_stats.json` file contains these stats and are checked in to git.
The stats are downloaded from a public BigQuery table made available
by Pypi. To download the stats and update the `download_stats.json` file:

```bash
go run ./gen_pypi_map bq -gcp <gcp-project-name>
```

The gcp-project-name can be any replit gcp project, because the table we are accessing `bigquery-public-data.pypi.file_downloads` is public. More info here: <https://packaging.python.org/en/latest/guides/analyzing-pypi-package-downloads/>

## Step 2: test modules

Next we test the packages in pypi we want to be able to guess. The default test
is:

0. use pkgutil to see what modules exists before installing the package
1. install the package
2. use pkgutil to see what new modules were added compared to before

We used to run this test on all modules on pypi. Now we have the option to
run it only on a subset of modules. The default is to collect the top
10000 packages. You can change this by passing in a different number to the
optional `-threshold` flag.

For example, to test the top 50000 packages:

```bash
go run ./gen_pypi_map/ test -threshold 50000
```

The test results for all tested packages will be stored in `pkgs.json` along with
the versions of the packages tested. `pkgs.json` is checked in to git. The command
will only test a packages if it's not already in the `pkgs.json` file or if
its latest version was not the one previously tested. If you
want to force a retest of the packages, you can use the `-force` flag.

## Step 3: generate sqlite db

Finally, we use the collected data to generate the lookup database. This is done with:

```bash
go run ./gen_pypi_map/ gen
```
