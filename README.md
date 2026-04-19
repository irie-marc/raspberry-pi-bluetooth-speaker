# Configure raspberry-pi as bluetooth-speaker

## 1. install packages
```
sudo apt update
sudo apt install pulseaudio-module-bluetooth bluez bluez-tools
```


## 3. add user pulse to bluetooth group
```
sudo usermod -aG bluetooth pulse
```

## 4. Force analog audio output
```
amixer cset numid=3 1
```

## 5. edit /etc/pulse/system.pa
- make sure the following lines are present
```
load-module module-native-protocol-unix auth-anonymous=1
load-module module-bluetooth-policy
load-module module-bluetooth-discover
```

## 5. edit /etc/pulse/daemon.conf
- make sure the following lines are present
```
allow-module-loading = yes
system-instance = yes
```

## 6. edit or create /etc/systemd/system/pulseaudio.service
- content should be:
```
[Unit]
Description=Sound Service
After=bluetooth.service
Requires=bluetooth.service

[Service]
# Note that notify will only work if --daemonize=no
Type=notify
ExecStart=/usr/bin/pulseaudio --daemonize=no --exit-idle-time=-1 --disallow-exit=true --system
Restart=always

[Install]
WantedBy=default.target
```

## 7. edit or create /etc/systemd/system/bt-agent.service
- content should be:
```
[Unit]
Description=Bluetooth Auth Agent
After=bluetooth.service
Requires=bluetooth.service

[Service]
ExecStart=/usr/bin/bt-agent -c NoInputNoOutput
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```


## 8. edit or create /etc/dbus-1/system.d/pulseaudio-bluetooth.conf
- content should be:
```
<busconfig>
  
 <policy user="pulse">  
  <allow send_destination="org.bluez"/>  
  <allow send_interface="org.bluez.Profile1"/>
  <allow send_interface="org.bluez.MediaEndpoint1"/>
  <allow send_interface="org.bluez.MediaPlayer1"/>
  <allow send_interface="org.freedesktop.DBus.ObjectManager"/>
  <allow send_interface="org.freedesktop.DBus.Properties"/>
 </policy>  
 
</busconfig> 
```


## 9. enable and start services
```
sudo systemctl daemon-reload
sudo systemctl enable pulseaudio bt-agent
sudo systemctl start pulseaudio bt-agent
```
