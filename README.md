# 🛡️ Amygdal

**Fork de Firefox centrado en privacidad y seguridad, con navegador nativo en catalán.**

![Language](https://img.shields.io/badge/UI-Catal%C3%A0-red) ![License](https://img.shields.io/badge/license-MPL--2.0-blue) ![Platform](https://img.shields.io/badge/platform-Windows-lightgrey)

> *"sigues ningú, sigues res, sigues inrastrejable"*
>
> Amygdal es un fork de Firefox pensado para demostrar, con una implementación real y funcional, cómo se puede diseñar un navegador privacy-first: toda la clasificación de reputación de webs se calcula **100% en local**. Ninguna URL, dominio o hash sale nunca del equipo del usuario hacia un servidor externo, lo que elimina por diseño la posibilidad de que un tercero (incluido el propio proyecto) pueda ver qué webs visita alguien.
>
> 🔗 Sitio del proyecto: [amygdal.llucomella.com](https://amygdal.llucomella.com)
>
> ## Por qué existe este proyecto
>
> Nace de una pregunta muy concreta de arquitectura de seguridad: ¿es posible dar protección real contra tracking, phishing y malware sin depender de servicios cloud de terceros que ven cada URL que visita el usuario? Amygdal es la respuesta práctica: un motor de reputación local, con estructuras de datos en memoria de acceso O(1), cache con TTL, y una capa de actores del propio Firefox (JSWindowActors) para inyectar señales de seguridad en la UI sin tocar la privacidad del usuario.
>
> ## Funcionalidades
>
> | Módulo | Qué hace |
> |---|---|
> | 🛡️ **Reputación web local** | 424 dominios clasificados en 4 niveles (Fiable / Desconegut / Sospitós / Perill), guardados en `Sets` en memoria (O(1)). Se muestra como *badges* en buscadores, barra de URL y tooltips de enlaces, e intercepta clics sospechosos antes de navegar. |
> | 🔒 **Buscador único: DuckDuckGo** | Google, Bing y Yahoo se eliminan de la configuración; no se pueden añadir otros motores, por diseño, para reducir superficie de tracking. |
> | 🚫 **Bloqueo de anuncios nativo** | uBlock Origin integrado como *system addon* no desinstalable, en lugar de reinventar un bloqueador propio. |
> | 🍪 **Eliminación de banners de cookies** | Detección en tres capas: ocultación CSS de los banners, limpieza del DOM, y listas de los 50 frameworks de consentimiento más usados (vía listas de uBlock). |
> | 📧 **Temp Mail integrado** | Autocompletado de correos temporales (API de Guerrilla Mail) en formularios de registro, para reducir la huella de identidad real del usuario. |
> | 🔑 **Generador de contraseñas** | Contraseñas fuertes y frases de paso integradas en el propio navegador. |
> | 📥 **Protección de descargas** | Verificación SHA-256 de cada descarga contra la base de datos local de URLhaus. |
> | 🔐 **Privacidad avanzada** | Total Cookie Protection, First-Party Isolation y borrado automático de datos de navegación. |
>
> ## Arquitectura
>
> ```
> AmygdalReputationService (browser/modules/)
>  ├─ 424 dominios trusted/suspicious/dangerous en Sets (O(1))
>  ├─ nsIURIClassifier (Safe Browsing local si hay DB disponible)
>  ├─ Cache LRU (TTL 10 min)
>  │
>  ├─→ AmygdalSERPChild/Parent      → badges en resultados de búsqueda
>  ├─→ AmygdalLinkTooltipChild/Parent → tooltips de reputación en enlaces
>  ├─→ ClickHandlerParent           → intercepción de clics sospechosos
>  ├─→ browser-amygdalSafety.js     → badge en la barra de URL + popups
>  └─→ DownloadIntegration          → verificación hash de descargas
>
> AmygdalCookieKillerChild   → eliminación de banners de cookies
> TempMail                   → correo temporal (Guerrilla Mail API)
> Password Generator         → generador de contraseñas nativo
> ```
>
> ## Stack técnico
>
> | Tecnología | Uso |
> |---|---|
> | JavaScript (`.sys.mjs`, `.js`) | Lógica principal: actors, servicios, extensiones, UI |
> | HTML/XUL | Interfaz del navegador, paneles, newtab |
> | CSS | Estilos de badges, paneles, newtab |
> | Python (`moz.build`) | Sistema de build de Firefox |
> | NSIS | Instalador de Windows |
> | Fluent (`.ftl`) | Localización / traducción |
> | JSON | Configuración de motores de búsqueda y políticas |
>
> ## Estructura del repositorio
>
> Este repositorio contiene **únicamente los ficheros creados o modificados** respecto al código fuente de Firefox Nightly (no el navegador completo). Para compilarlo hace falta aplicar estos ficheros sobre el árbol de código de Mozilla.
>
> <details>
 <summary><b>Ficheros creados</b>b></summary>summary>
 
 - `browser/modules/AmygdalReputationService.sys.mjs` — motor central de reputación
 - - `browser/actors/AmygdalSERPChild/Parent.sys.mjs` — badges en buscadores
   - - `browser/actors/AmygdalLinkTooltipChild/Parent.sys.mjs` — tooltips en enlaces
     - - `browser/actors/AmygdalCookieKillerChild.sys.mjs` — eliminación de cookie banners
       - - `browser/base/content/browser-amygdalSafety.js` — badge de barra de URL + popups
         - - `browser/base/content/browser-tempmail.js` — correo temporal
           - - `browser/extensions/amygdal/temp-mail/` — extensión Temp Mail
             - - `browser/extensions/amygdal/password-gen/` — generador de contraseñas
               - - `browser/extensions/newtab/data/content/amygdal-settings.js` — configuración newtab
                 - - `browser/extensions/ublock0/extension/` — uBlock Origin v1.70.0 (system addon)
                   - - `toolkit/components/downloads/AmygdalDownloadProtection.sys.mjs` — verificación hash de descargas
                     -
                     - </details>

                     <details>
                      <summary><b>Ficheros modificados</b>b></summary>summary>
                     
                     - `browser/app/profile/firefox.js` — preferencias de privacidad + registro del system addon
                     - - `browser/app/distribution/policies.json` — política de buscador único (DuckDuckGo)
                       - - `browser/actors/ClickHandlerParent.sys.mjs` — intercepción de clics sospechosos
                         - - `browser/components/BrowserGlue.sys.mjs` — registro de JSWindowActors
                           - - `browser/base/content/browser.js` — hook en `onLocationChange`
                             - - `browser/branding/unofficial/` — branding de Amygdal
                               - - `services/settings/dumps/main/search-config-v2.json` — DuckDuckGo por defecto
                                 - - `toolkit/components/downloads/DownloadIntegration.sys.mjs` — integración de verificación hash
                                   -
                                   - </details>

                                   ## Cómo compilar

                                   **Requisitos:** Windows 10/11, Mozilla Build, Visual Studio Build Tools (C++ desktop development) y el código fuente de Firefox Nightly.

                                   ```bash
                                   # 1. Descargar el código fuente de Firefox Nightly
                                   hg clone https://hg.mozilla.org/mozilla-central/ C:\firefox

                                   # 2. Copiar los ficheros de Amygdal sobre el código fuente
                                   #    manteniendo la estructura de directorios de este repositorio

                                   # 3. Abrir Mozilla Build
                                   C:\mozilla-build\start-shell.bat

                                   # 4. Compilar
                                   cd /c/firefox
                                   ./mach build

                                   # 5. Probar
                                   ./mach run

                                   # 6. Empaquetar
                                   ./mach package
                                   ```

                                   El instalador se genera en `obj-x86_64-pc-windows-msvc/dist/`.

                                   ## Licencia

                                   [Mozilla Public License 2.0](./LICENSE)

                                   ## Autoría

                                   Desarrollado por **Lluc Comella** ([@GodCatv2](https://github.com/GodCatv2)) junto con **Andualem Luis Cendoya** ([@Andu005](https://github.com/Andu005)).</summary>
                     </summary>
</details>
