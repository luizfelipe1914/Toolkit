#### Tornando os containers LXD disponíveis na rede local (sem nat do host hospedeiro)

Para tornarmos os containers LXD disponíveis em nossa rede local temos agumas opções de configurações disponíveis:

* *routed network*

* *macvlan*

* *bridged network*
  
  Nessa documentação, estarei utilizado a opção de *macvlan*. Para que essa configuração funcione é necessário que alguns requisitos sejam atendidos:
  
  * O switch no qual o host hospedeiro está instalado deve permitir mais de um endereço MAC por interface ethernet.
  
  * Se o host hospedeiro for uma máquina virtual, o modo promíscuo da rede deve estar habilitado.
  1. Primeiramente, devemos criar um *profile*(perfil) para os adaptadores de rede.
     1.1. Para listarmos os *profiles* existentes, utilize o comando abaixo:
     
     ```bash
     lxc profile list
     +---------+---------------------+---------+
     |  NAME   |     DESCRIPTION     | USED BY |
     +---------+---------------------+---------+
     | default | Default LXD profile | 7       |
     +---------+---------------------+---------+
     | macvlan |                     | 1       |
     +---------+---------------------+---------+
     ```

    1.2. Agora, criamos o *profile*:
        ```bash
        lxc profile create <profile_name>
        ```
        
    1.3. Verificamos as configurações do *profile* criado:

```bash
lxc profile show <profile_name>
```

     1.4. Agora adicionamos e vinculamos a interface de rede do *profile* à uma interface física do host hospedeiro:

```bash
lxc profile device add <profile_name> eth0 nic nictype=macvlan parent=<nic_hospedeiro>
```

1.5. Verificamos, novamente, as configurações do *profile* criado:

```bash
lxc profile show <profile_name>
```

Deverá ser gerada uma saída semelhante a esta:

```bash
user@host:~$ lxc profile show macvlan
config: {}
description: ""
devices:
  eth0:
    nictype: macvlan
    parent: enp2s0
    type: nic
name: macvlan
used_by:
- /1.0/instances/Ubuntu-Teste
```

2. Pronto! Já podemos iniciar containers com o *profile* criado:
   
   ```bash
   lxc <launch/copy> <container_image/container_in_hypervisor> <container_name> --profile default --profile <profile_criado>
   ```

Referências:

[How to make your LXD containers get IP addresses from your LAN using macvlan](https://blog.simos.info/how-to-make-your-lxd-container-get-ip-addresses-from-your-lan/)


