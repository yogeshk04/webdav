# What is WebDAV?
Overview: Web Distributed Authoring and Versioning or WebDAV is a protocol whose basic functionality includes enabling users to share, copy, move and edit files through a web server. It can also be used to support collaborative applications with features like file locking and revision tracking.

In the earlier days of the Web, people could only read/view Web content. Of course, someone had to write/create the content offline and then upload it later on. But it wasn't possible to connect to a Web server, open a document, and then edit it online a la Google Drive or Office 365.

Realizing the potential of the Web for remote collaboration activities like distributed authoring, wherein several authors could collaborate on the same document even while working from different parts of the world, an IETF working group was formed to develop extensions to HTTP (Hypertext Transfer Protocol) that would enable such activities. The resulting HTTP extension was called WebDAV.

As an extension of HTTP, WebDAV is typically served through port 80 for HTTP connections or port 443 for HTTPS connections. WebDAV HTTPS connections are encrypted by SSL (Secure Sockets Layer), making them suitable for 
confidential documents.

Important link: https://savannah.nongnu.org/bugs/?63364
==================================== Starting of mounting ====================================
Step 1: Install davfs2
	sudo rpm-ostree install davfs2 (sudo dnf install davfs2)
	
	Step 1.1: Download davfs2 version 1.6.1
		curl https://kojipkgs.fedoraproject.org//packages/davfs2/1.6.1/2.fc37/x86_64/davfs2-1.6.1-2.fc37.x86_64.rpm --output davfs2-1.6.1-2.fc37.x86_64.rpm
		sudo rpm-ostree install davfs2-1.6.1-2.fc37.x86_64.rpm
		sudo systemctl reboot
Step 2: Check the version of davfs
	mount.davfs -V
	
	Output: davfs2 1.6.1  <http://savannah.nongnu.org/projects/davfs2> ....

Step 3: Add the jfrog Credential Line into davfs secrets file
	sudo vi /etc/davfs2/secrets
		Add the API key line: https://siemens.jfrog.io/artifactory/jfrog-logs/ nikam xxxxxxx

Step 4: Create mount service
	cd /etc/systemd/system
	sudo vi var-mnt-jfroglogs.mount
		[Unit]
		Description=Mount WebDAV service

		[Mount]
		What=https://siemens.jfrog.io/artifactory/jfrog-logs
		Where=/var/mnt/jfroglogs
		Type=davfs
		TimeoutSec=30
	
	save the file

Step 5: Create automount service
	sudo vi var-mnt-jfroglogs.automount
		[Unit]
		Description=Mount WebDAV Service
		DefaultDependencies=no

		Requires=NetworkManager.service
		After=network-online.target
		Wants=network-online.target

		[Automount]
		Where=/var/mnt/jfroglogs

		[Install]
		WantedBy=remote-fs.target
	
	save the file

Step 6: Enable and run the systemd service
	systemctl enable var-mnt-jfroglogs.automount
	systemctl start var-mnt-jfroglogs.automount
	systemctl status var-mnt-jfroglogs.automount
	systemctl status var-mnt-jfroglogs.mount
	
Step 7: Check the mount
	ls -ls /mnt/jfroglogs/
		output: 
		total 1
		drwxr-xr-x.   7 root root 224 Jul 28 11:52 .
		drwxr-xr-x. 149 root root   0 Jul 28 11:52 artifactory
		drwxr-xr-x.  82 root root   0 Jul 28 11:53 distribution
		drwx------.   2 root root   0 Dec 21 12:10 lost+found
		drwxr-xr-x. 134 root root   0 Jul 28 15:40 pipelines
		drwxr-xr-x. 139 root root   0 Jul 28 11:53 xray
	
==================================== End of mounting ====================================
