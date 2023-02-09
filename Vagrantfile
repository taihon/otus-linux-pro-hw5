# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
	:nfss => {
		:box_name => "centos/7",
		:box_version => "2004.01",
		:ip_addr => "192.168.11.11"
	},
	:nfsc =>{
		:box_name => "centos/7",
		:box_version =>"2004.01",
		:ip_addr=>"192.168.11.12"
	}
}

Vagrant.configure("2") do |config|
	config.vm.box_version="2004.01"
	MACHINES.each do |boxname, boxconfig|
		config.vm.define boxname do |box|
			box.vm.box = boxconfig[:box_name]
			box.vm.hostname = boxname.to_s
			box.vm.network "private_network", ip: boxconfig[:ip_addr]
			if boxname.to_s == "nfss" then
				box.vm.provision "shell", inline: <<-'SHELL'
				yum install -y nfs-utils
				systemctl enable rpcbind nfs-server
				systemctl start rpcbind nfs-server
				systemctl enable firewalld
				systemctl start firewalld
				firewall-cmd --permanent --add-service=nfs3
				firewall-cmd --permanent --add-service=mountd
				firewall-cmd --permanent --add-service=rpc-bind
				firewall-cmd --permanent --add-port=20048/tcp
				firewall-cmd --reload
				mkdir /mnt/nfsstorage/upload -p
				chown -R nfsnobody:nfsnobody /mnt/nfsstorage/upload
				chmod -R 755 /mnt/nfsstorage/upload
				echo "/mnt/nfsstorage	*(rw,sync,all_squash,anonuid=65534,anongid=65534)" > /etc/exports
				systemctl restart nfs-server
				exportfs -r
				SHELL
			end
			if boxname.to_s == "nfsc" then
				box.vm.provision "shell", inline: <<-'SHELL'
				mkdir /media/nfs_share
				systemctl enable firewalld
				systemctl start firewalld
				#mount -t nfs 192.168.11.11:/mnt/nfsstorage /media/nfs_share/
				echo -e "[Unit]\nDescription = NFS share automount\n[Mount]\nWhat=192.168.11.11:/mnt/nfsstorage\nWhere=/media/nfs_share\nType=nfs\nOptions=defaults\nTimeoutSec=15\n[Install]\nWantedBy=multi-user.target">/etc/systemd/system/media-nfs_share.mount 

				echo -e "[Unit]\nDescription=nfs automount for nfs_share\n\n[Automount]\nWhere=/media/nfs_share\nTimeoutIdleSec=15\n\n[Install]\nWantedBy=multi-user.target" > /etc/systemd/system/media-nfs_share.automount
				systemctl daemon-reload
				systemctl start media-nfs_share.mount
				systemctl enable media-nfs_share.automount
				SHELL
				box.vm.provision "shell", reboot: true
			end
		end
	end
end
