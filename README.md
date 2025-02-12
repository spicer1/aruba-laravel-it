# Deployment di Laravel su Aruba

Effettuare il deployment di un progetto Laravel da un ambiente di sviluppo a quello di produzione su un server [Aruba](https://www.aruba.it/) richiede dei passaggi specifici.

Per questa guida mi riferisco a progetti che rispettino le seguenti condizioni:

 - **Laravel 11.x** o successivi
 - **Hosting Linux Basic** o **Advanced** (con accesso a **SSH**)

In altri casi i passaggi potrebbero essere diversi, o addirittura il deployment potrebbe risultare impossibile.

I passaggi da seguire sono:

 1. [Settaggio del file .env per la produzione](#env)
 2. [Upload dei file tramite FTP](#ftp)
 3. [Gestione degli .htaccess](#htaccess)
 4. [Recap](#recap)

Potresti riscontrare altri problemi, in tal caso consulta le [FAQ / Risoluzione di ulteriori problemi](#faq)

 - [Forbidden 403](#403)
 - [Error 505](505)
 - [Vite non carica gli assets](vite)
 - ["$variable is undefined"](undefined)
 - [Altri errori](altro)

<a name="env" id="env"></a>
# 1. Settaggio del file .env per la produzione

Il file `.env` deve essere modificato per funzionare correttamente in produzione.

## APP_ENV

La variabile `APP_ENV` che di default ha il valore `local` deve essere impostata su `production`.

> `APP_ENV` istruisce Laravel su come comportarsi in varie situazioni. Se settata su `production` chiederà una conferma per le migrazioni, impedirà il lancio di certi comandi e introdurrà altre misure di sicurezza. 

Quindi

    APP_ENV=production

## APP_DEBUG

La variabile `APP_DEBUG` che di default ha il valore `true` deve essere impostata su `false`.

Quindi

    APP_DEBUG=false

Eventualmente, **quando si carica il progetto per la prima** volta può avere senso mantenere `true` per verificare prima che tutto funzioni correttamente, per poi disabilitarlo in un secondo momento.
## APP_KEY

Assicurarsi che la variabile `APP_KEY` abbia un valore, altrimenti lanciare il comando:

    php artisan key:generate

## APP_URL

Assicurarsi che il dominio sul quale si sta caricando il progetto usi il protocollo `http` o `https`, quindi impostare l'url corretto.

    APP_URL=https://miosito.it

## Connessione al Database

Assicurarsi che il progetto punti correttamente al database. 

    DB_HOST=00.00.00.000
L'Host è indicato nella dashboard MySql

    DB_DATABASE=Sql0000000_1
Il nome del database è indicato nella sidebar a sinistra. è formato dal `nome utente` + `_1`, `_2`, `_3` ecc.

    DB_USERNAME=Sql0000000
Il nome utente è indicato nella dashboard MySql

    DB_PASSWORD=**********
Chiaramente inserire la password del database.
## Import del Database
Se non si ha accesso all'SSH, per importare il database occorre prima esportarlo in formato `.sql` dal Database Manager che si sta utilizzando in ambiente di sviluppo. Basterà poi importarlo nel database che abbiamo indicato nel file `.env`.

In caso di errori di importazione verificare manualmente il file `.sql`. L'errore più comune è legato alle **foreign keys** e a riferimenti cronologici non gestiti correttamente.

<a name="ftp" id="ftp"></a>
# Upload FTP

Dopo aver corretto il file `.env` e settato il database dovremo preparare il progetto per l'upload.

Per prima cosa lanciamo i seguenti comandi:

    php artisan clear-compiled 
    npm run build
    composer dump-autoload

## Upload
Il seguente metodo è quello più veloce per ottenere questo risultato, ma se si desidera caricare il progetto in altri modi funzionerà lo stesso.

 1. Zippare l'intero contenuto del progetto in un archivio `.zip` (altri
    formati potrebbero non essere letti dal client FTP o dal file
    manager di Aruba).
 2. Connettersi tramite FTP al server e caricare il file `zip` nella
    main root del dominio. Il risultato sarà simile a questo. E'
    possibile **eliminare** tutti i file precedentemente presenti nella
    root, ad esempio `/cgi-bin`, `index.php` e `ver.php`.
    
        /www.miosito.it
    	    mioprogetto.zip
 3. Accedere al **File Manager** di Aruba (si trova nella scheda Hosting Linux). Fare click destro sul file `.zip` che abbiamo caricato e fare `Estrai archivio -> Qui`. Ci metterà un po' di tempo in base alla grandezza del progetto (generalmente pochi minuti).
 4. Una volta scompattato il progetto è possibile rimuovere il file `.zip` e il risultato sarà simile a questo: 
    
        /www.miosito.it
    	    /app
    	    /bootstrap
    	    /config
    	    /database
    	    /lang
    	    /node_modules
    	    /public
    	    /resources
    	    /routes
    	    /storage
    	    /tests
    	    /vendor
    	    .editorconfig
    	    .env
    	    .gitattributes
    	    .gitignore
    	    artisan
    	    composer.json
    	    composer.lock
    	    package-lock.json
    	    package.json
    	    phpunit.xml
    	    postcss.config.js
    	    README.md
    	    tailwind.config.js
    	    vite.config.js
  
  <a name="htaccess" id="htaccess"></a>  	    
# Gestione degli .htaccess

Per permettere al server di puntare alla cartella `/public`, e quindi di far funzionare correttamente Laravel, occorre aggiungere un file `.htaccess` nella main root del progetto (in `/www.miosito.it`).

Il contenuto del file `.htaccess` deve essere il seguente:

    RewriteEngine on
    RewriteBase /
    
    RewriteCond %{REQUEST_URI} !(/$|\.)
    RewriteCond %{REQUEST_FILENAME} -d
    RewriteRule ^(.*[^/])$ $1/ [R=301,L]
    
    RewriteCond %{REQUEST_URI} !^/public/
    
    RewriteCond %{REQUEST_URI} !(/$|\.)
    RewriteCond %{DOCUMENT_ROOT}/public%{REQUEST_URI} -d
    RewriteRule ^(.*)$ /$1/ [R=301,L]
    
    RewriteCond %{REQUEST_URI} !^/public/
    RewriteRule ^(.*)$ /public/$1 [L]

La porzione più importante di questo `.htaccess` è sul finale, le ultime due righe istruiscono il redirect verso la cartella `/public`.
<a name="recap" id="recap"></a>
# Recap

Per ricapitolare i passaggi sono i seguenti:

 1. Aggiornamento dell'`.env` per la produzione
 2. Import del database
 3. Lancio dei comandi `php artisan optmize:clear`, `php artisan clear-compiled`, `npm run build`, `composer dump-autoload`
 4. Caricamento del progetto tramite `.zip` e scompattamento nella main root tramite File Manager
 5. Creazione del file [htaccess](#htaccess)

<a name="faq" id="faq"></a>
# FAQ / Risoluzione problemi

Possono capitare errori e problemi di varia natura nella fase di deployment. I più riscontrati e quelli su cui difficilmente ho trovato risposte online sono i seguenti:
<a name="403" id="403"></a>
## Forbidden 403
Se si riscontra un errore "Forbidden 403" con molta probabilità non è stato inserito/scritto correttamente il file `.htaccess` nella main root del progetto. Il risultato è un errore di reindirizzamento e l'impossibilità ad accedere correttamente alla cartella `/public`.
<a name="505" id="505"></a>
## ERROR 505
Se viene mostrata la pagina classica di Laravel con "ERROR 505" possiamo gioire, perchè essendo un asset di Laravel vuol dire che funziona correttamente ma non riesce ad ottenere le risorse desiderate. In questo caso basterà aggiornare il file `.env` e impostare la variabile `APP_DEBUG` su `true`. In questo modo sarà possibile verificare l'origine dell'errore e risolverlo direttamente in ambiente di produzione. Al termine ricordarsi di rimettere `APP_DEBUG=false`.
<a name="vite" id="vite"></a>
## Vite non carica gli assets
Se viene caricato correttamente il contenuto, ma non vengono caricati script e css con molta probabilità è stato caricato il file `hot` nella cartella `/public`. Basterà **rimuovere** questo file per far funzionare correttamente l'interfaccia.
<a name="undefined" id="undefined"></a>
## "$variable is undefined"
Se si riscontra un errore del tipo "**$variable is undefined**" e si esclude che l'errore provenga da una mancata comunicazione con il server o problemi causati da noi nel codice (ossia se non abbiamo la più pallida idea di come risolvere)... Con molta probabilità è un problema di **case sensitivity** delle classi.

Per risolvere basterà scandagliare tutta la cartella `/app` del nostro progetto e assicurarci che sia le `directory`, sia i file `.php` inizino tutti con la lettera maiuscola. Bisognerà anche modificare tutti i file `.php` assicurandoci che il namespace sia corretto e che il nome della classe sia corretto.

**Esempio errato:**

    /app
	    /View
		    /Components
			    /prova
				    /componente
Dove `componente.php` è così

    <?php
    
    namespace App\View\Components\prova;
    use Closure;
    use Illuminate\Contracts\View\View;
    use Illuminate\View\Component;
    
    class prova extends Component
    { 
	    ... 
    }
**Esempio corretto**

    /app
	    /View
		    /Components
			    /Prova
				    /Componente
Dove `Componente.php` è così

    <?php
    
    namespace App\View\Components\Prova;
    use Closure;
    use Illuminate\Contracts\View\View;
    use Illuminate\View\Component;
    
    class Prova extends Component
    { 
	    ... 
    }

Un possibile motivo per cui in ambiente di sviluppo queste classi funzionassero anche se il lowercase è che l'ambiente server Linux di Aruba è case sensitive.
<a name="altro" id="altro"></a>
## Altri errori
Nel caso si riscontrassero altri errori è fondamentale capire se questi hanno origine nel server o in Laravel. Per farlo basta confrontare i log di Aruba con quelli di Laravel. Un foglio di log pulito di Laravel a fronte di una lista infinita di codici errore di Aruba indicano che il problema è con Aruba. Al contrario un log pulito di aruba e un log sporco di Laravel indica che il problema risiede nel codice.
