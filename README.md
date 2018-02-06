### HugePage Configuration ###

	* Persistent huge pages are not enabled by default in most environments
	* The number of persistent huge pages on the host can be queried by checking /proc/meminfo. 

	~~~
	[root@locallaptop ~]# cat /proc/meminfo  |grep -i huge
	AnonHugePages:         0 kB
	ShmemHugePages:        0 kB
	HugePages_Total:       0
	HugePages_Free:        0
	HugePages_Rsvd:        0
	HugePages_Surp:        0
	Hugepagesize:       2048 kB
	[root@locallaptop ~]#
	~~~

	Sometimes 

	3) Lets check the meaning of each parameter 

	  3.1) `AnonHugePages` - refered as transparent huge pages // In first case there is `0` , transparent or annonymous huge pages. 
	  3.2) `HugePages_Total` - persistent huge page // In this case there is `0` Total huge pages are available. 

	4) Identifying contiguous areas of memory is a issue for all huge page sizes, but it is particularly problematic for larger huge page sizes such as 1 GB huge pages. 
	5) Allocating huge pages at boot time will ensure the correct number of huge pages is always available, while allocating them at run time can fail if memory has become too fragmented.
	6) To allocate huge pages at run time, the kernel boot parameters must be extended to include some huge page-specific parameters. 
	7) This can be achieved by modifying `/etc/default/grub` and appending the `hugepagesz`, `hugepages`, and `transparent_hugepages=never` arguments to `GRUB_CMDLINE_LINUX`.
	8) For example, To allocate,  2048 persistent 2 MB huge pages at boot time, run:
	~~~
 	echo 'GRUB_CMDLINE_LINUX="$GRUB_CMDLINE_LINUX hugepagesz=2M hugepages=2048 transparent_hugepage=never"' > /etc/default/grub
 	~~~
 	# cat /etc/default/grub
 	~~~
	GRUB_CMDLINE_LINUX="$GRUB_CMDLINE_LINUX hugepagesz=2M hugepages=2048 transparent_hugepage=never"
	~~~
	Important: Persistent huge pages are not usable by standard host OS processes. Ensure enough free, non-huge page memory is reserved for these processes.

	8) Reboot the host, then validate that huge pages are now available: wolla its worked. 
	~~~
	[root@localhost ~]# cat /proc/meminfo |grep Huge
	AnonHugePages:    212992 kB
	HugePages_Total:       0
	HugePages_Free:        0
	HugePages_Rsvd:        0
	HugePages_Surp:        0
	Hugepagesize:       2048 kB
	[root@localhost ~]# 
	~~~

	9) In this instance, the huge pages are mounted at /dev/hugepages. This mount point varies from platform to platform. If the above command did not return anything, the hugepages must be mounted manually. To mount the huge pages at /dev/hugepages, run:
	~~~
	[root@localhost ~]# mount |grep huge
	hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,seclabel)
	[root@localhost ~]# 
	~~~
	~~~
	# mkdir -p /dev/hugepages
	# mount -t hugetlbfs hugetlbfs /dev/hugepages
	~~~

	10) By default, an instance does not use huge pages for its underlying memory. , However, huge pages can bring important or required performance improvements for some workloads. 
	~~~
	$ openstack flavor set m1.large --property hw:mem_page_size=large
	~~~

	11) Different platforms offer different huge page sizes. For example: x86-based platforms offer 2 MB and 1 GB huge page sizes.
	~~~
	# openstack flavor set m1.large --property hw:mem_page_size=2Mb
	~~~
	OR 
	~~~
	openstack flavor set m1.large --property hw:mem_page_size=2048
	~~~

	# Note

	Explicitly requesting any page size will still result in a NUMA topology being applied to the instance, as described earlier in this document.
	~~~
	openstack flavor set m1.large --property hw:mem_page_size=any
	~~~

	12) To configure an image to use 1 GB huge pages, run:
	~~~
	openstack image set [IMAGE_ID]  --property hw_mem_page_size=1GB
	~~~

	13) 
