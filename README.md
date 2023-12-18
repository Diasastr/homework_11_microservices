# Розгортання Docker Swarm на AWS EC2

Цей README надає поетапні інструкції з налаштування Docker Swarm на AWS EC2. Він охоплює створення образів Docker, їх завантаження до Docker Hub, налаштування екземплярів AWS та розгортання Docker Swarm.

## Покрокові інструкції

### Частина 1: Налаштування Docker і Docker Hub

1. **Створила обліковий запис в Docker**:  
  [https://hub.docker.com/u/diana7hub]

3. **Створила локальні образи Docker**:  
   Створила Dockerfiles для веб додатків ( Nginx та Tomcat) :
   ```dockerfile
   FROM tomcat:latest
    
   # Delete the existing ROOT folder
   RUN rm -rf /usr/local/tomcat/webapps/ROOT
    
   # Copy the custom web application to the ROOT context
   COPY ./webapp/ /usr/local/tomcat/webapps/ROOT/
    
   # Expose port 8080
   EXPOSE 8080
   ```
   Та NGINX:
   ```dockerfile
   FROM nginx:latest

   # Copy the custom index.html to the container
   COPY index.html /usr/share/nginx/html/index.html
    
   # Expose port 80
   EXPOSE 80
   ```
      
   Потім створила образи Docker і тегнула їх, використовуючи наступні команди:

   ```bash
   docker build -t diana7hub/nginx-repo /nginx/Dockerfile
   docker build -t diana7hub/tomcat-repo /tomcat/Dockerfile
   ```
   
4. **Завантажила образи  на Docker Hub**:
   ```bash
   docker login
   docker push diana7hub/nginx-repo
   docker push diana7hub/tomcat-repo
   ```

### Частина 2: Налаштування AWS EC2

1. **Запустила EC2 екземпляри**:  
   Запустила три EC2 екземпляри - мастер і 2 воркери, створила security group
   
3. **SSH to EC2**:
   Використала EC2 connect метод для SSH в екземпляри

### Частина 3: Налаштування Docker Swarm на EC2

1. **Встановила Docker на екземпляри EC2**:  
   Підключилась до всіх інстансів і заранила команди:
   
   ```bash
   sudo yum update -y
   sudo yum install docker -y
   sudo service docker start
   sudo usermod -a -G docker ec2-user
   ```
   
2. **Встановила Docker Compose на екземпляри EC2**:
   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/download/v2.23.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   ```
   
3. **Ініціалізувала Docker Swarm:*:
   ```bash
   docker swarm init --advertise-addr [IP-адреса головного вузла]
   ```
   На робочих вузлах використала токен, який з'явився після ініціалізації менеджера.

### Частина 4: Розгортання додатків

1. **Створила docker-compose.yml на головному вузлі:**
   Файл можна знайти в репозиторії. Використовувала images зі свого репозиторію в Docker Hub.
   
3. **Розгорнула додатки:**
   ```bash
   docker stack deploy -c docker-compose.yml myapp
   ```
   
### Частина 5: Тестування та симуляція збоїв
1. **Тестувала розгортання:**
Доступ до додатків через публічні IP-адреси екземплярів EC2.

3. **Симулювала збій:**
Зупинила Docker на робочому вузлі та спостерігала за поведінкою сервісу. Swarm перерозподілив задачі на інші ноди.

   
   
  

   
   

   
   
   
