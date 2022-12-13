# CMPE 283 Assignment 3

## 1. This assignment was done by 

-	Nohar Reddy Gurrala (016707821)
-	Rama Sai Gurunadh Dara (016709251)

<br/>

Nohar Reddy Gurrala
-	Built the kernel
-	Installed the necessary kernel modules in the virtual machine
-	Included the discussed changes in the code
-	Reviewed the lecture on canvas and comprehended the procedures to be followed
-	Created documentation and updated README.md

Rama Sai Gurunadh Dara
-	Built the kernel
-	Researched and understood the assignment's code
-	Researched about CPU leaf nodes and CPU ID instructions
-	Recognized where to make changes and what modifications to make to carry out for the assignment
-	Created and compiled test files


<br />

Assignment 3 modifies the behavior of the cpu_id instruction for the following cases:

<br />

 For CPUID leaf node %eax=0x4FFFFFFE:
 
 -	Return the number of exits for the exit number provided (on input) in %ecx
 	
	-	This value should be returned in %eax 


For CPUID leaf node %eax=0x4FFFFFFF:

 -	Return the time spent processing the exit number provided (on input) in %ecx

	-	Return the high 32 bits of the total time spent for that exit in %ebx
	
	-	Return the low 32 bits of the total time spent for that exit in %ecx
 
 
<br /> 
 
## 2.	The steps used to complete the assignment.

-	This assignment has been performed in GCP (Google Cloud Platform).

-	Steps to perform for creation of assignment were.,

	1.	Create a GCP account and a project (if you haven't already)

	2.	In the local system, create an SSH key (helps in establishing the connection with VM)

```bash
~ ssh-keygen -t ed25519
```

After generating SSH copy the public key (which has .pub extension file), which is required while creating the VM.

Create of VM in GCP

```bash
gcloud compute instances create instance-1 
--project=mythical-legend-367621 
--zone=us-central1-a 
--machine-type=n2-standard-8 
--network-interface=network-tier=PREMIUM,subnet=default 
--metadata=ssh-keys=spartan:ssh-ed25519\ AAAAC3NzaC1lZDI1NTE5AAAAINOVrGHRedev1CrZOhCl96Y6qXv1zDiajEeM8T4Ukr8K\ spartan@IMS-063MBA.local 
--maintenance-policy=MIGRATE 
--provisioning-model=STANDARD 
--service-account=620889689365-compute@developer.gserviceaccount.com 
--scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append 
--create-disk=auto-delete=yes,boot=yes,device-name=instance-1,image=projects/ubuntu-os-cloud/global/images/ubuntu-1804-bionic-v20221018,mode=rw,size=10,type=projects/mythical-legend-367621/zones/us-central1-a/diskTypes/pd-ssd 
--no-shielded-secure-boot 
--shielded-vtpm 
--shielded-integrity-monitoring 
--reservation-affinity=any 
--min-cpu-platform "Intel Cascade Lake" 
--enable-nested-virtualization

```

<br />

Configuration:
    -	`Machine Type: n2-standard-8`
    <br />
    -	`CPU platform: Intel Cascade Lake`
    <br />
    -	`Enabling Nested Virtualization`
    <br />
    -  	`OS – Ubuntu`
    <br />
    -	`RAM – 32GB`
    <br />
    -	`ROM – 250GB`
<br />

Let’s start with fork the code into the Git account from https://github.com/torvalds/linux



After creating VM, SSH into the VM 

1.	Clone the repository from your repo

``` bash
~ git pull https://github.com/RamDara07/linux
```

2.	
``` bash
 ~  sudo apt-get install wget
 ```


3.	Install all packages required for building the kernel.
	``` bash
	~  sudo apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
	```
	
4.	Configure the kernel by copying an existing configuration file to a .config file.
``` bash
	~  cd linux
	~  cp -v /boot/config-$(uname -r) .config
```
5.	Build the kernel by using the following:
``` bash
	~  sudo make modules
	~  sudo make
```

<br />

While compiling the kernel, an error may occur saying No rule to make target `'debian/canonical-certs.pem'`, then execute the below commands:

``` bash
scripts/config --disable SYSTEM_TRUSTED_KEYS
scripts/config --disable SYSTEM_REVOCATION_KEYS
```

<br />

``` bash
~  sudo make modules_install

~ sudo make install
```


<br />

6.	Bootloader is automatically updated because of the make `install` and now we can `reboot` the system and `check` the kernel version


``` bash
sudo reboot
```

<br />

	`uname -mrs`	
	Output: Linux


<br />


7.	Make code changes in the Linux kernel code. Replace the files below with the files from this repository.
-	`vmx.c: /linux-6.0.7/arch/x86/kvm/vmx/vmx.c`
-	`cpuid.c: /linux-6.0.7/arch/x86/kvm/cpuid.c`

<br />

8.	Built and install modules again


-	`sudo make modules && sudo make modules_install`

<br />

9.	To load the newly created modules, run the following commands:

``` bash
sudo rmmod kvm_intel
sudo rmmod kvm
sudo modprode kvm
sudo modprobe kvm_intel
```

<br />

10.	To test the functionality, a nested virtual machine was created inside the GCP virtual machine. The steps to create a `nested virtual machine` are as follows:
-	Download the Ubuntu cloud image.img (QEMU compatible image) file from this `[ubuntu cloud images site]` in GCP VM (https://cloud-images.ubuntu.com/).

<br />

11.	
``` bash
	wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
```

<br />

12.	`Install` the required qemu packages
``` bash
sudo apt update && sudo apt install qemu-kvm -y
```

<br />


Move to the directory where .img file is downloaded. This ubuntu cloud image does not come with a default password for the vm. 

So, to change the password and login into the vm, perform these following steps(terminal):

1) 
```  bash
sudo apt-get install cloud-image-utils
```

<br />

2) 
```bash
	cat >user-data <<EOF
   	#cloud-config
   	password: newpass #new password here
    	chpasswd: { expire: False }
    	ssh_pwauth: True
    	EOF
```

<br />

3) 
```bash
cloud-localds user-data.img user-data

```

<br />

-	Now, to run this ubuntu image, execute this:
```bash
sudo qemu-system-x86_64 -enable-kvm -hda bionic-server-cloudimg-amd64.img -drive "file=user-data.img,format=raw" -m 512 -curses -nographic
```



<br />

-	**username**: `ubuntu`


-	**password**: `newpass`

<br />

-	Now to check the functionality, we need the cpuid utility package, for that

```bash
sudo apt-get update
sudo apt-get install cpuid

```
•	NOTE: Make sure two terminals are open:
o	the GCP VM terminal(T1)
o	the nested VM terminal(logged in)(T2)

<br />


## 3.	Comment on the frequency of exits – does the number of exits increase at a stable rate? Or are there more exits performed during certain VM operations? Approximately how many exits does a full VM boot entail?

<br />

Is there a steady growth in the number of exits?

- No. depending only on the call for instruction and the timing 

Are some VM actions associated with additional exits?

- Yes. Throughout the VM boot process. 

How many exits does a complete VM boot require?

- Varies based on the types of VM and distribution. I calculated it using my setup to be 4396839.

Exit(10) and Exit(12) Increase have been seen to occur steadily. As they are usually referred to, during our observation.


<br />

## 4.	Of the exit types defined in the SDM, which are the most frequent? Least?

- Of the exit types defined in the SDM, which are:

Most frequent

0x4ffffffe, Exit number 30. Total exits = 936899

EPT infraction The configuration of the EPT paging structures prevented an attempt to access memory using a guest-physical address.

Least frequent

- There are several basic exit reason(s) whose Total exits is 0. Second least exit reason is 29

 0x4ffffffe, Exit number 29. Total exits=2

```Javascript

0x4ffffffe, Exit number 30. Total exits = 936899		** Most Frequent **
0x4ffffffe, Exit number 49. Total exits = 167017
0x4ffffffe, Exit number 28. Total exits =  80101
0x4ffffffe, Exit number 10. Total exits =  42581
0x4ffffffe, Exit number 32. Total exits =  11016
0x4ffffffe, Exit number 48. Total exits =  10737
0x4ffffffe, Exit number  1. Total exits =  10591
0x4ffffffe, Exit number  7. Total exits =   7452
0x4ffffffe, Exit number 12. Total exits =   3887
0x4ffffffe, Exit number 31. Total exits =    221
0x4ffffffe, Exit number  0. Total exits =     11
0x4ffffffe, Exit number 54. Total exits =      9
0x4ffffffe, Exit number 18. Total exits =      7
0x4ffffffe, Exit number 55. Total exits =      3
0x4ffffffe, Exit number 29. Total exits =      2      		** Least Frequent **
0x4ffffffe, Exit number  2. Total exits =      0
0x4ffffffe, Exit number  8. Total exits =      0
0x4ffffffe, Exit number  9. Total exits =      0

```

