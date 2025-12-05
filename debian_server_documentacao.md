# Documentação de Instalação e Configuração do Debian Server

## 1. Download da ISO do Debian

- A ISO recomendada para servidor é a **netinst (Network Installation)**.
- Modelo mais comum: `debian-XX.X.X-amd64-netinst.iso`.
- Link oficial: [https://www.debian.org/distrib/netinst](https://www.debian.org/distrib/netinst)

## 2. Criação do Pendrive Bootável

- Utilizar Rufus, Ventoy ou Balena Etcher.
- Selecionar a ISO.
- Método de partição recomendado: GPT + UEFI.

## 3. Iniciando a Instalação

- Selecionar **Install** ou **Graphical Install**.
- Configurar idioma, teclado e layout.

## 4. Configuração de Usuário e Senha

### Regras do Debian para nome de usuário

- Deve começar com **letra minúscula**.
- Pode conter apenas **letras minúsculas** e **números**.
- Não pode ter espaços, acentos ou caracteres especiais.
- Máximo de 32 caracteres.

### Exemplos válidos

- `caio`
- `caioleal`
- `caiosantos`
- `suporte1`

### Exemplos inválidos

- `Caio` (maiúscula)
- `caio.suporte` (ponto)
- `caio-leal` (hífen)
- `caio leal` (espaço)
- `caio@server` (caractere especial)

## 5. Particionamento do Disco

### Observação sobre armazenamento

- Nesta instalação, está sendo utilizado um **HD** ao invés de um SSD.
- Isso não impede a instalação, porém:
  - A velocidade de leitura/escrita será menor.
  - O tempo de boot e instalação pode ser mais lento.
  - Para uso em servidor, um SSD é recomendado futuramente para melhorar desempenho.

(Adicionar conteúdo conforme você avançar na instalação.)

## 6. Configuração de Software (Seleção de Pacotes)

Durante a instalação, na etapa **Seleção de software**, foram feitas as seguintes escolhas para garantir um ambiente de servidor puro, sem interface gráfica:

###  Mantido

- **Servidor SSH**
- **Utilitários de sistema padrão**

###  Desmarcado

- Ambiente de área de trabalho do Debian
- GNOME
- Xfce
- GNOME Flashback
- KDE Plasma
- Cinnamon
- MATE
- LXDE
- LXQt
- Servidor web (Apache)

### Resultado

- O sistema será instalado **sem interface gráfica** e sem componentes de desktop.
- Ambiente leve, ideal para servidor.
- Consumo aproximado: \~100MB RAM / \~1% CPU.

## 7. Configuração de Rede

Durante os testes de conectividade entre a máquina host (Windows) e a VM Debian em modo Bridge, foi identificado o seguinte comportamento:

### Windows → Debian: Ping funcionando

O Windows consegue alcançar o servidor Debian corretamente.

###  Debian → Windows: Ping bloqueado

O Debian não conseguia pingar o Windows. Isso **não era falha no servidor Debian**, e sim no **firewall do Windows**, que bloqueia ICMP por padrão.

###  Solução aplicada

Foi necessário habilitar o recebimento de ping (ICMP Echo Request) no Firewall do Windows:

1. Acessar **Firewall do Windows com Segurança Avançada**.
2. Abrir **Regras de Entrada**.
3. Localizar e habilitar:
   - **File and Printer Sharing (Echo Request - ICMPv4-In)**
   - Ou **Compartilhamento de Arquivo e Impressora (Solicitação de Eco - ICMPv4-In)**
4. Após habilitar, o ping Debian → Windows passou a funcionar.

###  Alternativa via CMD (habilitar ICMP)

```
netsh advfirewall firewall add rule name="ICMP Allow" protocol=icmpv4:any,any dir=in action=allow
```

### Resultado

- A comunicação bidirecional entre Windows ↔ Debian foi restaurada.
- Confirmação de que a rede Bridge e o adaptador **Intel PRO/1000 MT** estão funcionando corretamente no VirtualBox. (Adicionar conteúdo mais tarde.)

## 8. Instalação do Sistema Base

(Adicionar conteúdo conforme for instalado.)

## 8. Instalação de Serviços (SSH, Web, Firewall, etc)

(Registrar o que for instalado.)

---

## 9. Configuração do Compartilhamento de Arquivos (Samba)

### 9.1 Criação do diretório compartilhado

O diretório utilizado para armazenar arquivos compartilhados foi criado em `/srv/compartilhado`.

```
sudo mkdir -p /srv/compartilhado
sudo chown -R caioleal:caioleal /srv/compartilhado
sudo chmod -R 775 /srv/compartilhado
```

### 9.2 Instalação do Samba

```
sudo apt install samba -y
```

### 9.3 Configuração do smb.conf

Arquivo editado: `/etc/samba/smb.conf`

Bloco adicionado ao final:

```
[compartilhado]
   path = /srv/compartilhado
   browseable = yes
   writable = yes
   guest ok = no
   valid users = caioleal
   force user = caioleal
```

### 9.4 Criação da senha SMB

```
sudo smbpasswd -a caioleal
```

### 9.5 Reinício do serviço Samba

```
sudo systemctl restart smbd
```

### 9.6 Acesso via Windows

Acesso realizado pelo explorador de arquivos do Windows:

```
\192.168.1.137\compartilhado
```

Credenciais utilizadas:

- Usuário: `debian\caioleal`
- Senha: definida via `smbpasswd`

### 9.7 Problema encontrado: permissão negada ao escrever

Ao tentar criar ou mover arquivos, o Windows exibiu mensagem de falta de permissão.

**Causa:** permissões Unix incorretas no diretório `/srv/compartilhado`.

**Solução aplicada:**

```
sudo chown -R caioleal:caioleal /srv/compartilhado
sudo chmod -R 775 /srv/compartilhado
```

**Resultado:** escrita e leitura funcionando corretamente.

### 9.8 Comunicação e autenticação validadas

- Conexão Windows → Debian funcionando.
- Autenticação SMB realizada com usuário `caioleal`.
- Transferência de arquivos bem-sucedida.

(Registrar o que for instalado.)

---

**Observação:** Conforme você for avançando, posso continuar expandindo a documentação para deixar pronta tanto para o GitHub quanto como projeto no LinkedIn.


## 10. Instalação e Configuração do Grafana

### 10.1 Problemas iniciais encontrados
Durante a tentativa de instalar o Grafana no Debian Trixie, o repositório antigo (`apt.grafana.com`) apresentava erros de assinatura e chave GPG inválida, tornando impossível instalar o pacote via `apt`.

### 10.2 Limpeza completa de repositórios quebrados
Para remover entradas antigas incorretas:

```
sudo rm -f /etc/apt/sources.list.d/grafana.list
sudo rm -f /usr/share/keyrings/grafana.gpg
```

Atualização do APT sem repositórios do Grafana:

```
sudo apt update
```

### 10.3 Adicionando o repositório correto do Grafana
O repositório oficial atualizado é:

```
https://packages.grafana.com/oss/deb
```

### 10.4 Instalação da chave GPG correta

```
wget -q -O - https://packages.grafana.com/gpg.key | sudo tee /usr/share/keyrings/grafana.gpg > /dev/null
```

### 10.5 Criando o arquivo de repositório

```
echo "deb [signed-by=/usr/share/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
```

Conferir conteúdo:

```
cat /etc/apt/sources.list.d/grafana.list
```

Saída esperada:

```
deb [signed-by=/usr/share/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main
```

### 10.6 Atualizar listas de pacotes

```
sudo apt update
```

O correto é aparecer:

```
Atingido: https://packages.grafana.com/oss/deb stable InRelease
```

### 10.7 Instalar o Grafana

```
sudo apt install grafana -y
```

### 10.8 Conclusão
Após correção dos repositórios e adição correta das chaves GPG, o sistema passa a reconhecer o pacote e a instalação funciona normalmente.

---

## 10. Correção de Erros do Prometheus (Configuração e Systemd)

Durante a configuração do Prometheus e integração com o Node Exporter no **Debian Server**, alguns erros foram encontrados e resolvidos. Abaixo está o registro detalhado do processo para referência futura.

### 10.1 Erro de sintaxe no arquivo `/etc/prometheus/prometheus.yml`

Ao reiniciar o Prometheus, o erro abaixo apareceu:

```
Error loading config (--config.file=/etc/prometheus/prometheus.yml)
yaml: line 35: could not find expected ':'
```

####  Causa
Indentação incorreta ou linhas mal formatadas no bloco `scrape_configs`.

####  Solução
1. Verificar linhas problemáticas:
```
nl -ba /etc/prometheus/prometheus.yml | sed -n '1,120p'
```

2. Corrigir o arquivo deixando exatamente assim:

```
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

3. Validar YAML:
```
promtool check config /etc/prometheus/prometheus.yml
```

---

### 10.2 Erro no serviço Systemd do Prometheus

Erro ao reiniciar:
```
Unknown section 'install'
```

####  Causa
O bloco `[Install]` estava escrito incorretamente.

####  Solução
Editar o serviço:
```
sudo nano /etc/systemd/system/prometheus.service
```

Conteúdo correto:
```
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/
Restart=always

[Install]
WantedBy=multi-user.target
```

Aplicar alterações:
```
sudo systemctl daemon-reload
sudo systemctl restart prometheus
sudo systemctl status prometheus
```

---

### 10.3 Verificação dos Targets

Após correção, acessar:
```
http://192.168.1.137:9090/targets
```

Resultado esperado:
- Prometheus → **UP**
- Node Exporter → **UP**

---

### 10.4 Integração com Grafana

Após tudo configurado no **Debian Server**, o Grafana reconheceu corretamente:
- Fonte de dados Prometheus
- Dashboard Node Exporter Full
- Instância `localhost:9100`

Dashboard exibindo métricas:
- CPU
- Memória
- Disco
- Uptime
- Load Average

Tudo funcionando perfeitamente.

---

###  Conclusão
A instalação do **Prometheus + Node Exporter + Grafana no Debian Server** foi concluída com sucesso.
O ambiente agora monitora recursos do sistema em tempo real e está pronto para expansão futura.
