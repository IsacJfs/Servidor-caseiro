

# Projeto Pessoal – Servidor Caseiro

Na jornada de um desenvolvedor Full Stack, o entendimento de redes e a habilidade para configurá-las de maneira eficaz é tão essencial quanto dominar algoritmos e estruturas de dados. Afinal, nossas aplicações não vivem em um vácuo; elas precisam comunicar-se com o mundo. Este artigo é especialmente útil para aqueles que estão dando seus primeiros passos em DevOps ou precisam de uma referência rápida para configurar uma rede no Lubuntu, um ambiente que é leve, mas ao mesmo tempo robusto para o desenvolvimento.

Aqui, compartilho um guia detalhado que elabora como identificar o seu dispositivo de rede, configurar um IP estático e lidar com as configurações de permissões de arquivos — habilidades fundamentais para manter suas aplicações e servidores caseiros funcionando sem imprevistos. Este conteúdo é ideal para o desenvolvedor que busca expandir sua expertise além do código, adentrando no gerenciamento e na configuração da infraestrutura que suporta as aplicações do dia a dia.

Mergulhe neste guia passo a passo e aprimore sua proficiência em redes com Lubuntu.

## 1ª Etapa – Instalação do Lubuntu

Iniciei baixando a ISO do Lubuntu diretamente do site oficial, na seção [Downloads – Lubuntu](https://lubuntu.me/downloads/), escolhendo a versão 22.04.3 LTS (Jammy Jellyfish).

Após o download, utilizei o programa [Rufus 4.3](https://rufus.ie/pt/) (versão para Windows x64) para criar um pendrive bootável. Aqui, deparei-me com o primeiro desafio: o Rufus configura o sistema de arquivos para FAT32 por padrão, o que não foi compatível com a BIOS mais antiga do meu notebook. Tive que refazer o processo, optando por NTFS, o que fez com que o pendrive fosse reconhecido sem problemas.

Com a mídia de instalação reconhecida, iniciei o processo de instalação do Lubuntu. O assistente é bastante intuitivo, mas a etapa de particionamento apresentou um obstáculo. O SSD, que já possuía uma distribuição Linux, não aceitou a simples opção "Erase disk". O manual de instalação sugere o uso do comando `sudo swapoff -a` em tais casos. Apesar de ter desmontado as partições, não foi suficiente para prosseguir com a instalação.

### Formatando o SSD pelo terminal

Aqui estão os comandos que usei para preparar o disco:

```sh
fdisk -l
```

Este comando lista as unidades conectadas ao computador. Foi possível identificar o disco desejado pelo tamanho da unidade. No meu caso, a saída foi algo semelhante a:

```
Disk /dev/sda: 223,57 GiB, 240057409536 bytes, 468862128 sectors
```

Posteriormente, executei o comando:

```sh
sudo dd if=/dev/zero of=/dev/sda bs=1M count=100
```

Este comando sobrescreve os primeiros 100MB do disco com zeros. Sem essa ação, o sistema não é capaz de reconhecer o disco como livre para instalação.

**Atenção**: O uso deste comando irá destruir todos os dados presentes na unidade especificada. Proceda com cautela.

#### Uma curiosidade:
Quando excluímos um arquivo de um HD, normalmente removemos apenas o seu identificador. Os dados permanecem até serem sobreescritos. É por isso que, com ferramentas adequadas, é possível recuperar arquivos cujas referências foram deletadas.

Com o SSD formatado, a instalação do Lubuntu pôde prosseguir, e deixei que o instalador administrasse a partição do disco.

### Conectando e configurando a rede

Após instalar o sistema e conectar ao WiFi, realizei as atualizações necessárias e procedi com a configuração da rede. Primeiro, configurei o firewall com o UFW (Uncomplicated Firewall), que acompanha a distribuição Lubuntu e muitas outras. O UFW facilita a criação de regras de firewall, determinando quais conexões são permitidas ou bloqueadas. Usei os seguintes comandos:

```sh
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo systemctl restart sshd
```

Em seguida, decidi configurar um IP estático para o computador ao se conectar via WiFi, pois meu modem não tem a função de atribuir um IP fixo a um endereço MAC. Escolhi um número mais alto dentro do intervalo de IPs para evitar conflitos com outros dispositivos.

Configurar um IP fixo pode ser um pouco mais complicado, pois não se pode garantir que o IP escolhido sempre estará disponível. No meu modem, os IPs variam de 1 a 32, e ele tenta alocar dispositivos começando do índice mais baixo. Por isso, selecionar um número maior pode ajudar a manter o IP desejado livre para o seu servidor.

# Configuração de Rede no Servidor Caseiro

Para iniciar a configuração da rede, é imprescindível identificar o nome do dispositivo de rede. No terminal, execute o comando:

```sh
ip address
```

Este comando listará todos os dispositivos de rede disponíveis. Se a conexão for por cabo Ethernet, o nome do dispositivo geralmente começa com "enp" (como enp3s0, enp5s0, etc.). Para conexões Wi-Fi, o nome do dispositivo frequentemente começa com "wlp" (no meu caso, era wlp3s0).

O próximo passo é localizar o arquivo de configuração da rede, localizado no diretório `/etc/netplan/`. O arquivo que encontrei foi o `01-network-manager-all.yaml`.

Para editar o arquivo, utilize o seguinte comando:

```sh
sudo nano /etc/netplan/01-network-manager-all.yaml
```

O editor de texto Nano será aberto. Se o arquivo estiver vazio, a tela será preta, o que é normal. Aqui está o conteúdo a ser inserido ou modificado:

```yaml
network:
  version: 2
  renderer: NetworkManager
  wifis:
    wlp3s0:
      dhcp4: no
      addresses: [192.168.1.x/24]
      routes:
       - to: default
         via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
      access-points:
        "nome-da-sua-rede":
          password: "sua-senha"
```

Substitua o "x" em `addresses: [192.168.1.x/24]` pelo endereço IP que deseja configurar como estático.

O yaml é bem rígido com identação, utilize espaços e não tab.

Para salvar as alterações, pressione `Ctrl+X`, confirme para salvar e, em seguida, saia do editor.

De volta ao terminal, aplique as alterações com o comando:

```sh
sudo netplan apply
```

Durante esse processo, encontrei um erro de permissão:

```
WARNING **: Permissions for /etc/netplan/01-network-manager-all.yaml are too open. Netplan configuration should NOT be accessible by others.
```

Para corrigir isso, ajustei as permissões do arquivo com o comando:

```sh
chmod 600 /etc/netplan/01-network-manager-all.yaml
```

Esse comando `chmod` modifica as permissões do arquivo. O número `600` indica que o proprietário do arquivo pode ler e escrever, mas não executar; enquanto outros usuários não têm nenhuma permissão.

Após a correção, executei novamente o comando `sudo netplan apply`.

Em seguida, iniciei a configuração com o `nmcli`, que é uma ferramenta de linha de comando para o NetworkManager.

Primeiro, desativei a conexão atual:

```sh
nmcli con down id "nome-da-conexão"
```

Depois, utilizei a sequência de comandos para configurar o IP estático:

```sh
nmcli con mod "nome-da-conexão" ipv4.addresses 192.168.1.x/24
nmcli con mod "nome-da-conexão" ipv4.gateway 192.168.1.1
nmcli con mod "nome-da-conexão" ipv4.dns "8.8.8.8,8.8.4.4"
nmcli con mod "nome-da-conexão" ipv4.method manual
nmcli con mod "nome-da-conexão" wifi-sec.key-mgmt wpa-psk
nmcli con mod "nome-da-conexão" wifi-sec.psk "sua-senha"
```

Para reativar a conexão, execute:

```sh
nmcli con up id "nome-da-conexão"
```

Para verificar se o endereço IP está correto, utilize:

```sh
ip addr show wlp3s0
```

Com estes passos, a configuração inicial da rede está completa.

