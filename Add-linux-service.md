# Create linux service to start karbowanecd

On this page you will find description how to run karbowanecd with JSON PRC as linux service. I use Ubuntu server 16.03 x64, but this description you can be applied to any of the linux versions with small changes.

1. Download latest linux version of karbowanec:

```
wget https://github.com/seredat/karbowanec/releases/download/v.1.5.1/karbo-cli-1.5.1-64bit.tar.gz
```

2. Unpack it to directory _/opt/karbo/_:

```
sudo mkdir /opt/karbo/
sudo tar -xf karbo-cli-1.5.1-64bit.tar.gz -C /opt/karbo/
```

3. To start service we will use user _karbo_, so lets create it and manage permissions:

```
sudo useradd karbo
sudo groupadd karbo
sudo usermod -a -G karbo karbo
sudo chgrp -R karbo /opt/karbo/
sudo chmod -R 770 /opt/karbo/
```

4. Create log file and add permision to write it:

```
sudo touch /var/log/karbowanecd
sudo chgrp -R karbo /var/log/karbowanecd
sudo chmod -R 770 /var/log/karbowanecd
```

5. Lets check if everything is ok. Try to run daemon with _karbo_ user permission:

```
sudo -u karbo /opt/karbo/karbowanecd --data-dir=/opt/karbo/.karbowanec --log-file=/var/log/karbowanecd --restricted-rpc --enable-cors=*  --enable-blockchain-indexes --rpc-bind-ip=0.0.0.0 --rpc-bind-port=32348 --fee-address=KaAxHCPtJaFGDq4xLn3fASf3zVrAmqyE4359zn3r3deVjCeM3CYq7K4Y1pkfZkjfRd1W2VPXVZdA5RBdpc4Vzamo1H4F5qZ
```

Stop it via entering `exit` inside daemon session.

6. You could pre-download blockchain bootstrap to speed-up process:

```
cd ./opt/karbo/.karbowanec
wget https://bootstrap.krbnodes.pp.ua/blockchain-$(date "+%Y-%m-%d").tar.gz
tar -xvzf blockchain-$(date "+%Y-%m-%d").tar.gz -С /opt/karbo/.karbowanec
```

7. To start _karbowanecd_ , we need to create service file in _/etc/systemd/system_:

```
nano /etc/systemd/system/karbowanecd.service
```

```
[Unit]
Description=karbowanecd
Documentation=http://karbo.io
After=syslog.target

[Service]
User=karbo
ExecStart=/opt/karbo/karbowanecd --data-dir=/opt/karbo/.karbowanec --log-file=/var/log/karbowanecd --restricted-rpc --enable-cors=*  --enable-blockchain-indexes --rpc-bind-ip=0.0.0.0 --rpc-bind-port=32348 --fee-address=KaAxHCPtJaFGDq4xLn3fASf3zVrAmqyE4359zn3r3deVjCeM3CYq7K4Y1pkfZkjfRd1W2VPXVZdA5RBdpc4Vzamo1H4F5qZ
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

Do not forget to change address to your wallet!

8. Run service:

```
sudo systemctl daemon-reload
sudo systemctl enable karbowanecd.service
sudo systemctl start karbowanecd.service
```

9. To check service status:

```
systemctl status karbowanecd.service
```

```
● karbowanecd.service - karbowanecd
   Loaded: loaded (/etc/systemd/system/karbowanecd.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2018-01-07 10:44:28 EET; 48min ago
     Docs: http://karbo.io
 Main PID: 14788 (karbowanecd)
   CGroup: /system.slice/karbowanecd.service
           └─14788 /opt/karbo/karbowanecd --data-dir=/opt/karbo/.karbowanec --log-file=/var/log/karbowanecd --restricted-rpc --rpc-bi
lines 1-7/7 (END)
```