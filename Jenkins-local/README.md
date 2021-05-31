# Local Jenkins
### First, install jdk on master node
```bash
sudo apt-get update
sudo apt-get install openjdk-8-jdk -y
java -version
```

### Second, install jenkins on master node
1. Add jenkins apt
```bash
sudo apt-get install default-jdk apt-transport-https wget gnupg -y
wget -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo su -
echo "deb https://pkg.jenkins.io/debian-stable binary/" > /etc/apt/sources.list.d/jenkins.list
exit
```

2. install jenkins
```bash
sudo apt-get update
sudo apt-get install jenkins -y
sudo systemctl enable jenkins
cat /var/lib/jenkins/secrets/initialAdminPassword
```

### On jenkins
1. Make freestyle trigger for github

2. Before build docker, you should give permission
```bash
sudo usermod -a -G docker jenkins
sudo systemctl restart jenkins
```
3. When you want to use su with jenkins

```bash
sudo echo "jenkins ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
```

4. When you want to use kube or helm in jenkins
```bash
sudo su jenkins
cat /home/kisec/.kube/config >> .kube/config
```


5. Make pipeline for push docker
```bash
node {
    def git_repo = 'user1'
    def docker_regi = 'docker-registry:30500'
    def docker_img = git_repo + '-php'
    def helm_repo = git_repo + '-php'
    def helm_port = '30501'
    
    
    stage('Make Dockerfile') {
        sh "echo 'FROM ubuntu:18.04' > Dockerfile"
        sh "echo 'RUN echo \'nameserver 8.8.8.8\' >> /etc/resolv.conf' >> Dockerfile"
        sh "echo 'RUN echo \'nameserver 8.8.4.4\' >> /etc/resolv.conf' >> Dockerfile"
         
        sh "echo 'RUN apt update' >> Dockerfile"
        sh "echo 'RUN apt upgrade -y' >> Dockerfile"
        
        sh "echo 'RUN apt install apache2 -y' >> Dockerfile"
        sh "echo 'RUN apt install git -y' >> Dockerfile"
        sh "echo 'ENV APACHE_RUN_USER www-data' >> Dockerfile"
        sh "echo 'ENV APACHE_RUN_GROUP www-data' >> Dockerfile"
        sh "echo 'ENV APACHE_LOG_DIR /var/log/apache2' >> Dockerfile"
        sh "echo 'ENV APACHE_PID_FILE  /var/run/apache2/apache2.pid' >> Dockerfile"
         
        sh "echo 'RUN apt install software-properties-common -y' >> Dockerfile"
         
        sh "echo 'RUN add-apt-repository ppa:ondrej/php' >> Dockerfile"
        sh "echo 'RUN apt update' >> Dockerfile"
         
        sh "echo 'ENV DEBIAN_FRONTEND=noninteractive' >> Dockerfile"
         
        sh "echo 'RUN apt install php7.3 php7.3-common php7.3-cli -y' >> Dockerfile"
        sh "echo 'RUN apt install php7.3-bcmath php7.3-bz2 php7.3-curl php7.3-gd php7.3-intl php7.3-json php7.3-mbstring php7.3-readline php7.3-xml php7.3-zip php7.3-mysql -y' >> Dockerfile"
        sh "echo 'RUN apt install libapache2-mod-php7.3' >> Dockerfile"
        
        sh "echo 'RUN mkdir /home/visual' >> Dockerfile"
        def git_clone = "echo \'RUN cd /home/visual && git clone https://github.com/kisecADCTF/" + git_repo + "\' >> Dockerfile"
        def move = "echo \'RUN mv /home/visual/"+ git_repo +"/* /home/visual\' >> Dockerfile"
        def remove = "echo \'RUN rm -rf /home/visual/"+ git_repo+"\' >> Dockerfile"
        
        sh git_clone
        sh move
        sh remove
        
        sh "echo 'RUN cp /home/visual/visual.conf /etc/apache2/sites-avaliable' >> Dockerfile"
        sh "echo 'RUN rm /home/visual/visual.conf' >> Dockerfile"
        
        sh "echo 'RUN mv /home/visual/* /var/www/html/' >> Dockerfile"
        
        sh "echo 'RUN ln -s /etc/apache2/sites-available/visual.conf /etc/apache2/sites-enabled/' >> Dockerfile"
        
        sh "echo 'EXPOSE 80' >> Dockerfile"
        
        sh "echo 'CMD [\"/usr/sbin/apache2ctl\", \"-D\", \"FOREGROUND\"]' >> Dockerfile"
    }
    

    stage('Build image') {
        sh 'docker build -t ' + docker_img + ' . --no-cache'
    }
     
    stage('Tag image') {
		sh 'docker tag ' + docker_img + ' ' + docker_regi + '/'+ docker_img +':${BUILD_NUMBER}'
    }

    stage('Push image') {
		sh 'docker logout'
		sh 'docker login '+ docker_regi +' -u kisec -p kisec1234'
		sh 'docker push ' + docker_regi + '/' + docker_img + ':${BUILD_NUMBER}'
    }
    
    stage('Remove docker image') {
        try {
            sh 'sudo docker rmi ' + docker_regi + '/' + docker_img ':${BUILD_NUMBER}'
            sh 'docker rmi $(docker images --filter "dangling=true" -q --no-trunc)'
        }
        catch (e) {
            echo 'No images'
        }
    }
    
    stage('Helm install') {
        sh'kubectl get pods'
        try {
            sh 'helm uninstall ' + helm_repo 
        }
        catch (e) {
            echo 'Helm not exists'
        }
        sh 'helm create ' + helm_repo
        sh 'echo "appVersion: ${BUILD_NUMBER}" >> ' + helm_repo + '/Chart.yaml'
        sh 'helm install --set image.repository=' + docker_regi + '/' + docker_img +',image.pullPolicy=Always,service.type=NodePort ' + helm_repo + ' ' + helm_repo
    }
    
    stage('Helm port change') {
        
        try {
            sh "kubectl patch svc \$(kubectl get svc | grep "+ helm_repo +" | cut -f 1 -d \' \' | head -n 1) --type=\'json\' -p \'[{\"op\":\"replace\",\"path\":\"/spec/ports/0/nodePort\",\"value\":"+ helm_port +"}]\'"
        }
        catch (e) {
            echo 'Port change failed'
        }
    }
    
    stage('Helm get port') {
        try {
            sh 'kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services ' + helm_repo
        }
        catch (e) {
            echo 'Check helm exits'
            sh 'helm list'
        }
    }
    
    
}
```
