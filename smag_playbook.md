# Trying out Apache Beam 

Doing this with the book __Building Big Data Pipelines with Apache Beam__ by _Jan Lukavský_ published by Packt, january 2022 [link](https://subscription.packtpub.com/book/data/9781800564930/pref/preflvl1sec03/what-this-book-covers)

## Pre

### WSL på windows

[TODO]

### Git repo

Jeg har lavet en fork af det oprindlige repo fra packt, i <https://github.com/TEC-CIT-Progs/Building-Big-Data-Pipelines-with-Apache-Beam.git>.

    git clone https://github.com/TEC-CIT-Progs/Building-Big-Data-Pipelines-with-Apache-Beam.git

### Første build

Kørte så følgende som anvist på side 5:

    ./mvnw clean install

Det tog 5-10 minutter og buildede alle kapitler og kørte test for dem.

Det tog lang tid, men alligevel lidt imponerende...
Jeg tror at testene kører som "single process", så uden Kafka... (tror jeg)

## Kapitel 1

Jeg kaster mig ud i første kapitel. 

### Fra "Setting up the environment for this book"

På side 31, finder jeg en opskrift på at sætte miljøet op.

#### Kubernetes / Minikube

Først skal vi have `minikube` som er Kubernetes i local-mode. Så jeg følger linket i bogen til <https://minikube.sigs.k8s.io/docs/start/>.  
Her vælger jeg 
 - [x] Linux
 - [x] x86-64
 - [x] Stable
 - [x] Binary download  

og kopierer to kommandoer fra boksen under:

    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube

Måske skal dette se lidt anderledes ud på windows/wsl [TODO]

Således hentes `minukube`, og installers derefter lydløst:

    $ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
    100 82.4M  100 82.4M    0     0  15.4M      0  0:00:05  0:00:05 --:--:-- 19.5M
    [sudo] password for soren: 

(muligvis forlanges password til `sudo`)

Man skal nu kunne starte og teste...

    $ minikube start
    😄  minikube v1.31.2 on Ubuntu 22.04


    ✨  Automatically selected the docker driver. Other choices: virtualbox, vmware, none, ssh
    📌  Using rootless Docker driver

    ❌  Exiting due to MK_USAGE: --container-runtime must be set to "containerd" or "cri-o" for rootless

Så på linus skal den åbenbart have parameteren `--container-runtime=containerd`:

    $ minikube start --container-runtime=containerd
    😄  minikube v1.31.2 on Ubuntu 22.04
    ✨  Automatically selected the docker driver. Other choices: virtualbox, vmware, none, ssh
    📌  Using rootless Docker driver
    👍  Starting control plane node minikube in cluster minikube
    🚜  Pulling base image ...
    💾  Downloading Kubernetes v1.27.4 preload ...

og den installerer selve kubernetes automatisk


Note: for at finde _hvor_ `docker-compose` containere er startet fra:

    docker inspect `docker ps -q` | grep "com.docker.compose.project.working_dir"


WAHH

Det skal specificeres at det er docker _og_ rootles (fordi jeg vist har insatlleret docker rootless)

    minikube start --driver=docker --container-runtime=containerd

ARGH

Den siger 

    💡  Suggestion: 

        Run the following:
        $ sudo mkdir -p /etc/systemd/system/user@.service.d
        $ cat <<EOF | sudo tee /etc/systemd/system/user@.service.d/delegate.conf
        [Service]
        Delegate=cpu cpuset io memory pids
        EOF
        $ sudo systemctl daemon-reload
    🍿  Related issue: https://github.com/kubernetes/minikube/issues/14871

Så det prøver jeg...  

Bum bum, måske en clean installering var bedre...?

`kubectl` installeres, og eksekveres gennem `minikube`

    minikube kubectl -- get po -A

Der er også et `minikube dashboard`, som måske er relevant senere. Fordi wsl ikke har en browser skal vi bare have en url:

    $ minikube dashboard --url=true
    🔌  Enabling dashboard ...
        ▪ Using image docker.io/kubernetesui/dashboard:v2.7.0
        ▪ Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
    💡  Some dashboard features require the metrics-server addon. To enable all features please run:

            minikube addons enable metrics-server


    🤔  Verifying dashboard health ...
    🚀  Launching proxy ...
    🤔  Verifying proxy health ...
    http://127.0.0.1:45261/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/

Tilbage til bogen...

På min linux har jeg ikke fået kubectl på (kunne jeg sikkert sagtens, men jeg kører den bare igennem minikube indtil videre)

    minikube kubectl config current-context

Men ok, der står jo der på side 31 at man skal gå til <https://kubernetes.io/docs/tasks/tools/>, hvor jeg trykker videre på [Install kubectl on Linux](https://kubernetes.io/docs/tasks/tools/)

    sudo apt-get update
    
    # apt-transport-https may be a dummy package; if so, you can skip that package
    sudo apt-get install -y apt-transport-https ca-certificates curl
    # Download the public signing key for the Kubernetes package repositories. The same signing key is used for all repositories so you can disregard the version in the URL:
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    # This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

    sudo apt-get update
    sudo apt-get install -y kubectl

#### Installer Kafka

Stadig s. 31

Kafka er en message que, lidt ligesom mqtt (Moquitto).

Bogen (det medfølgende repo) leverer et installaionsscript i `./env/install-kafka.sh`.

Nu løber jeg ind problemer med docker, som er manuelt installeret ... for at køre rootless... argh

