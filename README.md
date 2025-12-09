# tutorial_Yocto

Yocto Project é um projeto open-source de colaboração que ajuda desenvolvedores
a criar sistemas embarcados baseados em Linux. A sua principal motivação é
resolver o problema de ter que construir um sistema operacional do zero, com
todas as dependências corretas, para cada hardware diferente. Em vez de usar
distribuições genéricas como Ubuntu ou Fedora, que contêm muitos pacotes
desnecessários, o Yocto permite que você construa uma imagem “do zero”
contendo apenas o que seu projeto realmente precisa, garantindo um sistema
mínimo, seguro e eficiente.

Autor: André Felipe
---

# 1. Instalação de dependências

O Yocto requer um conjunto de pacotes de desenvolvimento. Dependendo da sua
distribuição Linux, você pode usar os seguintes comandos:

```bash
sudo apt-get install gawk wget git diffstat unzip texinfo gcc-multilib \
build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils \
debianutils iputils-ping python3-git python3-jsonpatch libyaml-dev
```
# 2. Clonar o Poky

Agora, vamos clonar o repositório principal do Poky e navegar para a versão
desejada:

```bash
 git clone git://git.yoctoproject.org/poky
```

```bash
cd poky
```
Subistitua por sua versão
```bash
git checkout -b meu-branch yocto-4.0
```
```bash
git branch
```
# 3. Configurar o Ambiente
Para que os comandos do BitBake funcionem, você precisa carregar as variáveis
de ambiente do Yocto.
```bash
source oe-init-build-env
```
Esse comando cria um diretório de build (build) e o configura para você. Após isso,
o prompt do seu terminal mudará para indicar que você está no ambiente de build
do Yocto.

# 4. Criação de uma Camada Customizada (layer)

Uma das melhores práticas no Yocto é manter suas personalizações em uma
camada separada do Poky. Isso facilita a atualização para novas versões do Yocto
sem perder seu trabalho.

# 4.1 Criar a Estrutura da Layer
```bash
cd ..   # Volte para o diretório raiz do Poky

mkdir meta-meuprojeto

cd meta-meuprojeto

mkdir recipes-core recipes-images recipes-kernel
```
# 4.2 Definir a Configuração da Layer

```bash

mkdir conf

nano conf/layer.conf

#Cole o que está abaixo dentro do seu nano:

# We have a conf and classes directory, add to BBPATH
BBPATH .= ":${LAYERDIR}"

# We have recipes, add to BBFILES
BBFILES += "${LAYERDIR}/recipes-*/**/*.bb \
            ${LAYERDIR}/recipes-*/**/*.bbappend"

BBFILE_COLLECTIONS += "meuprojeto"
BBFILE_PATTERN_meuprojeto = "^${LAYERDIR}/"
BBFILE_PRIORITY_meuprojeto = "6"

# This should only be added to a single layer where it is needed
LAYERSERIES_COMPAT_meuprojeto = "dunfell"  # ou a versão do Yocto que você está usando
```
# 5. Definição de Recipes para Pacote e Imagem
Uma recipe é o coração da construção de um pacote no Yocto.

# 5.1 Criar uma Recipe Simples

```bash
mkdir -p recipes-core/hello-world

nano recipes-core/hello-world/hello-world_1.0.bb

# No arquivo nano cole o código abaixo

DESCRIPTION = "A simple C program that prints 'Hello, Yocto!'"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0b288a8019b02a5c4e402"

SRC_URI = "file://helloworld.c"
S = "${WORKDIR}"

do_compile() {
    ${CC} helloworld.c -o helloworld
}

do_install() {
    install -d ${D}${bindir}
    install -m 0755 helloworld ${D}${bindir}
}
```

Criar o Arquivo Fonte

```bash
nano recipes-core/hello-world/helloworld.c

#Salve o código abaixo dentro do nano:
#include <stdio.h>

int main() {
    printf("Hello, Yocto!\n");
    return 0;
}
```
```bash
mkdir -p recipes-images/minha-imagem

nano recipes-images/minha-imagem/minha-imagem.bb

#Cole o arquivo abaixo no nano
DESCRIPTION = "Minha imagem minimalista com Yocto."

IMAGE_FEATURES += "empty-rootfs package-management"

# Inclua os pacotes que você quer na sua imagem
IMAGE_INSTALL += "packagegroup-core-boot \
                   hello-world \
                   kernel-devsrc \
                   kernel-image \
                   ${CORE_IMAGE_EXTRA_INSTALL}"
```

