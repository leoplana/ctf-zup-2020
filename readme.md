# :triangular_flag_on_post: CTF Zup 2020  Write up 1#


Este documento contém todas as respostas (passo a passo) e ferramentas utilizadas para resolução 
dos desafios do CTF Zup 2020, ou pelo menos aqueles que eu consegui resolver :rofl:

### Categorias
1. [Android](#MultiMarkdownOverview)
    - [Debug](#debug-me)
    - Decode
    - Db leak
    - Export
    - File access
    - Frida1
    - Intercept
    - Resources
    - Shared preferences
    - Smali
    - Strings
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

## Criptografia :key:

## Forense :mag:

## Misc :earth_americas:

## Pwns :computer:

## Aplicações Web :globe_with_meridians:



# Ferramentas #

### Android Studio ###
IDE para desenvolvimento Android disponível neste [link](https://redirector.gvt1.com/edgedl/android/studio/install/4.1.0.19/android-studio-ide-201.6858069-windows.exe) 
