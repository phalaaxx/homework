Describe the procedure of replacing a HDD containing partition part of a volume group (LVM)
-------------------------------------------------------------------------------------------

Поради липса на достатъчно дискове на машината (лаптоп), процедурата ще бъде демонстрирана с loop устройства. Първо създаваме 2 файла с размери по 1 GB:

	bozhin@ankaa:~$ dd if=/dev/zero of=pv.1 bs=1M seek=1024 count=0
	0+0 records in
	0+0 records out
	0 bytes (0 B) copied, 0.000328939 s, 0.0 kB/s

	bozhin@ankaa:~$ dd if=/dev/zero of=pv.2 bs=1M seek=1024 count=0
	0+0 records in
	0+0 records out
	0 bytes (0 B) copied, 0.000236974 s, 0.0 kB/s

След това създаваме loop0 и loop1 върху файловете:

	bozhin@ankaa:~$ sudo losetup /dev/loop0 pv.1
	bozhin@ankaa:~$ sudo losetup /dev/loop1 pv.2

Създаваме volume група с име vgtest върху loop0:

	bozhin@ankaa:~$ sudo vgcreate vgtest /dev/loop0
	  No physical volume label read from /dev/loop0
	  Writing physical volume data to disk "/dev/loop0"
	  Physical volume "/dev/loop0" successfully created
	  Volume group "vgtest" successfully created

Във volume групата все още няма logical volumes:

	bozhin@ankaa:~$ sudo lvs
	  LV           VG    Attr     LSize   Pool Origin Data%  Move Log Copy%  Convert
	  boot         ankaa -wi-ao-- 128.00m                                           
	  d-boot       ankaa -wi-a--- 128.00m                                           
	  d-root       ankaa -wi-a---   2.00g                                           
	  d-tmp        ankaa -wi-a---   5.00g                                           
	  d-usr        ankaa -wi-a---   8.00g                                           
	  d-var        ankaa -wi-a---   8.00g                                           
	  home         ankaa -wi-ao-- 200.00g                                           
	  mongodb      ankaa -wi-a---  30.00g                                           
	  root         ankaa -wi-ao--   2.00g                                           
	  sphinxsearch ankaa -wi-a---  15.00g                                           
	  squeeze      ankaa -wi-a---  15.00g                                           
	  src          ankaa -wi-a---  20.00g                                           
	  swap         ankaa -wi-ao--   4.00g                                           
	  tmp          ankaa -wi-ao--   5.00g                                           
	  usr          ankaa -wi-ao--   6.00g                                           
	  var          ankaa -wi-ao--   6.00g                                           

Създаваме logical volume с име "data":

	bozhin@ankaa:~$ sudo lvcreate -n vgtest/data -L 1000M
	  Logical volume "data" created

Създаваме файлова система върху новия volume и я монтираме в /mnt:

	bozhin@ankaa:~$ sudo mkfs.ext4 /dev/vgtest/data
	mke2fs 1.42.5 (29-Jul-2012)
	Discarding device blocks: done                            
	Filesystem label=
	OS type: Linux
	Block size=4096 (log=2)
	Fragment size=4096 (log=2)
	Stride=0 blocks, Stripe width=0 blocks
	64000 inodes, 256000 blocks
	12800 blocks (5.00%) reserved for the super user
	First data block=0
	Maximum filesystem blocks=264241152
	8 block groups
	32768 blocks per group, 32768 fragments per group
	8000 inodes per group
	Superblock backups stored on blocks: 
		32768, 98304, 163840, 229376

	Allocating group tables: done                            
	Writing inode tables: done                            
	Creating journal (4096 blocks): done
	Writing superblocks and filesystem accounting information: done


	bozhin@ankaa:~$ sudo mount /dev/vgtest/data /mnt

Създаваме тестов файл с размер 900MB върху новата файлова система и (не особено необходимо) проверяваме md5 сумата за сравнение по-късно:

	bozhin@ankaa:~$ sudo dd if=/dev/urandom of=/mnt/data.bin bs=1M count=900
	900+0 records in
	900+0 records out
	943718400 bytes (944 MB) copied, 67.6184 s, 14.0 MB/s

	bozhin@ankaa:~$ md5sum /mnt/data.bin 
	0ef4ee096bd80573db7c507eca0fd12f  /mnt/data.bin

Добавяме loop1 към volume групата:

	bozhin@ankaa:~$ sudo vgextend vgtest /dev/loop1
	  No physical volume label read from /dev/loop1
	  Writing physical volume data to disk "/dev/loop1"
	  Physical volume "/dev/loop1" successfully created
	  Volume group "vgtest" successfully extended


	bozhin@ankaa:~$ sudo vgs
	  VG     #PV #LV #SN Attr   VSize   VFree  
	  ankaa    1  16   0 wz--n- 465.76g 139.51g
	  vgtest   2   1   0 wz--n-   1.99g   1.02g

Свободното пространство в групата е 1.02GB (1000MB вече са заети от vgtest/data). Проверяваме и как е разпределено пространството върху physical volumes:

	bozhin@ankaa:~$ sudo pvs
	  PV         VG     Fmt  Attr PSize    PFree   
	  /dev/loop0 vgtest lvm2 a--  1020.00m   20.00m
	  /dev/loop1 vgtest lvm2 a--  1020.00m 1020.00m
	  /dev/sda1  ankaa  lvm2 a--   465.76g  139.51g


Преместваме physical extents от loop0 върху loop1:

	bozhin@ankaa:~$ sudo pvmove /dev/loop0 /dev/loop1
	  /dev/loop0: Moved: 7.2%
	  /dev/loop0: Moved: 38.0%
	  /dev/loop0: Moved: 100.0%


Проверяваме резултата:

	bozhin@ankaa:~$ sudo pvs
	  PV         VG     Fmt  Attr PSize    PFree   
	  /dev/loop0 vgtest lvm2 a--  1020.00m 1020.00m
	  /dev/loop1 vgtest lvm2 a--  1020.00m   20.00m
	  /dev/sda1  ankaa  lvm2 a--   465.76g  139.51g

Тъй, като loop0 вече не съдържа physical extents - можем да го премахнем от VG:

	bozhin@ankaa:~$ sudo vgreduce vgtest /dev/loop0
	  Removed "/dev/loop0" from volume group "vgtest"

И в резултат на това свободното пространство в групата отново е 20MB:

	bozhin@ankaa:~$ sudo vgs
	  VG     #PV #LV #SN Attr   VSize    VFree  
	  ankaa    1  16   0 wz--n-  465.76g 139.51g
	  vgtest   1   1   0 wz--n- 1020.00m  20.00m

За да сме сигурни, че в процеса не сме повредили данните (въпреки, че този тест в случая не е много точен заради FS cache):

	bozhin@ankaa:~$ md5sum /mnt/data.bin 
	0ef4ee096bd80573db7c507eca0fd12f  /mnt/data.bin


За да почистим създадените файлове след края на теста:

	bozhin@ankaa:~$ sudo umount /mnt
	bozhin@ankaa:~$ sudo lvchange -an vgtest
	bozhin@ankaa:~$ sudo vgremove vgtest
	Do you really want to remove volume group "vgtest" containing 1 logical volumes? [y/n]: y
	Do you really want to remove active logical volume data? [y/n]: y
	  Logical volume "data" successfully removed
	  Volume group "vgtest" successfully removed
	bozhin@ankaa:~$ sudo losetup -d /dev/loop0
	bozhin@ankaa:~$ sudo losetup -d /dev/loop1
	bozhin@ankaa:~$ rm pv.1 pv.2

