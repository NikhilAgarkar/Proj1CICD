pipeline {
    agent any

    stages {
        stage('Install and Configure Puppet Agent') {
            steps {
                script {
                    // Job 1: Install and configure puppet agent
                  
                    sh 'sudo apt-get update && sudo apt-get install -y puppet'
                    sh 'sudo puppet agent --server=puppet.example.com --waitforcert=60 --test'
                }
            }
        }

        stage('Install Docker using Ansible') {
            steps {
                script {
                    // Job 2: Push Ansible configuration to install Docker on the test server
                   
                    writeFile file: 'docker_install.yml', text: '''
                    ---
                    - name: Install Docker
                      hosts: 20.204.147.125
                      become: yes
                      tasks:
                        - name: Install Docker dependencies
                          apt:
                            name: "{{ item }}"
                            state: present
                          with_items:
                            - apt-transport-https
                            - ca-certificates
                            - curl
                            - software-properties-common

                        - name: Add Docker GPG key
                          apt_key:
                            url: https://download.docker.com/linux/ubuntu/gpg
                            state: present

                        - name: Add Docker APT repository
                          apt_repository:
                            repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable
                            state: present

                        - name: Install Docker
                          apt:
                            name: docker-ce
                            state: present
                    '''
                    sh 'ansible-playbook -i "20.204.147.125," -u proj1 docker_install.yml'
                }
            }
        }

        stage('Build and Deploy PHP Docker Container') {
            steps {
                script {
                    // Job 3: Build and deploy PHP Docker container
                    // Replace placeholders with actual Git repository URL
                    sh 'git clone https://github.com/NikhilAgarkar/Proj1CICD'
                    sh 'docker build -t my_php_app .'
                    sh 'docker tag my_php_app nikhilagarkar/my_php_app:latest'
                    sh 'docker push nikhilagarkar/my_php_app:latest'
                    sh 'docker run -d --name your_php_container -p 8080:80 nikhilagarkar/my_php_app:latest'
                }
            }
        }

        stage('Clean Up on Failure') {
            steps {
                script {
                    // Job 4: If Job 3 fails, delete the running container on the test server
                    catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                        sh 'docker stop your_php_container'
                        sh 'docker rm your_php_container'
                    }
                }
            }
        }
    }

  
}
