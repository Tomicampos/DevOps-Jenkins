pipeline {
    agent any  // Define que el pipeline puede ejecutarse en cualquier agente disponible.

    options {
        timeout(time: 2, unit: 'MINUTES')  // Establece un tiempo límite de 2 minutos para la ejecución de todo el pipeline.
    }

    environment {
        ARTIFACT_ID = "elbuo8/webapp:${env.BUILD_NUMBER}"  // Define una variable de entorno ARTIFACT_ID usando el número de build actual.
        VERSION = readFile('build.version').trim()  // Lee el archivo 'build.version', elimina espacios en blanco y asigna su contenido a la variable de entorno VERSION.
    }

    stages {
        stage('Docker Login') {  // Primer stage: Login en Docker
            steps {
                script {
                    // Utiliza las credenciales almacenadas en Jenkins (cred_docker_hub) para iniciar sesión en Docker Hub.
                    withCredentials([usernamePassword(credentialsId: 'credenciales_docker_hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        def registryCredentials = "-u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                        sh "docker login $registryCredentials"  // Ejecuta el comando de login de Docker.
                    }
                }
            }
        }

        stage('Building image') {  // Segundo stage: Construcción de la imagen Docker.
            steps {
                // Muestra el directorio de trabajo actual.
                // Lista los archivos del directorio con detalles.
                // Construye una imagen Docker etiquetada como 'testapp'.
                sh '''
                    pwd
                    ls -ltr
                    docker build -t testapp .
                '''
            }
        }

        stage('Run tests') {  // Tercer stage: Ejecuta pruebas en la imagen Docker construida.
            steps {
                sh "docker run testapp npm test"  // Ejecuta 'npm test' dentro de un contenedor Docker creado a partir de la imagen 'testapp'.
            }
        }

        stage('Deploy Image') {  // Cuarto stage: Etiquetado y subida de la imagen Docker a Docker Hub.
            steps {
                script {
                    def version = env.VERSION  // Obtiene la versión de la variable de entorno.
                
                // Etiqueta la imagen local 'testapp' con una nueva etiqueta que incluye la versión.
                // Sube la imagen etiquetada a Docker Hub.
                sh """
                    docker tag testapp tomicampos22/jenkins-devops:${version}
                    docker push tomicampos22/jenkins-devops:${version}
                    
                """
                }
            }
        }

        stage('Vulnerability Scan - Docker') {  // Quinto stage: Escaneo de vulnerabilidades en la imagen Docker.
            steps {
                script {
                    def version = env.VERSION  // Obtiene la versión de la variable de entorno.
                
                // Ejecuta el escáner de vulnerabilidades Trivy en la imagen.
                sh """
                    docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --severity=CRITICAL tomicampos22/jenkins-devops:${version}
                """
                }
            }
        }

        
        stage('Pass To microK8s') {
    steps {
        script {
            // Define la dirección IP del servidor remoto
            def remoteHost = '172.17.0.1'
            // Define el nombre de usuario para la conexión SSH
            def sshUser = 'user'
            // Define el comando kubectl que deseas ejecutar
            def kubectlCommand = '''
                kubectl create deployment jenkins-devops --image=jenkins-devops:${version}
                echo "Wait"
                sleep 10
                kubectl expose deployment jenkins-devops --port=3000
                wget https://raw.githubusercontent.com/tercemundo/platzi-scripts-integracion/master/webapp/nodePort.yml
                kubectl apply -f nodePort.yml
            '''
            // Ejecuta los comandos a través de SSH
            sshCommand(
                remote: remoteHost,
                user: sshUser,
                command: kubectlCommand
            )
        }
    }
}

        
    }
}