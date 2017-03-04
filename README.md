CentOS LiveCD customization role for Ansible
============================================
This role modifies a CentOS livecd image, enables sshd and injects a provided
ssh key so that the livecd can be accessed remotely. It probably works with
other RHEL based livecd images.

The role downloads the iso image, if it doesn't exist, to a cache directory.
It unpacks the two filesystem layers in the iso using libguestfs and squashfs
into a temporary working directory where the OS files are modified. The iso is
then repacked into the cache directory with a different name.
The required tools are installed by this role using Ansible's package module.

The variable **livecd_pubkey** needs to be set as its content is written to the
root user's authorized_keys file.


Details
-------
The input iso is extracted to *livecd_workdir*/iso after which the squashed
filesystem is extracted to *livecd_workdir*/squashfs. From here the LiveOS ext3
filesystem is mounted (read+write) to *livecd_workdir*/liveos. The SSHD
service is enabled by creating a symlink and the *livecd_pubkey* is injected to
the root user. The ext3 fs is then unmounted, squashed and the image is
recreated as a bootable iso image. 

Optionally (enabled by default) PXE bootable files are created to
*livecd_cache_dir*/tftpboot/ dir using the livecd-tools's livecd-iso-to-pxeboot
script. This script embeds the livecd iso content into initrd0.img which is
also extracted from the livecd image.

Variables:

 - livecd_pubkey: SSH pubkey. *Needs to be set!*
 - livecd_input_iso: Filename for the input LiveCD iso file
 - livecd_output_iso: Filenanme for the ouput iso file
 - livecd_url: URL for the iso file
 - livecd_cache_dir: Cache directory for input and output iso files
 - livecd_workdir: Temporary working directory
 - livecd_tools_state: latest/present (def: latest)
 - livecd_input_iso_checksum: Use Ansible's get_uri syntax (algo:checksum)
 - livecd_output_pxe: Set to false to disable PXE extraction (tftpboot dir)
 - livecd_use_fast_compression: Switch between fastest gzip or tightest LZMA2
 - livecd_delegate_to: delegate_to host (omitted by default)

Notes
-----
The tools have only been successfully tested on Fedora. Bugs were encountered
on CentOS 7 with the older version of the tools used.
The fast compression should be used for testing purposes only. The output 
initrd0.img can be in that case too large for PXE booting, at least when using
the iPXE bootloader.


Why?
----
This role can be used when PXE booting servers like for example with the role
[ansible-role-dhcp-kickstart][1]. A server can be temporarily booted into the
livecd and the host's raid configuration can be nuked that way by installing
the required tools and commanding it from Ansible over ssh.
But why? HP's cli tools only work a limited amount of distros and none
of the LiveCDs have sshd enabled by default for security reasons.

[1]: https://github.com/SoneraCloud/ansible-role-dhcp-kickstart/
