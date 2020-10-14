# :triangular_flag_on_post: CTF Zup 2020  Write up 1#


Este documento contém todas as respostas (passo a passo) e ferramentas utilizadas para resolução 
dos desafios do CTF Zup 2020, ou pelo menos aqueles que eu consegui resolver :rofl:

### Categorias
1. [Android](#MultiMarkdownOverview)
    - [Debug](#debug-me)
    - [Decode](#decode)
    - [Db leak](#db-leak)
    - [Export](#export)
    - [File access](#file-access)
    - Frida1
    - [Intercept](#intercept)
    - [Resources](#resources)
    - [Shared preferences](#shared-preferences)
    - Smali
    - [Strings](#strings)
2. [Criptografia](criptografia/index)
    - Not a caesar algorithm
3. [Forense](forense/index)
    - Pdf crypt
    - A lost pendrive
    - Unk
4. [Misc](misc/index)
    - Access log
    - Rotate
5. [Pwns](pown/index)
    - Pwn1
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

## Forense :mag:

## Misc :earth_americas:

## Pwns :computer:

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
