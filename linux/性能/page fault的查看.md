## ps命令

ps命令有很强的定制性，可以查看page faults

```
ps -o  maj_flt
```

## \time与/usr/bin/time

```
# \time ls
1006210223.txt	bpf_program.o			    index.html	     out.stacks		      test.scap
1.txt		denyhosts-3.0			    index.html.1     perl-5.32.1	      test.stp
2.txt		denyhosts-3.0.tar.gz		    install_bbr.log  perl-5.32.1.tar.gz       vda1.blktrace.0
a.out		dump.rd_bak			    install.sh	     rpcsvc-proto-1.4
bbr.sh		FlameGraph			    jupyter-echarts  rpcsvc-proto-1.4.tar.gz
bpf_program.c	gnu-getopt-1.0.14-5.el7.noarch.rpm  mysql	     test.c
0.00user 0.00system 0:00.00elapsed 100%CPU (0avgtext+0avgdata 2608maxresident)k
0inputs+0outputs (0major+127minor)pagefaults 0swaps

```

```
Command being timed: "ls"
	User time (seconds): 0.00
	System time (seconds): 0.00
	Percent of CPU this job got: 100%
	Elapsed (wall clock) time (h:mm:ss or m:ss): 0:00.00
	Average shared text size (kbytes): 0
	Average unshared data size (kbytes): 0
	Average stack size (kbytes): 0
	Average total size (kbytes): 0
	Maximum resident set size (kbytes): 2600
	Average resident set size (kbytes): 0
	Major (requiring I/O) page faults: 0
	Minor (reclaiming a frame) page faults: 126
	Voluntary context switches: 1
	Involuntary context switches: 0
	Swaps: 0
	File system inputs: 0
	File system outputs: 0
	Socket messages sent: 0
	Socket messages received: 0
	Signals delivered: 0
	Page size (bytes): 4096
	Exit status: 0
```

