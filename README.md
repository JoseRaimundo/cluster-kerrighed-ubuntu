


### Montando um Cluster 

### Instalando o Kerrighed

Antes de começar, é necessário instalar algumas dependências, o comando a seguir instalará tudo de uma vez.
		
	sudo apt-get install automake autoconf libtool pkg-config rsync bzip2 libncurses5 libncurses5-dev wget lsb-release xmlto patchutils xutils-dev build-essential libtool-bin bootstrap-vz

Pronto, agora sua máquina está preparada para instalar o Kerrighed. Abra o terminal e vá até paste a pasta src pelo terminal 

	cd /usr/src/

Neste diretório, faça o download do Kerrighed (por está em uma pasta do sistema, é necessário utilizar um usuário com permissões super).

> IMPORTANTE: Neste tutorial são apresentados todos os erros ocorridos durante a instalação do Kerrighed e também as respectivas soluções adotadas. Porém se você pode baixar diretamente os arquivos deste repositório e pular a seção de erros, pois as devidas correções já estão aplicadas. Caso queira pular, vá para a seção Instalação Direta.

	sudo wget http://gforge.inria.fr/frs/download.php/27161/kerrighed-3.0.0.tar.gz

Baixe também o kernel apropriado (durante a instalação do Kerrighed, ele faz o download automaticamente, porém o link utilizado pelo instalador está desativado): 

	sudo wget http://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.30.tar.bz2 

Descompacte o Kerrighed e kernel linux:

	sudo tar zxf kerrighed-3.0.0.tar.gz 

	sudo tar  -xvjf linux-2.6.30.tar.bz2 

Renomeie a pasta dos arquivos para kerrighed-src:

	sudo mv kerrighed-3.0.0 kerrighed-src

#### Primeiro error (Versão do GCC)

De acordo com a documentação do Kerrighed, basca executar o arquivo configure, porém ele irá utilizar o compilador gcc padrão do sistema operacional, e irá gerar um o erro: *cc1: error: code model kernel does not support PIC mode. Após algumas pesquisa, foi identificado que se trata de uma incompatibilidade dos tipos de variáveis do Kerrighed com as versões mais modernas do GCC. Para solucionar este problema, é necessário instalar uma versão mais antiga do GCC, as testadas neste tutorial foi a 4.4.8, 4.1.0 e 4.1.2. A versão 4.1.2 se mostrou mais apropriada por apresentar menos problemas.

Vá  ao diretório tmp e crie uma pasta chamada gcc.

	cd /tmp
	mkdir /tmp/gcc

Dentro do diretório gcc, baixe e descompacte os arquivos do gcc-4.1.2. Em seguida crie a pasta em que irá receber os arquivos compilados.


	cd /tmp/gcc/

	wget http://ftp.gnu.org/gnu/gcc/gcc-4.1.2/gcc-4.1.2.tar.bz2

	tar -xvjpf ./gcc-4.1.2.tar.bz2
	

Instale algumas dependências necessárias para instalação completa desta versão do gcc.

	sudo apt-get install linux-headers-$(uname -r) zlib1g zlib1g-dev zlibc gcc-multilib

#### Segundo error - conflitos de bibliotecas

Ao instalar o pacote *build-essentials* no inicio deste tutorial, foi instalado uma versão atualizada do GCC, logo será necessário realizar uma alteração no arquivo *configure* do GCC para evitar um problema de conflitos.
Para solucionar este problema, abra o arquivo configure (neste tutorial , utilizei o gedit, nele é possível configurar a exibição do número das linhas, preferences>view>display line numbers).

	gedit gcc-4.1.2/libstdc++-v3/configure

Na linha 8284, localize o seguinte trecho:

	sed -e 's/GNU ld version \([0-9.][0-9.]*\).*/\1/'`

E substitua por:

	sed -e 's/GNU ld (GNU Binutils for Ubuntu) \([0-9.][0-9.]*\).*/\1/'`

Salve e feche o arquivo.
Continuando com a instalação do gcc-4.1.2, execute os seguintes comandos:

	sudo ln -s /usr/lib/x86_64-linux-gnu/crt1.o /usr/lib/crt1.0
	
	sudo ln -s /usr/lib/x86_64-linux-gnu/crti.o /usr/lib/crti.o

	sudo ln -s /usr/lib/x86_64-linux-gnu/crtn.o /usr/lib/crtn.o


Agora é necessário recompilar o gcc, para isso vá para o diretório gcc que criamos, e crie e entre dentro da pasta que abrigará os arquivos compilados.

	cd /tmp/gcc/

	mkdir ./build
	
	cd build

Agora execute o comando que ira configurar a compilação de acordo com os requisitos desejados.

	../gcc-4.1.2/configure --program-suffix=-4.1 --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --disable-libunwind-exceptions --enable-__cxa_atexit --enable-languages=c,c++ --disable-multilib

A partir daqui é realizado a construção dos arquivos, durante a compilação aconteceu alguns erros. Neste tutorial são apresentadas as soluções para cada erro no momento em que o mesmo aparece durante a compilação.

Comando para compilação:

	sudo make -j 2 bootstrap MAKEINFO=makeinfo


Primeiro erro:

![erro 1](https://github.com/JoseRaimundo/cluster-kerrighed-ubuntu/blob/master/img/erro1.png?raw=true)


Para solucionar, edite o Makefile.
	
	gedit Makefile

Substituindo:

	CC = gcc
	CXX = g++

Por:
	
	CC = gcc -fgnu89-inline
	CXX = g++ -fgnu89-inline


Repita o comando de compilação:
	
	sudo make -j 2 bootstrap MAKEINFO=makeinfo

O próximo erro:

![enter image description here](https://github.com/JoseRaimundo/cluster-kerrighed-ubuntu/blob/master/img/erro2.png?raw=true)

Para solucionar, abra o arquivo linux-unwind.h na pasta do gcc-4.1.2.

	gedit gedit ../gcc-4.1.2/gcc/config/i386/linux-unwind.h

Na linha 139, substitua:
	
	struct siginfo *pinfo;

Por:

	siginfo_t *pinfo;

E na linha 141, substitua:
	
	struct siginfo info;

Por: 

	siginfo_t info;
	
Se você continuar, outro erro no mesmo arquivo irá acontecer.

![imagem error3](https://github.com/JoseRaimundo/cluster-kerrighed-ubuntu/blob/master/img/erro3.png?raw=true)

Substitundo na linha 58:

	sc = (struct sigcontext *) (void *) &uc_->uc_mcontext;

Por:

	// sc = (struct sigcontext *) (void *) &uc_->uc_mcontext;

Salve e feche o arquivo, e continue a compilação e finalmente instale o gcc.

	sudo make -j 2 bootstrap MAKEINFO=makeinfo
	
	sudo make install


### Compilando o Kerrighed



Volte para a pasta do Kerrighed e execute os seguintes comandos (o comando de configuração duas vezes).


	cd /usr/src/kerrighed-src/
	
	sudo ./autogen.sh

	sudo ./configure --sysconfdir=/etc CC=gcc-4.1
	
	sudo ./configure --sysconfdir=/etc CC=gcc-4.1


Informe para o Kerrighed quanto núcleos disponíveis sua máquina possuí:

	export CONCURRENCY_LEVEL=<número de núcleos>

Configure os drives e demais characteristics que requem suporte, para isso basta acessar a pasta do kernel e compilar o menu, irá aparecer uma janela para selecionar as opções desejadas :

	cd kernel
	
	make menuconfig

> IMPORTANTE: A configuração default é suficiente para o kernel utilizado, porém é necessário desmarcar as seguintes opções: 
>> Device drivers -> Macintosh device drivers
>>
>> Device drivers -> Network device support -> Ethernet (10 or 100 MBits)
>>
 >> Device drivers -> Network device support -> Ethernet (10 00 MBits)
 > 
>Volte para o menu principal, salve as configurações e saia do menu.

Execute o comando de compilação do código do Kerrighed setando a versão do GCC que instalamos.

	sudo make kernel CC=gcc-4.1


Irá aparecer o erro:

 ![imagem erro 4](https://github.com/JoseRaimundo/cluster-kerrighed-ubuntu/blob/master/img/erro4.png?raw=true)

Para solucionar, basta abrir o arquivo timeconst:

	sudo gedit _kernel/kernel/timeconst.pl 

Na linha 374, substitua:

	if (!defined(@val)) {

Por:

	if (!(@val)) {

Salve e feche o arquivo, agora abra o arquivo sumversion.c:

	gedit _kernel/scripts/mod/sumversion.c

E na primeira linha adicione a linha:

	#include <limits.h> 

Salve e feche o arquivo e continue com a compilação.
 
	sudo make kernel CC=gcc-4.1

Agora é só instalar e usar o Kerrighed:
	
	sudo make install CC=gcc-4.1

Para usar, você pode consultar a documentação presente no site oficial do [Kerrighed](http://www.kerrighed.org/).



----


----


		

		

### Instalação Direta
