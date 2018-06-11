install:
	#sudo cp 10-monitor.conf /usr/share/X11/xorg.conf.d/
	sudo cp nvidia-prime-boot.service /etc/systemd/system/
	sudo cp prime-socket.service /etc/systemd/system/
	sudo cp prime-select /usr/local/bin
	sudo rm -f /usr/local/bin/prime_socket
	sudo cp ../target/debug/prime_socket /usr/local/bin
	sudo chmod +x /usr/local/bin/prime-select
	sudo chmod +x /usr/local/bin/prime_socket
	sudo systemctl enable prime-socket.service
