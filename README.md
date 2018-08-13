


### Montando um Cluster 

### Instalando o Kerrighed

Antes de começar, é necessário instalar algumas dependências, o comando a seguir instalará tudo de uma vez.
		
	sudo apt-get install automake autoconf libtool pkg-config rsync bzip2 libncurses5 libncurses5-dev wget lsb-release xmlto patchutils xutils-dev build-essential libtool-bin

Pronto, agora sua máquina está preparada para instalar o Kerrighed. Abra o terminal e vá até paste a pasta src pelo terminal 

	cd /usr/src/

Neste diretório, faça o download do Kerrighed (por está em uma pasta do sistema, é necessário utilizar um usuário com permissões super).

	sudo wget http://gforge.inria.fr/frs/download.php/27161/kerrighed-3.0.0.tar.gz

Baixe também o kernel apropriado (durante a instalação do Kerrighed, ele faz o download automaticamente, porém o link utilizado pelo instalador está desativado): 

	sudo wget http://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.30.tar.bz2 

Descompacte o Kerrighed e kernel linux:

	sudo tar zxf kerrighed-3.0.0.tar.gz 

	sudo tar  -xvjf linux-2.6.30.tar.bz2 

Renomeie a pasta dos arquivos para kerrighed-src:

	sudo mv kerrighed-3.0.0 kerrighed-src

#### Primeiro error

De acordo com a documentação do Kerrighed, basca executar o arquivo configure, porém ele irá utilizar o compilador gcc padrão do sistema operacional, e irá gerar um o erro: *cc1: error: code model kernel does not support PIC mode. Após algumas pesquisa, foi identificado que se trata de uma incompatibilidade dos tipos de variáveis do Kerrighed com as versões mais modernas do GCC. Para solucionar este problema, é necessário instalar uma versão mais antiga do GCC, as testadas neste tutorial foi a 4.4.8, 4.1.0 e 4.1.2. A versão 4.1.0 se mostrou mais apropriada por apresentar menos problemas.

Vá ao diretório tmp e crie uma pasta chamada gcc.

	cd /tmp
	mkdir /tmp/gcc

Dentro do diretório gcc, baixe e descompacte os arquivos do gcc-4.1.0.


	cd /tmp/gcc/

	wget http://ftp.gnu.org/gnu/gcc/gcc-4.1.2/gcc-4.1.2.tar.bz2




























Agora execute separadamente os seguintes comandos: 

	cd kerrighed-src

	sudo ./autogen.sh 

		

