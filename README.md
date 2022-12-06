# CMPE 283 Assignment 2

1.

This assignment was done by 

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

## Assignment 2 modifies the behavior of the cpu_id instruction for the following cases:


For CPUID leaf node %eax=0x4FFFFFFC:
    -	Return the total number of exits (all types) in %eax


For CPUID leaf node %eax=0x4FFFFFFD:
    -	Return the high 32 bits of the total time spent processing all exits in %ebx
    -	Return the low 32 bits of the total time spent processing all exits in %ecx
    -	%ebx and %ecx return values are measured in processor cycles, across all VCPUs
 
 
 
2.

This assignment has been performed in GCP (Google Cloud Platform).

Steps to perform for creation of assignment were.,

1.	Create a GCP account and a project (if you haven't already)

2.	In the local system, create an SSH key (helps in establishing the connection with VM)

```
~ ssh-keygen -t ed25519
```

After generating SSH copy the public key (which has .pub extension file), which is required while creating the VM.

Create of VM in GCP

```
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

``` 
~ git pull https://github.com/RamDara07/linux
```

2.	
```
 ~  sudo apt-get install wget
 ```

3.	
```
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.0.7.tar.xz 
```

4.	`tar xvf linux-6.0.7.tar.xz`

5.	Install all packages required for building the kernel.
	```
	~  sudo apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
	```
	
6.	Configure the kernel by copying an existing configuration file to a .config file.
```
	~  cd linux-6.0.7
	~  cp -v /boot/config-$(uname -r) .config
```
7.	Build the kernel by using the following:
```
	~  sudo make modules
	~  sudo make
```
<br />

While compiling the kernel, an error may occur saying No rule to make target `'debian/canonical-certs.pem'`, then execute the below commands:
```
scripts/config --disable SYSTEM_TRUSTED_KEYS
scripts/config --disable SYSTEM_REVOCATION_KEYS
```

<br />

```
~  sudo make modules_install
```
<br />
```
~ sudo make install
```
<br />

8.	Bootloader is automatically updated because of the make `install` and now we can `reboot` the system and `check` the kernel version
```
sudo reboot
```
<br />
	`uname -mrs`
-	Output: Linux 6.0.7 x86_64

<br />
9.	Make code changes in the Linux kernel code. Replace the files below with the files from this repository.
-	`vmx.c: /linux-6.0.7/arch/x86/kvm/vmx/vmx.c`
-	`cpuid.c: /linux-6.0.7/arch/x86/kvm/cpuid.c`

<br />

10.	Built and install modules again
-	`sudo make modules && sudo make modules_install`

<br />
11.	To load the newly created modules, run the following commands:
```
~	sudo rmmod kvm_intel
~	sudo rmmod kvm
~	sudo modprode kvm
~	sudo modprobe kvm_intel
```

<br />

12.	To test the functionality, a nested virtual machine was created inside the GCP virtual machine. The steps to create a `nested virtual machine` are as follows:
-	Download the Ubuntu cloud image.img (QEMU compatible image) file from this `[ubuntu cloud images site]` in GCP VM (https://cloud-images.ubuntu.com/).

<br />

13.	
```
	wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
```

<br />

14.	`Install` the required qemu packages
```
sudo apt update && sudo apt install qemu-kvm -y
```
<br />

Move to the directory where .img file is downloaded. This ubuntu cloud image does not come with a default password for the vm. 

So, to change the password and login into the vm, perform these following steps(terminal):

1) 
``` 
sudo apt-get install cloud-image-utils
```
<br />
2) 
```
	cat >user-data <<EOF
   	#cloud-config
   	password: newpass #new password here
    	chpasswd: { expire: False }
    	ssh_pwauth: True
    	EOF
```
<br />

3) 
```
cloud-localds user-data.img user-data

```
<br />
-	Now, to run this ubuntu image, execute this:
```
sudo qemu-system-x86_64 -enable-kvm -hda bionic-server-cloudimg-amd64.img -drive "file=user-data.img,format=raw" -m 512 -curses -nographic
```
-	**username**: `ubuntu`
	<br />
-	**password**: `newpass`
	<br />
-	Now to check the functionality, we need the cpuid utility package, for that

```
sudo apt-get update
sudo apt-get install cpuid

```
•	NOTE: Make sure two terminals are open:
o	the GCP VM terminal(T1)
o	the nested VM terminal(logged in)(T2)

