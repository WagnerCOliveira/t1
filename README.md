Trabalho para disciplina Introdução a python - Ciencia de Dados.
===

Projeto de T1 - Contratos de Energia.
===

Atividade
===

Dado um arquivo de entrada, implemente um algoritmo que inicialize uma matriz tridimensional
que armazene os valores dos contratos de energia. A matriz deve ter as dimensões n × (m + 1) ×
(m + 1), onde cada elemento contratos[fornecedor][inicio][fim] representa o valor do contrato
oferecido pelo fornecedor para o período do mês inicial ao mês final. Se não houver contrato
específico para esse período, o valor deve ser ∞ (infinito).

Tabela de conteúdos
---
<!--ts-->   
   * [Tecnologias](#🛠-tecnologias-utilizadas)
   * [Criação Virtualenv](#criação-virtualenv)
   * [Instalação Pacotes](#instalação-de-pacotes)
   * [Acessando Virtualenv](#acessando-virtualenv---wsl-linux)
   * [Codigo](#código)
     * [Arquivo main.py](#código)
     * [Arquivo lib.py](#código)
   * [Referências](#referências)
   * [Contribuição](#contribuição)
   * [Autor](#autor)
   * [Licença](#licença)
<!--te-->

🛠 Tecnologias Utilizadas
---
As seguintes ferramentas foram usadas na construção do projeto:

- [Python 3.13.0](https://docs.python.org/pt-br/3/)
- [gdown==5.2.0](https://pypi.org/project/gdown/)
- [rich==13.9.4](https://github.com/Textualize/rich)


Criação Virtualenv
---


~~~bash
python3 -m venv .venv
~~~


Acessando Virtualenv - WSL, Linux
---


~~~bash
source .venv/bin/activate
~~~


Acessando Virtualenv - Windows
---


~~~bash
.venv/Scripts/activate.bat
~~~


Instalação de Pacotes
---


~~~bash
python -m pip install -r requirements.txt
~~~


Código
---


O app é responsavel por processar os seguintes passos:
  
  * Fazer o download de uma dataset
  * Processar o dataset
  * Imprimir o conteudo da matriz
  * Criar um arquivo csv com o conteudo da matriz


1. main.py


O código descrido neste arquivo faz as chamadas no pacote lib para executar as funções:


* download_dataset()
* preencher_matriz_contratos()
* imprimir_matriz()
* exportar_csv()

   
2. lib.py

O pacote lib contem a logica necessária para o preenchimento da matriz.


Imports

~~~python
import os                         # Responsavel pela interação com sistema operacional
import time                       # Funcionalidades com relação ao tempo
from rich.progress import track   # Renderizar barras de progresso
from gdown import download        # Google Drive Public File Downloader
~~~


Função **download_dataset()** verifica se o doiretorio dataset existe se não existe o cria,
e executa a função **download** do pacote **gdown**.

~~~python
def download_dataset(link: str,filename: str):
    '''
    Faz o donload do arquivo entrada.txt com dataset.
    '''
    # Criação do diretório dataset
    diretorio = str(filename).split('/')    
    if not os.path.exists(diretorio[0]):
       os.makedirs(diretorio[0])
    
    # Download do arquivo entrada.txt
    file_id = link.split('/')[5]          
    url = f'https://drive.google.com/uc?id={file_id}'
    download(url, filename) 
    
    return None
~~~

Função **preencher_matriz_contratost()** preenche uma matriz 3d dinamicamente de acordo com os
dados, do intervalo com os meses de contrato, uma quantidade de fornecedores e taxa, com os atribuidos da primera linha no arquivo entrada.txt, que feito download de um drive na internet, 
disponibilizado pelo professor

~~~python
def preencher_matriz_contratos(nome_arquivo: str):    
    with open(nome_arquivo, 'r') as arquivo:
        # lendo primeira linha para atribuir intervalo de meses, forncedores e taxa
        primeira_linha = arquivo.readline() 
        dados_primeira_linha = primeira_linha.strip().split()
        m = int(dados_primeira_linha[0]) # Qtd Meses Contratação
        n = int(dados_primeira_linha[1]) # Fornecedores
        t = float(dados_primeira_linha[2]) # Taxa
           
        # Carregando linhas do arquivo em uma lista
        linhas = arquivo.readlines() 
                
        # Ordenando a lista linhas pelos fornecedores
        fornecedores = []
        for linha in sorted(linhas): 
            fornecedores.append(linha)

        # Dividindo os fornecedores em blocos de processamento no formato de listas.
        # fazendo um calculo com base na varivel n, para descobrir o total de linhas para cada fornecedor e por consequencia ter o tamanho de cada bloco.
        bloco_fornecedores = []
        len_fornecedores = len(fornecedores)
        print(len_fornecedores)        
        for fornecedor in range(n):
            start = int(fornecedor * len_fornecedores/n)
            end = int((fornecedor+1) * len_fornecedores/n)            
            bloco_fornecedores.append(fornecedores[start:end])         
                
        # Processo de alimentar a matriz 3D
        matriz_3d = []
        
        for bloco in bloco_fornecedores:
            # instancio uma matriz 2d
            inf = float('inf')
            nlinhas = ncols = m + 1
            matriz = [inf] * nlinhas
            for i in range(nlinhas):
                matriz[i] = [inf] * ncols
            
            # Barra de progresso.
            len_bloco = len(bloco)
            id_fornecedor = bloco[0].split()[0]
            total = 0
            for _ in track(
                range(100), 
                description=f'Fornecedor {id_fornecedor}, {len_bloco} linhas.'
            ):
                # Para cada bloco adiciono o dado na matrix 2d
                for indice in bloco: 
                    matriz[int(indice.split()[1])][int(indice.split()[2])] = float(indice.split()[3])                                    
                
                time.sleep(0.01) # tempo simbolico contagem na barra de status
                total += 1 # contagem da porcentagem da barra de rolagem

            matriz_3d.append(matriz) # Adicionando matriz 2d na matriz 3d
            matriz = [] # limpando a matriz 2d
            
        arquivo.close() # fechando arquivo.

    return m, n, t, matriz_3d
~~~


Função **imprimir_matriz()** imprime na tela cada linha das matrizes.

~~~python
def imprimir_matriz(matriz, k=None):
    
    for i in matriz:
        for j in i:
            print(j)
        print(' ')

    return None
~~~

Função **exportar_csv()** Faz o export para um arquivo .csv da matriz 3d com uma linha em branco separando-as.

~~~python
def exportar_csv(nome_arquivo, matriz):
    
    # Testa de o direório existe, se não cria o mesmo.
    diretorio = str(nome_arquivo).split('/')    
    if not os.path.exists(diretorio[0]):
       os.makedirs(diretorio[0])
    
    # Lê a matriz imprimindo cada linha da matriz 3d em um arquivo .csv.
    with open(nome_arquivo, 'w') as arquivo:
        for i in matriz:
            for j in i:
                arquivo.write(str(j)+ '\n')
            arquivo.write('\n')

    return None
~~~

### Referências
---
- [Python Documentação](https://docs.python.org/pt-br/3/)
- [Google Download](https://pypi.org/project/gdown/)
- [Rich Textualize](https://github.com/Textualize/rich)

### Contribuições

- Divanilza Ferreira Flexa
- Emerson da Silva Maciel
- Isabele Benevinuto Sabóia
- Victor Lamark Costa Brasil
- Wagner da Costa Oliveira

### Autores

- Divanilza Ferreira Flexa
- Emerson da Silva Maciel
- Isabele Benevinuto Sabóia
- Victor Lamark Costa Brasil
- Wagner da Costa Oliveira

### Licença

- [GNU General Public License (GPL)](https://www.gnu.org/licenses/gpl-3.0.html)
