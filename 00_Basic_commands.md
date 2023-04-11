# BASIC SHELL COMMANDS
https://www.educative.io/blog/bash-shell-command-cheat-sheet

All'interno della Shell Ubuntu possiamo utilizzare alcuni comandi utili per gestire e muoverci nella macchina server

- `sudo` questo comando ci permette di lanciare le operazioni come superutenti, quindi con i massimi privilegi sul sistema
- `apt` questo comando ci permette di gestire i pacchetti, per esempio per lanciare un comando di aggiornaemnto scriveremo `apt update`. Nelle vecchie versioni di Ubuntu il comando era `apt-get`
- `apt udate` serve per vedere se ci sono aggiornamenti su pacchetti della macchina
- `apt upgrade` serve per lanciare l'aggiornamento. Se non si è super utenti bisogna aggiungere il comando sudo, es. `sudo apt upgrade`

### Muoversi all'interno del server

- `pwd` serve per vedere il percorso in cui ci si trova
- `cd` change directory, permette di spostarsi all'interno delle cartelle. Per uscire da una cartella `cd ..`, per entrare in una cartella `cd cartella` o `cd path/to/folder`
- `ll` per lisytare i contenuti di una directory
- `ls`per listare i contenuti di una directory
- `touch` per creare un file
- `mkdir` per creare una directory
- `grep` per cercare un contenuto
