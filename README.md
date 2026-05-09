# Amygdal
Privacy-focused Firefox fork with local website reputation scoring, built-in ad blocking, cookie banner removal, temp mail, and more. All classification is local — no URLs sent to external servers. Main Language of the Project is Catalan.

**sigues ningú, sigues res, sigues inrastrejable**

Amygdal és un fork de Firefox centrat en **privacitat** i **seguretat**, creat com a navegador en català. Tota la classificació de reputació és **LOCAL** — cap URL, domini o hash s'envia a cap servidor extern, aixo es fa aixi per tal de preservar la integritat de les consultes i que no hi hagi ningu que pogui mirar les peticions i saber quines webs es consulten..
 
🌐 [amygdal.llucomella.com](https://amygdal.llucomella.com)
 
---
 
## Funcionalitats
 
| Funcionalitat | Descripció |
|---------------|------------|
| 🛡️ Reputació web local | En total hi han 424 dominis classificats en Sets (O(1)), hi han 4 tipus de nivells, Fiable, Desconegut, Sospitós, Perill. Per fer-ho mes visual hem posat "Badges" als cercadors, barra URL, tooltips i intercepció de clics aixi quan el usuari navega de manera nativa ja te de una manera mes facil una idea del lloc web on entra. |
| 🔒 DuckDuckGo exclusiu | Hem posat com a únic motor de cerca DuckDuckGo(DDG). Altres buscadors com Google, Bing i Yahoo eliminats. No es poden afegir nous motors per una qüestio de privacitat |
| 🚫 Bloqueig d'anuncis | Hem afegit com a manera nativa la extensio de uBlock Origin es molt bona i no tenia sentit reinventar la roda, la hem integrat com a system addon (no desinstal·lable) |
| 🍪 Eliminació de cookies banners | Per tal de evitar rastreig, i que ens agafin informació mentre naveguem, Amygdal elimina les cookies pero per evitar incidencies de que deixi webs sense funcionar hem fet una detecció a tres capes, primer ocultem les finestres de acceptar les cookies (CSS Injection), despres netejem el DOM, i finalment analitzem els 50 frameworks mes populars de cookies per bloquejarlos amb les llistes de uBlock |
| 📧 Temp Mail | Per tal de millorar la privacitat del usuari, una bona praxis es utilitzar identitats "falses" que no siguin com la teva identitat real per evitar que els rastrejadors fagin un perfil de tu, per simplificar el us i fer-lo mes accesible a la gent, hem integrat de manera nativa a tots els formularis de inici/registre un boto per posar automaticament el correu temporal ho gem amb Guerrilla Mail que disposa de una API gratuita |
| 🔑 Generador de contrasenyes | Contrasenyes fortes i frases de pas |
| 📥 Protecció de descàrregues | Verificació SHA-256 contra base de dades local URLhaus |
| 🔐 Privacitat avançada | Total Cookie Protection, First-Party Isolation, esborrat automàtic |
 
## Arquitectura
 
```
AmygdalReputationService (browser/modules/)
  424 dominis trusted / suspicious / dangerous (Sets en memòria, O(1))
  nsIURIClassifier (Safe Browsing local, si DB disponible)
  Cache LRU (10 min TTL)
       |
       +→ AmygdalSERPChild/Parent ("Badges" injectats directament a sota dels resultats de cerca)
       +→ AmygdalLinkTooltipChild/Parent (tooltips en enllaços)
       +→ ClickHandlerParent (intercepció de clics)
       +→ browser-amygdalSafety.js (badge URL bar + popups)
       +→ DownloadIntegration (hash check de les descàrregues)
 
AmygdalCookieKillerChild (eliminació de banners de cookies)
TempMail (correu temporal via Guerrilla Mail API)
Password Generator (generador de contrasenyes)
```
 
## Fitxers del projecte
 
Aquest repositori conté **només els fitxers creats o modificats** per Amygdal. Per compilar el navegador necessites el codi font de Firefox Nightly i aplicar aquests fitxers a sobre.
 
### Fitxers creats
 
```
browser/modules/AmygdalReputationService.sys.mjs      → Motor central de reputació
browser/actors/AmygdalSERPChild.sys.mjs                → Badges als cercadors (child)
browser/actors/AmygdalSERPParent.sys.mjs               → Badges als cercadors (parent)
browser/actors/AmygdalLinkTooltipChild.sys.mjs          → Tooltips en enllaços (child)
browser/actors/AmygdalLinkTooltipParent.sys.mjs         → Tooltips en enllaços (parent)
browser/actors/AmygdalCookieKillerChild.sys.mjs         → Eliminació de cookie banners
browser/base/content/browser-amygdalSafety.js           → Badge barra URL + popups
browser/base/content/browser-tempmail.js                → Correu temporal
browser/extensions/amygdal/temp-mail/                   → Extensió Temp Mail
browser/extensions/amygdal/password-gen/                → Generador de contrasenyes
browser/extensions/newtab/data/content/amygdal-settings.js → Configuració newtab
browser/extensions/ublock0/extension/                   → uBlock Origin v1.70.0 (system addon)
toolkit/components/downloads/AmygdalDownloadProtection.sys.mjs → Hash check descàrregues
```
 
### Fitxers modificats
 
```
browser/app/profile/firefox.js                          → Preferències de privacitat + system addon
browser/app/distribution/policies.json                  → Política DuckDuckGo exclusiu
browser/actors/ClickHandlerParent.sys.mjs               → Intercepció de clics sospitosos
browser/actors/moz.build                                → Registre actors
browser/modules/moz.build                               → Registre ReputationService
browser/components/BrowserGlue.sys.mjs                  → Registre JSWindowActors
browser/base/content/navigator-toolbox.inc.xhtml        → Badge XUL a la barra
browser/base/content/browser.js                         → Crida onLocationChange
browser/branding/unofficial/                            → Icones, noms, branding Amygdal
browser/locales/en-US/installer/nsisstrings.properties  → Instal·lador en català
browser/extensions/newtab/prerendered/activity-stream.html → Pàgina nova pestanya
browser/extensions/moz.build                            → Registre extensions
chrome/browser/content/browser/built_in_addons.json     → Registre uBlock
services/settings/dumps/main/search-config-v2.json      → DuckDuckGo per defecte
toolkit/components/downloads/DownloadIntegration.sys.mjs → Hash check integrat
toolkit/components/downloads/moz.build                  → Registre AmygdalDownloadProtection
```
 
## Com compilar
 
### Requisits
- Windows 10/11
- [Mozilla Build](https://ftp.mozilla.org/pub/mozilla/libraries/win32/MozillaBuildSetup-Latest.exe)
- Visual Studio Build Tools amb C++ desktop development
- Codi font de [Firefox Nightly](https://hg.mozilla.org/mozilla-central/)
### Passos
 
```bash
# 1. Descarrega el codi font de Firefox Nightly
hg clone https://hg.mozilla.org/mozilla-central/ C:\firefox
 
# 2. Copia els fitxers d'Amygdal al codi font
# (copia els fitxers d'aquest repositori mantenint l'estructura de directoris)
 
# 3. Obre mozilla-build
C:\mozilla-build\start-shell.bat
 
# 4. Compila
cd /c/firefox
./mach build
 
# 5. Prova
./mach run
 
# 6. Empaqueta
./mach package
```
 
L'instal·lador es genera a `obj-x86_64-pc-windows-msvc/dist/`.
 
## Tecnologies
 
| Llenguatge | Ús |
|------------|-----|
| JavaScript (.sys.mjs, .js) | Lògica principal: actors, serveis, extensions, UI |
| HTML/XUL (.xhtml, .html) | Interfície del navegador, panels, newtab |
| CSS | Estils de badges, panels, newtab |
| Python (moz.build) | Sistema de build |
| NSIS (.nsi) | Instal·lador de Windows |
| Fluent (.ftl) | Localització / traducció |
| JSON | Configuració de motors de cerca, polítiques |
 
## Llicència
 
[Mozilla Public License 2.0](LICENSE)
 
## Autor
 
Lluc Comella — [@GodCatv2](https://github.com/GodCatv2)          |          Andualem Luis Cendoya - [@Andu005](https://github.com/Andu005)
