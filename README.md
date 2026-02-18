# Docker Batocera.Linux Build

## 📋 Descrição

Ambiente Docker otimizado e pronto para usar para compilação do **Batocera.Linux**. Este projeto fornece uma imagem Docker baseada em Ubuntu 24.04 com todas as dependências, ferramentas e configurações necessárias para fazer build do Batocera.Linux sem necessidade de instalar nada no seu sistema host.

## 🎯 Propósito

O Batocera.Linux é uma distribuição Linux specializada em emulação de jogos retrô. Compilar do zero requer inúmeras dependências e ferramentas específicas. Este projeto encapsula todo o ambiente de build em um container Docker, garantindo:

- **Ambiente consistente** entre diferentes máquinas
- **Isolamento** do sistema host
- **Reprodutibilidade** de builds
- **Facilidade de setup** - sem necessidade de configurar ferramentas localmente

## 🏗️ Arquitetura

### Componentes

#### [Dockerfile](Dockerfile)
- **Base**: Ubuntu 24.04 (latest LTS)
- **Arquitetura**: Suporte a 32-bit e 64-bit (multilib)
- **Localização**: pt_BR.UTF-8 (porta Brasil)
- **Timezone**: America/Sao_Paulo

#### [docker-compose.yml](docker-compose.yml)
- Orquestra o container com volumes mapeados
- Conectividade SSH habilitada
- Acesso interativo via shell

## 📦 Dependências Instaladas

### Ferramentas de Compilação
- `build-essential` - Compiladores C/C++ e ferramentas essenciais
- `gcc-multilib`, `g++-multilib` - Suporte a compilação 32-bit e 64-bit
- `cmake` - Sistema de build moderno
- `autoconf`, `automake` - Ferramentas de configuração
- `bison`, `flex` - Geradores de parser/lexer
- `libtool` - Suporte a bibliotecas compartilhadas

### Bibliotecas Essenciais
- `libncurses6`, `libncurses-dev` - Interface terminal
- `libssl-dev` - Criptografia SSL/TLS
- `libglib2.0-dev` - Biblioteca GLib
- Variantes i386 das bibliotecas principais

### Ferramentas de Sistema
- `git` - Controle de versão
- `mercurial` - Controle de versão (Hg)
- `subversion` - Controle de versão (SVN)
- `wget` - Download de arquivos
- `rsync` - Sincronização de arquivos
- `zip` - Compressão

### Ferramentas Especializadas
- `u-boot-tools` - Bootloader para embedded
- `device-tree-compiler` - Compilador de device trees (ARM)
- `mtools`, `dosfstools` - Utilitários de filesystem FAT
- `scons` - Sistema de build alternativo
- `imagemagick` - Processamento de imagens
- `graphviz` - Geração de gráficos
- `python3` - Suporte Python
- `default-jre` - Java Runtime
- `texinfo` - Documentação
- `gettext` - Internacionalização
- `bc` - Calculadora
- `cpio` - Arquivo/compressão

## 🚀 Como Usar

### Pré-requisitos
- Docker instalado
- Docker Compose instalado
- Espaço em disco suficiente (recomendado 50GB+)

### Instalação e Execução

1. **Clone o repositório**
```bash
git clone https://github.com/seu-usuario/docker-batocera.linux-build.git
cd docker-batocera.linux-build
```

2. **Configure os volumes** (opcional)
   
   Edite o arquivo [docker-compose.yml](docker-compose.yml) para apontar para o local correto do seu repositório Batocera.Linux:

```yml
volumes:
  - /caminho/para/seu/batocera.linux:/src/batocera.linux
  - /home/seu-usuario/.ssh:/root/.ssh
```

3. **Inicie o container**
```bash
docker-compose up -d
```

4. **Acesse o container**
```bash
docker-compose exec batocera_builder /bin/bash
```

Ou conecte-se diretamente:
```bash
docker exec -it batocera_builder /bin/bash
```

### Compilar o Batocera.Linux

Dentro do container:

```bash
cd /src/batocera.linux
./build.sh
```

## 📁 Estrutura de Volumes

| Host | Container | Propósito |
|------|-----------|-----------|
| `${HOME}/batocera.linux` | `/src/batocera.linux` | Código-fonte do Batocera |
| `${HOME}/.ssh` | `/root/.ssh` | Credenciais SSH |

## 🔧 Configurações

### Locale e Timezone
```dockerfile
LANG=pt_BR.UTF-8
LANGUAGE=pt_BR:pt
LC_ALL=pt_BR.UTF-8
TZ=America/Sao_Paulo
```

### Variáveis de Ambiente
- `FORCE_UNSAFE_CONFIGURE=1` - Workaround para erros de configuração do host-tar

## 📝 Regras de Negócio

1. **Ambiente Isolado**: O build acontece totalmente dentro do container
2. **Persistência**: Código-fonte e artefatos residem no host via volumes
3. **Conectividade SSH**: Suporte a chaves SSH para acesso a repositórios privados
4. **Multi-arquitetura**: Suporte simultaneo a 32-bit e 64-bit
5. **Localização**: Configurado para locale brasileiro por padrão

## ⚙️ Comandos Úteis

### Build da imagem Docker
```bash
docker-compose build
```

### Visualizar logs
```bash
docker-compose logs -f batocera_builder
```

### Parar o container
```bash
docker-compose stop
```

### Remover container e volumes
```bash
docker-compose down -v
```

### Executar comando específico
```bash
docker-compose exec batocera_builder [comando]
```

## 🐛 Troubleshooting

### Erro de permissão de SSH
Verifique que sua chave SSH está em `~/.ssh` com permissões corretas (600)

### Espaço em disco insuficiente
O build do Batocera requer bastante espaço. Considere aumentar o tamanho da partição Docker

### Container não inicia
Execute `docker-compose logs` para ver detalhes do erro

### 🌐 Servidor Web para Upgrades de Versão

O servidor web definido em [`docker-compose.webserver.yml`](docker-compose.webserver.yml) serve as imagens compiladas do Batocera.Linux (localizadas em `/src/batocera.linux/output`) na porta 8080. Isso permite que dispositivos Batocera façam upgrades de versão remotamente, apontando para a URL do servidor (ex: `http://<IP-do-host>:8080`).

Uso da variável `PLATFORM`

O `docker-compose.webserver.yml` usa a variável de ambiente `PLATFORM` para montar o volume que contém as imagens compiladas para uma arquitetura/plataforma específica. Se `PLATFORM` não for informada, o valor padrão `bcm2837` será usado.

Exemplo do mapeamento de volume dentro do `docker-compose.webserver.yml`:

```
- ${HOME}/batocera.linux/output/${PLATFORM:-bcm2837}/images/batocera/images/${PLATFORM:-bcm2837}:/src/batocera.linux/output
```

Como executar o servidor web especificando a plataforma (ex.: `bcm2837` — padrão):

Inline (uma execução):
```bash
PLATFORM=bcm2837 docker-compose -f docker-compose.webserver.yml up
```

Exportando a variável para a sessão atual:
```bash
export PLATFORM=bcm2837
docker-compose -f docker-compose.webserver.yml up
```

Usando um arquivo `.env` (opcional): crie um arquivo `.env` no diretório do projeto com a linha:
```
PLATFORM=bcm2837
```
e então execute:
```bash
docker-compose -f docker-compose.webserver.yml up
```

Plataformas disponíveis:
```
bcm2837
x86_64
rk3568
h700
```
Obs: Podem variar de acordo com o build executado no batocera.linux.


Para parar:
CTRL+C
```bash
docker-compose -f docker-compose.webserver.yml down
```

No dispositivo Batocera, para realizar o upgrade (via SSH):
```bash
batocera-upgrade http://<IP-do-host>:8080
```

Para mais informações sobre upgrades, consulte [Compilação do Batocera.Linux](https://wiki.batocera.org/compile_batocera.linux#compilation).

## 📦 Versões

- **Base OS**: Ubuntu 24.04 LTS
- **Docker Image**: melviserpa/batocera.linux-build:latest

## 🔗 Referências

- [Batocera.Linux](https://batocera.org)
- [Docker Documentation](https://docs.docker.com)
- [Docker Compose Documentation](https://docs.docker.com/compose)

---

**Última atualização**: 2024