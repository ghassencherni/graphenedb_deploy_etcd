node {

  git 'https://github.com/ghassencherni/graphenedb_deploy_etcd.git'
  withCredentials([usernamePassword(credentialsId: 'aws_credentials', usernameVariable: 'ACCESS_KEY', passwordVariable: 'SECRET_ACCESS')])
{

  if(action == 'Deploy ETCD') {


    stage('Getting "config"') {
      copyArtifacts filter: 'config', fingerprintArtifacts: true, projectName: 'graphenedb_deploy_eks', selector: upstream(fallbackToLastSuccessful: true)
    }
    stage('create secret containing etcd server certs and ca') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          export KUBECONFIG=config
          kubectl create secret generic etcd-client-certs --from-file=ca.crt=/var/lib/jenkins/secrets/ca.crt --from-file=cert.pem=/var/lib/jenkins/secrets/server.pem --from-file=key.pem=/var/lib/jenkins/secrets/server-key.pem
         """
    }
    stage('Deploy ETCD Helm') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          export KUBECONFIG=config
          helm repo add bitnami https://charts.bitnami.com/bitnami
          helm install bitnami/etcd --name graphenedb-etcd --values graphenedb.etcd.values --version 4.8.1
         """
    }
    
    stage('Get the ETCD Server URL') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          sleep 20
          export KUBECONFIG=config
          export SERVICE_IP=\$(kubectl get svc --namespace default graphenedb-etcd --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
          echo "etcd URL: https://\$SERVICE_IP:2379/"
         """
    }
  }

if(action == 'Destroy ETCD') {

    stage('Remove ETCD Helm') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          export KUBECONFIG=config
          helm del --purge graphenedb-etcd
          kubectl delete svc/graphenedb-etcd
         """
    }
    stage('Delete secret used to store etcd certs') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          export KUBECONFIG=config
          kubectl delete secret etcd-client-certs
         """
    }
   }
  }
}
