node {
    def build_ok = true
    stage('Criando Instancia AWS') {
        ansiblePlaybook(playbook: 'aws.yml', inventory: 'hosts_inventory', colorized: true)
    }
    try{
        stage('testando porta 22 da instancia') {
	sh '''
	  sleep 120
          IP_INSTANCE=\$(egrep -v "^$" hosts_inventory | tail -1 hosts_inventory)
          ssh -q -o StrictHostKeyChecking=no -i aws-jenkins-key.pem \$IP_INSTANCE exit
        if [ \$? -eq 0 ] ; then
          echo 'SSH esta ativo na instancia'
	  exit 0
        else
          echo 'Ocorreu um erro ao tentar se conectar na instancia. VERIFIQUE MANUALMENTE'
	  exit 1
        fi
        '''
	}
    } catch(e) {
        build_ok = false
        echo e.toString()  
    }
    try{
    stage('testando porta 80 da instancia') {
	sh '''
	  echo 'Aguardando o container iniciar.'
	  sleep 360
	  IP_INSTANCE=\$(egrep -v "^$" hosts_inventory | tail -1 hosts_inventory)
          curl \$IP_INSTANCE > /dev/null
        if [ \$? -eq 0 ] ; then
          echo 'A porta 80 esta ativa na instancia'
	  exit 0
        else
          echo 'Ocorreu um erro ao tentar se conectar na instancia. VERIFIQUE MANUALMENTE'
	  exit 1
        fi
        '''
    	}
    } catch(e) {
        build_ok = false
        echo e.toString()  
    }
    try{
    stage('Efetuando Payload') {
	sh '''
	  IP_INSTANCE=\$(egrep -v "^$" hosts_inventory | tail -1 hosts_inventory)
	  curl --header "Content-Type: application/json" --request POST --data '{"nome": "Jenkins name", "sobrenome": "Jenkins lastName", "email": "Jenkins email", "password": "Jenkins password", "defaultLanguage": "JenkinsdefaultLanguage", "dataNascimento": "Jenkins nascimento", "endereco": "Jenkins endereco", "areasInteresse": "Jenkins areasInteresse", "escola": "Jenkins escola", "statusEscola": "Jenkins statusEscola"}' http://\$IP_INSTANCE/api/users/cadastra
        '''
    	}
    } catch(e) {
        build_ok = false
        echo e.toString()  
    }
    try{
    stage('Enviando notificação para o RocketChat') {
    rocketSend channel: '#gupy-analistas', message: 'O deploy da aplicaç~ão MongApp terminou'
    	}
    } catch(e) {
        build_ok = false
        echo e.toString()  
    }
    if(build_ok) {
        currentBuild.result = "SUCCESS"
	echo 'I only execute on the master branch'
    } else {
        currentBuild.result = "FAILURE"
	echo 'I execute elsewhere'
    }
}
