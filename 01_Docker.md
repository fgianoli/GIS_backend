# INSTALLARE DOCKER
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04 

Docker è una piattaforma di virtualizzazione dei contenitori che consente agli sviluppatori di creare, distribuire e gestire applicazioni in modo rapido e affidabile. I contenitori Docker consentono di isolare un'applicazione e le sue dipendenze in un ambiente virtualizzato che può essere eseguito su qualsiasi sistema operativo in cui è installato Docker.

In pratica, Docker consente agli sviluppatori di creare un pacchetto di un'applicazione e di tutte le sue dipendenze in un contenitore, che può quindi essere eseguito su qualsiasi sistema operativo senza dover preoccuparsi di installare tutte le dipendenze manualmente. Ciò rende il processo di sviluppo e distribuzione di un'applicazione più rapido e semplificato, in quanto gli sviluppatori possono concentrarsi sulla creazione dell'applicazione senza dover preoccuparsi di tutti i dettagli di configurazione del sistema.


```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
apt-cache policy docker-ce
sudo apt install docker-ce

```

## Docker compose
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-22-04  

```
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose

chmod +x ~/.docker/cli-plugins/docker-compose
docker compose version

```
