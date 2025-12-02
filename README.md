# sel337_proj5

Eduardo Remoli de Souza Lopes - 14567332

Fábio Domingues Barreto da Silva - 14569011

# Init System e systemd 

O Init System é um estágio da inicialização do sistema operacional que ocorre logo após o carregamento do Kernel Linux. O *systemd* é uma escolha moderna de ferramenta que implementa o estágio Init System em uma distribuição Linux, sendo usada em várias distros populares como RedHat, Debian, Suse, CentOS, e também a Raspberry Pi OS utilizada no projeto.

O *systemd* é capaz de inicializar, monitorar, e finalizar serviços durante o boot, permitindo a criação de aplicativos que executem suas funções a partir do momento em que a OS é inicializada.

# Aplicação para blink de um LED

O arquivo *blink.sh*, escrito em bash script, começa o blink de um LED conectado à GPIO 18 da placa quando executado:

- `#!/bin/bash` aponta ao caminho do interpretador bash;
- `echo 18 > /sys/class/gpio/export` torna disponível o controle do pino GPIO 18 por meio do sistema de arquivos;
- `echo out > /sys/class/gpio/gpio18/direction` escreve out na direção do pino, configurando-o como saída;
- O restante do código é um loop que alterna o valor do pino entre 1 e 0 após um pequeno intervalo de tempo.

Já o arquivo *blinkoff.sh*, também escrito em bash script, tem o fim pretendido de ser executado sempre que o serviço parar, assim garantindo que o pino GPIO 18 seja unexported (por meio da linha de código `echo 18 > /sys/class/gpio/unexport`), possibilitando a repetição da execução de blink sem a necessidade de um reboot.

O unit file *blink.service* é responsável por incluir um serviço no systemd que utilize dos bash scripts escritos, sendo *blink.sh* definido como *ExecStart* e assim sendo executado na inicialização do serviço, e *blinkoff.sh* definido como *ExecStop* e assim sendo executado ao interromper o serviço:

```
[Unit]
Description= Blink LED
After=multi-user.target

[Service]
ExecStart=/home/sel/1132/p5/blink.sh
ExecStop=/home/sel/1132/p5/blinkoff.sh
user=SEL

[Install]
WantedBy=multi-user.target
```

Para o funcionamento do projeto, basta então copiar *blink.service* ao diretório de serviços do *systemd* (*/lib/systemd/system*), certificando-se de que os arquivos bash script estão no caminho indicado, e habilitar o serviço por meio do utilitário *systemctl* no terminal:

`sudo systemctl enable blink`

O serviço deve agora inicializar durante o boot da Raspberry Pi. Algumas outras utilidades do *systemctl* para o propósito de testes estão abaixo:

- `sudo systemctl stop blink` para parar o serviço;
- `sudo systemctl start blink` para iniciar o serviço;
- `systemctl status blink.service` para verificar o funcionamento e estado do serviço;
- `sudo systemctl disable blink` para desabilitar o serviço do boot;
- `sudo systemctl daemon-reload` recarrega o serviço, para caso alguma alteração tenha sido feita.
