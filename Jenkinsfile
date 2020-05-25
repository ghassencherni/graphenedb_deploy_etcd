node {

  git 'https://github.com/ghassencherni/mykveks_deploy_etcd.git'
  withCredentials([usernamePassword(credentialsId: 'aws_credentials', usernameVariable: 'ACCESS_KEY', passwordVariable: 'SECRET_ACCESS')])
{

  if(action == 'Deploy ETCD') {


    stage('Getting "config"') {
      copyArtifacts filter: 'config', fingerprintArtifacts: true, projectName: 'mykveks_deploy_eks', selector: upstream(fallbackToLastSuccessful: true)
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
          helm install bitnami/etcd --name mykveks-etcd --values mykveks.etcd.values --set auth.rbac.rootPassword=$etcd_admin_password --version $etcd_chart_version
         """
    }
    
    stage('Get the ETCD Server URL') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          sleep 20
          export KUBECONFIG=config
          ETCD_URL=https://\$(kubectl get svc --namespace default mykveks-etcd --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}"):2379
         """
    }
  }

if(action == 'Destroy ETCD') {

    stage('Remove ETCD Helm') {
      sh """
          export AWS_ACCESS_KEY_ID='$ACCESS_KEY'
          export AWS_SECRET_ACCESS_KEY='$SECRET_ACCESS'
          export KUBECONFIG=config
          helm del --purge mykveks-etcd
          kubectl delete svc/mykveks-etcd
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
