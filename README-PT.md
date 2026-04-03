vban - Ferramentas VBAN de linha de comando para Linux
======================================================

© 2015 Benoît Quiniou - quiniouben[at]yahoo(.)fr

O projeto vban é uma implementação de código aberto do protocolo VBAN.

O VBAN é um protocolo simples de áudio sobre UDP proposto pela VB-Audio, veja a [página da VBAN Audio](https://www.vb-audio.com/Voicemeeter/vban.htm).
Ele é composto por diversas ferramentas de linha de comando que permitem transmitir áudio proveniente de interfaces de áudio para um fluxo VBAN (vban_emitter), reproduzir um fluxo VBAN recebido em interfaces de áudio (vban_receptor) ou enviar texto pelo protocolo VBAN (vban_sendtext).

Até o momento, para ferramentas de áudio, foram implementados os backends de áudio Alsa, PulseAudio e Jack. Também existe uma saída FIFO (pipe), para permitir o encadeamento de ferramentas de linha de comando, e uma saída de arquivo (gravando dados PCM brutos).

Compilação e instalação
----------------------------

Dependendo dos backends de áudio que você deseja compilar, você precisará das bibliotecas e dos cabeçalhos de origem correspondentes.

Os nomes de pacotes usuais são:

* Alsa: libasound(X) e, eventualmente, libasound(X)-dev
* PulseAudio: libpulse(X) e, eventualmente, libpulse(X)-dev
* Jack: libjack(X) e, eventualmente, libjack(X)-dev

O vban é distribuído com scripts de compilação do autotools; portanto, para compilar, você precisa instalar o autoconf e o automake e executar:

	$ ./autogen.sh # provavelmente apenas uma vez
	$ ./configure # com ou sem opções (--help para obter a lista de opções possíveis)
	$ make # com ou sem opções

Para instalar, basta executar:
	
	# make install

Por padrão, as ferramentas vban serão compiladas com todos os 3 backends de áudio. Para desativá-los, as opções de configuração são:

	--disable-alsa
	--disable-pulseaudio
	--disable-jack

Uso
-----

Invocar vban_receptor ou vban_emitter sem nenhum parâmetro fornecerá dicas sobre como usá-los:

	Uso: vban_receptor [OPÇÕES]...

	-i, --ipaddress=IP : OBRIGATÓRIO. Endereço IP para obter o fluxo.
	-p, --port=PORTA : OBRIGATÓRIO. Porta para escutar.
	-s, --streamname=NOME : OBRIGATÓRIO. Nome do fluxo para reproduzir.
	-b, --backend=TIPO : Backend de áudio a ser usado. Os backends de áudio disponíveis são: alsa, pulseaudio, jack, pipe e file. O padrão é alsa.
	-q, --quality=ID : Indicador de qualidade da rede de 0 (baixa latência) a 4. Isso também interage com o tamanho do buffer jack. O padrão é 1.
	-c, --channels=LISTA : Canais do fluxo a serem usados. A LISTA tem o formato x,y,z,... O padrão é encaminhar o fluxo como está.
	-o, --output=NOME: OBSOLETO. Use -d.
	-d, --device=NOME: Nome do dispositivo de áudio. Este é o nome do arquivo para o backend de arquivo, o nome do servidor para o backend JACK, o dispositivo para ALSA e o nome do fluxo para PulseAudio.
	-l, --loglevel=NÍVEL: Nível de log, de 0 (FATAL) a 4 (DEPURAÇÃO). O padrão é 1 (ERRO).
	-h, --help: Exibe esta mensagem.

	Uso: vban_emitter [OPÇÕES]...
	
	-i, --ipaddress=IP: OBRIGATÓRIO. Endereço IP para o qual enviar o fluxo.
	-p, --port=PORTA: OBRIGATÓRIO. Porta a ser usada.
	-s, --streamname=NOME: OBRIGATÓRIO. Nome do fluxo a ser usado.
	-b, --backend=TIPO: DESATIVADO TEMPORARIAMENTE. Backend de áudio a ser usado. Somente o backend ALSA está funcionando neste momento.
	-d, --device=NOME: Nome do dispositivo de áudio. Este é o nome do arquivo para o backend de arquivos, o nome do servidor para o backend JACK, o dispositivo para o ALSA e o nome do fluxo para o PulseAudio.
	-r, --rate=VALOR: Taxa de amostragem do dispositivo de áudio. O padrão é 44100.
	-n, --nbchannels=VALOR: Número de canais do dispositivo de áudio. O padrão é 2.
	-f, --format=VALOR: Formato de amostragem do dispositivo de áudio (veja abaixo). O padrão é 16I (inteiro de 16 bits).
	-c, --channels=LISTA: Canais do fluxo a serem usados. A LISTA tem o formato x,y,z,... O padrão é encaminhar o fluxo como está.
	-l, --loglevel=NÍVEL: Nível de log, de 0 (FATAL) a 4 (DEBUG). O valor padrão é 1 (ERRO)
	-h, --help: exibe esta mensagem

	Formatos de bits reconhecidos: 8I, 16I, 24I, 32I, 32F, 64F, 12I, 10I
	
	Uso: vban_sendtext [OPÇÕES] MENSAGEM

	-i, --ipaddress=IP: OBRIGATÓRIO. Endereço IP para o qual enviar o fluxo.
	-p, --port=PORTA: OBRIGATÓRIO. Porta a ser usada.
	-s, --streamname=NOME: OBRIGATÓRIO. Nome do fluxo a ser usado.
	-b, --bps=VALOR: Indicador de taxa de bits de dados. Padrão: 0 (sem taxa de bits específica).
	-n, --ident=VALOR: Identificação do subcanal. Padrão: 0.
	-f, --format=VALOR: Formato de texto usado. Pode ser: 0 (ASCII), 1 (UTF8), 2 (WCHAR), 240 (USUÁRIO). padrão 1
	-l, --loglevel=LEVEL: Nível de registro, de 0 (FATAL) a 4 (DEBUG). O padrão é 1 (ERRO)
	-h, --help: exibe esta mensagem

Sobre a opção --channels, mais algumas dicas:
* Os índices dos canais variam de 1 a 256 (conforme especificado pelas especificações VBAN e, provavelmente, suficiente para qualquer placa de som ou configuração de entrada/saída)
* Você pode repetir canais
* Se você usar canais inexistentes, ouvirá silêncio, mas nenhum erro

Exemplos:

	vban_receptor -i IP -p PORTA -s NOME_DO_FLUXO -c1 # manter apenas o canal 1 e reproduzir em mono
	vban_receptor -i IP -p PORTA -s NOME_DO_FLUXO -c1,1,1,1 # manter apenas o canal 1 e reproduzir em 4 canais de saída (desde que seu dispositivo de saída seja capaz de fazê-lo)
	vban_receptor -i IP -p PORTA -s NOME_DO_FLUXO -c2,41,125,7,1,45 # selecionar alguns canais e reproduzi-los em 6 canais de saída (mesmo comentário)
	vban_emitter -i IP -p PORTA -s NOME_DO_FLUXO -c1,1,1,1 # usar o canal de áudio 1 como fonte (abrindo-o em mono, portanto, e criando um fluxo de 4 canais com cópias dos mesmos dados em todos os canais)
	vban_sendtext -i IP -p 6980 -sCommand1 "Strip(0).mute = 1;" # Silenciar a faixa 1 do VoiceMeeter Banana. Consulte o [manual do VoiceMeeter Banana](https://www.vb-audio.com/Voicemeeter/VoicemeeterBanana_UserManual.pdf) para obter mais informações.

LATÊNCIA
-------

O vban_receptor faz o possível para manter a latência em níveis razoáveis, de acordo com o parâmetro -q (--quality).

O tamanho do buffer é calculado de acordo com o parâmetro quality, seguindo a recomendação do documento de especificação do protocolo VBAN.
São:
* os dados são lidos/gravados na rede em blocos do tamanho do buffer
* para ALSA, o tamanho do buffer é usado para garantir uma latência adequada
* para PulseAudio, ele é usado diretamente para definir o tamanho do buffer de fluxo
* para JACK, ele é usado para definir um tamanho de buffer interno de tamanho duplo

GUI
---

Este projeto é composto apenas por ferramentas de linha de comando. Se você procura uma interface gráfica, pode consultar o projeto [VBAN-manager no GitHub](https://github.com/VBAN-manager/VBAN-manager)
