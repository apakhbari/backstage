# backstage
```
 _______  _______  _______  ___   _  _______  _______  _______  _______  _______ 
|  _    ||   _   ||       ||   | | ||       ||       ||   _   ||       ||       |
| |_|   ||  |_|  ||       ||   |_| ||  _____||_     _||  |_|  ||    ___||    ___|
|       ||       ||       ||      _|| |_____   |   |  |       ||   | __ |   |___ 
|  _   | |       ||      _||     |_ |_____  |  |   |  |       ||   ||  ||    ___|
| |_|   ||   _   ||     |_ |    _  | _____| |  |   |  |   _   ||   |_| ||   |___ 
|_______||__| |__||_______||___| |_||_______|  |___|  |__| |__||_______||_______|
```

## System Architecture
1. Back End
2. Front End
3. DB

- BE & FE can be in a single container but it is for dev purposes and not recommended.
- DB is postgres, but can use SQLite for dev purposes, if you use SQLIte it is going to be ephemeral so every time you restart it, data will lose

## Components
### Core
Deployed by default (e.g. software catalog)
1. SOftware catalog: all info about system & organization
2. Tech Docs: Documentatin inside backstage
3. Search: search across all backstage portal
4. Service Template: bootstrap services in backstage

### App (Integrations)
Extend capabilities of backstage. Deployed but require configuration (e.g. SSO / Analytics / Git)

### Plugins
Require installation, integration & configuration to your backstage application manually

Types of plugins:
- Service backed Plugins: Service is in same cluster/entity. deployed on different service in same system
- proxy backed plugin: redirect to proxy, which fetch info from some service outside of this cluster/entity

## Config files
- app-config.yaml --> app config. all settings are here, plugins are being add here
- catalog --> holds components in your system

# acknowledgment
## Contributors

APA 🖖🏻

## Links
Platform Engineering Series : https://youtube.com/playlist?list=PLGVPcLSzJXQos1O18dvKoW2XSczz2I2lH&si=Jf4lDdaGW4GMLQYt

```                                                                                                       
  aaaaaaaaaaaaa  ppppp   ppppppppp     aaaaaaaaaaaaa   
  a::::::::::::a p::::ppp:::::::::p    a::::::::::::a  
  aaaaaaaaa:::::ap:::::::::::::::::p   aaaaaaaaa:::::a 
           a::::app::::::ppppp::::::p           a::::a 
    aaaaaaa:::::a p:::::p     p:::::p    aaaaaaa:::::a 
  aa::::::::::::a p:::::p     p:::::p  aa::::::::::::a 
 a::::aaaa::::::a p:::::p     p:::::p a::::aaaa::::::a 
a::::a    a:::::a p:::::p    p::::::pa::::a    a:::::a 
a::::a    a:::::a p:::::ppppp:::::::pa::::a    a:::::a 
a:::::aaaa::::::a p::::::::::::::::p a:::::aaaa::::::a 
 a::::::::::aa:::ap::::::::::::::pp   a::::::::::aa:::a
  aaaaaaaaaa  aaaap::::::pppppppp      aaaaaaaaaa  aaaa
                  p:::::p                              
                  p:::::p                              
                 p:::::::p                             
                 p:::::::p                             
                 p:::::::p                             
                 ppppppppp                             
                                                       
```
