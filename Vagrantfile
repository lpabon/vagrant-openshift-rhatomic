# -*- mode: ruby -*-
# vi: set ft=ruby :

MINIONS = 3
DISKS = 3

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false
    #
    # Download RHEL Atomic from
    # https://access.redhat.com/downloads/content/293/ver=1/rhel---7/1.0.1/x86_64/product-downloads
    # Then once downloaded type:
    # $ vagrant box add --name rhel/atomic <downloaded rhel atomic box>
    # 
    config.vm.box = "rhel/atomic"

    # Override
    config.vm.provider :libvirt do |v,override|
        override.vm.synced_folder '.', '/home/vagrant/sync', disabled: true
    end

    # Make kub master
    config.vm.define :master do |master|
        master.vm.network :private_network, ip: "192.168.10.90"
        master.vm.host_name = "master"

        master.vm.provider :libvirt do |lv|
            lv.memory = 1024
            lv.cpus = 2
        end

    end

    # Make the glusterfs cluster, each with DISKS number of drives
    (0..MINIONS-1).each do |i|
        config.vm.define "atomic#{i}" do |atomic|
            atomic.vm.hostname = "atomic#{i}"
            atomic.vm.network :private_network, ip: "192.168.10.10#{i}"

            (0..DISKS-1).each do |d|
                driverletters = ('b'..'z').to_a
                atomic.vm.provider :libvirt do  |lv|
                    lv.storage :file, :device => "vd#{driverletters[d]}", :path => "disk-#{i}-#{d}.disk", :size => '500G'
                    lv.memory = 1024
                    lv.cpus =2
                end
            end

            if i == (MINIONS-1)
                # View the documentation for the provider you're using for more
                # information on available options.
                atomic.vm.provision :ansible do |ansible|
                    ansible.limit = "all"
                    ansible.playbook = "site.yml"
                    ansible.groups = {
                        "master" => ["master"],
                        "minions" => (0..MINIONS-1).map {|j| "atomic#{j}"},
                    }
                end
            end
        end
    end
end
