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
                    docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --severity=CRITICAL tomicampos22/testapp:${version}
                """
                }
            }
        }

        /*
        stage('Pass To microK8s') {  // Stage comentado: Implementación en Kubernetes (comentado para no ejecutarse).
            steps {
                sh '''
                sshpass -p 'master' ssh 172.17.0.1 -l root -o StrictHostKeyChecking=no "kubectl create deployment testapp --image=testapp:${version}"  // Crea un deployment en Kubernetes usando la imagen.
                echo "Wait"
                sleep 10  // Espera 10 segundos.
                sshpass -p 'master' ssh 172.17.0.1 -l root -o StrictHostKeyChecking=no "kubectl expose deployment testapp --port=3000"  // Expone el deployment en el puerto 3000.
                sshpass -p 'master' ssh 172.17.0.1 -l root -o StrictHostKeyChecking=no "wget https://raw.githubusercontent.com/tercemundo/platzi-scripts-integracion/master/webapp/nodePort.yml"  // Descarga un archivo de configuración de Kubernetes.
                sshpass -p 'master' ssh 172.17.0.1 -l root -o StrictHostKeyChecking=no "kubectl apply -f nodePort.yml"  // Aplica el archivo de configuración descargado.
                '''
            }
        }
        */
    }
}