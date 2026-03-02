Для того чтобы использовать SSH ключ который мы создавали на windows на нужно передать на сервер .pub ключ при помощью данной команды:
```CMD
type C:\Users\MAKI\Documents\Lab_Learning\SSH_key.pub | ssh maki@192.168.101.131 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

После успешной передачи ключа пробуем зайти на сервер:
```CMD
ssh -i C:\Users\MAKI\Documents\Lab_Learning\SSH_key maki@192.168.101.131
```

Должно выглядеть вот так после авторизации:
```CMD
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed Feb 25 02:37:51 PM UTC 2026

  System load:             0.0
  Usage of /:              24.6% of 18.53GB
  Memory usage:            13%
  Swap usage:              0%
  Processes:               220
  Users logged in:         1
  IPv4 address for ens160: 192.168.101.131
  IPv6 address for ens160: fd3f:b49d:94d5::f9f
  IPv6 address for ens160: fd3f:b49d:94d5:0:20c:29ff:feff:6d2c

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

71 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


Last login: Wed Feb 25 14:24:05 2026 from 192.168.101.201
maki@lablearning:~$
```

Раз мы успешно зашли на сервер, теперь следующий этап это отключить авторизацию по паролю: [[Укрепление безопасности сервера(Hardering)]]
