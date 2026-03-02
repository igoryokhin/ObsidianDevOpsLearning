При помощи данной команды мы содаем SSH ключи:
```CMD
ssh-keygen -t ed25519 -C "Lab_Learning"
```
Укажи куда сохранить ключ, не забывай что тебе не только нужно указать директорию куда сохранить SSH ключи, но и нужно указать имя ключа:
```CMD
Enter file in which to save the key (C:\Users\MAKI/.ssh/id_ed25519): C:\Users\MAKI\Documents\Lab_Learning\SSH_key
```

Дальше мы просто со всем соглашаемся(Enter) пока оставим без парольной фразы для простоты:

```CMD
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

Вот что ты должен увидеть при выполнении выше указаной команды:
```CMD
C:\Users\MAKI>ssh-keygen -t ed25519 -C "Lab_Learning"
Generating public/private ed25519 key pair.
Enter file in which to save the key (C:\Users\MAKI/.ssh/id_ed25519): C:\Users\MAKI\Documents\Lab_Learning\SSH_key
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\MAKI\Documents\Lab_Learning\SSH_key
Your public key has been saved in C:\Users\MAKI\Documents\Lab_Learning\SSH_key.pub
The key fingerprint is:
SHA256:r1aVECDl9KUYYUwZrib3zIRJ1B43v5ZWFZeFBHiyIj4 Lab_Learning
The key's randomart image is:
+--[ED25519 256]--+
|      o=X=.ooo.o*|
|     . *=+*o. ..o|
|      ..+ooB . . |
|     ..+o . + .  |
|    ..*.S. . +   |
|     +E= .. =    |
|       .+..o     |
|        ..       |
|       ..        |
+----[SHA256]-----+

C:\Users\MAKI>cd C:\Users\MAKI\Documents\Lab_Learning
```

Дальше нужно перенести созданый ключ на сервер: [[Передача SSH ключа на Linux сервер и авторизация по ключу]]