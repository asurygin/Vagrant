# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = { #задали массив с переменными?
  :Linux => { #назвали его на свой вкус, с таким названием будет создан каталог в machines, тире не использовать, только нижнее подчеркивание!
        :box_name => "centos/7", #подцепили готовый box centos/7 с оф сайта rhel
        :ip_addr => '192.168.133.154', #здесь задаем свой ip адресс
	:disks => { # перечисляем какие к контроллеру диски будут подключены, размерность и порт контроллера?
		
		:sata2 => {
                        :dfile => './sata2.vdi',
                        :size => 250, # Megabytes
						:port => 2
		},
        :sata3 => {
                        :dfile => './sata3.vdi',
                        :size => 250,
                        :port => 3
        },
        :sata4 => {
                        :dfile => './sata4.vdi',
                        :size => 250, # Megabytes
                        :port => 4
        }
	}
  },
}

Vagrant.configure("2") do |config| #стандартный синтаксис, что с этого момента будет читаться наш конфиг, то что было ранее определено не читается, если его не указать/не обратьися к нему после этого ключевого слова. config - как инструкция выполнения?

  MACHINES.each do |boxname, boxconfig| #здесь не все понятно, возможно считываем переменные? здесь теряюсь в догадках( возможно определяем, что теперь через boxconfig мы будем доставать заданные переменные

      config.vm.define boxname do |box| # выполняем инструкцию   config определяя boxname из инструкции MACHINES, обращаться к ней будем через box

          box.vm.box = boxconfig[:box_name] #в стандартном конфиге config.vm.box = "centos/7" , а сейчас инструкцию переопределили в box и подтянули к ней из MACHINES переменную box_name, для того чтобы бокс подцепить стардартный
          #box.vm.host_name = boxname.to_s # установить имя машины

          box.vm.network "forwarded_port", guest: 22, host: 5555 #сделать проброс порта 22 нашего Box на Local_host 2200

          box.vm.network "public_network", ip: boxconfig[:ip_addr], bridge: ["en1: Realtek PCIe GBE Family Controller"]		  #настройка VirtualBox, где public_network = bridged network (в этом случае потребуется указать какую свою реальную сетевую карту задействуете), а private_network=host-only access, так же подтягиваем из инструкции выше ip

          box.vm.provider :virtualbox do |vb| #задаются настройки виртуалки в оригинале config.vm.provider "virtualbox" do |vb|
            	  vb.customize ["modifyvm", :id, "--memory", "1024"] #задается память, в оригинале vb.memory = "512"
                  needsController = false #не ясно, возможно проверка есть ли контроллер дисков или принудительное отключение того, что было, а нужно это т к порты задаются с 1го, а в стандартной поставке "centos/7" уже есть 1 диск sda с первого порта
		  boxconfig[:disks].each do |dname, dconf| # определяем новую инструкцию, откуда вытягиваем инфу о дисках, видимо dname - это device name, обращаться будем к ней через dconf 
			  unless File.exist?(dconf[:dfile]) #если файлы были созданы, то подцепляем их к виртуалке
				vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]] #диски фиксированного размера, будут созданы в каталоге с vagrant, где запустились
                                needsController =  true #включаем контроллер
                          end

		  end
                  if needsController == true #если первое условие прошло удачно, добавляем диски
                     vb.customize ["storagectl", :id, "--name", "SATA1", "--add", "sata" ]
                     boxconfig[:disks].each do |dname, dconf|
                         vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                     end
                  end
          end
 	  box.vm.provision "shell", inline: <<-SHELL # как включается ssh, чтобы отличники на 6 сделали домашку, вписав сюда последовательно код сборки рейда
	      mkdir -p ~root/.ssh
              cp ~vagrant/.ssh/auth* ~root/.ssh
  	  SHELL

      end
  end
end
