# Сопровождение развёртывания на базе дистрибутива Asperitas


## Развёрнутые сервера (общий список)

Перейдите во вкладку _Deployed servers_. 

![](../../../images/all-servers.png)

При чистом старте это окно будет пустым. Так как в системе ещё не было создано ни одного сервера.


### Развёрнутые сервера (список одного развёртывания)

Перейдите во вкладку _Servers list_. 

![](../../../images/stack-servers.png)

## Обновление узлов 

### Обновление системы узла развёртывания

При обнаружении уязвимостей или выходе малых обновлений компонентов системы необходимо обновить пакеты, 
а также контейнеры системы, которые отвечают основным компонентам системы.

В первую очередь необходимо обновить систему 
~~~shell
sudo dnf update 
~~~

Далее обновите список образов контейнеров для установки: 
~~~shell
cat container_images | xargs -I {} sudo skopeo copy --src-tls-verify=false \
--dest-tls-verify=false docker://source-images-server/asperitos/{}:latest \
docker://localhost:13787/asperitos/{}:latest
~~~
### Обновление узлов вычисления 

Для начала необходимо сбекапить все данные 
~~~shell
mkdir backups-$(date "+%Y-%m-%d")
cd backups-$(date "+%Y-%m-%d")
sudo rsync -a /var/lib/openvswitch ./
sudo rsync -a /var/lib/config-data ./
sudo rsync -a /var/lib/kolla ./
sudo rsync -a /var/lib/tripleo-config ./
~~~

Так как при обновлении все контейнеры пересоздаются - то будет пересоздан и 
контейнер _nova_libvirt_, отвечающий за создание виртуальных машин. 
При этом все виртуальные машины не должны останавливаться. Проверьте 
наличие пакета systemd-container. 

### Обновление узлов управления 

Погасите horizon 
~~~shell
sudo systemctl stop tripleo_horizon
~~~

Для начала необходимо сбекапить все данные 
~~~shell
mkdir backups-$(date "+%Y-%m-%d")
cd backups-$(date "+%Y-%m-%d")
sudo rsync -a /var/lib/mysql ./
sudo rsync -a /var/lib/rabbitmq ./
sudo rsync -a /var/lib/openvswitch ./
sudo rsync -a /var/lib/config-data ./
sudo rsync -a /var/lib/kolla ./
sudo rsync -a /var/lib/tripleo-config ./
~~~
пше 
Сохраните старые образы контейнеров 
~~~shell
for im in $(sudo podman image ls | grep asperitos | grep latest | cut -d " " \
-f1 | uniq) ; do sudo podman tag $im:latest $im:$(date "+%Y-%m-%d") ; done
~~~

Обновите контейнеры с registry узла развёртывания
~~~shell
for im in $(sudo podman image ls | grep asperitos | grep latest | cut -d " " \
-f1 | uniq) ; do sudo podman pull $im:latest ; done
~~~