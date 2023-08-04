## Setup docker network 
    docker network create jenkins

## DID (docker in docker)
Kita akan menjalankan aplikasi React App menggunakan Docker container di dalam sebuah Docker container (lebih tepatnya di dalam Blue Ocean container–nanti akan dibahas). Praktik ini disebut dengan dind alias docker in docker. Jadi, silakan unduh dan jalankan docker:dind Docker image menggunakan perintah berikut.
### refer to jenkins.sh

    docker run \
      --name jenkins-docker \
      --rm \
      --detach \
      --privileged \
      --network jenkins \
      --network-alias docker \
      --env DOCKER_TLS_CERTDIR=/certs \
      --volume jenkins-docker-certs:/certs/client \
      --volume jenkins-data:/var/jenkins_home \
      --publish 2376:2376 \
      --publish 3000:3000 \
      docker:dind \
      --storage-driver overlay2

Berikut adalah penjelasan dari perintah di atas.


--name: Menentukan nama Docker container yang akan digunakan untuk menjalankan image.


--rm: Secara otomatis menghapus Docker container (yakni sebuah instance dari Docker image) saat dimatikan (shut down).


--detach: Menjalankan Docker container di background. Meski begitu, instance ini dapat dihentikan nanti dengan menjalankan perintah docker stop jenkins-docker.


--privileged: Menjalankan dind (docker in docker alias docker di dalam docker) saat ini memerlukan privileged access (akses istimewa) agar bisa berfungsi dengan baik. Persyaratan ini bisa jadi tak diperlukan dengan versi kernel Linux terbaru.


--network jenkins: Ini berhubungan dengan network yang dibuat pada langkah sebelumnya.


--network-alias docker: Membuat Docker di dalam Docker container tersedia sebagai hostname docker di dalam jenkins network.


--env DOCKER_TLS_CERTDIR=/certs: Mengaktifkan penggunaan TLS di Docker server. Ini direkomendasikan karena kita menggunakan privileged container. Environment variable ini mengontrol root directory di mana Docker TLS certificates dikelola.


--volume jenkins-docker-certs:/certs/client: Memetakan direktori /certs/client di dalam container ke Docker volume bernama jenkins-docker-certs.


--volume jenkins-data:/var/jenkins_home: Memetakan direktori /var/jenkins_home di dalam container ke Docker volume bernama jenkins-data. Ini akan memungkinkan Docker container lain dikelola oleh Docker container’s Docker daemon ini untuk mount data dari Jenkins.


--publish 2376:2376: Mengekspos Docker daemon port pada mesin host (komputer Anda). Ini berguna untuk mengeksekusi Docker command (perintah Docker) pada mesin host (komputer Anda) dalam mengontrol inner Docker daemon.


--publish 3000:3000: Mengekspos port 3000 dari docker in docker container.


docker:dind: Ini adalah image dari docker:dind itu sendiri. Image ini bisa diunduh sebelum dijalankan menggunakan perintah docker image pull docker:dind.


--storage-driver overlay2: Storage driver untuk Docker volume. Lihat halaman "Docker Storage drivers” untuk berbagai opsi yang didukung.

## Jalankan Blue Ocean (UX terbaru dari Jenkins).

    nano Dokckerfile
#
### refer to Dockerfile
    FROM jenkins/jenkins:2.346.1-jdk11
    USER root
    RUN apt-get update && apt-get install -y lsb-release
    RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
      https://download.docker.com/linux/debian/gpg
    RUN echo "deb [arch=$(dpkg --print-architecture) \
      signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
      https://download.docker.com/linux/debian \
      $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
    RUN apt-get update && apt-get install -y docker-ce-cli
    USER jenkins
    RUN jenkins-plugin-cli --plugins "blueocean:1.25.5 docker-workflow:1.28"

## Buat sebuah Docker image baru dari Dockerfile tadi dan berikan nama: myjenkins-blueocean:2.346.1-1

    docker build -t myjenkins-blueocean:2.346.1-1 .

## Setelah itu, jalankan myjenkins-blueocean:2.346.1-1 image sebagai container di Docker menggunakan perintah berikut.

    docker run \
      --name jenkins-blueocean \
      --detach \
      --network jenkins \
      --env DOCKER_HOST=tcp://docker:2376 \
      --env DOCKER_CERT_PATH=/certs/client \
      --env DOCKER_TLS_VERIFY=1 \
      --publish 8080:8080 \
      --publish 50000:50000 \
      --volume jenkins-data:/var/jenkins_home \
      --volume jenkins-docker-certs:/certs/client:ro \
      --volume "$HOME":/home \
      --restart=on-failure \
      --env JAVA_OPTS="-Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true" \
      myjenkins-blueocean:2.346.1-1 
