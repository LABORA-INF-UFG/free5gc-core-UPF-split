# Separação de Core e UPF no free5gc para testes ATSSS

Este repositório contém os arquivos de configuração e scripts modificados do projeto [free5gc](https://github.com/free5gc/free5gc) para permitir a execução da UPF (User Plane Function) em uma máquina separada do Core Network.

Este setup é parte de uma pesquisa de Iniciação Científica focada em testes de ATSSS (Access Traffic Steering, Switching, and Splitting).

## 🗺️ Topologia da Arquitetura

A arquitetura de rede virtualizada utilizada é a seguinte:

![Topologia da Rede](Topologia.png)

* **VM1 - UE / RAN:** Simula o equipamento do usuário e a rede de acesso.
* **VM2 - Core Network:** Executa as NFs do 5G Core (AMF, SMF, NRF, etc.).
* **VM3 - UPF:** Executa apenas a NF da UPF.

## ⚙️ Configuração

Este repositório é projetado para ser clonado em duas VMs (VM2 e VM3). As modificações principais foram feitas nos seguintes arquivos para refletir a topologia:

* **`config/smfcfg.yaml`**: Modificado para que a SMF (na VM2 @ `10.1.0.4`) possa se comunicar com a UPF (na VM3 @ `10.1.0.5`).
* **`config/upfcfg.yaml`**: Modificado para que a UPF (na VM3) faça o *bind* no endereço IP correto (`10.1.0.5`).
* **`run.sh`**: (Este arquivo deve ser dividido)
    * Na VM2, o script foi alterado para **não** iniciar a UPF.
    * Na VM3, um script foi criado para iniciar **apenas** a UPF.
* **`Makefile`**: (Opcional) O processo de build é separado:
    * Na VM2, executa-se `make`.
    * Na VM3, executa-se `make upf`.

## 🚀 Como Executar

**Pré-requisitos:**
* 3 Máquinas Virtuais (VMs) configuradas conforme a topologia.
* Dependências do free5gc (Go, bibliotecas C, etc.) instaladas em ambas VM2 e VM3.
* MongoDB instalado e rodando na VM2.

### 1. Na VM2 (Core Network)

1.  Clone este repositório:
    ```bash
    git clone https://github.com/IcaroC0der/free5gc-core-UPF-split free5gc-CORE
    cd free5gc-core
    ```
2.  Compile os NFs do Core:
    ```bash
    make
    ```
3.  Execute o script do Core:
    ```bash
    ./run.sh
    ```

### 2. Na VM3 (UPF)

1.  Clone este repositório:
    ```bash
    git clone https://github.com/IcaroC0der/free5gc-core-UPF-split free5gc-CORE
    cd free5gc-upf
    ```
2.  Compile apenas a UPF:
    ```bash
    make upf
    ```
3.  Execute o script da UPF:
    ```bash
    sudo ./run_upf.sh
    ```

Após estes passos, a SMF na VM2 deve conseguir estabelecer uma sessão PFCP com a UPF na VM3 através da rede `br-core-UPF`.
