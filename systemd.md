# Systemd

## systemctl

- List units (daemons)
	`systemctl list-units` \
	`systemctl list-units | grep .service`

- List units failed
	`systemctl --failed`

- Status
	`systemctl status <unit-name>`

- Show if unit is enable
	`systemctl is-enabled <unit-name>`

- Enable/Disable unit \
	`sudo systemctl enable <unit-name>` \
	`sudo systemctl disable <unit-name>` \
	Se iniciará el demonio al iniciar el sistema operativo. Para manipular
	tenemos que utilizar `sudo`.

- Start/Stop unit \
	`sudo systemctl start <unit-name>` \
	`sudo systemctl stop <unit-name>` \

- Restart unit
	`sudo systemctl restart <unit-name>`

- Session
	* Restart system
		`systemctl reboot`
	* Shut down system
		`systemctl poweroff`
	* Suspend
		`systemctl suspend`


## journalctl

- System logs
	`journalctl` Para ver los logs del sistema. Podemos filtrar sólo los logs de
	este arranque con la opción `-b`, `journalctl -b`.

- Unit logs
	`journalctl -u <unit-name>`
