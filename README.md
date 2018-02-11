# BoomVox est une enceinte connectée à l'assistant Google.

Démarrage

Pour ce projet, vous aurez besoin de ces composants :

    - Une Raspberry Pi 3 (+ carte microSD, souris, clavier, écran...)
    - Un speaker (3.5 mm connector)
    - Un microphone USB
    - Un bouton switch et des cables pour les pins de la Raspberry

Installation :

D'abord il faudra télécharger le Voice Kit microSD, disponible ici :
https://dl.google.com/dl/aiyprojects/voice/aiyprojects-latest.img.xz

Puis le déployer sur la microSD, à l'aide de Etcher.io par exemple.
https://etcher.io/

Ensuite, branchez votre Pi sur un moniteur, ainsi qu'une souris et un clavier.
Suite au boot :

1- Cliquez sur le logo de la raspberry en haut à gauche. Allez dans "preferences" et cliquez sur
"Raspberry Pi Configuration". Allez dans "interfaces" et activez le SSH, puis confirmez.
2- Cliquez sur le symbole Wi-Fi en haut à gauche, et cliquez sur le réseau adéquat et connectez-vous.
3- Double-cliquez sur “Start dev terminal”, et entrez "sudo leafpad /boot/config.txt".
Supprimez le "#" devant la ligne "dtparam=audio=on" et insérez un "#" devant les lignes en dessous.
Enregistrez et quittez leafpad.
"/boot/config.txt" doit maintenant ressembler à ça :

    - # Enable audio (loads snd_bcm2835)
    - dtparam=audio=on
    - #dtoverlay=i2s-mmap
    - #dtoverlay=googlevoicehat-soundcard

Maintenant, au tour de l'audio :

Entrez "sudo leafpad /etc/asound.conf" dans le terminal.
Supprimez tout et remplacez par :

     pcm.!default {
       type asym
       capture.pcm "mic"
       playback.pcm "speaker"
     }
     pcm.mic {
       type plug
       slave {
         pcm "hw:1,0"
       }
     }
     pcm.speaker {
       type plug
       slave {
         pcm "hw:0,0"
       }
     }

Enregistrez et fermez leafpad. Rebootez la Pi, lancez le terminal et entrez :
"leafpad /home/pi/voice-recognizer-raspi/checkpoints/check_audio.py".

En haut du fichier, changez la ligne "VOICEHAT_ID = ‘googlevoicehat’" en "VOICEHAT_ID = ‘bcm2835’".
Enregistrez et fermez.

Maintenant, il faut tester l'audio.

Allez sur le bureau, double-cliquez sur “Check audio” et suivez les instructions. Si vous
réussissez à entendre le son et à enregistrer votre voix, l'audio fonctionne.

Si vous rencontre des erreurs avec l'audio, vous pouvez vous référer à la page suivante de la 
documentation du Google Assistant : 
https://developers.google.com/assistant/sdk/prototype/getting-started-pi-python/configure-audio

Place à la configuration du cloud.

1- Ouvez le navigateur de la Raspberry et allez sur :
https://console.cloud.google.com/

Faites "Create a new project"
2- Dans la Cloud Console, activez le Google Assistant API
https://console.developers.google.com/apis/api/embeddedassistant.googleapis.com/overview

3- Créez un client OAUth 2.0 :
https://console.cloud.google.com/apis/credentials/oauthclient

4- Cliquez sur "Create credentials" et selectionnez "OAuth client ID" et suivez les instructions.

5- Dans la "Credentials list", trouvez vos identifiants et cliquez sur l'icône de téléchargement à droite.

6- Renommez fichier JSON que vous venez de télécharger en "assistant.json", déplacez-le dans "/home/pi/assistant.json"
dans le dev terminal, entrez : "systemctl stop voice-recognizer"

7- Allez dans "Activity Controls" :
https://myaccount.google.com/activitycontrols

et activez les éléments suivants : Web and app activity, Location history, Device information, Voice, audio activity

Et voilà ! Maintenant, place au test de votre assistant :

Dans le terminal, entrez : 
     src/main.py
Entrez votre login, appuyez sur le bouton et posez une question à Google.
Par exemple :
"Hey Google, what time is it ?"
Votre LED devrait s'allumer lorsque l'assistant Google enregistre et analyze vos requêtes.

Il ne reste plus qu'à construire coque.
