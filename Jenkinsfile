stage 'build'
node{
    checkout scm
    sh 'mvn clean package deploy' -Ddeploy.deploymentType=arm -Ddeploy.target=server1 -Ddeploy.targetType=server -Ddeploy.environment=Sandbox -DskipTests=true -Ddeploy.username=rpedada123 -Ddeploy.password=Rpedada1
}
