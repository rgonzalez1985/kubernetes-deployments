# INSTALACIÓN BÁSICA

1\. Descargar el instalador

```sh
curl -sSLf https://get.k0s.sh | sudo sh
```

**DOCUMENTACIÓN:** *https://docs.k0sproject.io/v1.22.5+k0s.1/*

2\. acceder con su (con sudo hay que poner la ruta completa)

3\. generar archivo de configuración:

```sh
k0s default-config > k0s.yaml
```

4\. Modificar el archivo de configuración k0s.yaml para utilizar calico como CNI por defecto

```yaml
    podCIDR: 10.244.0.0/16
    provider: calico
    serviceCIDR: 10.96.0.0/12
```

5\. Modificar el archivo de configuración k0s.yaml para crear una storageClass

```yaml
    storage:
      create_default_storage_class: true
      type: openebs_local_storage
```

6\. Ejecutar la configuración inicial

```sh
sudo /usr/local/bin/k0s install controller --single -c k0s.yaml
```

**NOTA:** *En el caso de necesitar instalar en un servidor con docker instalado, la sentencia de instalación es la siguiente:*

```sh
adminuser@localhost zalando]$ sudo k0s install controller -c /home/adminuser/k0s.yaml --enable-worker --cri-socket docker:unix:///var/run/docker.sock
```

\*\*NOTA: \*\**el usuario root no puede acceder a k0s a través de PATH. Escribir la ruta /usr/local/bin antes de k0s para tareas de root*

7\. Iniciar el cluster

```sh
sudo /usr/local/bin/k0s start
```

# INSTALAR Y CONFIGURAR KUBECTL

1\. Descargar kubectl

```ssh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

2\. Instalar kubectl

```ssh
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

3\. Configurar kubectl

```sh
sudo cp /var/lib/k0s/pki/admin.conf configcp /var/lib/k0s/pki/admin.conf config
mkdir $HOME/.kube
sudo cp /var/lib/k0s/pki/admin.conf $HOME/.kube/config
sudo chown adminuser $HOME/.kube/config
```

# ADMINISTRACIÓN

Hay que crear un tunel ssh hacia el puerto 6443 en local:

```ssh
ssh adminuser@192.168.1.74 -L 127.0.0.1:6443:0.0.0.0:6443
```

Ahora se puede administrar sin problemas utilizando Lens copiando el archivo $HOME/.kube/config a Lens

# Configurar el pod metrics-server

Ejecutar la siguiente configuración de firewall

```sh
sudo firewall-cmd --new-zone=k8s-localhost --permanent
sudo firewall-cmd --reload
# add calico network to trusted zone for allow all communications
sudo firewall-cmd --zone=trusted --add-source=10.244.0.0/16 --permanent
sudo firewall-cmd --reload
```

# configurar openebs-hostpath como storageClass por defecto
``` sh
 kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
 ```

 Se puede comprobar su activación por defecto lanzando el siguiente comando
 ```sh
 kubectl get storageclass
 ```
 Obteniendo la respuesta:
 ```txt
 openebs-hostpath (default)   openebs.io/local   Delete          WaitForFirstConsumer   false                  5m54s
 ```