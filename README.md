# 404notfoundhard_infra
404notfoundhard Infra repository

bastion_IP=35.207.140.251
someinternalhost_IP=10.156.0.3

testapp_IP=35.204.131.164
testapp_port=9292
### one line connect
```
ssh -i /home/user/.ssh/gcp_appUser.pub gcp_appUser@10.156.0.3 -o "proxycommand ssh -W %h:%p -i /home/user/.ssh/gcp_appUser.pub gcp_appUser@35.207.140.2"
```
#
additional task
#

#### first case
```
alias 2bastion='ssh -i /home/user/.ssh/gcp_appUser.pub gcp_appUser@35.207.140.251'
alias 2interalgcp='ssh -i /home/user/.ssh/gcp_appUser.pub gcp_appUser@10.156.0.3 -o "proxycommand ssh -W %h:%p -i /home/user/.ssh/gcp_appUser.pub gcp_appUser@35.207.140.2"'

funcsave 2bastion
funcsave 2internalgcp
```
#### second case
```
Host 35.207.140.251
  Hostname 35.207.140.251
  User gcp_appUser
  IdentityFile  ~/.ssh/gcp_appUser.pub
Host 10.156.0.*
  IdentityFile  ~/.ssh/gcp_appUser.pub
  User gcp_appUser
  ProxyCommand ssh -W %h:%p  gcp_appUser@35.207.140.251
```

## gcloud: create firewall rule
```
gcloud compute firewall-rules create allow-puma-service \
                                    --allow tcp:9292 \
                                    --description='allow connect to puma service' \
                                    --direction INGRESS \
                                    --target-tags=puma-server
```
## gcloud: create instance with startup script
#### first case with remote script
```
gcloud compute instances create reddit-app\
                                     --boot-disk-size=10GB \
                                     --image-family ubuntu-1604-lts \
                                     --image-project=ubuntu-os-cloud \
                                     --machine-type=g1-small \
                                     --tags puma-server \
                                     --restart-on-failure \
                                     --scopes storage-ro \
                                     --metadata startup-script-url=https://gist.githubusercontent.com/404notfoundhard/426db8a75e0a7fd8f4b37d1a8d57291a/raw/478068f6dbd73c3d04acbb248fe21e948d2ee1ef/startup_script.sh
```
#### second case with local script
```
gcloud compute instances create reddit-app\
                                     --boot-disk-size=10GB \
                                     --image-family ubuntu-1604-lts \
                                     --image-project=ubuntu-os-cloud \
                                     --machine-type=g1-small \
                                     --tags puma-server \
                                     --restart-on-failure \
                                     --metadata-from-file startup-script=/path/to/my/startup_script.sh
```
## ДЗ№6
###Опишите в коде терраформа добавление ssh ключа пользователя appuser1 в метаданные проекта. Выполните terraform apply и проверьте результат (публичный ключ можно брать пользователя appuser);  
```
  metadata {
    ssh-keys = "user3:${file(var.public_key_path)}"
    ssh-keys = "user2:${file(var.public_key_path)}"
  }
```
доступ будет только у последнего, в данном случае "user2". 
### Опишите в коде терраформа добавление ssh ключей нескольких пользователей в метаданные проекта (можно просто один и тот же публичный ключ, но с разными именами пользователей, например appuser1, appuser2 и т.д.). Выполните terraform apply и проверьте результат;  
```
  metadata {
    ssh-keys = "user3:${file(var.public_key_path)} user2:${file(var.public_key_path)}"
  }
```
доступ будет у всех перечисленных пользователей.
### Добавьте в веб интерфейсе ssh ключ пользователю appuser_web в метаданные проекта. Выполните terraform apply и проверьте результат; 
Если добавить пользователя в GCP console, то у него тоже есть доступ по ssh

* так же возможно добавление ssh ключей с помощью gcloud, пример:
```
$ cat ssh-keys-file
user3:ssh-rsa %pub_key_for_user3% user3
user2:ssh-rsa %pub_key_for_user2% user2

gcloud compute project-info add-metadata --metadata-from-file ssh-keys=/path/to/ssh-keys-files 
```
Но тут необходимо быть осторожным, потому что публичные ключи перезаписываются, так что если хотим сохранить старые публичные ключи, то они должны быть добавлены в этот файл.

### Добавление еще одного ресурса без помощи "count"
Проблемы: 
 - усложнение кода terraform
 - вероятность появления различных инстансов
 