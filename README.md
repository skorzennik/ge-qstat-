# SGK `qstat+` PERL wrapper around GridEngine qstat

## Description

  `qstat+` is a `PERL` scripts to improve the output of `qstat` on a system running the Grid Engine as the job scheduler/load manager.
  
   Release 1.01 Jul 1 2024, version 5.5/2
   

## Conceptual Description

  Runs a set of `qstat` commands and parse their output to produce a _better_ output that `qstat`

  I will run one or more of the following commands:
```
 qstat -xml -r -ext -u USER
 qstat -xml -f -u USER
 qstat -j JID
 qstat -F ssd_res,ssd_free,ssd_total -q all.q@@ssd-hosts -u nobody
 ruptime
```

 `qstat+` will also compute some derived quantities, like fraction of CPU usage and memory usage.
 On the HPC cluster `qstat+` was developed for, we have implemented a memory reservation and a local SSD usage request, hence `qstat+` can return information related to both.
  ...

## Practical Considerations

 We run Altair's Grid Engine version 8.8.1 with a customized configuration, hence `qstat+` is tailored to that configuration.
 It is my intention (_in my spare time_) to clean `qstat+` to clearly identify what is specific to our configuration.
 Among other things:
 * we do not use the `all.q` queue, but use other queue with short names
 * our compute nodes are all called compute-NN-MM, hence `qstat+` truncates it to NN-MM
 * we implemented memory reservation
 * we have local SSD

`qstat+` can be run in different (exclusive) modes:
  1. show summary (simple, extended or compact)
  2. show all jobs
  3. show queued jobs
  4. show running jobs
  5. show extra jobs (dr, Eqw, t)
  6. show info on a specific job
  7. show node list for a (set of) job(s)
  8. show node(s) with high load
  9. show global cluster status
 10. show empty slots, -esx: expanded info
 11. show node(s) that are down
 12. show over-reserved jobs, i.e.: resMem/maxVMem > 2.5, & age > 1 hr
 13. show oversubscribed jobs, i.e.: cpu% > 133%, & age > 1 hr
 14. show inefficient jobs, i.e.: cpu% < 33%, & age > 1 hr
 15. show SSD usage
 16. show help
 17. show examples

 and in *debug* mode, for debugging & testing.
 
### Usage info

```
usage: qstat+ [mode] [options]
  where modes are (exclusive list):
    -s|-sx|-sc      show summary (simple, extended or compact)
    -a              show all jobs
    -q              show queued jobs
    -r              show running jobs
    -X              show extra jobs (dr, Eqw, t)
    -j      JID     show info on a specific job
    -nlist  JID     show node list for a (set of) job(s)
    -hiload N       show node(s) with high load
    -gc             show global cluster status
    -es[x]          show empty slots, -esx: expanded info
    -down           show node(s) that are down
    -ores           show overreserved jobs, i.e.: resMem/maxVMem > 2.5, & age > 1 hr
    -osub           show oversubscribed jobs, i.e.: cpu% > 133%, & age > 1 hr
    -ineff          show inefficient jobs, i.e.: cpu% < 33%, & age > 1 hr
    -ssd            show SSD usage
    -h|-help--help  show help
    -examples       show examples

where options are:
    -u USER       limit to the specified user, you can use "*" or "all" to list everybody's jobs
                  or a coma-separated list
    -njobs        show counts in no. of jobs
    -npes         show counts in no. of PEs (slots)
    -nqpe         show jobs/PEs not jobs/tasks for queued jobs
    -nqxx         show jobs/tasks/PEs for queued jobs
    -load         show the nodes' load
    -sua          show the nodes' used/avail no of slots
    -age          show the age of the jobs, i.e., elapsed time vs starting/submit time
    -cpu          show the amount of CPU used by running job(s)
    -cpu%         show the amount of CPU/age/#PE in % (job efficiency)
    -cpur         show the ratio CPU/AGE, not scaled by #PE
    -mem          show the mean memory usage for running jobs, the total requested memory for queued jobs, in GB
    -memx         show more memory info for running jobs: reserved, mean, vmem and maxvmem (slow)
    -memr         show more memory info for running jobs: reserved, mean, maxvmem and res/mxvmem (slow)
    -io           show the I/O usage
    -iow          show the I/O wait usage (slow)
    -ioops        show the IOPs usage (slow)
    +lic          show license info (default), although not in detailed outputs
    -lic          do not show license info

    -noheader     do not show the header
    -nofooter     do not show the footer
    -raw          print raw values (for easier parsing)
    -queue QSPEC  limit to jobs in queue QSPEC (RE ok)
    -oresF  VAL   change the overreserved factor to VAL
    -osubT  VAL   change the oversubscribed threshold to VAL, in %
    -osubE  VAL   set the oversubscribtion excess threshold to VAL, in hr x slots
    -ineffT VAL   change the inefficient threshold to VAL, in %
    -ageT   VAL   change the minimum age threshold to VAL, in hour

    -v|-verbose   set verbose mode
    -warn         show warnings
    -wide         wide output (132 cols, implies -notty)
    -notty        do not use the width of the terminal, as returned by stty
    -check-load   check the instantanous load (-hiload only)

shorthands:
    +a   is expanded to -a -u $USER -nofooter             show all your jobs
    +a%                 -a -u $USER -nofooter -age -cpu%  ibidem, with cpu on % of age
    +ax%                -a -u $USER -nofooter -age -cpu% -load -sua -mem -io
    +ar%                -a -u $USER -nofooter -age -cpu% -memr -io
    +r                  -r -u $USER -nofooter             show all your running jobs
    +r%                 -r -u $USER -nofooter -age -cpu%  ibidem, with cpu on % of age
    +rr%                -r -u $USER -nofooter -age -cpu% -memr -io -iow -ioops
    +rx%                -r -u $USER -nofooter -age -cpu% -load -sua -memx -io -iow -ioops
    +q                  -q -u $USER -nofooter             show all your queued jobs
    +q%                 -q -u $USER -nofooter -age        ibidem but show age
    +qx%                -q -u $USER -nofooter -age -mem   ibidem plus mem info
    +X                  -X -u $USER -nofooter             show all your extra jobs
    +n                  -notty -wide -noheader -nofooter

qstat+: Ver 5.5/2 - Jun 2024
```

#### Usage Examples

```
% qstat+ -examples

examples:
 qstat+ -a -u hpc                   show all of hpc's jobs
 qstat+ -r -cpu% -u hpc             show all of hpc's running jobs, cpu in %
 qstat+ -r -cpu% -load -sua -u hpc  show all of hpc's running jobs, cpu in %, and
                                the nodes' load and slot usage/availability
 qstat+ -q                          show all the queued jobs
 qstat+ -j 8683280,8683285          show info on specific job IDs
 qstat+ -nlist 8683280,8683285      show nodes list for specific job IDs
 qstat+ -hiload 1.5                 show nodes whose load is 1.5 greater than the number of slots used
 qstat+ -ineffT 50 -ineff           show jobs that are below a 50% efficiency threshold
 qstat+ -osubT 200 -ageT 48 -osub   show jobs that are above a 200% usage threshold and are older than 48 hours
 qstat+ -gc                         show the global cluster status
 qstat+ -es                         show the empty slots
 qstat+ -down                       show which nodes are down

        +ax% -u all                 is equiv to -a -u all -cpu% -load -sua -mem
                             etc... for the +XXX shorthands

qstat+: Ver 5.5/2 - Jun 2024
```
### Customization

To run this on a different GE cluster, one needs to redefine some of the variables near the top of the ``PERL`` script:
```
```

### Debugging & Testing

#### Explanation

 The flag `-debug DIR` directs `qstat+` to read from files, located in the directory `DIR`, instead of running `qstat` and parsing its output, hence these files must contain the output of the commands `qstat+` would have run. 
 Examples of such files are provided in the `debug.tgz` archive, where 6 examples are available, including one when using the original SGE, not AGE 8.8.1. 

 One creates these files as follows: 
```
  qstat -xml -r -ext -u \* > dbg/qstat-r.xml
  qstat -xml -f -u \*      > dbg/qstat-f.xml
```
or
```
  qstat -j JID             > dbg/qstat-j-JID.txt
```
with a specific job id;

or to show SSD usage and reservation:
```
  qstat -F ssd_res,ssd_free,ssd_total -q all.q@@ssd-hosts -u nobody > /tmp/qstat-ssd.txt
```
#### Examples

Using the provided files, you can do the following:

 1. First untar the archive
```
tar xf debug.tgz

```
 2. Run `qstat+` in debug mode

  * Examples 0
```
qstat+ -debug ex.0
qstat+ -debug ex.0 -j 2343013
qstat+ -debug ex.0 -j 2343013.1
```
  * Examples 1: various modes
```
qstat+ -debug ex.1
qstat+ -debug ex.1 -gc
qstat+ -debug ex.1 -es
qstat+ -debug ex.1 -esx
qstat+ -debug ex.1 -down
qstat+ -debug ex.1 -ssd
qstat+ -debug ex.1 -sx
qstat+ -debug ex.1 -q
qstat+ -debug ex.1 -r
qstat+ -debug ex.1 -X
```
  * Examples 2: specific large MPI job
```
qstat+ -debug ex.2
qstat+ -debug ex.2 -j 2437501
qstat+ -debug ex.2 -nlist 2437501
```
  * Examples 3: specific user, job array
```
qstat+ -debug ex.3
qstat+ -debug ex.3 +a% -u erickoch
qstat+ -debug ex.3 -j 2436756
qstat+ -debug ex.3 -j 2436756.1
qstat+ -debug ex.3 -j 2436756.3
```
   * Examples 4: different user, lots of jobs
```
qstat+ -debug ex.4 +a% -u ingushin
qstat+ -debug ex.4 -j 2435983
qstat+ -debug ex.4 -nlist 2435983 -u ingushin
```
   * SGE example
```
qstat+ -debug sge -geType sge +a% -u all
```

3. Comparison file:

The file `test-debug.log` illustrates the output from these commands.
Click on the file to view it and see examples of `qstat+` output.

---

